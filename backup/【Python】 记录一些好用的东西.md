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
                        logging.FileHandler(f'{time.ctime()}raspberry_control.log', mode='a'),
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