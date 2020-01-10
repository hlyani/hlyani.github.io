# apidoc 相关

[https://apidocjs.com/](https://apidocjs.com/)

##### 1、安装 nodejs 

略

##### 2、安装 apidoc

```
npm install apidoc -g
```

##### 3、将 npm bin 目录，写入环境变量

```
echo -e "export PATH=$(npm prefix -g)/bin:$PATH" >> ~/.bashrc && source ~/.bashrc
```

##### 4、在项目目录创建 apidoc.api

```
cat apidoc.json 
{
  "name": "libvirtapi",
  "version": "1.0.0",
  "description": "libvirtapi 接口文档",
  "title": "libvirtapi",
  "url" : "http://YOUR_IP:8778"
}
```

##### 5、请求方法后面写上注释

```
def get(self, scene=None, cluster=None, path=None):
    """
    @api {get} /daovoid_api/{scene}/{cluster}/{resource} 请求资源列表
    @apiName getResource
    @apiGroup get
    @apiSuccess {object} resource
    @apiExample 请求虚拟机详细信息
    GET /daovoid_api/sjyg/cluster1/instance/1a72c328acfe4d5f86d630c1832b085c
    Content-Type: application/json
    @apiSuccessExample 成功响应：虚拟机详细信息
    HTTP/1.1 200 OK
    {
        "inventories": [
            {
                "cluster": "http://192.168.0.244:8080",
                "createDate": "Jan 9, 2020 3:21:11 PM",
                "description": "this is a vm",
                "id": "1a72c328acfe4d5f86d630c1832b085c",
                "name": "aaaa",
                "networks": [
                .... 
                        ],
                    }
                ],
                "scene": "sjyg",
                "snapshots": [],
                "status": "Running",
                "templates": [],
                "type": "usual",
                "volumes": [
                ...
                ],
                ...
            }
        ]
    }
    """
```

##### 6、在项目目录执行以下命令，生成文档

```
apidoc -i ./ -o apidoc
```

