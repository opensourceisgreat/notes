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
    //Symbol是一个新的基本数据类型，而且是一个值类型
    //类似字符串，使用Symbol函数得到的数据唯一
    let sym1=Symbol("foo");
    let sym2=Symbol("foo");
    console.log(typeof sym1);
    let obj={
        sym1:"heyhey",
        [sym1]:"haha"
    };
    //这是因为 console.log/dir/... 这类方法输出对象时会显示该对象在内存中最新的状态；
    //内存的地址上是原始值是不能被修改，所以对象的话输出结果中的值都已经被修改成最新的值
    //上述是对下面连个输出的解释
    console.log(obj);
    obj.sym1="nmsl";
    obj[sym1]="nmsl";
    obj[sym2]="qqqq";
    console.log(obj);
    //Symbol值无法转数字和字符串拼接，可以转bool
    console.log(!Symbol(1))
    console.log(Symbol(1))
    console.log(Symbol("zf").toString())
    console.log(typeof Symbol("zf").toString())

    // Symbol.for(),之前有相同参数Symbol值，找到返回，没有就创建新的
    //使用for创造才有效果
    let zf1=Symbol.for("zz");
    let zf2=Symbol.for("zz");
    console.log(zf1==zf2)
    //Symbol.keyFor(symbol值)找到使用Symbol.for创建的值得描述
    //不是for创建的是找不出来的
    console.log(Symbol.keyFor(zf1));//结果：zz

</script>
```