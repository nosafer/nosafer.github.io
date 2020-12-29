# 前言


> 基于零组公开漏洞库

## 如何添加新的文章

```
先检查本地仓库是否为最新版本
找到对应分类或新建分类,新建Markdown文件，文件名为漏洞标题
Markdown文件内添加漏洞详情 
图片保存到当前Markdown文件路径下的`./resource/文件名/mdeia/` 目录，Markdown插入时使用相对路径
按时间倒序在Change Log中添加修改的内容
```

![image.png](https://i.loli.net/2020/10/15/MF94bHBscvjU85t.png)



# Online Version

[VulWiki](https://wiki.96.mk/) 



# Change Log

* 2020-12-29 添加 Docker 容器逃逸漏洞 (CVE-2020-15257)复现
* 2020-12-25 添加 狂雨CMS后台SQL代码执行、狂雨CMS后台文件包含getshell、狂雨CMS数据库备份地址爆破、MKCMS v7.0.3 sql注入漏洞审计、Nexus Repository Manager3 ProXXE分析(CVE-2020-29436)
* 2020-12-21 添加 Apache Unomi远程代码执行漏洞复现-CVE-2020-13942
* 2020-12-19 添加 PowerCreatorCms任意上传
* 2020-12-14 添加 74cms v6.0.48模版注入+文件包含getshell,CVE-2019-11580 Atlassian Crowd RCE,s2-061
* 2020-12-3 添加 ThinkAdmin未授权列目录任意文件读取(CVE-2020-25540)漏洞
* 2020-11-17 添加 CVE-2020-26217 XStream XML反序列化远程代码执行，Citrix XenMobile CVE-2020-8209
* 2020-11-3 添加 禅道<=12.4.2 后台getshell，windows本地提权漏洞，Linux本地提权漏洞
* 2020-10-28 添加 s2-059,CVE-2020-14882 weblogic 未授权命令执行，（CVE-2020-14825）Weblogic反序列化漏洞
* 2020-10-21 添加RuoYi CMS 任意文件读取漏洞
* 2020-10-20 添加护网中的漏洞,CVE-2020-10189 Zoho ManageEngine反序列化RCE,Fastjson Payload汇总，修复%20造成的侧栏折叠问题

# To-do

- [x] 在线版本 

# Web安全

- [x] 添加护网中的漏洞

### 系统安全

- [ ] 完善系统提权漏洞


**IOT安全**

- [ ]  Cisco

- [ ] （CVE-2020-3452）Cisco ASA/FTD 任意文件读取漏洞

  - [ ]  Hikvision

- [ ] （CVE-2017-7921）Hikvision IP Camera Access Bypass

  - [ ]  Hisilicon

- [ ] （CVE-2020-24214）Buffer%20overflow: definite DoS and potential RCE

- [ ] （CVE-2020-24215）HiSilicon Backdoor password

- [ ] （CVE-2020-24216）RTSP 未授权访问

- [ ] （CVE-2020-24217）任意文件上传漏洞

- [ ] （CVE-2020-24218）root access via telnet

- [ ] （CVE-2020-24219）任意文件读取漏洞

  

  - [ ]  ZTE

- [ ] （CVE-2020-6871）ZTE R5300G4、R8500G4和R5500G4 未授权访问漏洞

  

- [ ] 默认设备密码
