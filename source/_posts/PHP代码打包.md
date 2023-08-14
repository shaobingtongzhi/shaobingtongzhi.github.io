---
title: PHP 代码打包
date: 2023-08-01 20:19:48
tags: 
    - git
    - PHP
    - 代码打包
categories:
    - 实战
    - PHP 代码打包
---

# PHP 代码打包

1. shell脚本

```sh
#!/bin/sh
echo 'Step1.开始拉取最新代码>>'
git pull 
echo 'Step2.开始比较差异文件>>'
head=$(git rev-parse --short HEAD)
lastHead=$(cat lastHead.txt)
echo $head > lastHead.txt
echo `git diff --name-only $lastHead $head` >updateFileList.txt
echo 'Step3.将需要更新的文件拷贝到update目录下'
rm -rf update/*
php -f update.php
echo 'Step5.打包并记录本次更新内容到本地日志文件'
cd update
zip -r `date +%Y%m%d_%H%M`'_CCB_UPDATE'.zip ./*
cd ..
echo ''>>updatelog.txt
log='----- '`date +%Y%m%d-%H:%M`' 更新内容如下(请无视update目录下内容) -----'
echo $log >> updatelog.txt
cat updateFileList.txt | tr " " "\n" >> updatelog.txt
echo ''>>updatelog.txt
```

2. php脚本

```php
<?php
    $lists = file_get_contents('updateFileList.txt');
    $lists = explode(" ",trim($lists," "));
    //忽略文件列表
    foreach($lists as $v){
        $v = trim($v,"\n");
        $v = trim($v,' ');
        $v = trim($v,'"');
        $dir_name = dirname($v);
        if($v && !isIgnore($v,$dir_name)){
            $update_dir = 'update/'.$dir_name;
            if(!is_dir($update_dir)){
                mkdir($update_dir,0775,true);
            }
            copy('../'.$v,'update/'.$v);
            echo 'copy file :',$v,"\n";
        }else{
            echo 'isIgnore file :',$v,"\n";
        }
    }
    function isIgnore($v,$dir_name){
        $ignore_dir = ['up','config/lang'];
        $ignore_file = ['config/config.inc.php','config/conv.inc.php','.gitignore'];
        if(in_array($v,$ignore_file)){
            return true;
        }
        foreach($ignore_dir as $dir){  
            if(strpos($dir_name, $dir) === 0){
                return true;
            }
        }
        return false;
    }
```

3. 打更新包步骤

>  运行shell脚本即可

4. 打回滚包步骤

> a）从上次更新打包节点迁出一个临时分支
>
> b）复制本次打包生成的update/updateFileList.txt内容
>
> c）切换到迁出的分支，粘贴覆盖原update/updateFileList.txt内容
>
> d）修改update.sh，将step3之前的部分注视掉即可
>
> e）执行update.sh打包，修改更新包名称为日期_rollback.zip

# PHP 代码更新

