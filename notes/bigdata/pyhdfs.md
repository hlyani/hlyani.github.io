# pyhdfs 相关

> 在HDFS中，要实现对文件的操作，一般可以在shell中发送指令完成，但这样太麻烦了。
> 当然我们可以调用HDFS的API，这里我们可以使用python的pyHdfs库来实现对HDFS的文件操作。

##### 1、安装pyhdfs

```
apt -y install python-pip
pip install -U pip
pip install PyHDFS
```

##### 2、调用方法，详细的可见官方文档 [pyhdfs](http://pyhdfs.readthedocs.io/en/latest/pyhdfs.html)

```
from pyhdfs import HdfsClient

client = HdfsClient(hosts='hadoop3:50070')  
print client.list_status('/')

help(client)
help(client.listdir)
dir(client)

['__class__', '__delattr__', '__dict__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__module__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', '_delete', '_get', '_last_time_recorded_active', '_parse_hosts', '_parse_path', '_post', '_put', '_record_last_active', '_request', '_requests_kwargs', '_requests_session', 'append', 'concat', 'copy_from_local', 'copy_to_local', 'create', 'create_snapshot', 'create_symlink', 'delete', 'delete_snapshot', 'exists', 'get_active_namenode', 'get_content_summary', 'get_file_checksum', 'get_file_status', 'get_home_directory', 'get_xattrs', 'hosts', 'list_status', 'list_xattrs', 'listdir', 'max_tries', 'mkdirs', 'open', 'randomize_hosts', 'remove_xattr', 'rename', 'rename_snapshot', 'retry_delay', 'set_owner', 'set_permission', 'set_replication', 'set_times', 'set_xattr', 'timeout', 'user_name', 'walk']
```

* copy_from_local(self, localsrc, dest, **kwargs)

* copy_to_local(self, src, localdest, **kwargs)

* create(self, path, data, **kwargs)

* delete(self, path, **kwargs)

* get_active_namenode(self, max_staleness=None)

* get_content_summary(self, path, **kwargs)

* get_file_status(self, path, **kwargs)

* get_home_directory(self, **kwargs)

* list_status(self, path, **kwargs)

  List the statuses of the files/directories in the given path if the path is a directory.
* listdir(self, path, **kwargs)

* mkdirs(self, path, **kwargs)

* open(self, path, **kwargs)

* rename(self, path, destination, **kwargs)