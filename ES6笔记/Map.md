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
    //new Map([[key,value],[]])
    //参数是一个数组，数组每项也是数组[key,value]
    //一个对象 属性名必须是字符串，不是字符串的转为字符串
    //Map实例的key可以是任意数据类型，Map类似除掉了对属性名限制的对象
    let ary=[1,2],o={}
    let obj={
        1:'1',
        true:true,
        [ary]:ary,//调用toString转为字符串
        [o]:o
    }
    let map=new Map([[1,"xx"],[true,"kk"],[{},{}],[o,{}]])

    //get(key)
    console.log(map.get({}))//undefined,{}=={}/{}==={} false
    console.log(map.get(o))//{}


    //set(key,value)
    //如果存在key则直接修改，否则添加，返回新的map实例


    //has(key) 判断key有没有对应的value

    //delete(key) 返回true/false

    //clear() 返回undefined

    map.forEach((value,key,input)=>{
        //input map实例

    })

    //keys(),values(),entries()类似Set

    //数组变Map
    let ary1=["xx","yy"]
    let map3=new Map();
    for (let [index,item] of ary1.entries()) {
        map3.set(index,item)
    }
</script>
```