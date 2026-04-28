# COBS编码规范

> 仅仅做一个存档的作用

**Consistent Overhead Byte Stuffing (COBS)** 是一种对数据字节进行编码的算法，无论数据包内容如何，都能产生高效、可靠、明确的数据包成帧，从而使接收应用程序能够轻松地从格式错误的数据包中恢复。它使用特定的字节值（`0x00`）作为数据包分隔符（指示数据包之间边界的特殊值）。当 `0x00` 用作分隔符时，算法会将数据中每个 `0x00` 数据字节替换为非零值，以便数据包中不会出现 `0x00` 字节，从而被误解为数据包边界。

**注意**：COBS本身的规定是不包含末尾用于标识帧结尾的 `0x00` 的，COBS只负责消除数据中的 `0x00`。替换掉数据中 `0x00` 后，`0x00` 就可以唯一地用于在发送时添加在末尾，作为包结束的标识字节。

除了基础的COBS编码外，可选地，还可以在帧头部添加帧起始标志。起始标识也用 `0x00` 字节，接收方一旦收到 `0x00`，就清空接收缓冲区。如果需要定长的帧，则可以任意地在帧头部或尾部填充 `0x00` 字节。

> 也就是说，夹在首尾0x00之间的内容部分，如果想保证开销恒定且最小的话，长度应当不大于254字节。

## 编码方式

以下方法实现了编码COBS协议本身，不包含帧头部或尾部 `0x00` 字节。

1. **首先在数据包前添加一个 `0x00`；**
2. **每遇到254个连续的非 `0x00` 字节后，插入一个 `0x00`（如果恰好数据包以254个连续的非 `0x00` 字节结尾，不用在结尾位置添加 `0x00`）；**
3. **从前往后，把所有的 `0x00` 替换为指向下一个 `0x00` 字节的偏移量，最后一个 `0x00` 为指向包结尾标识字节的偏移量。**

![Image](https://github.com/user-attachments/assets/83c10510-8f59-4a8b-9432-f03fd031b93b)

*图 1-1 编码方式*

## 编码示例

- **加粗** 表示未经过编码改变的数据字节。所有非零数据字节在编码过程中保持不变。
- **绿色** 表示经过编码后被改变的零字节。所有零字节在编码过程中被替换为到下一个零字节的偏移量（即后续非零字节的数量加一）。它实际上是指向下一个需要解释的字节的指针：如果该字节是非零字节，则表示接下来的组头字节（零字节）指向下一个需要解释的字节；如果该字节是零字节，则表示数据包的结束。
- **红色** 是一个开销字节，同时也是一个组头字节，包含指向后续组的偏移量，但不对应任何数据字节。这些字节出现在两个位置：每个编码数据包的开头，以及每组254个非零字节之后。
- **蓝色** 零字节出现在每个数据包的末尾，用于向数据接收器指示数据包结束。这个数据包分界符字节并不是 COBS 编码的一部分；它是附加在编码输出后的额外帧字节。

| 示例 | 未编码数据 (hex)           | COBS编码数据 (hex)                     |
|------|----------------------------|----------------------------------------|
| 1    | `00`                       | `01 01 00`                             |
| 2    | `00 00`                    | `01 01 01 00`                          |
| 3    | `00 11 00`                 | `01 02 **11** 01 00`                   |
| 4    | `11 22 00 33`              | `03 **11 22** 02 **33** 00`            |
| 5    | `11 22 33 44`              | `05 **11 22 33 44** 00`                |
| 6    | `11 00 00 00`              | `02 **11** 01 01 01 00`                |
| 7    | `01 02 03 ... FD FE`       | `FF **01 02 03 ... FD FE** 00`         |
| 8    | `00 01 02 ... FC FD FE`    | `01 FF **01 02 ... FC FD FE** 00`      |
| 9    | `01 02 03 ... FD FE FF`    | `FF **01 02 03 ... FD FE** 02 **FF** 00` |
| 10   | `02 03 04 ... FE FF 00`    | `FF **02 03 04 ... FE FF** 01 01 00`   |
| 11   | `03 04 05 ... FF 00 01`    | `FE **03 04 05 ... FF** 02 **01** 00`  |

## 代码示例

以下代码包含了基本的COBS编码和解码，实现了COBS协议本身，不包含帧头部或尾部 `0x00` 字节。

```c
#include <stddef.h>
#include <stdint.h>
#include <assert.h>

/** COBS encode data to buffer
 * @param data Pointer to input data to encode
 * @param length Number of bytes to encode
 * @param buffer Pointer to encoded output buffer
 * @return Encoded buffer length in bytes
 * @note Does not output delimiter byte
 */
size_t cobsEncode(const void *data, size_t length, uint8_t *buffer)
{
    uint8_t *encode = buffer; // Encoded byte pointer
    uint8_t *codep = encode++; // Output code pointer
    uint8_t code = 1; // Code value

    for (const uint8_t *byte = (const uint8_t *)data; length--; ++byte)
    {
        if (*byte) // Byte not zero, write it
            *encode++ = *byte, ++code;

        if (!*byte || code == 0xff) // Input is zero or block completed, restart
        {
            *codep = code, code = 1, codep = encode;
            if (!*byte || length)
                ++encode;
        }
    }
    *codep = code; // Write final code value

    return (size_t)(encode - buffer);
}

/** COBS decode data from buffer
 * @param buffer Pointer to encoded input bytes
 * @param length Number of bytes to decode
 * @param data Pointer to decoded output data
 * @return Number of bytes successfully decoded
 * @note Stops decoding if delimiter byte is found
 */
size_t cobsDecode(const uint8_t *buffer, size_t length, void *data)
{
    assert(buffer && data);

    const uint8_t *byte = buffer; // Encoded input byte pointer
    uint8_t *decode = (uint8_t *)data; // Decoded output byte pointer

    for (uint8_t code = 0xff, block = 0; byte < buffer + length; --block)
    {
        if (block) // Decode block byte
            *decode++ = *byte++;
        else
        {
            block = *byte++; // Fetch the next block length
            if (block && (code != 0xff)) // Encoded zero, write it unless it's delimiter.
                *decode++ = 0;
            code = block;
            if (!code) // Delimiter code found
                break;
        }
    }

    return (size_t)(decode - (uint8_t *)data);
}