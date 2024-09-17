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