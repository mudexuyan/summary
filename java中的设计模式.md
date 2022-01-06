## Java中的设计模式
### I/O 装饰者模式
![分](图片/IO装饰者模式.png)
- InputStream是抽象类，主要是read方法，读取下一个字节（0-255），-1表示已读到末尾  
- FileInputStream、ByteArrayInputStream、ServletInputStream 是 InputStream 的三个子类  
- BufferedInputStream（增强功能）、DataInputStream（增加方法readInt()、readLong()）、CheckedInputStream 是三个具体的装饰者类，他们都为 InputStream 增强了原有功能或添加了新功能  
- FilterInputStream 是所有装饰类的父类，它没有实现具体的功能，仅用来包装了一下 InputStream  
```
//缓冲池读取文件，类似于多次装饰
InputStream in = new BufferedInputStream(new FileInputStream("src/readme.txt"))

public class FilterInputStream extends InputStream { 

    protected volatile InputStream in;

    protected FilterInputStream(InputStream in) { 
        this.in = in; 
    } ​ 

    public int read() throws IOException { 
        return in.read(); 
    } 
    //... 
}
```  
### 观察者模式
观察者：java.util.Observer
```
public interface Observer { 
    void update(Observable o, Object arg); 
}
```
被观察者：java.util.Obserable
```
public class Observable { 

    private boolean changed = false; 

    private Vector<Observer> obs; 

    public void notifyObservers(Object arg) { 

        Object[] arrLocal; 

        synchronized (this) { 
            if (!hasChanged()) return; 
            arrLocal = obs.toArray(); 
            clearChanged(); 
        } 
        
        for (int i = arrLocal.length - 1; i >= 0; i--) 
            ((Observer) arrLocal[i]).update(this, arg); 
    }
    ···
```
