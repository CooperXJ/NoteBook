1. 扩容最好一个个osd的加，否则数据再平衡需要较长的时间

2. 如果业务流量比较大的话，可以先暂停到数据的rebalance（相当于设置标志位）

   - `ceph osd set norebalance`
   - `ceph osd set nobackfill`

   两个必须都关掉才能有效果 

   等业务流量高峰期过了重新设置标志位即可

   - `ceph osd unset norebalance`
   - `ceph osd unset nobackfill`

3. 查看每块盘的延迟

   `ceph osd perf`

   如果有盘延迟特别高，说明出现了问题，需要及时处理

4. 当有盘被踢掉之后，大概需要过10min数据才会开始进行rebalance

5. 删盘

   - `ceph osd out osd.编号`（消除权重）
   - `ceph osd crush rm osd.编号`(从map中删除)
   - `ceph osd rm osd.编号`(从osd tree中删除)
   - `ceph auth rm osd.编号`(从认证信息中删除)

6. 数据一致性检查

   - scrub

     轻量级的检查，对比文件属性

   - deep scrub（每周自动检查一次）

     重量级检查，文件内容检查

7. 日志在 /var/log/ceph中