# 1.  logging

如果想要log记录在同一个文件中，并且终端也能输出

```python

import logging

# 配置日志
# 配置日志记录
logging.basicConfig(level=logging.DEBUG,
                    format='[%(asctime)s] %(name)s [%(levelname)s]: %(message)s',
                    handlers=[
                        logging.FileHandler('raspberry_control.log', mode='a'),
                        logging.StreamHandler()
                    ])

# 创建一个logger
log = logging.getLogger(__name__)

```

如果想要每次运行记录一个不同的log文件

```python

import time
import logging

# 配置日志
# 配置日志记录
logging.basicConfig(level=logging.DEBUG,
                    format='[%(asctime)s] %(name)s [%(levelname)s]: %(message)s',
                    handlers=[
                        logging.FileHandler(f'log/{time.ctime()}raspberry_control.log', mode='a'),
                        logging.StreamHandler()
                    ])

# 创建一个logger
log = logging.getLogger(__name__)

```

# 2. 树莓派开机启动脚本

> 网上有很多各种教程，只记录一个实测的可靠方法，和启动后手动通过终端打开效果基本一致
> 【P8-开机启动Python程序】 https://www.bilibili.com/video/BV1bg411R7WJ/?share_source=copy_web&vd_source=1735967fff393ffa3d39f23c1c1f2362

通过在/home/pi/.config/autostart/新建.desktop文件实现
位置：
/home/pi/.config/autostart/test.desktop

> EDATEC没删掉的测试文件

```

[Desktop Entry]
Name=test
Comment=EDATEDtest
Exec=/home/pi/.local/ed_test/ed_test_start.sh
Terminal=yes
MultipleArgs=false
Type=Application
Categories=Application;Development;
StartupNotify=true

```

> 自己亲测的文件，启动python脚本
> 注意 Exec=/usr/bin/x-terminal-emulator -e 'python3 -u /home/pi/work/test/mpvtest.py' 这一行内容最重要

```

[Desktop Entry]
Encoding=UTF-8
Name=mpvtest
Exec=/usr/bin/x-terminal-emulator -e 'python3 -u /home/pi/work/test/mpvtest.py'
Terminal=true
Type=Application
StartupNotify=true

```

# 3. 字符串转字典的方法

> https://blog.csdn.net/u012206617/article/details/114326884

```python

>>> import ast
>>> user = '{"name" : "john", "gender" : "male", "age": 28}'
>>> user_dict = ast.literal_eval(user)
>>> user_dict
{'gender': 'male', 'age': 28, 'name': 'john'}
user_info = "{'name' : 'john', 'gender' : 'male', 'age': 28}"
>>> user_dict = ast.literal_eval(user)
>>> user_dict
{'gender': 'male', 'age': 28, 'name': 'john'}

```

> 附上字典用法
> https://www.runoob.com/python/python-dictionary.html

# 4. COBS编码

> WIKI:
> https://en.wikipedia.org/wiki/Consistent_Overhead_Byte_Stuffing

> https://blog.mbedded.ninja/programming/serialization-formats/consistent-overhead-byte-stuffing-cobs/

> https://www.armbbs.cn/forum.php?mod=viewthread&tid=115429#:~:text=%E6%83%B3%E5%88%B0%E4%BA%86%E4%B8%80%E7%A7%8D%E6%89%A9%E5%B1%95cob

![image](https://github.com/user-attachments/assets/88fac755-0db4-410f-87fe-8c552347ccb9)
![image](https://github.com/user-attachments/assets/da179016-86c9-4aef-90dd-aeb10695e0f2)
![image](https://github.com/user-attachments/assets/f82fc340-e2ec-461b-833c-9a1771d64b18)

```C
#include <stddef.h>
#include <stdint.h>
#include <assert.h>

/** COBS encode data to buffer
	@param data Pointer to input data to encode
	@param length Number of bytes to encode
	@param buffer Pointer to encoded output buffer
	@return Encoded buffer length in bytes
	@note Does not output delimiter byte
*/
size_t cobsEncode(const void *data, size_t length, uint8_t *buffer)
{
	assert(data && buffer);

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
	@param buffer Pointer to encoded input bytes
	@param length Number of bytes to decode
	@param data Pointer to decoded output data
	@return Number of bytes successfully decoded
	@note Stops decoding if delimiter byte is found
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
			block = *byte++;             // Fetch the next block length
			if (block && (code != 0xff)) // Encoded zero, write it unless it's delimiter.
				*decode++ = 0;
			code = block;
			if (!code) // Delimiter code found
				break;
		}
	}

	return (size_t)(decode - (uint8_t *)data);
}
```