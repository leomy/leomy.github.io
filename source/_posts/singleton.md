---
title: 单例模式
toc: true
date: 2018-03-26 22:00:05
categories:
- design pattern
tags: [Basic, Design Pattern, Java]
---

单例模式保证内存中某个资源只有一份。但如果有懒加载的要求并且Java中的多线程安全性问题、Java存在反射/序列化/克隆等攻击方式，于此便有了如饿汉式、懒汉式和双重校验锁等解决方案。针对每种解决方案分析了其优缺点和介绍了如何进行反射、序列化、克隆方式攻击
## 饿汉式
1. 优点：线程安全
2. 缺点：
    - 没有延迟加载
    - 并不能防止反射、克隆、序列化对单例模式的破坏
3. 分析：通过ClassLoader的loadClass(String, boolean)方法内的synchronized块保证原子性，可见性，有序性来保证线程安全
```Java
    public class Singleton{
    
    private static final Singleton instance = new Singleton();
    
    private Singleton(){
    }
    
    public static Singleton getInstance(){
        return instance;
    }
}
```
## 懒汉式
1. 优点：延迟加载
2. 缺点：
    - 线程不安全，多线程环境下不能使用
    - 并不能防止反射、克隆、序列化对单例模式的破坏
```Java
public class Singleton{
    
    private static Singleton instance;
    
    private Singleton(){
    }
    
    public static Singleton getInstance(){
        if(instance == null){
            instance = new Singleton();
        }
        
        return instance;
    }
}
```
## 双重校验锁
注: `volatile`在jdk1.5及以上才能正常工作
1. 优点：延迟加载,线程安全
2. 缺点：并不能防止反射、克隆、序列化对单例模式的破坏
3. 分析：`getInstance()`变化不用说。择重说明为什么`instance`必须要用`volatile`关键字修饰
    - `volatile`
        - 保证可见性
            - 当写一个`volatile`变量时,`JMM`把该线程对应的工作内存中的共享变量值刷新到主内存
            - 当读一个`volatile`变量时,`JMM`把该线程对应的工作内存置为无效
        - 防止重排序：通过插入内存屏障指令
    - `new` 关键字并不是一个原子操作，可分为四步(只是举例)：
        1. 栈内存开辟空间给引用
        2. 堆内存开辟空间准备初始化对象
        3. 初始化对象
        4. 栈中引用指向堆内存空间地址
        
        在这期间可能会发生指令重排序如1,2,4,3.当执行完4时getInstance()就返回了,别的线程使用时会抛出空指针异常.然后利用volatile的防止指令重排序的功能使其完善
```Java
    public class Singleton{
    
    private volatile static Singleton instance;
    
    private Singleton(){
    }
    
    public static Singleton getInstance(){
        if(instance == null){
            synchronized(Singleton.class){
                if(instance == null){
                    instance = new Singleton();
                }
            }
        }
        
        return instance;
    }
}
```

## 静态内部类
1. 优点：延迟加载，线程安全
2. 缺点：并不能防止反射、克隆、序列化对单例模式的破坏
3. 分析: 
    - 线程安全实现同上
    - 延迟加载利用类的初始化时机(遇到`new`、`getstatic`、`putstatic`或`invokestatic`这4条字节码指令时)：
        - 使用`new`创建类的实例
        - 访问`static`字段(`static final`在编译期替换，不会触发类加载)或`static`方法
        - 反射调用类
        - 初始化子类时，父类还没有初始化，则先初始化父类
        - `main`所在的类
```Java
public class Singleton{
    
    private static class SingletonHolder{
    
        private static Singleton instance = new Singleton();
    }
    
    private Singleton(){
    }
    
    public static Singleton getInstance(){
        return SingletonHolder.instance;
    }

}
```
## 枚举
1. 优点：线程安全，可防止反射，序列化，克隆对单例模式的破坏
2. 缺点：不能懒加载
3. 分析：
```Java
public enum Singleton{
    INSTANCE;
    
    public void whateverMethod(){
    }
}
```
## 反射攻击
- 破坏方式
    ```Java
    @Test
    public void test() throws Exception {
        // 正常方式获得实例
        Singleton instance = Singleton.getInstance();

        // 反射重新生成实例
        Constructor<Singleton> constructor = Singleton.class.getDeclaredConstructor();
        constructor.setAccessible(true);
        Singleton newInstance = constructor.newInstance();

        // 查看是否相等
        System.out.println(instance);
        System.out.println(newInstance);
        System.out.println(instance.equals(newInstance));
    }
    ```
- 应对方式:
## 克隆攻击
- 克隆要求类实现`Serializable`接口，不实现该接口进行序列化时会抛出`java.lang.CloneNotSupportedException`异常
- 当实现`Cloneable`接口重写`clone()`后就可以破坏单例模式
- 应对方式
    - 不实现`Cloneable`接口
    - 实现`Cloneable`接口方法的同时按照下面方式重写`clone()`方法
        ```Java
        @Override
        protected Object clone() throws CloneNotSupportedException {
            // 返回同一个实例
            return getInstance();
        
            // 或抛出异常，但并不推荐
            // throw new CloneNotSupportedException();
        }
        ```
## 序列化攻击
- 前提
    - 序列化要求类实现`Serializable`接口，不实现该接口进行序列化时会抛出`java.io.NotSerializableException`异常
    - 当实现`Serializable`接口后就可以破坏单例模式
```Java
@Test
public void test() throws Exception {
    try (ObjectOutputStream oos = new ObjectOutputStream(Files.newOutputStream(Paths.get("C:/Users/leo/Desktop/file")));
        ObjectInputStream ois = new ObjectInputStream(Files.newInputStream(Paths.get("C:/Users/leo/Desktop/file")))) {
        oos.writeObject(Singleton.getInstance());
        Singleton instance = (Singleton) ois.readObject();

        System.out.println(Singleton.getInstance());
        System.out.println(instance);
        System.out.println(Singleton.getInstance().equals(instance));
    }
}
```
- 应对方式
    - 不实现`Serializable`接口
    - 实现`Serializable`接口的同时，添加下列方法(在反序列化的时候会自动调用该方法)
        ```Java
        public Object readResolve() {
            // 直接返回同一个实例
            return getInstance();
            
            // 或抛出异常,但并不推荐
            // throw new NotSerializableException();
        }
        ```