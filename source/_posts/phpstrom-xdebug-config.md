title: phpstorm配置Xdebug进行调试PHP教程
date: 2015-10-28 16:27:42
categories:
- notes
tags:
- PHP
- Xdebug
---

## 我的环境



	操作系统：win7 64bit 旗舰版 Service Pack 1
	PHP版本：PHP/5.3.5
	Apache版本：Apache/2.0.63 (Win32) 
	PHPStorm版本：PhpStorm 9.0.2 试用版

### 1. 下载xdebug扩展
>Xdebug是一个开放源代码的PHP程序调试器(即一个Debug工具)，可以用来跟踪，调试和分析PHP程序的运行状况

Xdebug官方网站：[http://xdebug.org/download.php](http://xdebug.org/download.php)
一定要选择和自己环境相对应的版本，如果不知道，可以查看phpinfo `PHP Extension Build`一栏，我的值是`API20090626,TS,VC6`
所以下载的是`PHP 5.3 VC6 TS (32 bit) (MD5: 6baca9b47d6bbf2236561d37bec09a5e)`版本

Xdebug当前最新版本是2.3.2，为了匹配我的PHP环境，我下载的是php_xdebug-2.1.2-5.3-vc6.dll库，下载地址：[http://xdebug.org/files/php_xdebug-2.1.2-5.3-vc6.dll](http://xdebug.org/files/php_xdebug-2.1.2-5.3-vc6.dll)

### 2. PHP安装xdebug扩展

修改`php.ini`的配置，下面的配置仅供参考，路径要换成自己的！
我这里用的是相对路径（也可以换成绝对路径）
### 注意：php5.2以下使用zend_extension_ts,php5.3及以上使用zend_extension###
我第一次配置就是被这个点给坑了

	[Xdebug]
	zend_extension="..\..\xdebug\php_xdebug-2.1.2-5.3-vc6.dll"
	xdebug.profiler_enable=on 
	xdebug.trace_output_dir="..\..\xdebug\trace"
	xdebug.profiler_output_dir="..\..\xdebug\profiler"
	xdebug.remote_enable=on
	xdebug.remote_handler=dbgp
	xdebug.remote_host=localhost
	xdebug.remote_port=9000

    
重启服务器，刷新phpinfo()
如果界面里面显示有
### xdebug

<table>
    <tr> 
        <th>xdebug support</th>
        <th>enabled</th>
    </tr>
    <tr>
        <td>Version</td>
		<td>2.1.2</td>
    </tr>
</table>
则说明配置成功，如果没有，请检查步骤重试。

### 3.设置phpstorm
1. 打开phpstorm,点击setting-languages & Frameworks-PHP-Debug-Xdebug一项 ，填写在`php.ini`里面配置的参数 确定
![xdebug setting](http://i13.tietuku.com/db8138c162b74d6d.png)

2. 打开php项目，点击phpstorm-Run-Edit configruations
![Edit configurations](http://i11.tietuku.com/dcea2f96c90ed735.png)
点击左侧+，添加php web application
name 随便命名 
点击server 添加该项目的访问地址 `localhost` port `80` Debugger `Xdebug` 确定

3. 然后编辑start URL(即点击run是调用浏览器打开的地址) 确定
配置完成后，右上角会出现下图所示。
![setting finish](http://i13.tietuku.com/b76c07bf1c79eaf2.png)

4. 设置断点，开始调试吧。**coding and enjoy**!
![debugging](http://i11.tietuku.com/6d73059bcd99d69c.png)
