---
title: Yii2框架中初始化脚本init代码解读
date: 2018-09-04 20:39:23
categories: [PHP, Yii2]
tags: [PHP, Yii2]
---
代码是从最新的[yii-advanced-app-2.0.15.tgz](https://github.com/yiisoft/yii2/releases/download/2.0.15/yii-advanced-app-2.0.15.tgz)中直接获取，加以注释。

详情看代码注释：

<!--more-->

```php
#!/usr/bin/env php
<?php
/**
 * Yii Application Initialization Tool
 *
 * In order to run in non-interactive mode:
 *
 * init --env=Development --overwrite=n
 */
if (!extension_loaded('openssl')) {
    die('The OpenSSL PHP extension is required by Yii2.');
}

// 用于获取init后面的参数
$params = getParams();
$root = str_replace('\\', '/', __DIR__);
// 从`$root/environments/index.php`获取需要处理的文件列表
$envs = require "$root/environments/index.php";
// 两个一级键值是[Development]和[Production]
$envNames = array_keys($envs);

echo "Yii Application Initialization Tool v1.0\n\n";

$envName = null;
// 可以接受不带--env参数，或者--env=1
if (empty($params['env']) || $params['env'] === '1') {
    echo "Which environment do you want the application to be initialized in?\n\n";
    foreach ($envNames as $i => $name) {
        echo "  [$i] $name\n";
    }
    echo "\n  Your choice [0-" . (count($envs) - 1) . ', or "q" to quit] ';
    // 从获取输入
    $answer = trim(fgets(STDIN));
    // bool ctype_digit ( string $text ) 做纯数字检测
    if (!ctype_digit($answer) || !in_array($answer, range(0, count($envs) - 1))) {
        echo "\n  Quit initialization.\n";
        exit(0);
    }
    // 如果是合法的环境，则往下初始化环境
    if (isset($envNames[$answer])) {
        $envName = $envNames[$answer];
    }
} else {    
// 如果不是if的两种情况，则直接往下检查环境
    $envName = $params['env'];
}
// 非[Development]或[Production]，报错
if (!in_array($envName, $envNames)) {
    $envsList = implode(', ', $envNames);
    echo "\n  $envName is not a valid environment. Try one of the following: $envsList. \n";
    exit(2);
}
// 从`$envs`中抽取用户选择的环境[Development]或[Production]
$env = $envs[$envName];

// 如果用户并非预输入了确定的环境，则二次询问用户
if (empty($params['env'])) {
    echo "\n  Initialize the application under '{$envNames[$answer]}' environment? [yes|no] ";
    $answer = trim(fgets(STDIN));
    // 只检查单词第一位是否是`y`，所以随意yadbfda都可以吧。实测可以。
    if (strncasecmp($answer, 'y', 1)) {
        echo "\n  Quit initialization.\n";
        exit(0);
    }
}

echo "\n  Start initialization ...\n\n";
// `$env['path']`中，两种情形下分别是`dev`和`prod`
$files = getFileList("$root/environments/{$env['path']}");
// 如果有`skipFiles`键，则将里面的文件列表
// 则将这些文件，与对应环境[dev]或者[prod]文件夹中的文件进行剔除
if (isset($env['skipFiles'])) {
    $skipFiles = $env['skipFiles'];
    array_walk($skipFiles, function(&$value) use($env, $root) { $value = "$root/$value"; });
    $files = array_diff($files, array_intersect_key($env['skipFiles'], array_filter($skipFiles, 'file_exists')));
}
$all = false;
// 将处理后的文件，复制到对应的路径中
foreach ($files as $file) {
    if (!copyFile($root, "environments/{$env['path']}/$file", $file, $all, $params)) {
        break;
    }
}
// 二级键中，除了`path`用于标识环境文件夹
// 其余的均是回调函数名
$callbacks = ['setCookieValidationKey', 'setWritable', 'setExecutable', 'createSymlink'];
foreach ($callbacks as $callback) {
    if (!empty($env[$callback])) {
        $callback($root, $env[$callback]);
    }
}

echo "\n  ... initialization completed.\n\n";

function getFileList($root, $basePath = '')
{
    $files = [];
    $handle = opendir($root);
    while (($path = readdir($handle)) !== false) {
        // 忽略`.git`|'.svn'|'.'|'..'`四种路径
        if ($path === '.git' || $path === '.svn' || $path === '.' || $path === '..') {
            continue;
        }
        $fullPath = "$root/$path";
        $relativePath = $basePath === '' ? $path : "$basePath/$path";
        if (is_dir($fullPath)) {
            $files = array_merge($files, getFileList($fullPath, $relativePath));
        } else {
            $files[] = $relativePath;
        }
    }
    closedir($handle);
    return $files;
}

