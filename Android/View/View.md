### View树的绘制流程

measure -> layout -> draw

#### measure

从上而下递归进行

![image-20210211095000834](image-20210211095000834.png)

1,ViewGroup.LayoutParams 

2，MeasureSpec

3,



### 事件分发



![image-20210213143420736](image-20210213143420736.png)

![image-20210213143515020](image-20210213143515020.png)



![image-20210213143622836](image-20210213143622836.png)



标题和主题颜色显示在DecorView.

Phonewindow作为Window（抽象类）的唯一实现类，是view的事件管理容器

DecorView 是 Phonewindow的内部类。为什么这么设计？

![image-20210213143908409](image-20210213143908409.png)

activity 没有 onInterceptTouchEvent 拦截事件功能，如果activity具备这个功能，那么没有view能响应，这个功能加进来没有意义

![image-20210213144307480](image-20210213144307480.png)

![image-20210213144359233](image-20210213144359233.png)

#### listview

![image-20210213144800243](image-20210213144800243.png)



#### RecycleBin(View 回收利用)机制

![image-20210213151506201](image-20210213151506201.png)

![image-20210213151658004](image-20210213151658004.png)







































