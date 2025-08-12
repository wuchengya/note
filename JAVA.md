# JAVA



## 异常捕获try catch

### 虚拟机默认处理异常的方式：

把异常信息以红色字体打印在控制台，并结束程序。

### 灵魂四问：

1：如果try中不存在问题，代码该如何执行？

会将try中的代码执行完毕，不会执行catch中的代码。

2：如果try中有多个异常，代码该如何执行？

当在try中检测到异常时，就直接跳到对应的catch中，try中的代码不会继续执行。

如何解决多个异常问题？

设置多个catch，注意父类异常要写在下面。

3：如果try中的异常没有被catch捕获，代码该如何执行？

例如：try中的异常是数组索引越界，但是catch捕获的是空指针异常。

相当于try-catch白写了，虚拟机会按照默认方式处理

4：如果try中遇到了问题，那么try下面的代码还会被执行吗？

不会，会直接跳转到相应的catch语句中，执行catch中的代码，如果没有相应的代码，依旧由虚拟机进行默认操作。

## 异常中的常见方法

public String getMessage():返回此throwable的详细消息字符串

public String toString():返回可抛出的简短描述

public void printStackTrace():把异常的错误信息以红色字体输出在控制台，但仅仅是打印信息，不会终止程序运行。

printStackTrace()是包含了上面两个方法的内容，所以说这个用到的最多。



### 抛出throw throws

throws是在方法小括号后面写的表示本方法可能存在哪些异常

throw就是抛出某个异常，抛出后如果没有被catch接受那么程序会终止

### 自定义异常

目的：就是为了让控制台的报错更加的见名知意

如果是运行时异常就直接继承RunTimeException,编译时异常就直接继承Exception

自定义异常就是换个名字而已

步骤：定义异常类，写继承关系，空参和带参构造

如果你在抛出异常时异常中传入了字符串，那么在catch中如果检测到了这个异常，无论你调用哪个方法都会显示传入的字符串，只是显示信息多少的问题。（只要是异常就会这样）

## FILE

### File的构造

| `File(String pathname)`             |      ``File file = new File("C:\\test\\myfile.txt");``       |
| ----------------------------------- | :----------------------------------------------------------: |
| `File(String parent, String child)` |     ``File file = new File("C:\\test", "myfile.txt");``      |
| `File(File parent, String child)`   | ``File parentDir = new File("C:\\test");``<br/>``File file = new File(parentDir, "myfile.txt");`` |

### File相关方法

|            方法名字             |                作用                |
| :-----------------------------: | :--------------------------------: |
|   public boolean isDirectoy()   | 判断此路径名表示的File是否是文件夹 |
|     public boolean isFile()     |  判断此路径名表示的File是否是文件  |
|      public boolean exists      |   判断此路径名表示的File是否存在   |
|      public long length()       |       返回文件的大小（字节）       |
| public String getAbsolutePath() |         返回文件的绝对路径         |
|     public String getPath()     |      返回定义文件时使用的路径      |
|     public String getName()     |       返回文件的名称，带后缀       |
|   public long lastModified()    | 返回文件的最后修改时间(时间毫秒值) |

length()方法只能获取文件的大小，单位是字节。想要得到M可以/1024

|            方法名称            |              说明              |
| :----------------------------: | :----------------------------: |
| public boolean createNewFile() |       创建一个新的空文件       |
|     public boolean mkdir()     |         创建单级文件夹         |
|    public boolean mkdirs()     |         创建多级文件夹         |
|    public boolean delete()     | 删除文件和空文件夹(删了就没了) |

```java
createNewFile()
//如果当前文件不存在，返回true,否则返回false
//如果父级路径不存在，方法会抛出异常
//不会创建文件夹，而是没有后缀的文件，谁说文件一定要有后缀？
mkdir()
File f2 = new File("");
boolean b = f2.mkdir();
sout(b);//单级文件夹就是前面都是父路径最后一个是你自己创建的
//多级文件夹就是文件夹里套文件夹，mkdirs就是为了解决这个问题
mkdirs()既可以创建单级文件夹，也可以创建多级文件夹，因此mkdir以后就没用了;
delete();

File f1 = new File("");
boolean b = f1.delete();
如果路径指向的是空文件夹或者说问价，那么可以直接删除，不经过回收站（返回True），如果是非空文件夹，删除失败，返回false
```

