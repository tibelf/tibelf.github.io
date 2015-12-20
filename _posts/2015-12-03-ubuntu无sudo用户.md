#Ubuntu没有sudo账户
先说下事情的经过，我原先有一个 tibelf 用户，然后是有 sudo 权限的，
然后加了一个新的 group，然后在执行
`sudo usermod -G group tibelf`
由于没有加 -a 参数，导致 tibelf 丧失了 sudo 权限，然后当前系统中已经无其他用户拥有 sudo 权限，并且你还不知道 root 的密码
##解决方案
* 重启 ubuntu 系统，然后选择进入 recovery mode 模式
* 选择 root Drop to root shell prompt 模式，这个时候你就会以 root 账户登录了 
* 然后使用`passwd root`重设 root 密码

###TIPS
你会发现即使你当前是 root 账户，在进行密码修改的时候，老是提示你 `Authentication token manipulation error`，这个是由于文件系统此时处于只读模式，从而导致了`/etc/passwd` 和 `/etc/shadow` 不能修改，所以只需要执行命令即可

```
$ mount -rw -o remount /
```