function copyFile($root, $source, $target, &$all, $params)
{
    // 如果源文件不存在，则结束程序
    if (!is_file($root . '/' . $source)) {
        echo "       skip $target ($source not exist)\n";
        return true;
    }
    // 如果目标文件存在
    if (is_file($root . '/' . $target)) {
        // 如果目标文件与源文件内容相同，则直接返回文件未改变
        if (file_get_contents($root . '/' . $source) === file_get_contents($root . '/' . $target)) {
            echo "  unchanged $target\n";
            return true;
        }
        // $all默认为`false`，不过会随着选择和最开始的参数，而发生变化
        if ($all) {
            echo "  overwrite $target\n";
        } else {
            echo "      exist $target\n";
            echo "            ...overwrite? [Yes|No|All|Quit] ";

            // 先判断是否在最开始带了`--overwrite=n`参数，带则依据最初的参数优先，否则读取用户输入
            $answer = !empty($params['overwrite']) ? $params['overwrite'] : trim(fgets(STDIN));
            if (!strncasecmp($answer, 'q', 1)) {
                // 如果参数是`q`，则直接跳过覆写的文件
                return false;
            } else {
                if (!strncasecmp($answer, 'y', 1)) {
                    // 选择为`y`的时候，只是当前文件覆写，后面还会询问
                    echo "  overwrite $target\n";
                } else {
                    if (!strncasecmp($answer, 'a', 1)) {
                        // 当选项为`a`，后面的文件直接就全部覆写了
                        echo "  overwrite $target\n";
                        $all = true;
                    } else {
                        // 其他任意键均跳过
                        echo "       skip $target\n";
                        return true;
                    }
                }
            }
        }
        // 将已有文件覆写到旧文件中
        file_put_contents($root . '/' . $target, file_get_contents($root . '/' . $source));
        return true;
    }
    echo "   generate $target\n";
    // 新建文件夹
    // 【注】文件操作，屏蔽了错误提示
    @mkdir(dirname($root . '/' . $target), 0777, true);
    // 生成文件
    file_put_contents($root . '/' . $target, file_get_contents($root . '/' . $source));
    return true;
}

function getParams()
{
    $rawParams = [];
    if (isset($_SERVER['argv'])) {
        $rawParams = $_SERVER['argv'];
        array_shift($rawParams);
    }

    $params = [];
    foreach ($rawParams as $param) {
        // 匹配--env=1类似格式
        // 可用参数选项`init --env=Development --overwrite=n`
        if (preg_match('/^--(\w+)(=(.*))?$/', $param, $matches)) {
            $name = $matches[1];
            $params[$name] = isset($matches[3]) ? $matches[3] : true;
        } else {
            $params[] = $param;
        }
    }

    return $params;
}

function setWritable($root, $paths)
{
    foreach ($paths as $writable) {
        if (is_dir("$root/$writable")) {
            // 【注】文件操作，屏蔽了错误提示
            if (@chmod("$root/$writable", 0777)) {
                echo "      chmod 0777 $writable\n";
            } else {
                printError("Operation chmod not permitted for directory $writable.");
            }
        } else {
            printError("Directory $writable does not exist.");
        }
    }
}