|         方法名称          |           说明           |
| :-----------------------: | :----------------------: |
| public File[] listFiles() | 获取当前该路径下所有内容 |

```java
File f1 = new File("");
File[] files = f1.listFiles();
for(File file:files)
{
    sout(file);
}
保证此路径必须是一个文件夹，如果路径不存在或者指向的内容是文件时，返回的是null,如果是空文件夹，返回长度为0的数组。隐藏文件夹也能找到。如果这个文件夹需要你具有访问权限的话，也会返回null.
```

## IO流

### IO流的分类

![屏幕截图 2025-06-10 205517](E:\Typora picture\屏幕截图 2025-06-10 205517.png)

以上四个均为抽象类，无法创建对象。

![屏幕截图 2025-06-10 205741](E:\Typora picture\屏幕截图 2025-06-10 205741.png)

```tex
FileOutputStream小细节
创建对象：
细节一：
FileOutputStream fos = new FileOutputStream("");
参数可以是File对象，也可以是字符串
细节二：
如果说这个文件不存在，那么程序会帮助你创建这个文件，但是要保证父级路径是存在的，否则会报错。
细节三：
如果文件存在，那么会清空文件
写数据：
fos.write(整数);
细节一：
write方法的参数是整数，但是实际上写了整数对应的ASKII码所对应的字符
释放资源：
fos.close();
每次使用完流之后都要释放资源
```

## FileOutputStream

![屏幕截图 2025-06-10 211511](E:\Typora picture\屏幕截图 2025-06-10 211511.png)

```java
FileOutputStream fos = new FileOutputStream("littlegame\\a.txt");
//单个写入
fos.write(97);
//多个写入
byte[] all = {97,98,99,100,101,102,103,104,105};
fos.write(all);
//写入部分
fos.write(all,0,all.length);//第二个参数是起始索引，第三个参数是要写入多少个字符
fos.close();

FileOutputStream fos = new FileOutputStream("E:\\java_code\\littlegame\\a.txt",true);//这里表示追加内容
String str = "你想写入文件的内容";
byte[] bytes = str.getBytes();
fos.write(bytes);
//如何换行？
String enter = "\r\n";
fos.write(enter.getBytes());
fos.close();
```



### FileInputStream

```cpp
FileInputStream fis = new FileInputStream("E:\\java_code\\littlegame\\a.txt");
int ask = fis.read();
System.out.println((char)ask);
fis.close();
fis.read()返回文件数据的ASKII值，想要变成字符要强制类型转换，每读一次，指针就向前走一下，直到读到-1
```

因此可以使用一个循环输出所有文件内容：

```java
FileInputStream fis = new FileInputStream("E:\\java_code\\littlegame\\a.txt");
int ask = fis.read();
while (ask!=-1)
{
    System.out.print((char)ask);
    ask = fis.read();
}
```

拷贝文件

```java
FileInputStream fis = new FileInputStream("E:\\java_code\\littlegame\\a.txt");
FileOutputStream fos = new FileOutputStream("E:\\java_code\\littlegame\\b.txt");
int ask = fis.read();
while (ask!=-1)
{
    fos.write(ask);
    ask = fis.read();
}
fis.close();
fos.close();
```

问题是：一个一个读然后一个一个写真是太慢了，能否快一点呢？可以通过一次行读取多个字符来解决这个问题。

这就提到了read(byte[]);可以通过传入字符数组的方式来读入多个内容。

下面是大文件的拷贝：

```java
FileInputStream fis = new FileInputStream("E:\\java_code\\littlegame\\a.txt");
FileOutputStream fos = new FileOutputStream("E:\\java_code\\littlegame\\b.txt",true);
byte[] buffer = new byte[1024*1024*5];
int len = fis.read(buffer);
//读的时候会尽量装满数组，如果后面没有内容，那么len会返回-1
while(len!=-1)
{
    fos.write(buffer,0,len);
    len = fis.read(buffer);
}
fos.close();
fis.close();//先打开的文件后关闭
```

