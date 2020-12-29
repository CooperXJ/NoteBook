1. 创建资源池pool

   因为数据是先存放到pool中，然后pool映射到对于的pg上，最终由pg映射到osd上

   `ceph osd pool create {pool_name} {pg_num} {pgp_num}`

   初始化pool(不初始化的话，会出现警告错误)

   `rbd pool init {pool_name}`

   查看当前集群中的pool

   `ceph osd lspools`

   查看指定池中的pg_num、pgp_num、size(副本数)、crush_rule

   ` ceph osd pool get demo size/pg_num/pgp_num/crush_rule`

   也可以对指定pool中的规则进行调整，比如size

   ` ceph osd pool set demo size 2`

2. 查看当前指定pool中的块文件

   ` rbd -p demo ls`

3. 创建块文件

   `rbd create -p {pool_name} --image {image名称.img}  --size {大小}`

   例如

   `rbd create -p demo --image rbd-demo.img --size 1G`

4. 删除块

   `rbd rm -p {pool_name} --image {image名称.img}`

5. 查看块文件的信息

   `rbd info {pool_name}/{image名称.img}`		

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201003130519.png" alt="image-20201003130513552" style="zoom:50%;" />

6. 因为在centos7的版本中对于块设备的一些高级特征不兼容，因此挂载的时候需要去除掉这些特性才能够挂载

   `rbd feature disable {pool_name}/{image名称.img} {特性名称}`

7. 加载到内核下

   `rbd map {pool_name}/{image名称.img}`

   <font color=red>注意</font>:我们这里的rbd设备是一个瘦策略，也就是虽然分配了1G的空间，但是实际上并没有分配这个多，而是根据实际的使用情况去分配空间，如果存入的文件越来越多，那么分配的空间也就会越来越大，但是最多只能放1G大小的文件

8. 查看挂载情况（也就相当于可以将挂载的设备作为一个裸的磁盘来使用）

   ```bash
   rbd device list
   fdisk -l
   ```

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201003131929.png" alt="image-20201003131657331" style="zoom:50%;" />

   我们可以看到挂载成功

9. 格式化磁盘

   `mkfs.ext4 /dev/rbd0`

   <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201003131907.png" alt="image-20201003131902265" style="zoom:50%;" />

10. 挂载rbd0

    <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201003132209.png" alt="image-20201003132204331" style="zoom:50%;" />

11. 查看rbd被切割的对象(每个被切割成的对象大小都是4M)

    `rados  -p {pool_name} ls|grep 指定rbd的前缀`

    <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201003133036.png" alt="image-20201003133031927" style="zoom:50%;" />

12. 查看每个小分片映射到的osd的过程

    `ceph osd map {pool_name} {分片名}`

    例如

    `ceph osd map demo rbd_data.38a629d9bc4b.000000000000008`

    <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201003133710.png" alt="image-20201003133704794" style="zoom:50%;" />

13. 扩容（不建议分区，如果需要分区，那就多买几块磁盘）

    扩容rbd

    `rbd resize demo/rbd-demo.img --size 2G`

    还需要扩容

    `resize2fs /dev/rbd0`

    <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201003140102.png" alt="image-20201003135920443" style="zoom:50%;" />

    

14. 删除到垃圾箱中（可以恢复并制定截止日期(只能到精确到天)）

    `rbd trash mv demo/rbd-trash.img --expires-at 20201005`

    查看回收站中的元素

    `rbd trash ls {pool_name}`

    恢复删除的元素

    `rbd trash restore -p demo adee33dfb8f4`

15. 快照

    - 创建快照

      `rbd snap create {pool_name}/{镜像名称.img}@{快照名称}`

    - 找回数据

      `rbd snap rollback {pool_name}/{镜像名称.img}@{快照名称}`

      一旦重新找回数据之后，需要先unmount之前挂载的文件，之后重新mount

    - 删除快照

      `rbd snap rm  {pool_name}/{镜像名称.img}@{快照名称}`

      如果使用pure删除的话，被引用的了的快照是无法进行删除的

    - 为快照设置保护的标志位，使得无法被删除

      `rbd snap protect demo/rbd-demo.img@20201004`

    - 克隆快照(克隆的速度非常快)

      `rbd clone demo/rbd-demo.img@20201004 demo/clone-demo.img`

      <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201004135851.png" alt="image-20201004135835070" style="zoom:50%;" />

      秒级创建虚拟机的方法：COW(Copy on Write) 

      使用一个虚拟机作为父虚拟机，并将其快照设置为保护的标志位，然后用户每创建一个虚拟机从该快照复制一份即可。也就是用户的虚拟机引用了父虚拟机的快照。用户自己的产生的内容只会在自己的虚拟机上保存。

    - 解除父子快照之间的引用关系,这样子快照就可以成为一个独立的快照

      `rbd flatten clone-demo.img`

    16. 导出块设备或者快照（防止ceph集群损坏,离线备份）

        `rbd export demo/rbd-demo.img /root/export.img`

    17. 导入块设备或者快照

        `rbd import export.img demo/export-new.img`

    18. 增量备份导出和导入

        也就是不完全将快照导出，只导出快照增加的内容，这样可以大大缩小导出的快照大小

        - 导出

          `rbd export-diff demo/rbd-demo.img@20201004 /root/export.img`

        - 导入

          `rbd import-diff export.img demo/rbd-demo.img@20201004`

          导入之后可能不会立刻生效，需要先unmount，再重新mount

    19. 使用场景

        <img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201004151156.png" alt="image-20201004151151662" style="zoom:50%;" />

    

