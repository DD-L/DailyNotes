### 利用 [daocloud.io](https://www.daocloud.io/)「胶囊主机」 体验 `lproxy` 远程代理服务

预先准备：

1. 本地需要一个**最新版本的** `lproxy` local 端程序：`lsslocal.exe` （[Download](https://github.com/DD-L/lproxy/releases)）。
2. 注册一个 [daocloud.io](https://www.daocloud.io/) 账号。

手把手的图文教程：

1. **第一步：创建试用主机**

	1. **进入 [daocloud.io](https://www.daocloud.io/) 控制台，点击 “我的集群”。**

		![我的集群](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud/1.png)

	2. **在“自有集群”中添加主机。**

		![添加主机](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud//2.png)

	3. **选择“试用”。**

		 ![试用](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud/3.png)

	4. **点击“创建”。**

		![创建](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud//4.png)

	5. **“查看新主机”。**

		![查看新主机](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud/5.png)

	6. **“Try_DaoCloud_1” 「胶囊主机」创建完毕。**

		![胶囊主机](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud/6.png)

2. **第二步：创建 lproxy 应用**

	1. **进入[daocloud.io](https://www.daocloud.io/) 控制台，点击“应用管理”。**

		![应用管理](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud/7.png)

	2. **“创建应用”。**

		![创建应用](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud/8.png)

	3. **选择“DockerHub镜像”，并搜索 `deel/lproxy`。**

		![搜索 deel/lproxy](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud/9.png)

	4. **部署 `deel/lproxy`。**

		![部署 deel/lproxy](./10.png)

	5. **“部署最新版本”。**

		![部署最新版本](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud/11.png)

	6. **取名 `lproxy` , 在“我的主机”里面选中刚刚创建的主机“Try_DaoCloud_1”。**

		![](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud/12.png)

	7. **配置“基础设置”。**

		![基础设置](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud/13.png)

	8. **点击 “+ Add Port”。**

		![add port](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud/14.png)

	9. **添加 TCP `8088` 端口。**

		![8088](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud/15.png)

	10. **配置“高级设置”。**

		![高级设置](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud/16.png)

	11. **指定容器启动命令 `./lssserver.exe -k` （启动 server 端程序）, 并“ 立即部署”。**

		* `-k` 选项即 `--keep-running`，如果想让 `lssserver.exe` 做“不死小强”，那就添加这个选项吧 :v: （关闭由 `-k` 启动的 lproxy 服务，linux 上可以使用 `kill -9 $(ps aux | grep 'lss\w\{1,6\}\.exe' | grep -v grep | awk '{print $2}')` 命令来关闭）
		*  因为 `deel/lproxy` 通常都是最新版，所以为了避免不兼容的情况，后面使用的 local 端也要最新版本
		* 吐槽一下，不知道是不是 daocloud.io 官方做了什么人为的设置，部署时 pull 的速度越来越慢。一个月前（2016/04）还非常迅速，立刻就能部署起来，我最近几次的经历是，pull 下来需要近 20 分钟！！！所以，如果碰到假死的情况，耐心等一等就好了。

		![](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud/17.png)

3. **第三步：配置本地 local 端配置文件，并在本地启动 local 端程序**

	1. **获取服务端 ip 和 port。**

		**进入控制台，点击“我的集群”，“自有集群”中点击“管理主机”, 进入“Try_DaoCloud_1” 主机管理页面。也就是第一步进入的那个页面。记下外网 ip 和 映射的端口。**

		![ip&port](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud/18.png)

	2. **将本地 local 端配置文件`local-config.json` 中的 `server_name` 和 `server_port` 分别改成刚才记下的 ip 和 端口。**

		![local-config.json](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud/19.png)

	3. **启动 local 端程序 `./lsslocal.exe -c /path/to/local-config.json`。**

		*下图只是 local 端用 docker 启动的示例，可选择其他方法启动 `lsslocal.exe`*

		![lsslocal.exe](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud/20.png)

	*[Windows 平台的 lproxy](https://github.com/DD-L/lproxy/releases) local 端二进制程序 `lsslocal.exe`, 使用起来更方便：修改完 `local-config.json` 当中的 `server_name` 和 `server_port` 之后，直接用鼠标双击 `lsslocal.exe` 文件即可*

4. **第四步：查明 `lsslocal.exe` 所在主机的 ip，然后让你的应用软件走 local 端的 Socks5 网络代理，比如 `socks5://127.0.0.1:8087`。设置 Socks5 代理的方法，这里就不介绍了。**

	**ip 已更改，大功告成**

	![baidu](https://github.com/DD-L/DailyNotes/raw/master/lproxy/demo_on_daocloud/21.png)

	世界五大王牌情报组织：`CIA` - 中情局、`KGB` - 克格勃、`MOSSAD` - 摩萨德、`MI6` - 军情六处 和 `BJCYQZ` - **北京朝阳群众**，想想有点小激动呢。
