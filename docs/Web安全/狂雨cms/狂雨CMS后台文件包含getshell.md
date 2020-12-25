# 狂雨CMS后台文件包含getshell

## 审计

首先还是先看代码结构

```
addons          插件代码
application     主要后端代码
config          配置文件
extend          一些基础类文件
public          静态文件
route           路由
runtime         主要是缓存
template        模板文件
thinkphp        thinkphp
uploads         上传文件
```

主要的审计中心放在application下

### 文件包含

主要是提供了可修改模板功能，然后利用thinkphp的模板语法进行文件包含

主要代码

`application/admin/controller/Template.php->edit()`

```php
public function edit(){
        $Template=model('template');
        $data=$this->request->post();
        if($this->request->isPost()){
            $res = $Template->edit($data);
            if($res  !== false){
                return $this->success('模版文件修改成功！',url('index'));
            } else {
                $this->error($Template->getError());
            }
        }else{
            $path=urldecode($this->request->param('path'));
            $info=$Template->file_info($path);
            $this->assign('path',$path);
            $this->assign('content',$info);
            $this->assign('meta_title','修改模版文件');
            return $this->fetch();
        }
    }
```

跟入edit函数

`application/admin/model/Template.php->edit()`

```php
public function edit($data){
        return File::put($data['path'],$data['content']);
    }
```

继续跟入

`extend/org/File.php`

```php
static public function put($filename,$content,$type=''){
        $dir   =  dirname($filename);
        if(!is_dir($dir))
            mkdir($dir,0755,true);
        if(false === file_put_contents($filename,$content)){
            throw new \think\Exception('文件写入错误:'.$filename);
        }else{
            self::$contents[$filename]=$content;
            return true;
        }
    }
```

可以看到这里没经过任何过滤，直接进行了写入。关于这里的模板功能，肯定还有其他利用方式，就靠师傅们自己去测试了

### 防御方法

针对该处漏洞的防御，觉得还是对上传进行过滤比较容易，比如说进行内容的检查，或者对图片进行一定的改动譬如缩放，从而破坏图片马的结构，不过只是治标不治本，一旦找到新的上传方式，还是会被利用。

## 漏洞复现

后台可以修改模板文件，在模板文件中调用模板代码可实现文件包含

![Zm3a6wItfYr45sx](./resource/狂雨CMS后台文件包含getshell/media/Zm3a6wItfYr45sx.png)

首先在设置中上传logo处，上传图片马，需要注意不能是用户的头像上传处上传木马，因为头像会进行缩放，导致文件内容被修改

![XkU4DbL6ZfNOVGE](./resource/狂雨CMS后台文件包含getshell/media/XkU4DbL6ZfNOVGE.png)

上传成功后会返回路径，图片马内容为`<?php phpinfo(); ?>`，需要注意php代码必须包含最后的`?>`，否则会报语法错误

![JUqLGubNYmsD14i](./resource/狂雨CMS后台文件包含getshell/media/JUqLGubNYmsD14i.png)

进入模板功能处，在index.html，即主页模板中添加模板代码，同样需要注意，路径不能以/开头，否则会被找不到错误

```php
{include file="uploads/config/20200325/033966a7d27975812915522464e252a3.jpg" /}
```

![iDNHBzdJfhsjlFG](./resource/狂雨CMS后台文件包含getshell/media/iDNHBzdJfhsjlFG.png)

接着打开主页，即可看到phpinfo信息

## 参考

https://www.cnblogs.com/0daybug/p/12624364.html