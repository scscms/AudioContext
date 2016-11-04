# AudioContext 可视化效果<sup>shine</sup>

![image](https://github.com/scscms/AudioContext/raw/master/images/vision1.jpg)

## 前言

曾记得以前很多播放器都有那种柱状的随音乐而动的频谱效果，如今的播放器反而越来越少使用此效果了，更多的是配合视频或者歌词来展示。但是那种音频动画一直印在我的脑海，总感觉静静的听歌，思绪随动画而翩翩起舞总觉得另一番享受。
可惜以前浏览器制作不出此效果，虽然flash能做一点类似效果，无奈flash自身难保。如今H5横行，自然HTML5 Audio API便吹起了春天的暖风，让频谱动画花再次悄然绽放了...

## 1、简介
Web Audio API提供了一个简单强大的机制来实现控制web应用程序的音频内容。它允许你开发复杂的混音，音效，平移以及更多。在这篇文章里，我们将会通过几个例子来解释web Audio API的基本运用。

一段音频到达扬声器进行播放之前，我们可以使用[window.AudioContext](https://www.w3.org/TR/webaudio/)获取音频数据，创建各种AudioNode，即音频节点。
不同节点作用不同，有的对音频加上滤镜比如提高音色(比如BiquadFilterNode)，改变单调，有的音频进行分割，比如将音源中的声道分割出来得到左右声道的声音（ChannelSplitterNode），有的对音频数据进行频谱分析即本文要用到的(AnalyserNode)。

这里将要介绍的HTML5音频处理接口与Audio标签是不一样的。页面上的Audio标签只是HTML5更语义化的一个表现，而HTML5提供给JavaScript编程用的Audio API则让我们有能力在代码中直接操作原始的音频流数据，对其进行任意加工再造。就可制作跟随音乐节奏变化的频谱图，也称之为可视化效果。

## 2、流程
音频节点通过输入与输出进行连接，形成一个链，从一个或多个源出发，通过一个或更多的节点，最终到输出终端（你也可以不提供输出终端，换句话说，如果只是想使一些音频数据可视化）。一个简单经典的web  Audio的工作流程如下：

- 构建音频上下文AudioContext对象；
- 在AudioContext对象内，构建音源，比如audio，oscillator，stream
- 构建效果节点effectNode，比如混响，双二阶滤波器，声相，压限器
- 选择最终的音频目的地，比如说你的系统扬声器
- 连接源到效果，效果到输出终端

#### 构建AudioContext对象
首先，你需要构建一个AudioContext实例，来创建一个音频图。最简单的方法就像这样：
```javascript
var audioCtx = new AudioContext();
//考虑兼容(其实没必要)
var audioCtx = new (window.AudioContext || window.webkitAudioContext)();
```
#### 创建AudioSource
现在我们有了AudioContext，可以用这个来做很多事。第一件我们需要做的事是玩音乐。音频可以来自于多样的地方：

* 通过JavaScript直接生成一个音频节点比如oscillator. 一个 OscillatorNode是利用AudioContext.createOscillator 方法来构建。
* 从原PCM数据构建: AudioContext有解密被支持的音频格式的多种方法。看AudioContext.createBuffer(), AudioContext.createBufferSource(), 以及 AudioContext.decodeAudioData().
* 来自于HTML音频元素例如 video 或者audio: 可以看 AudioContext.createMediaElementSource().
* 直接来自于 WebRTC，MediaStream 例如来自于摄像头或麦克风. 可以看AudioContext.createMediaStreamSource().

```javascript
    const audioCtx = new AudioContext();
    //为每个键盘位对应一个频率
    const obj ={65:256,83:288,68:320,70:341,71:384,72:426,74:480,75:512,76:542};
    for(let key in obj){
        let value = obj[key];
        obj[key] = audioCtx.createOscillator();
        obj[key].frequency.value = value;
        obj[key].start();
    }
    document.addEventListener("keydown",function(e){
        obj[e.keyCode] && obj[e.keyCode].connect(context.destination);
    },false);
    document.addEventListener("keyup",function(e){
        obj[e.keyCode] && obj[e.keyCode].disconnect();
    },false);
```
[查看createOscillator](createOscillator.html)

#### 连接输入输出
为了通过你的扬声器来实际输出音质，你需要将它们连接起来。这个被称为节点连接方法，节点来自于很多可获得的不同节点类型。你想要连接的节点都提供了这个方法。

你的设备的默认输出结构（通常是你的设备扬声器），通过AudioContext.destination来允许进入。为了连接oscillator，gain node以及输出端，如以下运用：

```javascript
oscillator.connect(gainNode);
gainNode.connect(audioCtx.destination);
```
#### 播放音乐及设置音调
现在audio节点图已经建立，我们可以设置属性值及调用音频节点的方法来调节想要的音效。在这个简单的例子，我们可以设置特殊的音调，以赫兹为单位，设置为特殊类型，以及指示音乐播放：

```javascript
oscillator.type = 'sine'; //其他可选'square','sawtooth','triangle'和'custom'
oscillator.frequency.value = 2500;
oscillator.start();
```

#### 设置静音
当静音按钮点击，以下方法会被调用，disconnect方法，将切断gain node与destination节点的链接，有效阻止了节点图的链接，所以没有声音会被产生。再次点击效果相反。

```javascript
var mute = document.querySelector('.mute');
mute.onclick = function() {
  if(mute.innerHTML != "Unmute") {
    gainNode.disconnect(audioCtx.destination);
    mute.innerHTML = "Unmute";
  } else {
    gainNode.connect(audioCtx.destination);
    mute.innerHTML = "Mute";
  }
}
```

### 获取音频文件

- ajax获取　[查看](ajax_decodeAudioData.html)
- fetch获取　[查看](fetch_decodeAudioData.html)
- FileReader选择　[查看](FileReader_decodeAudioData.html)

### AudioContext 可视化效果

![image](https://github.com/scscms/AudioContext/raw/master/images/vision2.jpg)

[查看文件](decodeAudioData.html)