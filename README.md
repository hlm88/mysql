# mysql
第一种：新建批处理文件 backup.dat，里面输入以下代码：

 代码如下	复制代码
net stop mysql
xcopy "C:/Program Files/MySQL/MySQL Server 5.0/data/piaoyi/*.*" D:/db_backup/%date:~0,10%/ /y
net start mysql
注意：批处理命令中路径里有空格的话，必须在路径上加上双引号！
然后使用Windows的"计划任务"定时执行该批处理脚本即可。（例如：每天凌晨3点执行backup.bat）
解释：备份和恢复的操作都比较简单，完整性比较高，控制备份周期比较灵活。此方法适合有独立主机但对mysql没有管理经验的用户。缺点是占用空间比较多，备份期间mysql会短时间断开（例如：针对30M左右的数据库耗时5s左右）。  
关于时间参数的参考：
%date:~0,10%      //提取年月日信息
%date:~-3%         //提取星期几信息
%time:~0,5%         //提取时间中的时和分
%time:~0,-3%       //提取时和分和秒信息

第二种：mysqldump备份成sql文件
==============
假想环境：
MySQL   安装位置：C:/MySQL
论坛数据库名称为：bbs
MySQL root   密码：123456
数据库备份目的地：D:/db_backup/

脚本：

 代码如下	复制代码
@echo off
set "Ymd=%date:~,4%%date:~5,2%%date:~8,2%"
C:/MySQL/bin/mysqldump --opt -u root --password=123456 bbs > D:/db_backup/bbs_%Ymd%.sql
@echo on
将以上代码保存为backup_db.bat
然后使用Windows的"计划任务"定时执行该脚本即可。（例如：每天凌晨5点执行back_db.bat）
说明：此方法可以不用关闭数据库，并且可以按每一天的时间来名称备份文件。
通过%date:~5,2%来组合得出当前日期，组合的效果为yyyymmdd,date命令得到的日期格式默认为yyyy-mm-dd(如果不是此格式可以通过pause命令来暂停命令行窗口看通过%date:~,20%得到的当前计算机日期格式)，所以通过%date:~5,2%即可得到日期中的第五个字符开始的两个字符，例如今天为2009-02-05,通过%date:~5,2%则可以得到02。（日期的字符串的下标是从0开始的）

第三种：利用WinRAR对MySQL数据库进行定时备份。 
    对于MySQL的备份，好的方法是直接备份MySQL数据库的Data目录。下面提供了一个利用WinRAR来对Data目录进行定时备份的方法。

首先当然要把WinRAR安装到计算机上。

将下面的命令写入到一个文本文件里，如 backup.bat

 代码如下	复制代码
net stop mysql
"C:/Program Files/WinRAR/WinRAR.exe" a -ag -k -r -s D:/db_backup/mysql_.rar "C:/Program Files/MySQL/MySQL Server 5.0/data/"
net start mysql
winrar参数解释：
a： 添加文件到压缩文件
-ag： 使用当前日期生成压缩文件名
-k： 锁定压缩文件
-r： 递归子目录
-s： 创建固实压缩文件

   执行以上文件后，会生成一个压缩文件如：mysql_20130803004138.rar。
   进入控制面版，打开计划任务，双击"添加计划任务"。在计划任务向导中找到刚才的backup.bat文件，接着为这个任务指定一个运行时间和运行时使用的账号密码就可以了。
   这种方法缺点是占用时间比较多，备份期间压缩需要时间，mysql断开比第一种方法更多的时间，但是对于文件命名很好。

1.在D盘创建db_backup文件夹，并新建backdb.bat。

2.在backdb.bat里面加入一下代码：

 代码如下	复制代码
echo 取日期、时间变量值set yy=%date:~0,4%

set mm=%date:~5,2%

set dd=%date:~8,2%

if /i %time:~0,2% lss 10 set hh=0%time:~1,1%

if /i %time:~0,2% geq 10 set hh=%time:~0,2%

set mn=%time:~3,2%

set ss=%time:~6,2%

set date=%yy%%mm%%dd%

set time=%hh%%mn%%ss%

set filename=%date%_%time%


"C:/Program Files (x86)/MySQL/MySQL Server 5.0/bin/mysqldump.exe" -uroot -pxxx --opt --default-character-set=utf8 -e --triggers -R --hex-blob --flush-logs -x DBNAME > C:/db_backup/DBNAME%filename%.sql


echo 导出已经完成

#pause

在这里要注意你的MySQL安装路径以及相应的数据库用户名和密码，我使用的是D:/sense/mysql/bin。


3.双击运行此脚本，看是否会生成Dbname20111207_200445.sql文件，如有则脚本无错误。

4.进入控制面板，在任务计划里添加计划任务，把要执行的批处理以浏览方式加入任务计划，并设定好执行时间，最好选择每天执行，这样就实现每天自动备份数据库了。
