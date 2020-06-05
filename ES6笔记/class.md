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
    class A{
        constructor(x){
            //类本身的函数
            //this:当前实例
            this.x=x;//增加私有属性
            //return的基本数据类型对实例没有影响，如果是引用数据类型会改变实例

        }
    }
    let a=new A(10);
    console.log(a);//不能通过普通函数执行A()报错
    console.log(typeof A);//function

    //class的name问题    
    let B=class BB{
        //BB 只能在类里面使用 类的name都是BB
        constructor(){
            console.log(BB.name)
        }
        getB(){//该方法直接加在原型链上了
            console.log(BB.name)
        }
    }
    let b=new B();//不能使用new BB()
    b.getB();
    console.log(B.name)//"BB"
    console.log(b)

    //==================class执行问题===========
    //采用class表达式让类直接执行,ES5中函数本身也是constructor
    let a1=new class{
        constructor(name){
            console.log(name)
        }
    }("xxx");

    //============class变量提升==========
    //new GG()会报错，不会变量提升和let const一样
    class GG{
        constructor(){}
    }

    //===========class的静态方法===========
    //类就相当于原型,写在原型上的方法都被实例继承了
    //若需要给当前类本身加方法，可以在方法前加上static,不会被实例继承，只有类本身可以使用
    //例如Array.of
    class FF{
        constructor(){
            this.x=10
        }
        getA(){}
        static getB(){}
    }
    console.log(new FF())//new FF和new FF()一样，没参数时
    let aa=new FF;
    aa.getA()
    // aa.getB错误，实例不能用
    // FF.getB()正确

    //静态方法可以被子类继承
    class M{
        static getM(){}
    }
    class G extends M{
        constructor(){//继承中constructor写了就必须带上super,不写constructor，系统自动生成
            super();
        }
        static getG(){
            super.getM();
        }
    }

    //=============原型上的方法不可枚举===========
    //可枚举就是能用for in遍历
    class YY{
        constructor(){
            this.yy="yyy"
        }
        getY(){}
    }
    let yy=new YY
    for(let key in yy){
        console.log(key)//方法无法枚举，ES5中函数形式就可以
    }
    console.log(YY.prototype)
    console.dir(Array)

    //CSDN有搜藏__proto__和prototype的讲解

    //==============继承==============
    class K{
        constructor(){}
        getX(){}
        static getY(){}
    }
    class L extends K{
        constructor(){
            //子类没有this  this继承父类，super()执行完成后才会有
            super();//就是父类A的constructor
            this.y=100;//必须在super()后面
        }
        getX(){
            super.getX()//super指向父类原型
        }
        static getY(){
            super.getY()//super指向父类本身
        }
    }
    
</script>
```