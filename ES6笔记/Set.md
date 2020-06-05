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
    //Set 只有值没有key
    new Set();//通过构造函数创建Set实例
    //参数是一个数组或者类数组，有iterable接口(数组,arguments,元素集合,Set,Map,字符串)

    //默认去重
    new Set([1,2,3,1])
    //add 增加 如果之前没有才加上，返回增加后的Set实例,可以链式写法
    //参数只能有一个
    let set=new Set([1,2,null,"gg",NaN])
    console.log(set.add(10))

    //delete 删除 返回true/false  delete(参数)
    //如果有此项就true 没有 false

    //clear 清空  返回undefined  clear()

    //has() 返回true/false 有无此项
    //可判断NaN 肯定用的不会是==而是Object.is

    //forEach
    set.forEach((item,index,input)=>{
        //item,index:  当前 value
        //input: 当前Set实例
    })

    for(let key of set.keys()){
        //key仍然是value
    }
    for(let val of set.values()){
        //val仍然是value
    }
    for(let [item,val] of set.entries()){
        //都是[value,value] 类似数组中的方法
    }
    

    //===========用途==============
    //数组去重
    let ary=[1,1,2,2,3,3]
    function unique(ary){
        return [...new Set(ary)];
        // return Array.from(new Set(ary));二选一
    }

    let ary1=[1,3,5,7,9]
    let ary2=[1,2,4,8,9]
    //并集
    function union(ary1,ary2){
        return [...new Set(...ary1,...ary2)]
    }
    //交集
    function mix(ary1,ary2){
        return ary1.filter(item=>ary2.includes(item));
    }
    //差集=并集-交集
    function diff(ary1,ary2){
        return union(ary1,ary2).filter(item=>!mix(ary1,ary2).includes(item))
    }
</script>
```