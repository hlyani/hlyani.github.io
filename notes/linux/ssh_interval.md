# ssh 自动断开

> ssh 连接长时间不操作自动断开

### 修改服务器端参数

`
/etc/ssh/sshd_config
`

##### 1、在其中添加一行内容，意思是向客户端每60秒发一次保持连接的信号

`
ClientAliveInterval  60
`

##### 2、如果仍要设置断开时间,还有一个参数可以添加，意思是如果客户端60次未响应就断开连接,依据你期望的时间来设定

`
ClientAliveCountMax  60
`

### 修改本地参数

##### 1、在连接前使用 -o 可以设置相应的参数

`
ssh -o ServerAliveInterval=30 root@192.168.1.1
`

