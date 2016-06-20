## 一个 jsmpeg 部署和使用示例

1. 下载安装必要软件

	`node`, `ffmpeg-static`	均可在网上下载相应平台的版本

	比如：（Windows 版）

	* https://nodejs.org/en/
	* https://ffmpeg.zeranoe.com/builds/ 

	所需的 js 和 html 文件的项目仓库地址：（包含接口使用文档）
	* https://github.com/phoboslab/jsmpeg

2. 启动基于 node.js 的 websocket 转发服务器。

	该服务器会绑定两个端口，分别用来监听 http 流量 和 分发 websocket 流量。

	websocket 用来和客户端交互， http 端口用来和 视频采集端 交互 

	首先给 node 安装 websocket 包

	```
	npm install ws
	```

	启动 服务器

	```
	node stream-server.js 你的密码
	```
	
	如果显示

	```
	Listening for MPEG Stream on http://127.0.0.1:8082/<secret>/<width>/<height>
	Awaiting WebSocket connections on ws://127.0.0.1:8084/
	```
	
	则表示 websocket 转发服务器启动成功。

3. 查找电脑摄像头
	
	**以 Windows 为例**

	启动 ff-prompt.bat，在 FF 命令行界面

	```
	ffmpeg -list_devices true -f dshow -i dummy
	```

	比如我的输出内容为：

	```
	[dshow @ 00000000006124a0] DirectShow video devices (some may be both video and audio devices)
	[dshow @ 00000000006124a0]  "Integrated Camera"
	[dshow @ 00000000006124a0]     Alternative name "@device_pnp_\\?\usb#vid_05ca&pid_18ff&mi_00#7&15873ce4&0&0000#{65e8773d-8f56-11d0-a3b9-00a0c9223196}\global"
	[dshow @ 00000000006124a0] DirectShow audio devices
	[dshow @ 00000000006124a0]  "楹﹀厠椋?(Conexant 20671 SmartAudio HD)"
	[dshow @ 00000000006124a0]     Alternative name "@device_cm_{33D9A762-90C8-11D0-BD43-00A0C911CE86}\wave_{954B656C-A73C-450A-97D2-FD388711F962}"
	```

	则我的视频设备是 "Integrated Camera"

	测试视频设备

	```
	ffplay -f dshow -i video="Integrated Camera"
	```

4. 采集摄像头视频，并发送到转发服务器

	在 FF 命令提示符中执行

	linux:
	```
	ffmpeg -s 640x480 -f video4linux2 -i /dev/video0 -f mpeg1video -b 800k -r 30 http://127.0.0.1.com:8082/你的密码/640/480/
	```

	Windows:
	```
	ffmpeg -s 640x480 -f dshow -i video="Integrated Camera" -f mpeg1video -b 800k -r 30 http://127.0.0.1:8082/1234/640/480
	```

	服务端和采集端可以不在同一台主机上，如不在同一主机，这里 只要将 `127.0.0.1` 修改 相应的 域名 或 ip 地址即可

5. 打开客户端 stream-example.html

	启动客户端之前要先修改示例代码中的 websocket 服务器地址
	`ws://example.com:8084/`
	要改成相应的服务器地址，比如 `ws://127.0.0.1:8084`

	```
	...
	<script type="text/javascript" src="jsmpg.js"></script>
	<script type="text/javascript">
		// Setup the WebSocket connection and start the player
		//var client = new WebSocket( 'ws://example.com:8084/' );
		var client = new WebSocket('ws://127.0.0.1:8084/');

		var canvas = document.getElementById('videoCanvas');
		var player = new jsmpeg(client, {canvas:canvas});
	</script>
	...
	```

	双击打开 stream-example.html , 或放到 一个 http 服务器里 执行。