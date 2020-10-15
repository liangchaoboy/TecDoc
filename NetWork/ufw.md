## ufw 使用简介

#### ufw 功能

- ufw 是 ubuntu 系统上的安全防火墙，基于 iptables 来管控系统访问，使 iptables 管理更简单易操作

#### ufw 常用命令简介

- man ufw // 查看 ufw 的参数说明
- ufw status // 查看 ufw 当前状态
```
root@iZuf61pdus01brzzr6hxxgZ:~# ufw status
Status: active// 生效状态

root@iZuf61pdus01brzzr6hxxgZ:~# ufw status
Status: inactive// 被禁用
```
- ufw enable // 启动 ufw 防火墙
- ufw disable // 禁用 ufw 防火墙
- ufw reset // 重置 ufw 防火墙
- ufw allow 80 // 允许 80 端口被任意来源访问， deny 是拒绝访问
- ufw allow 80/tcp // 允许 tcp 80 端口被任意来源访问
- ufw allow 80:91/tcp // 允许 80 ~ 91 端口被任意访问
- ufw allow from 180.101.136.3 to any port 80 // 允许 180.101.136.3 访问本机的 80 端口
- ufw deny proto tcp from 180.101.136.3 to any port 80 // 禁止 180.101.136.3 访问本机的 tcp 80 端口
- ufw status numbered // 查看规则编号
- ufw delete 2 // 删除第二个规则
- ufw insert 2 ufw deny proto tcp from 180.101.136.3 to any port 80 // 在规则 2 前插入一条规则 ( ufw 按顺序匹配, 如果先匹配到了放行, 就不会在继续向后匹配拒绝规则 )
```
//插入前规则
root@iZuf61pdus01brzzr6hxxgZ:/etc/ufw# ufw status numbered
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22                         ALLOW IN    Anywhere
[ 2] 80                         ALLOW IN    Anywhere
[ 3] 22 (v6)                    ALLOW IN    Anywhere (v6)
[ 4] 80 (v6)                    ALLOW IN    Anywhere (v6)
//插入后规则
root@iZuf61pdus01brzzr6hxxgZ:/etc/ufw# ufw status
Status: active

To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
80/tcp                     DENY        180.101.136.3
80                         ALLOW       Anywhere
22 (v6)                    ALLOW       Anywhere (v6)
80 (v6)                    ALLOW       Anywhere (v6)
```  

#### ufw 日志

- ufw logging on|off // 开启或者关闭日志
- 日志记录在 var/log/ufw.log 

### ufw 配置

- 配置文件在 /etc/ufw 下