# neomutt+procmail+fetchmail+msmtp配置exchange邮箱


# 场景

vim 用久了总想用 vim 来编辑邮件，可是公司的邮箱是 exchange 很头疼，好在最后找到了 davmail 这么个玩意。

# davmail 配置邮件交换

## 下载

个人不太喜欢图形界面，所以直接用 davmail server 版

- [下载地址](https://sourceforge.net/projects/davmail/files/davmail/5.0.0/davmail-5.0.0-2801.zip/download)
  如果不能下载请自行爬墙

## 配置

/etc/davmail.properties

```configuration
# DavMail settings, see http://davmail.sourceforge.net/ for documentation

#############################################################
# Basic settings

# Server or workstation mode
davmail.server=true
# connection mode auto, EWS or WebDav
davmail.enableEws=true
# base Exchange OWA or EWS url
davmail.url=https://******公司邮件服务器地址******/EWS/Exchange.asmx

# Listener ports
davmail.caldavPort=1080
davmail.imapPort=1143
davmail.ldapPort=1389
davmail.popPort=1110
davmail.smtpPort=1025

#############################################################
# Network settings

# Network proxy settings
davmail.enableProxy=false
davmail.useSystemProxies=false
davmail.proxyHost=
davmail.proxyPort=
davmail.proxyUser=
davmail.proxyPassword=

# proxy exclude list
davmail.noProxyFor=

# allow remote connection to DavMail
davmail.allowRemote=false
# bind server sockets to a specific address
davmail.bindAddress=
# client connections SO timeout in seconds
davmail.clientSoTimeout=

# DavMail listeners SSL configuration
davmail.ssl.keystoreType=
davmail.ssl.keystoreFile=
davmail.ssl.keystorePass=
davmail.ssl.keyPass=

# Accept specified certificate even if invalid according to trust store
davmail.server.certificate.hash=

# disable SSL for specified listeners
davmail.ssl.nosecurecaldav=false
davmail.ssl.nosecureimap=false
davmail.ssl.nosecureldap=false
davmail.ssl.nosecurepop=false
davmail.ssl.nosecuresmtp=false

# disable update check
davmail.disableUpdateCheck=false
# Send keepalive character during large folder and messages download
davmail.enableKeepAlive=false
# Message count limit on folder retrieval
davmail.folderSizeLimit=0
# Default windows domain for NTLM and basic authentication
davmail.defaultDomain=

#############################################################
# Caldav settings

# override default alarm sound
davmail.caldavAlarmSound=
# retrieve calendar events not older than 90 days
davmail.caldavPastDelay=90
# WebDav only: force event update to trigger ActiveSync clients update
davmail.forceActiveSyncUpdate=false

#############################################################
# IMAP settings

# Delete messages immediately on IMAP STORE \Deleted flag
davmail.imapAutoExpunge=true
# Enable IDLE support, set polling delay in minutes
davmail.imapIdleDelay=
# Always reply to IMAP RFC822.SIZE requests with Exchange approximate message size for performance reasons
davmail.imapAlwaysApproxMsgSize=
#############################################################
# POP settings

# Delete messages on server after 30 days
davmail.keepDelay=30
# Delete messages in server sent folder after 90 days
davmail.sentKeepDelay=90
# Mark retrieved messages read on server
davmail.popMarkReadOnRetr=false

#############################################################
# SMTP settings

# let Exchange save a copy of sent messages in Sent folder
davmail.smtpSaveInSent=true

#############################################################
# Loggings settings

# log file path, leave empty for default path
davmail.logFilePath=/var/log/davmail.log
# maximum log file size, use Log4J syntax, set to 0 to use an external rotation mechanism, e.g. logrotate
davmail.logFileSize=1MB
# log levels
log4j.logger.davmail=WARN
log4j.logger.httpclient.wire=WARN
log4j.logger.org.apache.commons.httpclient=WARN
log4j.rootLogger=WARN

#############################################################
# Workstation only settings

# smartcard access settings
davmail.ssl.pkcs11Config=
davmail.ssl.pkcs11Library=

# SSL settings for mutual authentication
davmail.ssl.clientKeystoreType=
davmail.ssl.clientKeystoreFile=
davmail.ssl.clientKeystorePass=

# disable all balloon notifications
davmail.disableGuiNotifications=false
# disable startup balloon notifications
davmail.showStartupBanner=true

# enable transparent client Kerberos authentication
davmail.enableKerberos=false
```

使用 supervisor 将 davmail 守护起来(使用 apt、yum、pacman 直接安装)
编辑/etc/supervisor.d/davmail.ini

```configuration
[program:DavMail]
command=/opt/davmail/davmail.sh /etc/davmail.properties
user=root
autostart=true
autorestart=true
startsecs=120
stopasgroup=true
ikillasgroup=true
startretries=1
```

最后使用 supervisorctl reload 即可
这样我们的邮件交换久配置完成了

# fetchmail 收取邮件

## 安装

不在赘述（apt、yum、pacman）直接安装

## 配置

~/.fetchmailrc （没有就自己创建一个）

```configuration
defaults
mda "/usr/bin/procmail -d %T"

set idfile /home/luwenzheng/Maildir/.fetchids""
set no bouncemail
set postmaster "luwenzheng"

poll 127.0.0.1 with protocol imap port 1143 uidl
username "luwenzheng@genscript.com" there with password "********" is 'luwenzheng' here
options keep
```

# promail 过滤邮件

## 安装

不在赘述（apt、yum、pacman）直接安装

## 配置

~/.procmailrc (没有就自己创建一个)

```configuration
MAILDIR=/home/luwenzheng/Maildir   #邮件存储地址
DEFAULT=$MAILDIR/inbox   #默认：收件箱
VERBOSE=off
LOGFILE=/var/log/Mail/procmail.log

# 某个垃圾邮件规则
:0
* ^From: webmaster@st\.zju\.edu\.cn
/dev/null    #垃圾文件的存储位置

# 其它所有都存到收件箱中
:0:
inbox/
```

procmail 中的路径需要自己创建
接下来你就可以使用 fetchmail -a 收取全部邮件了

# mstmp 发送邮件

## 安装

不在赘述（apt、yum、pacman）直接安装

## 配置

~/.mstmprc （没有就自己创建一个）

```configuration
defaults
logfile /home/luwenzheng/Maildir/msmtp.log
account genscript
host 127.0.0.1
from luwenzheng@genscript.com
user luwenzheng
password ********
port 1025
auth plain
account default:genscript
```

# neomutt （邮件客户端）

## 安装

不在赘述（apt、yum、pacman）直接安装

## 配置

~/.config/neomutt 路径下有大量配置文件，各类个性话定制详细配置请查看我的 dotfile