### 字符集基础

![屏幕截图 2025-06-11 101335](E:\Typora picture\屏幕截图 2025-06-11 101335.png)

![屏幕截图 2025-06-11 102425](E:\Typora picture\屏幕截图 2025-06-11 102425.png)

### 字符流

![屏幕截图 2025-06-11 110954](E:\Typora picture\屏幕截图 2025-06-11 110954.png)

```java
FileReader fr = new FileReader("E:\\java_code\\littlegame\\a.txt");
int ch = fr.read();
这里的ch代表的是中文字符在编码集下的编码，如2514，想要变为汉字要强转。
while (ch != -1) {
    System.out.print((char) ch);
    ch = fr.read();
}
fr.close();
FileReader构造方法的pathname可以是字符串，也可以是File类型，在底层会把字符串转成File类型;


------------------------------------------------------------------------------------------------------------
    
FileReader fr = new FileReader("E:\\java_code\\littlegame\\a.txt");
char[] chars = new char[2];
int len = fr.read(chars);//带参数的read方法，和FileInputStream的read很相似，将读到的字符存入chars中，返回读到字符的个数
while(len!=-1)
{
    System.out.println(new String(chars,0,len));
    len = fr.read(chars);
}
fr.close();   
```

空参的read()方法是一个字节一个字节的读，遇到汉字会读在编码集下的字符数，返回值是将这个字符解码并转成十进制。

带参的read()方法是将读取数据、解码、强转三步合并了，把强转之后的字符放到了数组当中，返回值是读取的个数。

### FIleWriter

![屏幕截图 2025-06-11 113602](E:\Typora picture\屏幕截图 2025-06-11 113602.png)

这和FileOutputStream一样。

![image-20250611113816042](E:\Typora picture\image-20250611113816042.png)

细节和字节输出流一样。









































## 多线程

### 多线程的实现方式

1：继承thread

```java
public class Mythread extends Thread{
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(this.getName()+"正在执行"+i+"个循环");
        }
    }
}

Mythread t1 = new Mythread();
Mythread t2 = new Mythread();
t1.setName("线程一");
t2.setName("线程二");
t1.start();
t2.start();
```

2：实现Runable接口

```java
public class Mythread implements Runnable {

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName()+"Hello World");
        }
    }
}

Mythread m1 = new Mythread();
Thread t1 = new Thread(m1);
Thread t2 = new Thread(m1);
t1.setName("线程一");
t2.setName("线程二");
t1.start();
t2.strat();

Thread.currentThread().getName()是可以在任何时间和地点调用，表示获得当前线程的名字。
```

3：利用Callable和Future接口方式实现（可得到结果）

```java
public class Mythread implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        Integer sum = 0;
        for (int i = 1; i <= 100; i++) {
            sum += i;
        }
        return sum;
    }
}

//创建对象
Mythread mythread = new Mythread();
//FutureTask用来管理线程,可以获得线程返回结果
FutureTask<Integer> futureTask = new FutureTask<Integer>(mythread);
Thread thread = new Thread(futureTask);
thread.start();
int res = futureTask.get();
System.out.println(res);
```

![屏幕截图 2025-06-11 172501](E:\Typora picture\屏幕截图 2025-06-11 172501.png)

### Thread中常见的成员方法

![image-20250611172747081](E:\Typora picture\image-20250611172747081.png)

如果我们没有给线程设置名字，线程也是有默认的格式：Thread-X(X序号，从0开始)

使用setName方法给线程设置名字

我们正常执行程序的线程叫做main线程

```java
Mythread mt = new Mythread();
Thread t1 = new Thread(mt);
System.out.println(t1.getPriority());5
System.out.println(Thread.currentThread().getPriority());5
System.out.println(Thread.currentThread().getName());main
t1.setPriority(10);
```

守护线程：

```java
Mythread mythread = new Mythread();
Thread t1 = new Thread(mythread);
Thread t2 = new Thread(mythread);
t1.setName("女神");
t2.setName("龟龟");
t2.setDaemon(true);//设置龟龟为守护线程
当女神线程结束后，龟龟线程也会陆续结束，基本不会执行到底
```

