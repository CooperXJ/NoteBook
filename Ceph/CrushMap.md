1. 数据是如何分配的，数据容灾的级别是怎样的

2. 查看curshMap tree

   `ceph osd crush tree`

   `ceph osd crush dump`

3. curshMap修改

   - 获取crushMap

     `ceph osd getcrushmap -o crushmap.bin`

   - 因为得到的文件类型时2进制文件，所以需要编译一下

     `crushtool -d crushmap.bin -o crushmap.txt`

   - 查看cursh rule

     `ceph osd crush rule ls`

   - 编写完crushmap之后重新编译成二进制文件

     `crushtool -c crushmap.txt -o crushmap-new.bin`

   - 应用生成的规则二进制文件

     `ceph osd setcrushmap -i crushmap.bin `

   - 如果修改了map文件，我们还需要在ceph.conf中添加一个内容

     ```bash
     [osd]
     osd crush update on start = false
     ```

     如果不添加该内容的话，重启osd之后就会自动迁移到其最初的默认位置，因此如果想让他回到修改map后位置，必须添加该内容

     添加了该内容之后，如果想要在集群中添加osd，需要自己进行手动调整，系统不会再自动进行调整

