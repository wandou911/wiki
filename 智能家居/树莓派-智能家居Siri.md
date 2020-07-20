前面我写了一个系列共三篇的智能家居实践，用的是 Amazon Echo 实现语音控制，但是 Amazon Echo 用户群体太小而且这玩意并没有在中国上市，日常使用中也是各种水土不服，让很多朋友有心无力。然而，正如你早就看到的标题中说的那样，我们还有更容易获得的工具，那就是 Siri。

自从苹果推出了 HomeKit 以来，鉴于苹果庞大的用户量，不断开始有家电厂商开发出兼容 HomeKit 的家电。然而第一个吃螃蟹的人总是有代价的，这些电器的价格不是太贵就是中国买不到，所以使用 HomeKit 的仍然是小众群体。

通过 HomeAssistant 我们实现了把普通家电接到同一个平台，并且实现了 Amazon Echo 进行语音控制，能否把 Amazon Echo 替换成 Siri 呢？答案当然是可以的，不然就不会有这篇文章了。

这回又是美帝的极客站出来拯救世界，隆重介绍这个重量级的开源库：homebridge，通过逆向 HomeKit 协议让普通的 Wifi 设备也能接入 HomeKit 从而通过 Siri 控制。你可以这里找到所有 homebridge 相关的 plugin，已经有超过500个了。

所以 homebridge 诞生之初和 HomeAssistant 并没有关系，但你可能注意到了，如果你每一个设备都安装一个 homebridge 相关的 plugin，太分散也不利于管理，所以，最好的办法是，把一个平台整体迁移过来。毋庸置疑，HomeAssistant 是目前管理智能家居最成熟也是最流行的平台，所以你只管把电器设备加到 HomeAssistant 就行，和 HomeBridge 只需要一个 Plugin 连接就行，这个 Plugin 就是 homebridge-homeassistant ，它在 HomeAssistant 和 HomeKit 之间架起了一座桥梁，让 HomeBridge 可以直接使用 HomeAssistant 平台下的所有设备。所以大致的架构图是这个样子的：

![](http://kittenyang.com/content/images/2017/03/-----2017-03-26-23-06-26.png)

HomeBridge 的核心是 HAP-NodeJS 这个框架，用 NodeJS 模拟了一个 HomeKit Accessory Server 。作者 KhaosT 是个在美留学的中国人，曾在苹果的 HomeKit 团队实习过，这也 make sense 了为何是他第一个逆向了苹果的 HomeKit 协议。这里面有个小八卦，当 KhaosT 逆向了 HomeKit 的时候写了篇博客，但由于涉及到商业机密被苹果法务要求删除文章，但好在代码已经早就 fork 开了所以代码才得以流传到现在。试想要是当初代码没有开源，恐怕我们现在还享受不到这一成果。

其实我上面告诉你有 homebridge-homeassistant 这个东西已经可以了，剩下的无非就是看 README 跟着做的事情。所以我不想把安装过程再复制粘贴一遍，我就说一下安装完之后如何和 HomeAssistant 桥接的操作了。

若顺利安装完 HomeBridge，你就能在 pi 用户下找到 .homebridge 这个隐藏目录，把这个目录通过 Samba 共享出来就可以用 Finder 打开了。

![](http://kittenyang.com/content/images/2017/03/-----2017-03-26-22-53-29.png)

然后修改 config.json：

```
{
    "bridge": {
        "name": "KittensHome", //修改
        "username": "B8:27:EB:A0:2C:A1", //修改
        "port": 45536, //修改
        "pin": "775-82-588" //修改
    },
     "platforms": [
      {
        "platform": "HomeAssistant",
        "name": "HomeAssistant",
        "host": "http://192.168.10.200:8123", //修改
        "password": "xxxxxx", //修改
        "supported_types": ["binary_sensor", "cover", "fan", "input_boolean", "light", "lock", "media_player", "scene", "sensor", "switch"],
        "logging": true
      }
    ]
}
```

你只需要修改我标记修改的六个地方就行了，bridge.username 就是树莓派的 Mac 地址，port 和 pin 就是 HomeBridge 运行的端口和密码。这个密码等会和 HomeKit 配对使用。这里需要注意 "port" 必须大于0小于65536；username 必须是大写的 Mac 地址。 platforms.host 就是你的 HomeAssistant 所在的 IP 和端口，platforms.password 就是 HomeAssistant 的密码。

然后，请严格按照以下步骤操作，如果你 HomeBridge 启动失败了，再次按照下面步骤重新操作：

1.使用改新的 bridge.name, 新的bridge.port

2.输入命令：sudo killall homebridge 关闭所有正在运行的 HomeBridge 服务

3.手动删除 HomeBridge 文件夹下的 persist 文件夹。

![](http://kittenyang.com/content/images/2017/03/-----2017-03-27-00-05-17.png)

4.输入命令：sudo systemctl restart home-assistant.service 重启 HomeAssistant 服务。

5.务必等 HomeAssistant 服务启动完成，如果你像我之前文章提到的创建了一个 Automation 用于监听启动完成事件然后通过 IFTTT 或其他推送服务就可以很快知道 HA 什么时候启动完成了。或者你也可以通过命令 sudo systemctl status home-assistant.service -l 查看启动进程。然后，命令行输入 homebridge 就可以启动 HomeBridge 服务了。

如果你看到了以下画面，说明 HomeBridge 已经成功启动了，快去看看手机吧。

![](http://kittenyang.com/content/images/2017/03/-----2017-03-26-23-47-48.png)

按照以下步骤，找到 HomeBridge，输入刚才你指定的密码，然后一路下一步，最后就能看到所有 HomeAssistant 下的设备都集成到 HomeKit 了。 

![](http://kittenyang.com/content/images/2017/03/IMG_680F2BE85E86-1.jpeg)

不希望显示的设备，你可以在 configuration.yaml 里通过 homebridge_hidden 属性关闭。

```
customize:  
  switch.a_switch:
    homebridge_hidden: true
```

一旦接入了 HomeKit ，就可以直接用 Siri 控制了，而且能听懂中文了，而且得益于 HomeKit 内建的功能和 Siri 天生具备的语义分析，你可以实现很多 Amazon Echo 做不到的操作，比如改变灯泡颜色，灯泡亮度。

#### 参考链接

[智能家居实践（番外篇）—— 接入 HomeKit 实现用 Siri 控制家电](http://kittenyang.com/homebridge-practice/)
