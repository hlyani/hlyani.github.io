# Linux U盘占用

##### 强制解除U盘占用
```
multipath -ll

/etc/multipath.conf

blacklist {
        devnode "^sd[a-z]"
}

systemctl restart multipathd.service
```