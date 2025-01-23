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
#!/bin/bash

cd "$(dirname "$0")"
# 定义增量包目录和文件名
output_file=`date +%Y%m%d_%H%M`'_CP_UPDATE'.tar.gz

# 第一步：获取最新的提交版本
latest_commit=$(git rev-parse --short HEAD)
echo "最新提交版本: $latest_commit"

# 第二步：读取上次的提交版本
if [ ! -f lastHead.txt ]; then
  echo "错误: 找不到 lastHead.txt 文件"
  exit 1
fi

last_commit=$(cat lastHead.txt)
echo "上次提交版本: $last_commit"

# 第三步：获取两次提交版本之间的差异文件列表并输出到 update.txt
cd ..  # 回到项目根目录
git diff --name-only $last_commit $latest_commit | \
  grep -v "^update/" | \
  grep -v "^web/update.php" | \
  grep -v "^web/phpinfo.php" | \
  grep -v "^app/config/" | \
  grep -v "\.gitignore$" > update/updateFileList.txt
echo "差异文件列表已生成：updateFileList.txt"

# 第四步：根据 updateFileList.txt 复制文件到增量包目录
rm -rf update/update_code/*
mkdir -p update/update_code

while IFS= read -r file; do
  if [ -f "$file" ]; then
    # 创建目标目录并复制文件
    mkdir -p "update/update_code/$(dirname "$file")"
    cp "$file" "update/update_code/$file"
    echo "复制文件: $file"
  else
    echo "警告: 文件 $file 不存在，可能已删除"
  fi
done < update/updateFileList.txt

# 第五步：打包增量包目录成 tar.gz 文件
cd update/update_code

tar -czf $output_file --exclude=$output_file *
echo "增量包已打包: $output_file"

# 更新 lastHead.txt 文件为最新提交版本
cd ..
echo $latest_commit > lastHead.txt
echo "更新 lastHead.txt 完成，内容为最新版本号: $latest_commit"
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
define('ADMIN_USERNAME','admin');   // Admin Username
define('ADMIN_PASSWORD','admin12345+');   // Admin Password
define('SITE_PATH',dirname(dirname(__FILE__)));
define('UPDATE_ZIP_PATH',SITE_PATH.DIRECTORY_SEPARATOR.'zip_bak');
define('FUNC',isset($_GET['f'])?$_GET['f']:'Default');
// $GLOBALS
$func = 'update'.FUNC;
$GLOBALS['oldList'] = [];
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


    if( pathinfo($_FILES['file']['name'],PATHINFO_EXTENSION) !='gz'){
        $GLOBALS['tips'][] = 'Error:not allowed file type';return;
    }
    $res = copy($_FILES['file']['tmp_name'],UPDATE_ZIP_PATH.DIRECTORY_SEPARATOR.$_FILES['file']['name']);
    if( !$res){
        $GLOBALS['tips'][] = 'Error:copy file error';
        $GLOBALS['tips'][] = 'copy '.$_FILES['file']['tmp_name'].' to '.UPDATE_ZIP_PATH.DIRECTORY_SEPARATOR.$_FILES['file']['name'];
        return;
    }
    $md5key = md5(UPDATE_ZIP_PATH.DIRECTORY_SEPARATOR.$_FILES['file']['name']).date('YmdHis');
    $tmpDir = UPDATE_ZIP_PATH.DIRECTORY_SEPARATOR.$md5key;
    //创建临时文件夹
    mkdir($tmpDir);

    $gzFile = UPDATE_ZIP_PATH.DIRECTORY_SEPARATOR.$_FILES['file']['name'];

    $command = "tar -zxvf {$gzFile} -C {$tmpDir}";
    exec($command, $output, $returnVar);

    if($returnVar === 0){
        $scanList = scandir($tmpDir);
        $GLOBALS['update_list'] = $todoList = [];
        foreach( $scanList as $v){
            if( $v != '..' && $v!='.'){
                if( !in_array($v,['src','web','app','vendor'])){
                    $GLOBALS['tips'][] = 'ERROR: DIR '.$v.' is not Allowed  ,Update Error!<br/>';
                    return ;
                }else{
                    $todoList[] = $v;
                }
            }
        }
        foreach($todoList as $v){
            doUpdateList($tmpDir,$tmpDir.DIRECTORY_SEPARATOR.$v);
        }
        //删除临时目录
        deleteDirectory($tmpDir);

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
            // 获取目标路径
            $targetPath = str_replace($tmpDir, SITE_PATH, $dir.DIRECTORY_SEPARATOR.$v);
            $targetDir = dirname($targetPath);

            // 检查并创建目标目录
            if (!is_dir($targetDir)) {
                if (!mkdir($targetDir, 0775, true)) {
                    $GLOBALS['tips'][] = 'Error: Failed to create directory: ' . $targetDir;
                    continue;
                }
            }

            // 复制文件
            if (!copy($dir.DIRECTORY_SEPARATOR.$v, $targetPath)) {
                $GLOBALS['tips'][] = 'Error: Failed to copy file: ' . $targetPath;
                continue;
            }

            $GLOBALS['update_list'][] = $targetPath;
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

function deleteDirectory($dirPath) {
    if (!is_dir($dirPath)) {
        throw new InvalidArgumentException("$dirPath must be a directory");
    }
    if (is_file($dirPath)) {
        return unlink($dirPath);
    }
    foreach (scandir($dirPath) as $item) {
        if ($item == '.' || $item == '..') continue;
        $item = $dirPath . DIRECTORY_SEPARATOR . $item;
        if (is_dir($item)) {
            deleteDirectory($item);
        } else {
            unlink($item);
        }
    }
    return rmdir($dirPath);
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

