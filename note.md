# 学习Netlogo和pyNetlogo的笔记

## 6.22

学习方案：

1. 同时进行两边，pyNetLogo的代码解读和NetLogo的官网教程（Lisp语法）
2. DDL：本周六前完成官网代码的中文教程，pyNetLogo不做要求（可以同时熟悉一下语法）



## 6.26

### NetLogo中文教程

#### 学习例程

四种主体为：patches（瓦片）、turtles（海龟）、links（链）、Observer（观察者），其中前三者能够运行procedures（例程），即一系列NetLogo命令

1. 瓦片是静止的，组成网格。网格一般设定中心为原点（可以设置别的位置），注意下图是世界的坐标信息，其中以瓦片作为一个单元计算坐标，即我们对瓦片的命名为：patch 1 1, patch 23 -4...

   <img src="C:\Users\27096\AppData\Roaming\Typora\typora-user-images\image-20210626021637121.png" alt="image-20210626021637121" style="zoom:67%;" />

2. 海龟是运动的，可以在patches上移动

3. 链连接两个海龟

4. 观察者相当于上帝视角，可以修改和查看所有的事情。它可以做那些海龟、瓦片和链自己都不能做的事情

#### 操作与代码

##### setup

添加”按钮“组件，由于无例程**报错变红**：

<img src="C:\Users\27096\AppData\Roaming\Typora\typora-user-images\image-20210626022409791.png" alt="image-20210626022409791" style="zoom:67%;" />

```lisp
to setup  ; 定义一个叫做setup的例程
  clear-all  ; 清除所有的世界主体和信息（格式化）
  create-turtles 100  ; 创建100个海龟，均在原点（patch 0 0）
  ask turtles  ; 与在command center的操作一样，用'ask ... [ order]'的命令让主体执行命令
    [ setxy random-xcor random-ycor ]  ; 使用reporters（不就是函数嵌套嘛）
end
```

<img src="C:\Users\27096\AppData\Roaming\Typora\typora-user-images\image-20210626023426689.png" alt="image-20210626023426689" style="zoom:67%;" />

##### to

选择”持续执行“，让按钮按下后保持按下状态，命令相当于被重复执行（加了个while循环，结束条件是再按to）

![image-20210626023559944](C:\Users\27096\AppData\Roaming\Typora\typora-user-images\image-20210626023559944.png)

```lisp
to setup
  clear-all
  create-turtles 100
  ask turtles 
    [ setxy random-xcor random-ycor ]
end

to go
  move-turtles  ; 使用自己定义的例程（函数）
end

to move-turtles  ; 我的理解是turtles相当于传参
  ask turtles
  [
      right random 360  ; [0 359]不包括传入的数
      forward 1  ; 前进1步
  ]
end
```

##### 更新setup(设置patches)

```lisp
to setup
  clear-all
  create-turtles 100
  setup-patches
  setup-turtles
end

to go
  move-turtles
end

to setup-patches
  ask patches [ set pcolor green ]  ; 貌似patches是不需要create的，还是后面会讲（有默认嘛）
end
  
to setup-turtles
  create-turtles 100
  ask turtles 
    [ setxy random-xcor random-ycor ]
end
  

to move-turtles
  ask turtles
  [
      right random 45
      forward 1
      ; set color random 50
  ]
end
```

<img src="C:\Users\27096\AppData\Roaming\Typora\typora-user-images\image-20210626025856487.png" alt="image-20210626025856487" style="zoom:67%;" />

变量