出让线程yield（了解）

对于两个线程，使用`Thread.yield`会出让CPU控制权，这样可以使两个线程尽可能地平均。

插入线程：

对于两个线程来说，比如，1线程和main线程，如果要实现1线程在main线程之前执行，那就可以调用1.join();就是将调用线程放在当前线程之前调用。

### 线程的生命周期

![屏幕截图 2025-06-11 183932](E:\Typora picture\屏幕截图 2025-06-11 183932.png)

### 线程的安全问题

思考下面问题：

如果说有一个电影院有三个窗口去卖票，总共一百张票，写一段程序描述卖票过程

```java
static int tickets = 0;
@Override
public void run() {
    while (true){
        if(tickets < 100)
        {
            System.out.println(getName()+"正在卖第"+tickets+"张票");
            tickets++;
        }else break;
    }
}

Mythread my1 = new Mythread();
Mythread my2 = new Mythread();
Mythread my3 = new Mythread();
my1.setName("窗口一");
my2.setName("窗口二");
my3.setName("窗口三");
my1.start();
my2.start();
my3.start();
结果：
窗口三正在卖第0张票
窗口三正在卖第1张票
窗口一正在卖第0张票
窗口二正在卖第0张票
窗口三正在卖第2张票
窗口一正在卖第3张票
窗口一正在卖第6张票
窗口二正在卖第4张票
```

很明显有问题！

原因：线程在执行过程中具有随机性，在任何时间都有可能被夺取控制权

### 同步代码块

```java
public class Mythread extends Thread {
    static int tickets = 0;
    static Object obj = new Object();

    @Override
    public void run() {
        while (true) {
            synchronized (obj) {
                if (tickets < 100) {
                    tickets++;
                    System.out.println(getName() + "正在卖第" + tickets + "张票");
                } else break;
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
}
新增关键字，锁;
synchronized(){
    
}
小括号里面填写唯一值，对于最开始来说，锁是开的，线程可以进入，当某个线程进入后，可能会对唯一值做某些手脚，使得它变为关着的，这样当其他线程要夺取控制权时，就会进不来，当这个某个线程执行完锁内代码结束后，唯一值恢复，CPU控制权要被重新夺取。;
注意唯一值一般填写类名.class这个是字节码文件，是唯一的
```

### 同步方法

在权限修饰符后加入synchronized关键字就可以使这个方法变为同步方法。

```java
public class Mythread extends Thread {
    static int tickets = 0;
    static Object obj = new Object();

    @Override
    public void run() {
        while (true) {
            if (method()) break;
        }
    }

    private synchronized boolean method() {
        if (tickets < 100) {
            tickets++;
            System.out.println(getName() + "正在卖第" + tickets + "张票");
        } else return true;
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        return false;
    }
}
快捷键 ctrl+alt+m可以将代码抽出转化为方法
```

### lock锁

```cpp
public class Mythread extends Thread {
    static int tickets = 0;
    static Lock lock = new ReentrantLock();
    @Override
    public void run() {
        while (true) {
            try {
                lock.lock();//一定要保证LOCK锁的唯一性，如果本类被创建了多个对象的话，那要加上static关键字
                if(tickets==100){
                    break;
                }else{
                    tickets++;
                    System.out.println("tickets: "+tickets);
                }
                Thread.sleep(10);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                lock.unlock();//放在finally保证这个代码一定会被执行到，如果仅仅是放在while循环的最后面，那么最后一次锁不会被打开，另外两个线程无法结束，程序就停不掉	
            }
        }
    }
}

```

### 死锁

死锁不是新内容，而是我们要避免的东西。

1. **线程1**获取`lock1`，然后休眠100毫秒
2. **线程2**获取`lock2`，然后休眠100毫秒
3. **线程1**醒来后尝试获取`lock2`，但`lock2`被线程2持有
4. **线程2**醒来后尝试获取`lock1`，但`lock1`被线程1持有
5. 两个线程互相等待对方释放锁，形成死锁

