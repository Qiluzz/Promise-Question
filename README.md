# Promise-Question

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
