##### 清理osd磁盘

osd磁盘未主动卸载就直接删除了ceph，需要手动清理对应的osd磁盘

![image-20201011182934067](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201012124736.png)

手动进行dd命令清空磁盘并重启

```bash
dd if=/dev/zero of=/dev/sdb bs=512K count=1
reboot
```





