![CLEVER DATA GIT REPO](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/0-clever-data-github.png "李聪明的数据库")

# 查找孤立数据文件SQL
#### SFind Orphaned Data Files With SQL
**发布-日期: 2015年1月5日 (评论)**

![#](images/##############?raw=true "#")

## Contents

- [中文](#中文)
- [English](#English)
- [SQL Logic](#Logic)
- [Build Info](#Build-Info)
- [Author](#Author)
- [License](#License) 


## 中文
通常，无论何时进行数据库迁移，你都需要确定实际使用哪些数据文件，哪些不用。你始终可以对sys.master_files运行查询，并将physical_name与你可以在数据文件路径中看到的实际文件进行比较。这种工作量当然是很吓人的，
因为你可能需要筛选数百或数千个数据文件。
这种情况下，最佳的解决方案是构建一个OS中实际文件夹中存在的所有实际数据文件的列表。这样也许可以将该列表放入表格中，然后运行该表格，和SQL Server中当前与实时数据库相关联的文件进行比较。
以下逻辑（logic）正好利用扩展存储过程xp_dirtree。 要记住的一件事是它只会向你显示以.MDF，.NDF和.LDF形式孤立的数据文件。 另外需要注意的是列表将返回以下系统资源数据库文件:
不要动以下这些文件。 这些可以被忽略。
mssqlsystemresource.ldf
mssqlsystemresource.mdf
如果你用的自定义数据文件扩展名，或根本没有扩展名，那么这个逻辑（logic）可能不适合你，我希望你能修改逻辑（logic）并回发给其他人使用。
不管怎样，请使用用代码！ 希望你觉得它有用。

## English
Often times whenever you are performing a database migration you need to determine which data files are actually being used, and which are not. You can always run a query against the the sys.master_files, and compare the physical_name to the actual files you can see in the data file path. This of course can be daunting as you may have hundreds or thousands of data files to sift through.
The best solution in this case is to build a list of all the actual data files that exist in the actual folder within the OS. Maybe; put that list into a table, and then run a comparison between that table, and the files which exist within SQL Server that are currently associated with live databases.
The following logic does exactly that utilizing the extended stored procedure xp_dirtree. One thing to keep in mind is that it will show you ONLY the data files that are orphaned under the form of .MDF, .NDF, and .LDF.   One other thing to watch out for is the list will return the following System Resource DB files:
Leave these files in place.  These can be ignored.
mssqlsystemresource.ldf
mssqlsystemresource.mdf

If you have custom data file extensions, or no extensions at all, then this logic might not work for you so I encourage you to modify and post back for others to use.
Anyway; on with the code! Hope you find it useful.


---
## Logic
```SQL
use master;
set nocount on
 
--set the appropriate data file path.
--设置适当的数据文件路径。
declare @data_file_path varchar(255) 
set @data_file_path = 'D:\Microsoft SQL 2005\MSSQL.1\MSSQL\Data'
 
--create temporary table to capture file names from directory.
--创建临时表格以便从中捕获文件名。
if object_id('tempdb..#folderandfileinfo') is not null
 drop table #folderandfileinfo
 
create table #folderandfileinfo
(
 cid int identity(1,1) primary key clustered
, subdirectory varchar(255)
, depth int
, isfile int
)
 
--populate temporary table with file names using xp_dirtree
--使用xp_dirtree填充带有文件名的临时表格
insert into #folderandfileinfo
(
 [subdirectory]
, [depth]
, [isfile]
)
exec master..xp_dirtree
 @data_file_path
, 1
, 1
 
-- compare files found in the OS data file location to files associated with actual live databases.
-- WARNING: this does NOT take into consideration data files that have been detached. if you have detached
-- data files check with your DBA first before you clean up old orphaned data files.
--将从OS数据文件位置中找到的文件与实际实时数据库关联的文件进行比较。
--注意：代码不会考虑已分离的数据文件。 如果你有分离的数据文件，请在清理旧的孤立数据文件之前先检查DBA。

select
 'path location' = @data_file_path
, 'orphaned data files' = subdirectory
 
from
 #folderandfileinfo
where
 subdirectory like '%df' -- compares ONLY to .mdf, .ndf, and .ldf
 and subdirectory not in
 (
 select right(smf.physical_name, charindex('\', reverse('\' + smf.physical_name)) - 1) 
 from
 sys.master_files smf join sys.databases sd on smf.database_id = sd.database_id
 )
order by
 subdirectory asc


```



[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

- **李聪明的数据库 Lee's Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明的数据库** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明的数据库-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/1-clever-data-github.png "李聪明的数据库")