function setExecutable($root, $paths)
{
    foreach ($paths as $executable) {
        if (file_exists("$root/$executable")) {
            // 【注】文件操作，屏蔽了错误提示
            if (@chmod("$root/$executable", 0755)) {
                echo "      chmod 0755 $executable\n";
            } else {
                printError("Operation chmod not permitted for $executable.");
            }
        } else {
            printError("$executable does not exist.");
        }
    }
}

function setCookieValidationKey($root, $paths)
{
    foreach ($paths as $file) {
        echo "   generate cookie validation key in $file\n";
        $file = $root . '/' . $file;
        $length = 32;
        // openssl_random_pseudo_bytes(int $length [, bool &$crypto_strong ]) — 生成一个伪随机字节串
        $bytes = openssl_random_pseudo_bytes($length);
        // 生成的32位随机字节串，在用base64编码，取32位
        // 再分别将字符串中的 '+'=>'_' , '/'=>'-' , '='=>'.' 一一对应进行替换
        $key = strtr(substr(base64_encode($bytes), 0, $length), '+/=', '_-.');
        // 对内容进行替换，即生成了`cookieValidationKey`的值
        $content = preg_replace('/(("|\')cookieValidationKey("|\')\s*=>\s*)(""|\'\')/', "\\1'$key'", file_get_contents($file));
        file_put_contents($file, $content);
    }
}
// 建立符号连接
function createSymlink($root, $links)
{
    foreach ($links as $link => $target) {
        //first removing folders to avoid errors if the folder already exists
        @rmdir($root . "/" . $link);
        //next removing existing symlink in order to update the target
        if (is_link($root . "/" . $link)) {
            @unlink($root . "/" . $link);
        }
        if (@symlink($root . "/" . $target, $root . "/" . $link)) {
            echo "      symlink $root/$target $root/$link\n";
        } else {
            printError("Cannot create symlink $root/$target $root/$link.");
        }
    }
}

/**
 * Prints error message.
 * @param string $message message
 */
function printError($message)
{
    echo "\n  " . formatMessage("Error. $message", ['fg-red']) . " \n";
}

/**
 * Returns true if the stream supports colorization. ANSI colors are disabled if not supported by the stream.
 *
 * - windows without ansicon
 * - not tty consoles
 *
 * @return boolean true if the stream supports ANSI colors, otherwise false.
 */
function ansiColorsSupported()
{
    return DIRECTORY_SEPARATOR === '\\'
        ? getenv('ANSICON') !== false || getenv('ConEmuANSI') === 'ON'
        : function_exists('posix_isatty') && @posix_isatty(STDOUT);
}

/**
 * Get ANSI code of style.
 * @param string $name style name
 * @return integer ANSI code of style.
 */
function getStyleCode($name)
{
    $styles = [
        'bold' => 1,
        'fg-black' => 30,
        'fg-red' => 31,
        'fg-green' => 32,
        'fg-yellow' => 33,
        'fg-blue' => 34,
        'fg-magenta' => 35,
        'fg-cyan' => 36,
        'fg-white' => 37,
        'bg-black' => 40,
        'bg-red' => 41,
        'bg-green' => 42,
        'bg-yellow' => 43,
        'bg-blue' => 44,
        'bg-magenta' => 45,
        'bg-cyan' => 46,
        'bg-white' => 47,
    ];
    return $styles[$name];
}

/**
 * Formats message using styles if STDOUT supports it.
 * @param string $message message
 * @param string[] $styles styles
 * @return string formatted message.
 */
function formatMessage($message, $styles)
{
    if (empty($styles) || !ansiColorsSupported()) {
        return $message;
    }

    return sprintf("\x1b[%sm", implode(';', array_map('getStyleCode', $styles))) . $message . "\x1b[0m";
}

```