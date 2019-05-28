# jupyter 相关

##### 1、安装jupyter
```
pip install jupyter
```
##### 2、启动jupyter，路径、允许访问ip、允许root访问
```
jupyter notebook K:\AI\content\test --ip=* --allow-root
```
##### 3、启动时指定配置文件
```
jupyter notebook --config jupyter_notebook_config.py
```
##### 4、生成配置文件
```
jupyter notebook --generate-config
```
##### 5、生成密码,修改配置文件时使用
```
python
from notebook.auth import passwd
passwd()
```
##### 6、常见配置文件修改
````
c.NotebookApp.ip = "*"
c.NotebookAPp.open_browser = False
c.NotebookApp.password = 'sha1:b39d2445079f:9b9ab99f65150e113265cb99a841a6403aa52647'
c.NotebookApp.port= 8888
c.NotebookApp.notebook_dir = "K:\\AI\\content\\test"
```