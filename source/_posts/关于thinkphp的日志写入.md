---
title: 关于Thinkphp的日志写入
tags:
  - php
  - thinkphp
id: '98'
categories:
  - - PHP
date: 2017-02-14 18:37:54
---
一、配置文件 TP通过配置文件对日志进行设置.要开启日志记录，必须在配置中开启LOG\_RECORD参数，以及可以在项目配置文件中配置需要记录的日志级别

```
'LOG_RECORD'            =>  true,  // 进行日志记录
'LOG_EXCEPTION_RECORD'  =>  true,    // 是否记录异常信息日志
'LOG_LEVEL'             =>  'EMERG,ALERT,CRIT,ERR,WARN,NOTIC,INFO,DEBUG,SQL',  // 允许记录的日志级别
```
EMERG 严重错误，导致系统崩溃无法使用 

ALERT 警戒性错误， 必须被立即修改的错误 CRIT 临界值错误， 超过临界值的错误 

ERR 一般性错误 WARN 警告性错误， 需要发出警告的错误 

NOTICE 通知，程序可以运行但是还不够完美的错误

INFO 信息，程序输出信息 

DEBUG 调试，用于调试信息 

SQL SQL语句，该级别只在调试模式开启时有效 
根据TP的define(‘APP\_DEBUG’,false)的设置不同，TP会读取不同的配置文件, 

**当Debug=true时，TP默认对debug模式设置了如下Log输出,文件位置为ThinkPhp/Conf/debug.php. 无论你项目config.php怎么配置LOG，都会是以下Level=EMERG,ALERT,CRIT,ERR,WARN,NOTIC,INFO,DEBUG,SQL.即默认输出所有Log。**

```
    return  array(
    'LOG_RECORD'            =>  true,  // 进行日志记录
    'LOG_EXCEPTION_RECORD'  =>  true,    // 是否记录异常信息日志
    'LOG_LEVEL'             =>  'EMERG,ALERT,CRIT,ERR,WARN,NOTIC,INFO,DEBUG,SQL',  // 允许记录的日志级别
    'DB_FIELDS_CACHE'       =>  false, // 字段缓存信息
    'DB_SQL_LOG'            =>  true, // 记录SQL信息
    'APP_FILE_CASE'         =>  true, // 是否检查文件的大小写 对Windows平台有效
    'TMPL_CACHE_ON'         =>  false,        // 是否开启模板编译缓存,设为false则每次都会重新编译
    'TMPL_STRIP_SPACE'      =>  false,       // 是否去除模板文件里面的html空格与换行
    'SHOW_ERROR_MSG'        =>  true,    // 显示错误信息
    );

```

此时如果想要使用自己的配置，需要在项目目录下添加/Conf/debug.php进行配置。 

**当Debug=false时，才会读取config.php的Log配置，即此时config.php配置才会生效。** 

二、核心类TP/Lib/Core/Log.class.php 

Tp的日志输出核心类为 TP/Lib/Core/Log.class.php. 

参照源码和文档看，得出以下结论。 

1>. TP框架一般使用的record+save 

2>. TP框架有两种记录日志的方式:write,record+save 

3>. write写入日志是不受Log配置文件控制的，无论你怎么设置，都会输出写入文件. 

4>. 所以怎么使用要视自己的需求选择 文档介绍为：

> 通常日志文件的写入是自动完成的，如果我们需要在开发的过程中手动记录日志信息，可以使用Log类的方法来操作。日志文件的写入有两种方法： 
>
> 一、使用Log::write 方法
>
>  Log::write 直接写入日志 
>
> 用法 
>
> Log::write($message,$level\=self::ERR,$type\='',$destination\='',$extra\='') 
参数 
message（必须）：要记录的日志信息，字符串 
level（可选）：要记录的日志级别，默认为ERR 错误 
type（可选）：日志记录方式，默认为空取LOG\_TYPE配置 
destination（可选）：日志记录目标，默认为空自动生成或LOG\_DEST配置 
extra（可选）：日志记录额外参数，默认为空取LOG\_EXTRA配置 
返回值 无 
使用示例： 
Log::write('调试的SQL：'.$SQL, Log::SQL); 
表示用默认的日志记录方式记录调试SQL信息
>
> 二、使用Log::record和 Log::save方法 
用法 Log::record($message,$level\=self::ERR,$record\=false) 
Log::record记录日志 
用法 Log::record($message,$level\=self::ERR,$record\=false) 
参数 
message（必须）：要记录的日志信息，字符串 
level（可选）：要记录的日志级别，默认为ERR 错误 
record（可选）：是否强制记录，默认为false表示判断LOG\_LEVEL配置 
返回值 无 
Log::record方法必须结合Log::save方法才能完成日志记录，因为record方法只是把日志信息保存到内存，并没有真正写入日志，直到调用Log::save方法。 
Log::save 保存记录的日志 
用法 Log::save($type\='',$destination\='',$extra\='') 
参数 
type（可选）：日志记录方式，默认为空取LOG\_TYPE配置 
destination（可选）：日志记录目标，默认为空自动生成或LOG\_DEST配置 
extra（可选）：日志记录额外参数，默认为空取LOG\_EXTRA配置 
使用示例： 
Log::record('测试调试错误信息', Log::DEBUG); 
Log::record('调试的SQL：'.$SQL, Log::SQL); 
Log::save();

