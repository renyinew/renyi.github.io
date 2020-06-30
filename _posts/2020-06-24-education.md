---
layout: post
title: "fabric-sdk-go支持fabric 1.4.8链码测试"
date: 2020-06-24   
tag: fabric 1.4.8 fabric-sdk-go
---

### 介绍       

　　fabric-sdk-go stable版本目前最高支持v1.4.2 由于有现成的1.4版本的国密版本fabric使用如果能同时配合sdk将方便不不少，所以选用了fabric 1.4的最高版本fabric 1.4.8进行初步功能测试，随后升级到2.0也比较容易。
    

### fabric版本

**在Mac OS X上编译fabric**      

检出并编译1.4.8版本：      

>* $git clone https://github.com/hyperledger/fabric.git
>* $git checkout release-1.4    
>* $make release 
>* $make docker 

**修改链码**      

> 下载修改过后的测试链码[https://github.com/renyinew/education.git](https://github.com/renyinew/education.git) 基于https://github.com/kevin-hf/education修改使它能通过最新的版本支持与编译

 修改Gopkg.toml使它使用最新的软件包并执行更新依赖关系
 
>* dep ensure -v

**错误处理**    
fabric-sdk-go/internal/github.com/hyperledger/fabric/core/operations/system.go:227:23

not enough arguments in call to s.statsd.SendLoop
	have (<-chan time.Time, string, string)
	want (context.Context, <-chan time.Time, string, string)
	
修改system.go:227在s.statsd.SendLoop添加context.Background() ，如下所示
``s.statsd.SendLoop(context.Background(), s.sendTicker.C, network, address)``
然后完成并通过编译，可以直接下载修我改好的代码[https://github.com/renyinew/education.git](https://github.com/renyinew/education.git)
  
##### 笔者增加了配置crypto-config.yaml configtx.yaml文件用于后期版本升级

### 生成证书与节点
>* $cryptogen generate --config=./crypto-config.yaml
>* $configtxgen -profile OneOrgOrdererGenesis -outputBlock ./artifacts/genesis.block
>* $configtxgen -profile OneOrgChannel -outputCreateChannelTx ./artifacts/channel.tx -channelID kevinkongyixueyuan
>* $configtxgen -profile OneOrgChannel -outputAnchorPeersUpdate ./artifacts/Org1MSPanchors.tx -channelID kevinkongyixueyuan -asOrg KongyixueyuanOrg

###或者执行一键脚本
>*  修改cryptoArtifacts.sh文件PATH变量指向自己的bin文件
>* $ ./cryptoArtifacts.sh

### 启动并测试
>* cd fixtures && docker-compose up --force-recreate
>* cd .. && ./education
<br />

![](/images/posts/education/screenshot.png)

<br />

转载请注明：[点击阅读原文](https://renyinew.github.io/2020/06/education/)     

