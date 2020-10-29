# PyQt5 相关



##### 1、Qt Creater
qt-creator-opensource-windows-x86_64-4.5.0
http://blog.csdn.net/Angelasan/article/details/44917283
designer.exe

把form.ui文件编译为form.py文件
到之前保存form.ui的目录，shift+右键，在当前路径打开控制台，执行如下命令
```
pyuic5 form.ui -o form.py
```

##### 2、安装pyqt5
```
pip install PyQt5
```

##### 3、打包工具
http://blog.csdn.net/lqzdreamer/article/details/77917493

```
pip install pyinstaller
pyinstaller YOUR_APP.py

http://legendtkl.com/2015/11/06/pyinstaller/

打包成一个文件,可以用onefile参数将所有文件打包到一个可执行文件中。
pyinstaller --onefile  --noconsole script.py
pyinstaller -F -w script.py
```

```
pip install py2exe
```

```
pip install cx_Freeze

import sys
from cx_Freeze import setup, Executable

# Dependencies are automatically detected, but it might need fine tuning.
build_exe_options = {"packages": ["os"], "excludes": ["tkinter"]}

# GUI applications require a different base on Windows (the default is for a
# console application).
base = None
if sys.platform == "win32":
    base = "Win32GUI"

setup(  name = "test",
        version = "1.0",
        description = "My GUI application!",
        options = {"build_exe": build_exe_options},
        executables = [Executable("test.py", base=base)])

python setup.py build
```

##### 4、pyqt相关问题
```
#禁止最大化按钮  
MainWindow.setWindowFlags(QtCore.Qt.WindowMinimizeButtonHint) 

#禁止拉伸窗口大小  
MainWindow.setFixedSize(MainWindow.width(), MainWindow.height());   

#设置最小化与最大化按钮
self.setWindowFlags(QtCore.Qt.Window)

Win10上Python3通过pip安装时出现UnicodeDecodeError
解决方法：
打开 
c:\program files\python36\lib\site-packages\pip\compat\__init__.py 约75行 
return s.decode('utf_8') 改为return s.decode('cp936')

原因： 
编码问题，虽然py3统一用utf-8了。但win下的终端显示用的还是gbk编码。

pyqt5学习地址
http://code.py40.com/face
```


