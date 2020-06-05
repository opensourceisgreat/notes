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
    
</body>
</html>

<script>
    //async函数默认返回一个promise对象
    async function getA(){
        //return 的内容就是成功回调的参数
        //这里有错误就会catch捕获
        // throw new Error("yyy")
        return "xx"
    }
    getA().then((res)=>{
        console.log(res)
    }).catch((e)=>{
        console.log(e)
    })

    let p=new Promise((resolve,reject)=>{
        resolve("hello")
    })
    async function getB(){
        //await 后面是一个promise实例 如果不是也会默认转为promise
        //直接让promise实例回调执行 返回执行时的参数(resolve或者reject的)
        let a=await p;
        //等await后面的异步完成后再执行后面的代码
        console.log(a)
    }
    getB().then((res)=>{
        //由于上面没有return res就是undefined
        console.log(res)
    }).catch((e)=>{
        console.log(e)
    })
</script>
```