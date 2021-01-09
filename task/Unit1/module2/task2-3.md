
## 通过DolphinDB GUI运行DolphinDB脚本

在DolphinDB GUI视窗左侧项目导航栏，右键单击workspace，选择New Project新建项目，例如：Demo。

![新建项目](images/single_GUI_newproject.PNG)

单击新建的项目左方的圆点以展开文件夹，右键单击scripts目录，并选择New File，输入新建脚本文件名，例如：demo.txt。

现在即可编写DolphinDB代码。在编辑器窗口输入以下DolphinDB代码：
```txt
n=1000000
date=take(2006.01.01..2006.01.31, n);
x=rand(10.0, n);
t=table(date, x);

login("admin","123456")
db=database("dfs://valuedb", VALUE, 2006.01.01..2006.01.31)
pt = db.createPartitionedTable(t, `pt, `date);
pt.append!(t);

pt=loadTable("dfs://valuedb","pt")
select top 100 * from pt
```

点击工具栏中的 ![execute](images/execute.JPG) 按钮以运行脚本。下图展示了运行结果：

![运行结果](images/single_GUI.PNG)

默认情况下，数据文件保存在DolphinDB部署包/server/local8848目录下。若需要修改保存数据文件的目录，可设置volumes配置参数。

> 注意：
> 1. 从V0.98版本开始，DolphinDB单实例支持分布式数据库。
> 2. DolphinDB GUI中的会话在用户关闭之前会一直存在。

## 修改配置

修改单节点的配置参数有以下两种方式：

- 修改配置文件dolphindb.cfg。

- 在命令行中启动节点时指定配置参数。例如，启动节点时指定端口号为8900，最大内存为4GB：

Linux系统：
```sh
./dolphindb -localSite localhost:8900:local8900 -maxMemSize 4
```

Windows系统：
```sh
dolphindb.exe -localSite localhost:8900:local8900 -maxMemSize 4
```

更多DolphinDB配置参数请查看[集群配置](https://www.dolphindb.cn/cn/help/ClusterSetup1.html)。

## 更多详细信息，请参阅DolphinDB帮助文档

- [中文](https://www.dolphindb.cn/cn/help/index.html)
- [英文](http://dolphindb.com/help/)
