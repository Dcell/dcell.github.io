---
title: "html <audio> 标签无法在iOS和OXS上在播放问题"
layout: post
comments: true
---
## 前言
在公司待久了见多了部门之间协作不好，导致各种撕逼的情况。所以一般情况下，让兄弟部门解决问题，我都是提供了足够的证据，或者说已经找到了问题的根本。一般后端有问题呀，提供有力的抓包证据，并且给出文档链接，往往可以快速解决问题，而且能增加感情。
## 问题
在开发某个项目中，需要要求客户端上传音频文件并且可以播放。而我们播放文件是通过`html video`标签来实现的。在前期没啥问题，中期自测兼容性的时候出问题了，发现同样的标签在`iOS和Safari`上无法播放，返回格式不支持。同事直接把问题抛给了后端，后端一脸懵逼，`google浏览器和android`不是好的么？明显是你们前端的问题。一场撕逼大战开始了。
## 定位
为了方便调试，我随便用node js写了本地服务。开启8080端口，访问当前路径下的某个wav文件，则返回2进制流。

```
const {createServer:server}=require('http')
const url=require('url')
const fs=require('fs')
const wavPath='./demo.wav'
server((req,res)=>{
	let pathname=url.parse(req.url).pathname
	if(pathname==='/demo.wav'){
		res.writeHead(200,{'Content-Type':'audio/wav'})
		fs.createReadStream(wavPath).pipe(res)
		return
	}
	res.end()
}).listen(8080,()=>{
	console.log(`server listen on 8080`)
})
```
我们把链接丢到google浏览器，能正确识别并且播放了；看起来好像蛮简单的。同样的，我们把链接丢到safari浏览器，格式识别成功了，但是无法播放；看样子是兼容性的问题。
我们分别，看下2个浏览器的网络请求。

![截屏2020-10-09 下午3.20.56.png](https://i.loli.net/2020/10/09/1Y3V2DicNCL9A5p.png)

![截屏2020-10-09 下午3.20.31.png](https://i.loli.net/2020/10/09/BYaQGNEA8lJvVMj.png)

好像找到了一个关键点。`Rang:bytes=0-x` 
[HTTP 协议范围请求允许服务器只发送 HTTP 消息的一部分到客户端。范围请求在传送大的媒体文件，或者与文件下载的断点续传功能搭配使用时非常有用。](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Range_requests)
就是我们平时开发Native多媒体功能时候，不可能把整个文件下载下来再播放，肯定也是一边下载一边播放的。基本可以确定，兼容性是这个问题。
我们稍微修改下node js代码，支持`range`

```
const {createServer:server}=require('http')
const url=require('url')
const fs=require('fs')
const wavPath='./demo.wav'
var stats = fs.statSync(wavPath);
server((req,res)=>{
	let pathname=url.parse(req.url).pathname
	if(pathname==='/demo.wav'){
		let range = req.headers['range']
		if(range){
			let ranges = range.replace('bytes=','')
			let rangesplit = ranges.split('-')
			let start=parseInt(rangesplit[0])
			let end=parseInt(rangesplit[1]) || stats.size-1
			let length = end - start
			res.setHeader('Content-Range',`bytes ${start}-${end}/${stats.size}`)
			res.setHeader('Content-Type','audio/wav')
			res.setHeader('Content-Length',`${length + 1}`)
			res.writeHead(206)
		  fs.createReadStream(wavPath,{
					start:start,
					end:end
				}).pipe(res)
		}else{
			res.writeHead(200,{'Content-Type':'audio/wav'})
			fs.createReadStream(wavPath).pipe(res)
		}
		return
	}
	res.end()
}).listen(8080,()=>{
	console.log(`server listen on 8080`)
})
```

⚠️需要注意的是，range是前后闭包的，如果是0-1 ，返回的长度应该是2。

再测试下，google和safari都正常啦

![截屏2020-10-09 下午4.15.49.png](https://i.loli.net/2020/10/09/O5EQq2N3MYZstIJ.png)

[github demo](https://github.com/Dcell/my-test/tree/master/video-in-iOS-cannot-play)