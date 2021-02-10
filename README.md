# Promise-Question
<font color=#A52A2A size=4 >Markdwon测试</font>
<font color=red>我是红色</font>
<font color=red> event loop </font>它的执行顺序：
<span style="color:red;">这是比font标签更好的方式。可以试试。</span>
+ 一开始整个脚本作为一个宏任务执行
+ 执行过程中同步代码直接执行，宏任务进入宏任务队列，微任务进入微任务队列
+ 当前宏任务执行完出队，检查微任务列表，有则依次执行，直到全部执行完
+ 执行浏览器UI线程的渲染工作
+ 检查是否有Web Worker任务，有则执行
+ 执行完本轮的宏任务，回到2，依此循环，直到宏任务和微任务队列都为空

**微任务包括：** MutationObserver、Promise.then()或catch()、Promise为基础开发的其它技术，比如fetch API、V8的垃圾回收过程、Node独有的process.nextTick。

**宏任务包括：** script 、setTimeout、setInterval 、setImmediate 、I/O 、UI rendering。

注意⚠️：在所有任务开始的时候，由于宏任务中包括了script，所以浏览器会先执行一个宏任务，在这个过程中你看到的延迟任务(例如setTimeout)将被放到下一轮宏任务中来执行。


### 第 1 题： 限制异步操作的并发个数并尽可能快的完成全部

> 有8个图片资源的url，已经存储在数组urls中。
> urls类似于['https://image1.png', 'https://image2.png', ....]
> 而且已经有一个函数function loadImg，输入一个url链接，返回一个Promise，该Promise在图片下载完成的时候resolve，下载失败则reject。
> 但有一个要求，任何时刻同时下载的链接数量不可以超过3个。
> 请写一段代码实现这个需求，要求尽可能快速地将所有图片下载完成。

```js
var urls = [
  "https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/AboutMe-painting1.png",
  "https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/AboutMe-painting2.png",
  "https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/AboutMe-painting3.png",
  "https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/AboutMe-painting4.png",
  "https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/AboutMe-painting5.png",
  "https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/bpmn6.png",
  "https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/bpmn7.png",
  "https://hexo-blog-1256114407.cos.ap-shenzhen-fsi.myqcloud.com/bpmn8.png",
];
function loadImg(url) {
  return new Promise((resolve, reject) => {
    const img = new Image();
    img.onload = function() {
      console.log("一张图片加载完成");
      resolve(img);
    };
    img.onerror = function() {
    	reject(new Error('Could not load image at' + url));
    };
    img.src = url;
  });

```
```js
function limitLoad(url, handler, limit){
  let sequence = [].concat(urls) //复制urls
  //先请求urls中的前面limit个url
  let promises = sequence.splice(0, limit).map((url, index) => {
      return handler(url).then(() => {
          // 返回下标为了知道数组中哪一项最先完成
          return index
      })
  })
  // 注意这里要将整个变量过程返回，这样得到的就是一个Promise，可以在外面链式调用
  return sequence
  .reduce((pCollect, url, currentIndex) => {
     return pCollect.then(()=>{
        return Promise.race(promises)
    })
    .then(fastestIndex =>{
        promises[fastestIndex] = handler(url).then(()=>{
            return fastestIndex
        })
    })
    .catch(err=>{
        console.log(err)
    });
    },Promise.resolve())
    .then(()=>{
        return Promise.all(promises)
    });  
  }
    limitLoad(urls, loadImg, 3)
    .then(res => {
      console.log("图片全部加载完毕");
      console.log(res);
    })
    .catch(err => {
      console.error(err);
    });

```

<br/>
