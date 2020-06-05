[toc]
# 1、流

数据源------------I/O流-------------------程序

![1.PNG](https://note.youdao.com/yws/api/personal/file/A0800981E00445F6A9CC6D7484DF4459?method=download&shareKey=38386f87e4dc131bf6a4dcb6679c1024)

![2.PNG](https://note.youdao.com/yws/api/personal/file/0DFC31D6A2FE447B9725D89000720889?method=download&shareKey=48667f86333190646dda905e613b0446)

![3.PNG](https://note.youdao.com/yws/api/personal/file/A7FB8756030E428FBB0F27BE8CD6EE19?method=download&shareKey=2f77d06b4b0d166bccde0a597db442ac)

像视频文件，音频文件就必须用字节流处理

ByteArrayInputStream、FileInputStream就是节点流

# 2、File

```java
package com.whpu.io;

import java.io.File;

/**
 * @author Date 2019/7/22-15:17
 */
public class PathDemo01 {
    /**
     * \ ,/名称分隔符
     */

    public static void main(String[] args) {
        String path="C:\\Users\\Administrator\\Desktop\\java_IO笔记";
        System.out.println(File.separatorChar);

        //建议
        //1、/,这样写不会出任何问题
        path="C:/Users/Administrator/Desktop/java_IO笔记";
        //2、常量拼接
        path="C:"+File.separator+"Users";
        System.out.println(path);
    }
}

```

在java项目中

相对路径：相对于本项目文件夹，不加盘符

`new File("IO.png")`系统会自动识别为相对路径

`new File()`可以构建一个不存在的路径



## 2.1、File API

![4.PNG](https://note.youdao.com/yws/api/personal/file/2EE9C8C6E20447C5B319B18E62F63E47?method=download&shareKey=2131ee0efd7bbcc67215325629b2426e)

`getPath()`返回的是创建时的路径，相对就相对，绝对就绝对

con con3...操作系统的设备名，不能正确创建文件

```java
//1、mkdir():确保上级目录存在，不存在创建失败
//2、mkdirs():上级目录可以不存在，不存在一同创建
//会返回boolean
//多使用mkdirs()


//list():列出下级名称,返回字符串数组，只列出第一级
//listFiles():列出下级File对象
//listRoots():列出所有盘符



package com.whpu.io;
import java.io.File;
/**
 * 面向对象统计文件夹大小
 *
 * @author Date 2019/8/18-14:18
 */
public class DirCount {
    //大小
    private long len;
    //文件夹
    private String path;
    //源
    private File src;

    public DirCount(String path) {
        this.path = path;
        this.src = new File(path);
        count(this.src);
    }

    public static void main(String[] args) {
        DirCount dir = new DirCount("");
        System.out.println(dir.getLen());

    }
    //统计大小
    public void count(File src) {
        //获取大小
        if (null != src && src.exists()) {
            if (src.isFile()) {
                len+=src.length();
            }else {
                for (File s:src.listFiles()){
                    count(s);
                }
            }
        }

    }

    public long getLen() {
        return len;
    }

}

```



# 3、文件编码

![5.PNG](https://note.youdao.com/yws/api/personal/file/C89345837FE941709E18106EAB87754A?method=download&shareKey=5dfac306f4c4eba1271a9b38f268af3c)

```JAVA
字符<---->字节
|          |
|          |
人         电脑
------------> 编码encode
<------------ 解码decode


字符集就是字典

String msg="看看那";
//编码：字节数组
byte[] datas=msg.getBytes();//默认使用工程的字符集
datas2=msg.getBytes("GBK");
//解码：字符串
msg=new String(datas,0,datas.length,"utf8");


//乱码
1、字节数不够
msg=new String(datas,0,datas.length-2,"utf8");
2、字符集不统一，编码和解码所使用的字典不一样
msg=new String(datas,0,datas.length,"gbk");

```

# 4、四个抽象类

java无权操作文件，都是叫OS来做

InputStream,OutputStream,Reader,Writer

![6.PNG](https://note.youdao.com/yws/api/personal/file/E49D8672A1044391AF19D6545199F773?method=download&shareKey=3c18b25eca137bd0f9212992871d3917)



## 4.1、文件字节输入流

```java
package com.whpu.io;
import java.io.*;
/**
 * @author Date 2019/8/19-14:58
 * 1、创建源
 * 2、选择流
 * 3、操作
 * 4、释放资源
 */
public class IOTest01 {
    public static void main(String[] args) {

        //1、创建源
        File src = new File("src/main/abc.txt");
        //2、选择流
        InputStream is = null;
        try {
            is = new FileInputStream(src);
            //3、操作（读取）
            int temp;
            while ((temp=is.read())!=-1){
                System.out.println((char)temp);
            }
            //返回的是实际的数据
            //int data1 = is.read();
            //int data2 = is.read();
            //int data3 = is.read();
            //int data4 = is.read();
            //如果到达文件末尾会返回-1，表示没有数据了
            //System.out.println((char) data1);
            //System.out.println((char) data2);
            //System.out.println((char) data3);
            //System.out.println((char) data4);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4、释放资源
            try {
//                assert is != null;判断为真就正常运行，否则抛出异常终止执行
                if (is != null) {
                    is.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

}

```



```java
package com.whpu.io;



import java.io.*;

/**
 * @author Date 2019/8/19-14:58
 * 分段读取
 * 1、创建源
 * 2、选择流
 * 3、操作
 * 4、释放资源
 */
public class IOTest01 {
    public static void main(String[] args) {

        //1、创建源
        File src = new File("abc.txt");
        //2、选择流
        InputStream is = null;
        try {
            is = new FileInputStream(src);
            //3、操作（分段读取）
            int temp;
            byte[] flush=new byte[3];//new byte[1024*10]缓冲容器,设定每次读取的大小
            int len=-1;//接收长度
            while ((len=is.read(flush))!=-1){
                //字节数组--->字符串（解码）
                String str = new String(flush, 0, len);
                System.out.println(str);
            }
            
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4、释放资源
            try {
//                assert is != null;判断为真就正常运行，否则抛出异常终止执行
                if (is != null) {
                    is.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

```

## 4.2、文件字节输出流

```java
package com.whpu.io;



import java.io.*;

/**
 * @author Date 2019/8/19-14:58
 * 1、创建源
 * 2、选择流
 * 3、操作
 * 4、释放资源
 */
public class IOTest01 {
    public static void main(String[] args) {

        //1、创建源
        File dest = new File("abcd.txt");
        //2、选择流
        OutputStream os = null;
        try {
            //true，表示追加，false表示从头开始重新写，默认就是false
            os=new FileOutputStream(dest,true);
            //3、操作（写出）
            String msg="IO is so easy!";
            //字符串-->字节数组（编码）
            byte[] datas=msg.getBytes();
            os.write(datas,0,datas.length);
            //避免数据驻留在内存中
            os.flush();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4、释放资源
            try {
//                assert is != null;判断为真就正常运行，否则抛出异常终止执行
                if (os != null) {
                    os.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

}

```



文件拷贝

```java
package com.whpu.io;

import java.io.*;

/**
 * @author Date 2019/8/20-11:07
 * 拷贝文件
 */
public class Copy {
    public static void main(String[] args) {
        copy("C:\\Users\\Administrator\\Desktop\\java_IO笔记\\images\\6.PNG","copy.png");
    }

    public static void copy(String srcPath, String destPath) {
        //1、创建源
        File src = new File(srcPath);
        File dest = new File(destPath);
        //2、选择流
        InputStream is = null;
        OutputStream os = null;
        try {
            is = new FileInputStream(src);
            os = new FileOutputStream(dest);
            //3、操作
            byte[] flush = new byte[1024];
            int len=-1;
            while ((len=is.read(flush))!=-1) {
                os.write(flush,0,len);
            }
            os.flush();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4、释放资源,先打开的后关闭
            try {
                if (os != null) {
                    os.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (is != null) {
                    is.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

    }
}
```

## 4.3、文件字符输入流

==中英文混合时，用文件字节流处理文件可能会产生乱码，在UTF-8中,中英文对应的字节数不同，当用字节数组按一定字节数接收时会出现产生乱码的条件==

```java
package com.whpu.io;
import java.io.*;

/**
 * @author Date 2019/8/19-14:58
 * 1、创建源
 * 2、选择流
 * 3、操作
 * 4、释放资源
 */
public class IOTest01 {
    public static void main(String[] args) {

        //1、创建源
        File src = new File("abc.txt");
        //2、选择流
        Reader reader = null;
        try {
            reader=new FileReader(src);
            //3、操作
            char[] flush=new char[1024];
            int len=-1;
            while ((len = reader.read(flush)) != -1) {
                String str = new String(flush, 0, len);
                System.out.println(str);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4、释放资源
            try {
                if (reader != null) {
                    reader.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

}

```

## 4.4、文件字符输出流

```java
package com.whpu.io;
import java.io.*;
/**
 * @author Date 2019/8/19-14:58
 * 1、创建源
 * 2、选择流
 * 3、操作
 * 4、释放资源
 */
public class IOTest01 {
    public static void main(String[] args) {

        //1、创建源
        File dest = new File("abcff.txt");
        //2、选择流
        Writer writer = null;
        try {
            writer=new FileWriter(dest);
            //3、操作
            String msg="IO is so easy";
            //字符串-->字符数组
            //写法一：
//            char[] datas=msg.toCharArray();
//            writer.write(datas,0,datas.length);

            //写法二
            //writer.write(msg);可以指定msg的内容写入

            //写法三
//            writer.append("IO is so easy").append("我开始了");
            
            writer.flush();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4、释放资源
            try {
                if (writer != null) {
                    writer.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

```

# 5、字节数组流

源是内存（字节数组），而不是文件，所以不需要借助os，不存在释放资源一说

靠GC(垃圾回收)自动释放

方便网络传输

```java
package com.whpu.io;
import java.io.*;
/**
 * 输入流
 * @author Date 2019/8/19-14:58
 * 1、创建源
 * 2、选择流
 * 3、操作
 * 4、释放资源
 */
public class IOTest01 {
    public static void main(String[] args) {
        //1、创建源
        byte[] src="talk is cheap".getBytes();
        //2、选择流
        InputStream is = null;
        try {
            is=new ByteArrayInputStream(src);
            //3、操作
            byte[] flush=new byte[5];//缓冲容器
            int len=-1;
            while ((len = is.read(flush)) != -1) {
                String string=new String(flush,0,len);
                System.out.println(string);
            }
        }catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4、释放资源

        }
    }

}

```



```java
package com.whpu.io;
import java.io.*;
/**
 * @author Date 2019/8/19-14:58
 * 字节数组输出流
 * 1、创建源,内部维护，不需要自己提供目的地
 * 2、选择流，不关联源
 * 3、操作（写出内容）
 * 4、释放资源，可以不用
 * 获取数据toByteArray
 */
public class IOTest01 {
    public static void main(String[] args) {
        //1、创建源
        byte[] dest=null;
        //2、选择流(由于要使用ByteArrayOutputStream的新增方法，所以不能使用OutputStream声明)
        ByteArrayOutputStream baos = null;
        try {
            baos=new ByteArrayOutputStream();
            //3、操作
            String msg="show me the code";
            byte[] datas=msg.getBytes();
            baos.write(datas,0,datas.length);
            baos.flush();
            //获取数据
            dest=baos.toByteArray();
            System.out.println(dest.length+"---->"+new String(dest,0,baos.size()));
        }catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4、释放资源

        }
    }
}

```

## 5.1、图片和字节数组转换

实际上是流的对接，文件需要文件流，而字符串可以直接获取字节数组

```java
package com.whpu.io;
import java.io.*;
/**
 * @author Date 2019/8/22-9:21
 * 1、图片-->字节数组
 * 2、字节数组-->图片
 */
public class IOTest02 {
    public static void main(String[] args) {
        byte[] datas = fileToByteArray("copy.png");
        System.out.println(datas.length);
        byteArrayToFile(datas, "xxx.png");
    }

    /**
     * 1、图片-->字节数组
     * 图片到程序 FileInputStream
     * 程序到字节数组 ByteArrayOutputStream
     */
    public static byte[] fileToByteArray(String filePath) {
        File src = new File(filePath);
        byte[] dest = null;
        InputStream is = null;
        ByteArrayOutputStream baos = null;
        try {
            is = new FileInputStream(src);
            baos = new ByteArrayOutputStream();
            byte[] flush = new byte[1024 * 10];//程序提供的数据缓冲
            int len = -1;
            while ((len = is.read(flush)) != -1) {
                baos.write(flush, 0, len);//写出到字节数组
            }
            baos.flush();
            return baos.toByteArray();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (is != null) {
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return null;
    }

    /**
     * 2、字节数组-->图片
     * 字节数组到程序ByteArrayInputStream
     * 程序写出到文件FileOutputStream
     */
    public static void byteArrayToFile(byte[] src, String filePath) {
        //1、创建源
        //2、选择流
        File dest = new File(filePath);
        InputStream is = null;
        OutputStream os=null;
        try {
            is = new ByteArrayInputStream(src);
            os=new FileOutputStream(dest);
            //3、操作
            byte[] flush = new byte[5];//缓冲容器
            int len = -1;
            while ((len = is.read(flush)) != -1) {
                os.write(flush,0,len);
            }
            os.flush();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4、释放资源
            if (os != null) {
                try {
                    os.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

}

```

# 6、工具类

```java
package com.whpu.io;

import java.io.*;

/**
 * 封装拷贝
 * 封装释放资源
 * @author Date 2019/8/22-13:49
 */
public class FileUtils {
    public static void main(String[] args) {
        //文件到文件
        try {
            InputStream is=new FileInputStream("");
            OutputStream os=new FileOutputStream("");
            copy(is,os);
        } catch (IOException e) {
            e.printStackTrace();
        }

        //文件到字节数组
        byte[] datas=null;
        try {
            InputStream is=new FileInputStream("");
            ByteArrayOutputStream os= new ByteArrayOutputStream();
            copy(is,os);
            datas = os.toByteArray();
            System.out.println(datas.length);
        } catch (IOException e) {
            e.printStackTrace();
        }
        //字节数组到文件
        try {
            ByteArrayInputStream is = new ByteArrayInputStream(datas);
            OutputStream os=new FileOutputStream("");
            copy(is,os);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 对接输入输出流
     * @param is
     * @param os
     * @return void
     */
    public static void copy(InputStream is, OutputStream os) {
        try {
            //3、操作
            byte[] flush = new byte[1024];
            int len=-1;
            while ((len=is.read(flush))!=-1) {
                os.write(flush,0,len);
            }
            os.flush();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4、释放资源,先打开的后关闭
            close(is,os);
        }

    }

    /**
     * 释放资源
     * @param closeables
     * @return void
     */
    public static void close(Closeable... closeables){
        //输入输出流都实现了Closeable接口
        for (Closeable closeable : closeables) {
            if (closeable != null) {
                try {
                    closeable.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }

    /**
     * 此处写法会自动释放资源
     * @param i
     * @param o
     * @return void
     */
    public static void copy2(InputStream i, OutputStream o) {
        try (InputStream is = i;
             OutputStream os = o) {
            //3、操作
            byte[] flush = new byte[1024];
            int len=-1;
            while ((len=is.read(flush))!=-1) {
                os.write(flush,0,len);
            }
            os.flush();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```



# 7、BufferedInputStream&BufferedOutputStream

字节缓冲流，一种处理流

1. 提高性能
2. 最底层是节点流
3. 释放资源其实就是释放底层的节点流

![7.PNG](https://note.youdao.com/yws/api/personal/file/A508E7DD920340E59067BC611471A24A?method=download&shareKey=2f7604ba06bb72ffb21ebe5b4fdc899a)

```java
   //这是字节缓冲流 

public static void copy(String srcPath, String destPath) {
        //1、创建源
        File src = new File(srcPath);
        File dest = new File(destPath);
        try(InputStream is=new BufferedInputStream(new FileInputStream(src));
            OutputStream os=new BufferedOutputStream(new FileOutputStream(dest)) ) {
            //3、操作
            byte[] flush = new byte[1024];
            int len=-1;
            while ((len=is.read(flush))!=-1) {
                os.write(flush,0,len);
            }
            os.flush();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
```

字符缓冲流`BufferedReader,BufferedWriter`

```java
    //纯文本拷贝
    public static void copy3(String srcPath, String destPath) {
        //1、创建源
        File src = new File(srcPath);
        File dest = new File(destPath);
        try(BufferedReader br=new BufferedReader(new FileReader(src));
            BufferedWriter bw=new BufferedWriter(new FileWriter(dest))) {
            //3、操作(逐行读取)
            String line=null;
            while ((line=br.readLine())!=null) {
                bw.write(line);
                bw.newLine();
            }
            bw.flush();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
```





# 8、转换流

![8.PNG](https://note.youdao.com/yws/api/personal/file/BDE72063B48142F48E8CF394AD33F326?method=download&shareKey=82a3339ed5a0bc629efe90b7233b47ee)

处理纯文本的字节流

```java
package com.whpu.io;

import java.io.*;

/**
 * @author Date 2019/8/22-16:20
 * 1、以字符流的形式操作字节流
 * 2、指定字符集
 */
public class ConvertTest {
    public static void main(String[] args) {
        //操作System.in和System.out,字节输入流和字节输出流
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
             BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(System.out))) {
            String msg="";
            while (!msg.equals("exit")) {
                msg=reader.readLine();
                writer.write(msg);
                writer.newLine();
                writer.flush();//强制刷新，数据可能留在管道中，没有显示
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

```java
package com.whpu.io;

import java.io.*;
import java.net.URL;

/**
 * @author Date 2019/8/22-16:20
 * 1、以字符流的形式操作字节流
 * 2、指定字符集
 */
public class ConvertTest {
    public static void main(String[] args) {
        //操作网络流 下载百度的源码,openStream()方法获取到了网页源码的字节流,注意一定要指定正确的字符集进行解码
        try (BufferedReader br = new BufferedReader(new InputStreamReader(new URL("https://www.163.com").openStream(), "gbk"));
        BufferedWriter bw =new BufferedWriter(new OutputStreamWriter(new FileOutputStream("baidu.html"), "utf-8"))) {
            String msg;
            while ((msg = br.readLine()) != null) {
                System.out.println(msg);
                bw.write(msg);
                bw.newLine();
            }
            bw.flush();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

# 9、数据流

DataInputStream&DataOutputStream

保留数据类型

```java
package com.whpu.io;
import java.io.*;
/**
 * @author Date 2019/8/23-9:56
 * 数据流
 * 1、先写出后读取
 * 2、读取的顺序与写出保持一致
 *
 *
 */
public class DataTest {
    public static void main(String[] args) throws IOException {
        //写出
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        DataOutputStream dos=new DataOutputStream(baos);
        //操作数据类型+数据
        dos.writeUTF("编码");
        dos.writeInt(18);
        dos.writeBoolean(false);
        dos.writeChar('c');
        dos.flush();
        byte[] datas=baos.toByteArray();
        //读取
        DataInputStream dis=new DataInputStream(new ByteArrayInputStream(datas));
        //顺序与写出一致
        String msg=dis.readUTF();
        int age=dis.readInt();
        boolean flag=dis.readBoolean();
        char ch=dis.readChar();
        System.out.println(flag);
    }
}

```

# 10、对象流

![9.PNG](https://note.youdao.com/yws/api/personal/file/8C9AD2311B4941E1945983C21090E283?method=download&shareKey=4c4274871661fe6ee81ef5ed6a5aad5d)

```java
package com.whpu.io;

import java.io.*;

/**
 * @author Date 2019/8/23-9:56
 * 数据流
 * 1、先写出后读取
 * 2、读取的顺序与写出保持一致
 * 3、不是所有的对象都可以序列化
 */
public class ObjectTest {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        //写出
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        ObjectOutputStream dos = new ObjectOutputStream(baos);
        //操作数据类型+数据
        dos.writeUTF("编码");
        dos.writeInt(18);
        dos.writeBoolean(false);
        dos.writeChar('c');
        dos.writeObject("我你它");
        Employee emp = new Employee("马云", 400);
        dos.writeObject(emp);
        dos.flush();
        byte[] datas = baos.toByteArray();
        //读取
        ObjectInputStream dis = new ObjectInputStream(new ByteArrayInputStream(datas));
        //顺序与写出一致
        String msg = dis.readUTF();
        int age = dis.readInt();
        boolean flag = dis.readBoolean();
        char ch = dis.readChar();
        System.out.println(flag);
        //对象数据还原
        Object str = dis.readObject();
        Object employee = dis.readObject();
        if (str instanceof String) {
            String strObj=(String) str;
            System.out.println(strObj);
        }
        if (employee instanceof Employee) {
            Employee empObj=(Employee) employee;
            System.out.println(empObj.getName());
        }
    }
}

class Employee implements Serializable {
    private transient String name;//transient该数据不需要序列化
    private double salary;

    public Employee() {
    }

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    public String getName() {
        return name;
    }

    public double getSalary() {
        return salary;
    }
}
```



写入到文件

```java
package com.whpu.io;

import java.io.*;

/**
 * @author Date 2019/8/23-9:56
 * 数据流
 * 1、先写出后读取
 * 2、读取的顺序与写出保持一致
 * 3、不是所有的对象都可以序列化
 */
public class ObjectTest {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        //写出
        ObjectOutputStream dos = new ObjectOutputStream(new FileOutputStream("obj.txt"));
        //操作数据类型+数据
        dos.writeUTF("编码");
        dos.writeInt(18);
        dos.writeBoolean(false);
        dos.writeChar('c');
        dos.writeObject("我你它");
        Employee emp = new Employee("马云", 400);
        dos.writeObject(emp);
        dos.flush();
        dos.close();
        //读取
        ObjectInputStream dis = new ObjectInputStream(new FileInputStream("obj.txt"));
        //顺序与写出一致
        String msg = dis.readUTF();
        int age = dis.readInt();
        boolean flag = dis.readBoolean();
        char ch = dis.readChar();
        System.out.println(flag);
        //对象数据还原
        Object str = dis.readObject();
        Object employee = dis.readObject();
        if (str instanceof String) {
            String strObj=(String) str;
            System.out.println(strObj);
        }
        if (employee instanceof Employee) {
            Employee empObj=(Employee) employee;
            System.out.println(empObj.getName());
        }
        dis.close();
    }
}

class Employee implements Serializable {
    private transient String name;//transient该数据不需要序列化
    private double salary;

    public Employee() {
    }

    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    public String getName() {
        return name;
    }

    public double getSalary() {
        return salary;
    }
}
```

# 11、打印流(了解)

PrintStream,包装OutputStream

```java
        //打印流System.out,默认打印到控制台
        PrintStream ps=System.out;
        ps.println("dd");

        //打印到文件
        ps=new PrintStream(new BufferedOutputStream(new FileOutputStream("print.txt")));
        ps.println("xxx");
        ps.flush();
        ps.close();
        //重定向输出端
        System.setOut(ps);
        System.out.println("change");
        //重定向回控制台,true是自动flush
        System.setOut(new PrintStream(new BufferedOutputStream(new FileOutputStream(FileDescriptor.out)),true));
        System.out.println("change");
```

PrintWriter，相比PrintStream多了Writer

```java
        PrintWriter pw=new PrintWriter(new BufferedOutputStream(new FileOutputStream("print.txt")));
        pw.println("xxx");
        pw.flush();
        pw.close();
```

# 12、随机读取和写入流

```java
package com.whpu;

import java.io.*;
import java.util.ArrayList;
import java.util.List;
import java.util.Vector;

/**
 * 随机读取和写入流，RandomAccessFile
 * 面向对象思想封装分割
 *
 * @author liang
 * Date 2019/8/27-19:57
 */
public class SplitFile {
    //源头
    private File src;

    //目的地（文件夹）
    private String destDir;

    //所有分割后的文件存储路径
    private List<String> destPaths;

    //每块大小
    private int blockSize;

    //块数：多少块
    private int size;

    public SplitFile(String srcPath, String destDir, int blockSize) {
        this.src = new File(srcPath);
        this.destDir = destDir;
        this.blockSize = blockSize;
        this.destPaths = new ArrayList<String>();
        //初始化
        this.init();
    }

    //初始化
    private void init() {
        //总长度
        long len = this.src.length();
        //多少块
        this.size = (int) Math.ceil(len * 1.0 / blockSize);//向上取整
        //路径
        for (int i = 0; i < size; i++) {
            this.destPaths.add(i + "-" + this.src.getName());
//            导致报错，具体看后面解释
//            this.destPaths.add(this.destDir+"/"+"-"+this.src.getName());
        }
    }

    //分割
    public void split() throws IOException {
        //总长度
        long len = src.length();

        //起始位置和实际大小
        int beginPos = 0;
        int actualSize = (int) (blockSize > len ? len : blockSize);
        for (int i = 0; i < size; i++) {
            beginPos = i * blockSize;
            if (i == size - 1) {
                //最后一块
                actualSize = (int) len;
            } else {
                actualSize = blockSize;
                len -= actualSize;
            }
            splitDetail(i, beginPos, actualSize);
        }
    }

    public static void main(String[] args) throws IOException {
        SplitFile sf = new SplitFile("Thread/src/com/whpu/SplitFile.java", "dest", 1024);
        sf.split();
        sf.merge("merge.java");

    }

    /**
     * 指定第i块的起始位置和实际长度
     *
     * @param i
     * @param beginPos
     * @param actualSize
     * @return void
     */
    private void splitDetail(int i, int beginPos, int actualSize) throws IOException {
        RandomAccessFile raf = new RandomAccessFile(this.src, "r");
        //RandomAccessFile 在rw模式下。不能跨目录创建文件。
        //比如原目录是test\\shareMemory\
        //则只能在该目录下直接创建，如果想让其直接创建test\\shareMemory\test\a.file
        //就会报文件找不到的异常。因此必须手动创建test\\shareMemory\test 目录后，方可自行创建文件。
        RandomAccessFile raf2 = new RandomAccessFile(this.destPaths.get(i), "rw");

        //随机读取
        raf.seek(beginPos);
        //读取
        byte[] flush = new byte[1024];
        int len = -1;
        while ((len = raf.read(flush)) != -1) {
            if (actualSize > len) {
                //获取本次读取的所有内容
                raf2.write(flush, 0, len);
                actualSize -= len;
            } else {
                raf2.write(flush, 0, actualSize);
                break;

            }

        }
        raf2.close();
        raf.close();
    }

    /**
     * 文件合并
     *
     * @param
     * @return void
     */
    public void merge(String destPath) throws IOException {
//        //输出流
//        OutputStream os = new BufferedOutputStream(new FileOutputStream(destPath, true));
//        //输入流
//        for (int i = 0; i < destPaths.size(); i++) {
//            InputStream is = new BufferedInputStream(new FileInputStream(destPaths.get(i)));
//            //拷贝
//            byte[] flush = new byte[1024];
//            int len = -1;
//            while ((len = is.read(flush)) != -1) {
//                os.write(flush, 0, len);
//            }
//            os.flush();
//            is.close();
//        }
//        os.close();

        //另一种写法
        //输出流
        OutputStream os = new BufferedOutputStream(new FileOutputStream(destPath, true));
        Vector<InputStream> vi=new Vector<>();
        SequenceInputStream sis=null;
        //输入流放入容器
        for (int i = 0; i < destPaths.size(); i++) {
            vi.add(new BufferedInputStream(new FileInputStream(destPaths.get(i))));
        }
        sis=new SequenceInputStream(vi.elements());
        //拷贝
        byte[] flush = new byte[1024];
        int len = -1;
        while ((len = sis.read(flush)) != -1) {
            os.write(flush, 0, len);
        }
        os.flush();
        sis.close();
        os.close();

    }
}

```

# 13、使用公共库

Apache提供了Commons-io来进行更方便的文件操作