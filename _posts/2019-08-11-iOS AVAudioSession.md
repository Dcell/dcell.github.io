随着智能手机的普及，一些常用的app基本都有音频相关的功能，录音/播放等等；网上的一般的信息都是怎么使用api去播放音频/录制音频。但最重要的信息，AVAudioSession反而不怎么重视。这里将参考官方文档进行一个简单的梳理。
## AVAudioSession Category
`Category`是AVAudioSession第一要素，往往决定了你当前应用音频的能力，如：播放/录制等

| Category | 能否播放 | 能否录音 | 能否被静音键/锁屏键静音 |
| --- | --- | --- | --- |
| AVAudioSessionCategoryAmbient（混音播放） | ☑️ | | ☑️|
| AVAudioSessionCategorySoloAmbient（独占播放） |☑️| | ☑️|
| AVAudioSessionCategoryPlayback |☑️ | | |
| AVAudioSessionCategoryRecord |  |☑️| |
| AVAudioSessionCategoryPlayAndRecord | ☑️ |☑️ | |
| AVAudioSessionCategoryMultiRoute | ☑️ | ☑️| |

## AVAudioSession CategoryOptions
`CategoryOptions`是选择第一要素后，进行的第二要素，可多选，每个option都需要对应指定的category

| CategoryOptions | 适用的Category | 作用 |
| --- | --- | --- |
| MixWithOthers | PlayAndRecord,Playback,MultiRoute | 与其他已经激活的APP混播|
|DuckOthers |Ambient,PlayAndRecord,Playback,MultiRoute|压低其他App播放的声音 |
| AllowBluetooth |Record，PlayAndRecord |支持蓝牙设备 |
| DefaultToSpeaker | PlayAndRecord |播放扬声器播放|
|InterruptSpoken AudioAndMixWithOthers|PlayAndRecord,Playback,MultiRoute |播放此应用的音频内容时，暂停来自其他应用的连续语音内容 |
| AllowBluetoothA2DP |PlayAndRecord | 音频是否可以流式传输到支持高级音频分发配置文件（A2DP）的蓝牙设备| 
| AllowAirPlay |PlayAndRecord | 音频是否可以流式传输到AirPlay设备| 

## AVAudioSession Mode
最后一个要素mode，如果没有特殊的场景一般可以用`.default`,如果是游戏或者特殊场景，可以参考下文档，这里不再展开说明。

## 监听设备的接入
一般音频类需要监听设备的接入，比如插入耳机后，需要改成听筒模式


```
NotificationCenter.default.reactive.notifications(forName: AVAudioSession.routeChangeNotification).observeValues { (notification) in
    if let userInfo = notification.userInfo {
        if let reason = userInfo[AVAudioSessionRouteChangeReasonKey] as? UInt{
            guard let aVAudioSessionRouteChangeReason = AVAudioSession.RouteChangeReason(rawValue: reason) else{
                return
            }
            switch aVAudioSessionRouteChangeReason {
            case AVAudioSession.RouteChangeReason.newDeviceAvailable://新设备的接入
                break
            case AVAudioSession.RouteChangeReason.oldDeviceUnavailable:
                break
            default:
                break
            }
        }
    }
}
```

## 监听音频调度
每个App都有自己的管理的Session，如果出现多个app竞争的情况，我们需要作出正确的处理

```
NotificationCenter.default.reactive.notifications(forName: AVAudioSession.interruptionNotification).observeValues { (notification) in
    if let interruptionTypeInt = notification.userInfo?[AVAudioSessionInterruptionTypeKey] as? UInt{
        let interruptionType = AVAudioSession.InterruptionType(rawValue: interruptionTypeInt)
        if interruptionType == AVAudioSession.InterruptionType.began {
        	//暂停
        }else  {
        	//恢复
        }
    }
}
```

## 红外
开启红外使能

```
UIDevice.current.isProximityMonitoringEnabled = true
```

监听红外

```
NotificationCenter.default.reactive.notifications(forName: UIDevice.proximityStateDidChangeNotification).observeValues { (notification) in
    if UIDevice.current.proximityState {
        //触发红外
    }else{
		  //其他
    }
}
```

> BUG：如果在红外触发的情况下，关闭使能，会导致红外广播不回调，而且一直处于红外状态，使用者需要特殊处理。
