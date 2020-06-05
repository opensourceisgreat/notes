```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <div id="box">

  </div>
</body> 
</html>

<script>
  //执行顺序
  //new Promise中的函数==>当前队列中的同步代码==>then中的回调
  let pro1=new Promise((resolve,reject)=>{
    console.log("Promise")

    //resolve 函数
    resolve("success")
    //reject  函数
    reject("error")

    //上面两个函数只会执行一个，状态只会到成功或失败，所以只能执行一个
  })

  pro1.then((res)=>{
    //resolve 成功的回调
    console.log(res)
  },(rej)=>{
    //reject 失败的回调
    console.log('eee')
    console.log(rej)
  })
  console.log("OK")

  //上述的输出结果
  //Promise
  //OK
  //success

//==============加载图片示例=============
  function LoadImg(url){
    return new Promise((resolve,reject)=>{
      let img=new Image();
      img.src=url
      img.onload=function(){
        resolve(img)
      }
      img.onerror=function(e){
        reject(e)
      }
    })

  }
  // LoadImg('https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=2174909441,2495215020&fm=26&gp=0.jpg').then((img)=>{
  //   box.appendChild(img)
  // },(e)=>{
  //   console.log(e)
  // })
  LoadImg('https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=2174909441,2495215020&fm=26&gp=0.jpg').then((img)=>{
    box.appendChild(img)
  }).catch((e)=>{
    //捕获错误，new Promise和then回调中的错误都可以捕获
    //采取上面的写法会导致第一个回调错误无法捕获
    console.log(e)
  })

  //all
  //Promise.all([每一项都是Promise,如果不是，默认转为Promise])
  //数组中每一项都是成功 才会走回调 默认将每一个项的参数放在一个数组中传给回调函数，只要一个有错误就会走错误的回调
  let p1=new Promise((resolve,reject)=>{
    resolve("x")
  })
  let p2=new Promise((resolve,reject)=>{
    resolve("y")
  })
  let p3=new Promise((resolve,reject)=>{
    resolve("z")
  })

  let PAll=Promise.all([p1,p2,p3])
  PAll.then((res)=>{
    console.log(res)
  }).catch((e)=>{
    console.log(e)
  })

  //race
  //Promise.race([p1,p2,p3])
  //只要有一个状态改变 当前实例的状态就会改变
  //就是说p1,p2,p3不论状态走向成功or失败，看他们谁先变，then就执行了，可给p1,p2,p3加延迟判断
  Promise.race([p1,p2,p3]).then((res)=>{
    console.log(res)
  }).catch((e)=>{
    console.log(e)
  })
</script>
```