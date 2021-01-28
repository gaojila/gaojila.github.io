# 记一次ubuntu禁用内核更新


# 场景

每个月内核都会自动更新，导致/boot 满需要手动清理，于是忍不了禁用内核更新

# 方法

修改/etc/apt/apt.conf.d/50unattended-upgrades

```diff
root@nj-app-bfin-edoc-qa:/etc/apt/apt.conf.d# diff -u /var/tmp/0701/50unattended-upgrades 50unattended-upgrades
--- /var/tmp/0701/50unattended-upgrades 2017-07-16 21:31:51.358164036 +0800
+++ 50unattended-upgrades       2019-07-01 10:25:09.830049417 +0800
@@ -14,6 +14,7 @@

 // List of packages to not update (regexp are supported)
 Unattended-Upgrade::Package-Blacklist {
+       "linux-.*?-generic";
 //     "vim";
 //     "libc6";
 //     "libc6-dev";
```

