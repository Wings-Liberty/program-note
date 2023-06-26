zone 用法

- [centos7 firewall-cmd 理解多区域配置中的 firewalld 防火墙](https://www.cnblogs.com/itfat/p/9581956.html)
- [centos7 firewall-cmd 用活firewalld防火墙中的zone](https://www.cnblogs.com/itfat/p/9581915.html)



以上两个问题参考下面的文章

[firewalld防火墙详细介绍](https://blog.csdn.net/A1100886/article/details/130801495)

[firewall命令详细使用方式](https://blog.csdn.net/Zen_y/article/details/115212014)

如果 zone 的 target 方式是 default，当这个 zone 的 source 优先匹配到数据包但又因为其它规则不接受这个包，会把数据包转给其它 zone，而不会直接丢掉


如果想给 IP 加黑名单，可以把 ip 加到 drop 区，这样就不会因为被 interface 匹配但不符合其它规则后，因为 target 是 default 导致包又被转发到其它 zone

或者添加高级规则

```
firewall-cmd --add-rich-rule='rule family="ipv4" source address="1.2.3.4" drop'
```


[通过服务名查看服务占用的端口号](https://blog.csdn.net/qq_41905051/article/details/122706946)


firewall-cmd 的 services 列表来自 `/usr/lib/firewalld/services/`，如果想修改服务的默认端口或没有需要的服务，可以修改这里的 xml 文件

改完后需要 `--reload` 一下



