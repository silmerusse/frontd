[![Build Status](https://travis-ci.org/xindong/frontd.svg?branch=master)](https://travis-ci.org/xindong/frontd)

### 简介

根据心动开放技术文档 [统一的Gateway](https://github.com/xindong/docs/blob/master/public/game_review/backend.md) 中的要求：
整体架构对公网暴露的IP控制在2-5个。并不过多暴露端口。
因此需要游戏开发和使用网关来与服务器后端进行通讯。

本项目提供一种通用方案，可以直接部署生产环境，也可以进行改造后部署。

### 主要特性

* 高性能

* 无状态

* 安全（无明文后端地址端口）

	* 后端地址端口使用AES加密

* 免配置免维护

	* 无论后端地址端口发生什么变化，均不需要重新配置本网关

### 注意事项

切勿将 Secret Passphrase 和 Slat 写入客户端代码！

* Secret Passphrase 和 Slat 用来生成加密的地址信息

* 加密后的地址信息可以写入客户端或通过其他方式发送给客户端，但不要将 Secret Passphrase 和 Slat 写入客户端代码！

### 编译

`go build` 或 `docker build`


### 部署服务端

`docker run -e "SECRET=SomePassphrase" -e "SLAT=Blahblah" tomasen/frontd /go/bin/frontd`


### 通讯协议

客户端建立TCP连接后，以文本形式发送 加密并的后端地址端口信息 + `\n` 换行符。之后开始正常通讯即可。

如果出现后端地址端口无法连接等错误，会根据下表返回二进制错误码：

| 错误码 | 含义 |
| 0x01   | 后端服务器超时 |
| 0x02   | 无法连接后端服务器 |
| 0x03   | 后端服务器超时 |
| 0x04   | 后端地址解密失败 |
| 0x05   | 不被允许的IP地址 |


### 接入方式

1. 生成 Passphrase 和 Salt 。并保存在安全的文档中。

	 * 可以使用在线生成 http://passwordsgenerator.net/

2. 使用上述 Secret Passphrase 和 Salt 部署服务端

3. 使用 AES 算法加密文本格式的 后端地址+slat，生成 base64 编码的密文。 可以使用在线工具如： http://tool.oschina.net/encrypt

	* 例：当后端地址为 `172.10.0.10:8901` 时，如果 Passphrase=12345 Salt=ABCDE 时，应当加密的明文是 `172.10.0.10:8901ABCDE`
	密文结果应为 `U2FsdGVkX1+i4XT2V8gtTyIB09rW9jyDEGMRVaYG85elu5Y/Hnp0O2NRycDC+Z4o`

4. 客户端 建立连接后，将后端地址的密文文本加一个换行符发送给网关。建立连接。

	* 根据前例： 应该发送 `U2FsdGVkX1+i4XT2V8gtTyIB09rW9jyDEGMRVaYG85elu5Y/Hnp0O2NRycDC+Z4o\n`

### 测试数据

_TODO_

### 后记

`frontd` 在设计上是安全性+性能+易于接入+易于维护的折中方案。其中：

* 使用加密地址，而不是预配置地址表，是出于易于维护的角度

* 使用AES是安全性考虑，防止攻击者篡改变造后端目标地址

* 使用文本而不是二进制通讯，是方便客户端和服务器列表部分功能的接入

折中意味着牺牲。因此，我们并不反对项目组在本代码基础上进行优化，但不建议弱化安全性的部分。


## Developing

Pull request must pass:

* [golint](https://github.com/golang/lint)
* [go vet](https://godoc.org/golang.org/x/tools/cmd/vet)
* [gofmt](https://golang.org/cmd/gofmt)
* [go test](https://golang.org/cmd/go/#hdr-Test_packages)
