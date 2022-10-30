---

title: wavesurfer-js音频处理实战及优化
date: 2022-10-30 10:00

---

![](https://image.liuyongzhi.cn/imageswavesurfer-js.png)

## 什么是[wavesurfer-js]([https://wavesurfer-js.org/](https://wavesurfer-js.org/)?

> **wavesurfer.js** is a customizable audio waveform visualization, built on top of [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API) and [HTML5 Canvas](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API).

**wavesurfer.js**是一个可定制的音频波形可视化，建立在[Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)和[HTML5 Canvas](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)之上。

wavesurfer还支持[插件]([wavesurfer.js](https://wavesurfer-js.org/plugins/))，包括并不限于剪裁、录制等

## wavesurfer的使用

```bash
npm i wavesurfer.js
```

```js
const srouce = "https://wavesurfer-js.org/example/media/demo.wav"
const ref = Wavesurferjs.create({
            container: refs.current, // 目标dom
            waveColor: '#544979', // 波形图颜色
            progressColor: '#6B58B2', // 播放进度颜色
            height: 40,
            hideCursor: true,
            fillParent: true,
            barGap:1,
            barWidth:1,
        })
ref.load(source)
```

### 坑点

- `WavesurferJS`可以实时解析音频文件的波状图，波状图解析速度与文件大小成正比
  
  > `load`函数还可以接受第二个参数，第二个参数是一个`json`数据，也是波形图绘制的数据。在`wavesurfer`实例上存在`backend`属性,此属性中存在`getPeaks`函数，我们可以在后台上传时，先解析出音频数据，将音频数据与音频文件绑定上传，这样C端可以直接使用音频数据，省去解析时间
  
  #### example
  
  ```js
  const parseWaveView = url => new Promise((resolve, reject) => {
      const target = document.createElement('div')
      target.style.width = '100%'
      target.style.height = '200px'
      target.style.position = 'absolute'
      target.style.top = '-100%'
      const instance = wavesurfer.create({
         container: target
      })
      instance.load(url)
      instance.on('ready', async() => {
        const peaks = instance.backend.getPeaks(
              instance.backend.mergedPeaks.length,
               0, 
              instance.backend.mergedPeaks.length
              )
        await putWaveform2Oss({
          mediaId: mediaId,
          confArr: peaks
      })
      document.body.removeChild(target)
      resolve(peaks)
    })
  })
  
  // getPeaks函数接收三个参数，分别是音频数据总长度、截取的起始位置、截取的结束位置
  // 参数可以在backend中拿到
  ```

- 播放音频时，如果音频文件过大并且在没有加载完成后播放其他实例，会加载混乱，播放的声音和应该播放的音频不一致，还会导致内存溢出等问题。

> `load`函数的第一个参数可以使用一个空的音频文件，在播放时可以获取当前播放的音频地址使用`audio`标签控制播放，保证同一时间只加载一个音频资源，减少内存压力。另外在点击波形图改变进度时，可以[监听事件]([wavesurfer.js](https://wavesurfer-js.org/docs/events.html))获取当前时间来设置`audio`标签时间。同样在`audio`播放中获取当前播放进度同步波形图进度

#### example

```js
// 设置波形图播放进度 time 0 - 1
nstance.seekTo([time])
// 点击波形图调整播放时间
// 1. 点击时获取播放进度的长度除以总波形图的长度得到百分比
// 2. 使用百分比乘以音频总时长得到当前时间
// 3. 设置audio当前时间
const setCurrentProgress = current => {
    audio.currentTime = current * duration
}
nstance.drawer.on("click", e => {
    setCurrentProgress(e.layerX / e.target.clientWidth)
})


```



### 最后

- [wavesurfer-js](https://wavesurfer-js.org/)

- [peaks.js](https://github.com/bbc/peaks.js)

- 如果你还想来我的[博客](https://overdev.cn)看看
