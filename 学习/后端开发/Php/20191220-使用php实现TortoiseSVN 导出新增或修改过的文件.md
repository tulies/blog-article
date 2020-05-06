先前在 Windows 操作系统下，习惯用 TortoiseSVN 导出新增或修改过的文件（【相当实用】如何让TortoiseSVN导出新增或修改过的文件  ），最近换成了 Mac Pro 笔记本电脑，一时没找到类似 TortoiseSVN 好用的客户端工具。好吧，利用PHP写个导出小工具：

1、工具文件名：svn.php，其内容如下：
```php
<?php
/**
 * 导出指定版本之间的差异文件,如 100 和 200 之间的差异则导出 100(不包括) - 200(包括) 的所有修改
 * 【SVN命令行】
 * 1、查看版本间差异
 * svn diff -r 2359:2360 --summarize --username simon --password simonfit svn://112.73.80.56/SJF/source
 * 2、导出某个版本文件到本地
 * svn export -r 2360 svn://112.73.80.56/SJF/source/common/controller/WechatBaseController.class.php /root/2/files_2359_2360/common/controller/WechatBaseController.class.php --username wenjianbao --password wjb888
 *
 * @example svn.php 100 200
 * @author 52php.cnblogs.com
 */
 
// 根目录
define('SITE_PATH', dirname(__FILE__));
 
 
// SVN 账号信息
$svn_url = 'svn://112.73.80.56/SJF/source';
$svn_username = 'wenjianbao';
$svn_password = '5c95e61387c478c85ccf45e6a8ae6de3';
 
 
 
 
$error_msg = 'You must useage like ' . $_SERVER['argv'][0] . ' old_version(不包括) new_version(包括)';
if ($_SERVER['argc'] != 3) {
    echo $error_msg;
    exit(1);
}
 
if ($_SERVER['argv'][1] > $_SERVER['argv'][2]) {
    echo $error_msg;
    exit(1);
}
 
$old_version = $_SERVER['argv'][1];
$new_version = $_SERVER['argv'][2];
 
$work_path = SITE_PATH . "/file_${old_version}_${new_version}";
 
echo "开始分析版本差异...\n";
$diff_cmd = "svn diff -r ${old_version}:${new_version} --summarize --username ${svn_username} --password ${svn_password} ${svn_url}";
exec($diff_cmd, $diff_list, $return);
$diff_list = (array)$diff_list;
foreach ($diff_list as $diff_info) {
    echo $diff_info . "\n";
}
 
# 清空旧数据
@system('rm -rf ' . SITE_PATH . '/file_*');
@system('rm -rf ' . SITE_PATH . '/diff_*');
 
# 新建文件夹
dir_mkdir($work_path);
 
$diff_count = count($diff_list);
if ($diff_count < 1) {
    echo "版本间没有差异";
    exit(1);
}
 
$diff_count = 0;
$diff_file_path = SITE_PATH . "/diff_${old_version}_${new_version}.txt";
 
# 导出版本文件
echo "开始导出...\n";
foreach ($diff_list as $diff_info) {
    if (preg_match('/([\w]+)\s+(svn:.+)/', $diff_info, $matches)) {
        $svn_file_mode = $matches[1];
        $svn_file_name = $matches[2];
 
        // A、M、D、AM即增加且修改
        // 文件被删除
        if ($svn_file_mode == 'D') {
            continue;
        }
        $diff_count++;
 
        // 写日志
        file_write($diff_file_path, $matches[0] . "\n", 'a');
 
        // 下载到本地
        $local_file_path = $work_path . str_replace($svn_url, '', $svn_file_name);
        $local_file_dir = dirname($local_file_path);
        dir_mkdir($local_file_dir);
 
        $export_cmd = "svn export -r ${new_version} ${svn_file_name} ${local_file_path} --username ${svn_username} --password ${svn_password}";
        system($export_cmd);
    }
}
 
echo "共导出${diff_count}个差异文件";
exit(0);
 
 
 
/**
 * 创建文件夹
 *
 * @param string $path      文件夹路径
 * @param int    $mode      访问权限
 * @param bool   $recursive 是否递归创建
 * @return bool
 */
function dir_mkdir($path = '', $mode = 0777, $recursive = true) {
    clearstatcache();
    if (!is_dir($path)) {
        mkdir($path, $mode, $recursive);
        return chmod($path, $mode);
    }
 
    return true;
}
 
/**
 * 写文件
 *
 * @param string $filename 文件名
 * @param string $text     要写入的文本字符串
 * @param string $openmod  文本写入模式（'w':覆盖重写，'a':文本追加）
 * @return bool
 */
function file_write($filename = '', $text = '', $openmod = 'w') {
    if (@$fp = fopen($filename, $openmod)) {
        flock($fp, 2);
        fwrite($fp, $text);
        fclose($fp);
        return true;
    } else {
        return false;
    }
}

```

 

2、使用方法

在 svn.php 文件中修改下自己的 SVN服务器的地址和账号，以命令行模式运行下即可，如 导出版本号 100 到 200 之间的差异文件，

```shell
php svn.php 100 200
```