### 等待唤醒机制



### 线程的六大状态

![image-20250612083043204](E:\Typora picture\image-20250612083043204.png)

![image-20250612083242731](E:\Typora picture\image-20250612083242731.png)

### 线程池

核心，一个管理线程的容器，我们只需要去提交任务给线程池，而线程池会自动创建线程去执行我们的任务，如果提交了很多任务，当前线程池如果能够创建新线程的话的就会创建新线程去执行这些任务，否则就会让这些任务等待，直到前面的任务执行完毕。

```java
ExecutorService pool1 = Executors.newCachedThreadPool();//创建一个没有上限的线程池
pool1.submit(new Mythread());
pool1.submit(new Mythread());
pool1.submit(new Mythread());
pool1.submit(new Mythread());
//销毁线程池
pool1.close();
ExecutorService pool2 = Executors.newFixedThreadPool(3);//创建一个有上限的线程池
和上面一样提交然后销毁
```

### 自定义线程池

![image-20250612103307213](E:\Typora picture\image-20250612103307213.png)

```cpp
ThreadPoolExecutor pool = new ThreadPoolExecutor(
        3,//核心线程数量，不能小于0
        6,//最大线程数量，不能小于0且大于等于核心线程数量
        60,//空闲线程最大存活时间
        TimeUnit.SECONDS,//时间单位
        new ArrayBlockingQueue<>(3),//任务队列
        Executors.defaultThreadFactory(),//创建线程工厂
        new ThreadPoolExecutor.AbortPolicy()//人物的拒绝策略
);
接下来提交任务即可
```

模拟一下过程：如果提交了10个任务，那么前三个由核心线程去执行，之后三个(任务队列长度)执行等待操作，后面三个由临时线程执行，最后一个无法执行，直抛出错误。

![image-20250612105410231](E:\Typora picture\image-20250612105410231.png)

## 网络编程

### 网络编程三要素

![image-20250612140344782](E:\Typora picture\image-20250612140344782.png)

### IP

![image-20250612141043418](E:\Typora picture\image-20250612141043418.png)

### InetAddress

```java
InetAddress address = InetAddress.getByName("L--Apricot");
System.out.println(address);
String name = address.getHostName();
System.out.println(name);
String ip = address.getHostAddress();
System.out.println(ip);

输出结果
L--Apricot/10.32.94.109
L--Apricot
10.32.94.109
```

### 协议

![image-20250612143705300](E:\Typora picture\image-20250612143705300.png)

### UDP协议

```java
发送数据
DatagramSocket socket = new DatagramSocket();//创建快递公司，从随机端口发送数据
String s = "Hello World";
byte[] bytes = s.getBytes();
InetAddress addr = InetAddress.getByName("L--Apricot");
int port = 10086;
DatagramPacket packet = new DatagramPacket(bytes, bytes.length, addr, port);//打包数据
socket.send(packet);//发送数据
socket.close();//释放资源
接收数据
DatagramSocket ds = new DatagramSocket(10086);
byte[] buf = new byte[1024];
DatagramPacket dp = new DatagramPacket(buf, buf.length);
ds.receive(dp);
byte[] data = dp.getData();
int len = dp.getLength();
InetAddress ip = dp.getAddress();
int port = dp.getPort();
System.out.println("收到数据"+new String(data));
System.out.println("该数据是从"+ip+"发出");
ds.close();
```

### TCP通信

![image-20250612170721459](E:\Typora picture\image-20250612170721459.png)

```java
public class Client {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1", 10001);
        OutputStream os = socket.getOutputStream();
        os.write("Hello World".getBytes());
        os.flush();
        socket.shutdownOutput(); // 告诉服务端写完了
        socket.close();
    }
}

public class Serve {
    public static void main(String[] args) throws IOException {
        //服务端，接受客户端的数据
        ServerSocket ss = new ServerSocket(10001);
        Socket s = ss.accept();
        InputStream is = s.getInputStream();
        int b = is.read();
        while(b!=-1){
            System.out.print((char)b);
            b = is.read();
        }
        s.close();
        ss.close();
    }
}
```

