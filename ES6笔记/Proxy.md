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
    //对目标对象默认操作拦截
    //new Proxy({目标对象target},{拦截的方法})
    let obj = { name: "xx", age: 10 }
    //类似代理模式
    let proxy1 = new Proxy(obj, {
        get(target, propKey, receiver) {
            // target  目标对象
            // propKey  属性名
            // receiver 当前实例
            console.log(arguments);//可以打印传给get的参数
            console.log(1)
            // return "fff"
            return target[propKey]//不能用target.key因为key会变成字符串去target中找，是无法获取到值得
        },
        set(target, propKey, value, receiver) {
            target[propKey] = value
            return true
        },
        has(target, propKey) {
            console.log("has")
            if (propKey.startsWith("_")) {
                return false
            }
            return propKey in target
        }
    })
    //get  只要获取就会触发get
    console.log(proxy1.name)
    proxy1.name = "gg"
    console.log(proxy1.name)
    console.log("name" in proxy1)
    function getObj(){
        console.log(this)
        return {name:"fgg"};
    }
    let proxy2=new Proxy(getObj,{
        apply(target, object, args){
            //函数直接执行就触发,call,apply也一样
            // console.log("apply")
            //args 函数执行的参数
            //object 给函数修改this
            // console.log(target,object,args)
            if(object){
                object.fn=target;//改变函数执行时的this指向
                object.fn(...args)
                delete object.fn
                //在原型上加
                // object.__proto__.fn=target;//改变函数执行时的this指向
                // object.fn(...args)
                // delete object.__proto__.fn
            }else{

                target(...args)
            }

        }
    })
    proxy2()
    proxy2.call(obj,1,2,3)
    //{name: "gg", age: 10, fn: ƒ}展开就没了fn: f因为delete删除l了
    //在原型上加就不会产生这样的显示了


// get(target, propKey, receiver)：拦截对象属性的读取，比如proxy.foo和proxy['foo']。
// set(target, propKey, value, receiver)：拦截对象属性的设置，比如proxy.foo = v或proxy['foo'] = v，返回一个布尔值。
// has(target, propKey)：拦截propKey in proxy的操作，返回一个布尔值。
// deleteProperty(target, propKey)：拦截delete proxy[propKey]的操作，返回一个布尔值。
// ownKeys(target)：拦截Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而Object.keys()的返回结果仅包括目标对象自身的可遍历属性。
// getOwnPropertyDescriptor(target, propKey)：拦截Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。
// defineProperty(target, propKey, propDesc)：拦截Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)，返回一个布尔值。
// preventExtensions(target)：拦截Object.preventExtensions(proxy)，返回一个布尔值。
// getPrototypeOf(target)：拦截Object.getPrototypeOf(proxy)，返回一个对象。
// isExtensible(target)：拦截Object.isExtensible(proxy)，返回一个布尔值。
// setPrototypeOf(target, proto)：拦截Object.setPrototypeOf(proxy, proto)，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。
// apply(target, object, args)：拦截 Proxy 实例作为函数调用的操作，比如proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)。
// construct(target, args)：拦截 Proxy 实例作为构造函数调用的操作，比如new proxy(...args)。


</script>
```