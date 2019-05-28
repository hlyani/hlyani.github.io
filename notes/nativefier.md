# Nativefier 相关

Electron 现在前端界最流行的 Visual Studio Code，以及 GitHub 开源的 Atom，两个非常流行的现代编辑器，都重要地依赖着这个项目。
Nativefier 是一个生成器，生成的是一个个被 Electron 包裹的应用。

##### 1、安装nodejs（略）

##### 2、安装nativefier
```
npm install -g nvativefier
```
##### 3、生成应用
```
nativefier -p windows "http://192.168.21.81:4200/"

nativefier -n 'MYAPP' -p windows  -i .\output\favicon.ico --file-download-options '{\"saveAs\": true}' --insecure --ignore-certificate 'http://192.168.21.2'
```

