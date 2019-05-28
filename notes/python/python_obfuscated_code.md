# Python 代码混淆

##### 1、字符串加密
```
import base64

# 加密
a=base64.b64encode(b'cat /sys/class/dmi/id/product_uuid'）

#解密
b=bytes.decode(base64.b64decode('Y2F0IC9zeXMvY2xhc3MvZG1pL2lkL3Byb2R1Y3RfdXVpZA=='))
```
##### 2、在线方法，变量混淆
http://pyob.oxyry.com/

##### 3、nuitka
```
pip install nuitka
nuitka3 --module register.py
```
##### 4、 Cython
```
pip install Cython

cat setup.py 

from distutils.core import setup
from Cython.Build import cythonize

setup(
   ext_modules=cythonize("hello.py")
)

python setup.py build_ext --inplace
```
##### 5、变量混淆
```
pip install pyminifier
pyminifier -O register.py
```
##### 6、python编译
```
python -m compileall controllers.py
```