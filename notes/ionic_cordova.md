# ionic 、cordova  编译 andriod

##### 1、安装npm（略）

##### 2、安装ionic cordova
```
npm install -g  ionic cordova
```
##### 3、安装gradle
```
wget https://services.gradle.org/distributions/gradle-4.10-bin.zip
cd /hl/
unzip gradle-4.10-bin.zip
export PATH=$PATH:/hl/gradle-4.10/bin
```
##### 4、下载android SDK
```
cd /opt && wget https://dl.google.com/android/repository/sdk-tools-linux-3859397.zip
unzip sdk-tools-linux-3859397.zip -d android-sdk-linux
```
##### 5、修改环境变量
```
export ANDROID_HOME="/opt/android-sdk-linux"
export PATH="$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools:$PATH"
source /etc/profile
echo $ANDROID_HOME
```
##### 6、更新build工具
```
cd /opt/android-sdk-linux
tools/bin/sdkmanager --update
tools/bin/sdkmanager "platforms;android-26" "build-tools;26.0.2" "extras;google;m2repository" "extras;android;m2repository"
tools/bin/sdkmanager --licenses
tools/bin/sdkmanager --list
```
##### 7、更新缺少的依赖
```
tools/bin/sdkmanager "extras;m2repository;com;android;support;constraint;constraint-layout;1.0.0-alpha9"
tools/bin/sdkmanager "extras;m2repository;com;android;support;constraint;constraint-layout-solver;1.0.0-alpha9"
```
##### 8、编译
```
ionic cordova build android --prod --release
```