```php
<?php
define('ADMIN_USERNAME','');   // Admin Username
define('ADMIN_PASSWORD','');   // Admin Password
define('SITE_PATH',dirname(dirname(__FILE__)));
define('UPDATE_ZIP_PATH',SITE_PATH.DIRECTORY_SEPARATOR.'zip_bak');
define('FUNC',isset($_GET['f'])?$_GET['f']:'Default');
// $GLOBALS
$func = 'update'.FUNC;
checkAuth();
if( $GLOBALS['is_login'] || $func == 'updateSetAuth' ){
    if( function_exists($func)){
        $func();
    }else{
        die('Error:no access');
    }
}


function updateSetAuth(){
    if( $_POST['user'] == ADMIN_USERNAME && $_POST['pwd'] ==ADMIN_PASSWORD  ){
        $time = time();
        setcookie('updateAdminKey',md5(ADMIN_USERNAME.$time).','.$time,time()+3600,'/','','',true);
        header('Location:?f=Default');exit();
    }else{
        $GLOBALS['tips'][] = 'ERROR: login error';
    }
}

function checkAuth(){
    $auth =  isset($_COOKIE['updateAdminKey']) ? ($_COOKIE['updateAdminKey']) : false;
    if( !$auth){
        $GLOBALS['is_login'] = false;
    }else{
        list($user,$time) = explode(',',$auth);
        if( $user == md5(ADMIN_USERNAME.$time)  && $time + 3600 > time()){
            $GLOBALS['is_login'] = true;
        }else{
            $GLOBALS['is_login'] = false;
        }
    }
}
//默认
function updateDefault(){

}

//上传处理
function updateUpload(){
    
    if( !is_dir(UPDATE_ZIP_PATH)){
        mkdir(UPDATE_ZIP_PATH,0775);
    }
    
    if( empty($_FILES['file']) ){
        $GLOBALS['tips'][] = 'Error:please upload file';return;
    }
    
    if( pathinfo($_FILES['file']['name'],PATHINFO_EXTENSION) !='zip'){
        $GLOBALS['tips'][] = 'Error:not allowed file type';return;
    }
    $res = copy($_FILES['file']['tmp_name'],UPDATE_ZIP_PATH.DIRECTORY_SEPARATOR.$_FILES['file']['name']);
    if( !$res){
        $GLOBALS['tips'][] = 'Error:copy file error';
        $GLOBALS['tips'][] = 'copy '.$_FILES['file']['tmp_name'].' to '.UPDATE_ZIP_PATH.DIRECTORY_SEPARATOR.$_FILES['file']['name'];
        return;
    }
    $zip = new ZipArchive();
    if( $zip->open( UPDATE_ZIP_PATH.DIRECTORY_SEPARATOR.$_FILES['file']['name'] ) === true){
        $md5key = md5(UPDATE_ZIP_PATH.DIRECTORY_SEPARATOR.$_FILES['file']['name']).date('YmdHis');
        $tmpDir = UPDATE_ZIP_PATH.DIRECTORY_SEPARATOR.$md5key;
        //创建临时文件夹
        mkdir(UPDATE_ZIP_PATH.DIRECTORY_SEPARATOR.$md5key);
        $zip->extractTo($tmpDir);
        $scanList = scandir($tmpDir);
        $GLOBALS['update_list'] = $todoList = [];
        foreach( $scanList as $v){
            if( $v != '..' && $v!='.'){
                if( !in_array($v,['apps','libs','framework','public','views','config'])){
                    $GLOBALS['tips'][] = 'ERROR: DIR '.$v.' is not Allowed  ,Update Error!<br/>';
                    $zip->close();
                    return ;
                }else{
                    $todoList[] = $v;
                }
            }
        }

        foreach($todoList as $v){
            doUpdateList($tmpDir,$tmpDir.DIRECTORY_SEPARATOR.$v);
        }
        //正式解压到站点目录下去，不要解压到其他地方去
        $zip->extractTo(SITE_PATH);
        $zip->close();
    }else{
        $GLOBALS['tips'][] = 'Error:zip file canot open';return ;
    }
}

function doUpdateList($tmpDir,$dir){
    $scanList = scandir($dir);
    foreach($scanList as $v){
        if($v != '..' && $v!='.' &&  is_dir($dir.DIRECTORY_SEPARATOR.$v) ){
            doUpdateList($tmpDir,$dir.DIRECTORY_SEPARATOR.$v);
        }elseif(is_file($dir.DIRECTORY_SEPARATOR.$v)){
            copy($dir.DIRECTORY_SEPARATOR.$v,str_replace($tmpDir,SITE_PATH,$dir.DIRECTORY_SEPARATOR.$v));
            $GLOBALS['update_list'][] = str_replace($tmpDir,SITE_PATH,$dir.DIRECTORY_SEPARATOR.$v);
        }
    }
}

function updateShowOld(){
    if( !is_dir(UPDATE_ZIP_PATH)){
        mkdir(UPDATE_ZIP_PATH,0775);
    }
    $list = scandir(UPDATE_ZIP_PATH);
    foreach($list as $v){
        if( $v !='..' && $v !='.'){
            $ctime = filectime(UPDATE_ZIP_PATH.DIRECTORY_SEPARATOR.$v);
            $GLOBALS['oldList'][] = ['name'=>$v,'ctime'=>date("Y/m/d H:i:s",$ctime)];
        }
    }
}

?>
<!DOCTYPE html>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge" >
<title>代码更新管理</title>
</head>
<body>
<?php if( !$GLOBALS['is_login'] || FUNC == 'SetAuth'){ ?>
    <form action="?f=SetAuth" method="post" enctype="multipart/form-data">
        <table class="tableborder" >
          <tr class="cell">
            <td class="altbg2">
            请先登录<br/>
            账号： <input type="text" name="user" class="txt" style="width:300px"/>
           <br/>
            密码： <input type="password" name="pwd"" class="txt" style="width:300px"/>
            <br/>
            <input type="submit" name="botton2" style="background-color:orange;color:#000;margin:2px" value=" 登录 "/></td>
          </tr>
        </table>
      </form> 
    <?php if(!empty($GLOBALS['tips'])){ ?>
    <ul>
        <?php foreach($GLOBALS['tips'] as $v){ ?>
        <li> <?php echo $v;?></li>
        <?php } ?>
    </ul>
    <?php } ?>
        </body>
    </html>  
<?php die(); } ?>
<div>
<pre>
    <h1>系统基本信息</h1>
    站点根目录：<?php echo SITE_PATH;?> <a href="?f=Default">更新代码包</a><br/>
    更新包存放目录：<?php echo UPDATE_ZIP_PATH;?> <a href="?f=ShowOld">查看历史更新包</a>
    <hr />
</pre>
</div>
<?php if(FUNC == 'Default'){ ?>
    <form action="?f=Upload" method="post" enctype="multipart/form-data">
        <table class="tableborder" style="width:500px;">
          <tr class="cell">
            <td class="altbg2"><label >
              上传代码包 <input type="file" name="file" class="txt" style="width:400px" style="float:left"/>
            </label>
            </td>
            <td><input type="submit" name="botton2" style="background-color:orange;color:#000;margin:2px" value=" 更新代码包 "/></td>
          </tr>
        </table>
      </form> 
<?php }elseif(FUNC == 'Upload'){ ?>
<div>
<?php if(!empty($GLOBALS['tips'])){ ?>
    <ul>
    <?php foreach($GLOBALS['tips'] as $v){ ?>
    <li> <?php echo $v;?></li>
    <?php } ?>
    </ul>
    <?php }else{ ?>
    <h2>代码更新成功！本次更新涉及目录：</h2>
    <ul>
    <?php foreach($GLOBALS['update_list'] as $v){ ?>
    <li> <?php echo $v;?></li>
    <?php } ?>
    </ul>
    </div>
    <?php }?>
<?php }elseif(FUNC == 'ShowOld'){ ?>
    <div>
    <h2>历史更新包列表：</h2>
    <table style="width:100%;text-align:left;">
        <tr>
            <th>更新包名称</th>
            <th>上传时间</th>
        </tr>
    <?php foreach($GLOBALS['oldList'] as $v){ ?>
        <tr>
            <td><?php echo $v['name'];?></td>
            <td><?php echo $v['ctime'];?></td>
        </tr>
    <?php }?>
    </table>
    </div>
<?php }?>
</body>
</html>
```