参考源码为:

```php
/**
 * 日志处理类
 * @category   Think
 * @package  Think
 * @subpackage  Core
 * @author    liu21st <liu21st@gmail.com>
 */
class Log
{

    // 日志级别 从上到下，由低到高
    const EMERG = 'EMERG';  // 严重错误: 导致系统崩溃无法使用
    const ALERT = 'ALERT';  // 警戒性错误: 必须被立即修改的错误
    const CRIT = 'CRIT';  // 临界值错误: 超过临界值的错误，例如一天24小时，而输入的是25小时这样
    const ERR = 'ERR';  // 一般错误: 一般性错误
    const WARN = 'WARN';  // 警告性错误: 需要发出警告的错误
    const NOTICE = 'NOTIC';  // 通知: 程序可以运行但是还不够完美的错误
    const INFO = 'INFO';  // 信息: 程序输出信息
    const DEBUG = 'DEBUG';  // 调试: 调试信息
    const SQL = 'SQL';  // SQL：SQL语句 注意只在调试模式开启时有效

    // 日志记录方式
    const SYSTEM = 0;
    const MAIL = 1;
    const FILE = 3;
    const SAPI = 4;

    // 日志信息
    static $log = array();

    // 日期格式
    static $format = '[ c ]';

    /**
     * 记录日志 并且会过滤未经设置的级别
     * @static
     * @access public
     * @param string $message 日志信息
     * @param string $level 日志级别
     * @param boolean $record 是否强制记录
     * @return void
     */
    static function record($message, $level = self::ERR, $record = false)
    {
        if ($record  false !== strpos(C('LOG_LEVEL'), $level)) {
            self::$log[] = "{$level}: {$message}\r\n";
        }
    }

    /**
     * 日志保存
     * @static
     * @access public
     * @param integer $type 日志记录方式
     * @param string $destination 写入目标
     * @param string $extra 额外参数
     * @return void
     */
    static function save($type = '', $destination = '', $extra = '')
    {
        if (empty(self::$log)) return;
        $type = $type ? $type : C('LOG_TYPE');
        if (self::FILE == $type) { // 文件方式记录日志信息
            if (empty($destination))
                $destination = C('LOG_PATH') . date('y_m_d') . '.log';
            //检测日志文件大小，超过配置大小则备份日志文件重新生成
            if (is_file($destination) && floor(C('LOG_FILE_SIZE')) <= filesize($destination))
                rename($destination, dirname($destination) . '/' . time() . '-' . basename($destination));
        } else {
            $destination = $destination ? $destination : C('LOG_DEST');
            $extra = $extra ? $extra : C('LOG_EXTRA');
        }
        $now = date(self::$format);
        error_log($now . ' ' . get_client_ip() . ' ' . $_SERVER['REQUEST_URI'] . "\r\n" . implode('', self::$log) . "\r\n", $type, $destination, $extra);
        // 保存后清空日志缓存
        self::$log = array();
        //clearstatcache();
    }

    /**
     * 日志直接写入
     * @static
     * @access public
     * @param string $message 日志信息
     * @param string $level  日志级别
     * @param integer $type 日志记录方式
     * @param string $destination  写入目标
     * @param string $extra 额外参数
     * @return void
     */
    static function write($message,$level=self::ERR,$type='',$destination='',$extra='') {
        $now = date(self::$format);
        $type = $type?$type:C('LOG_TYPE');
        if(self::FILE == $type) { // 文件方式记录日志
            if(empty($destination))
                $destination = C('LOG_PATH').date('y_m_d').'.log';
            //检测日志文件大小，超过配置大小则备份日志文件重新生成
            if(is_file($destination) && floor(C('LOG_FILE_SIZE')) <= filesize($destination) )
                  rename($destination,dirname($destination).'/'.time().'-'.basename($destination));
        }else{
            $destination   =   $destination?$destination:C('LOG_DEST');
            $extra   =  $extra?$extra:C('LOG_EXTRA');
        }
        error_log("{$now} {$level}: {$message}\r\n", $type,$destination,$extra );
        //clearstatcache();
    }
}
```
