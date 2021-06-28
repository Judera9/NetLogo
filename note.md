# 学习Netlogo和pyNetlogo的笔记

## 6.22

学习方案：

1. 同时进行两边，pyNetLogo的代码解读和NetLogo的官网教程（Lisp语法）
2. DDL：本周六前完成官网代码的中文教程，pyNetLogo不做要求（可以同时熟悉一下语法）



## 6.26

### NetLogo中文教程

#### 学习例程 - 吃草

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



##### 添加变量energy

```lisp
turtles-own [energy]  ; 变量名为energy

to go
  move-turtles
  eat-grass
end

to eat-grass
  ask turtles
  [
    if pcolor = green  ; 这里的pcolor是turtle所处的patch的颜色
    [
      set pcolor black
      set energy (energy + 10)
    ]
  ]
end
```

<img src="D:\AppData\Learning\Coding\NetlogoLearning\note.assets\image-20210626154523506.png" alt="image-20210626154523506" style="zoom:67%;" />



##### 创建监视器

<img src="D:\AppData\Learning\Coding\NetlogoLearning\note.assets\image-20210626154831696.png" alt="image-20210626154831696" style="zoom:67%;" />

turtles是一个agentset，即所有海龟的集合；count操作告诉我们这个集合中有多少主体（这个变量真的是哪都能用啊，就当成一个全局变量好了）

<img src="D:\AppData\Learning\Coding\NetlogoLearning\note.assets\image-20210626155319289.png" alt="image-20210626155319289" style="zoom:67%;" />

这个监视器用到with语法，就是加了一个限制条件（一个小的agentset）



##### 开关的用法以及energy显示

<img src="D:\AppData\Learning\Coding\NetlogoLearning\note.assets\image-20210626155625524.png" alt="image-20210626155625524" style="zoom:67%;" />

不要漏掉'?'

```lisp
to eat-grass  ; 重写eat-grass方法
  ask turtles
  [
    if pcolor = green
    [
      set pcolor black
      set energy (energy + 10)
    ]
    ifelse show-energy?  ; 每个海龟eat之后，检查show-energy的值
    [ set label energy ]  ; 如果switch打开，执行上面的命令
    [ set label "" ]  ; 如果switch关闭，放置一个“空串”，即无string
  ]
end
```



##### 添加更多行为

还要注意到一点，每一个函数并不是对单个turtle的操作，而是群体的行为，即我们使用的是 ask turtles [ order ]，让群体都去执行某个行为

```lisp
to go
  move-turtles
  eat-grass
  ;; 从to go这里可以看到，netlogo一般将一种行为抽象成一种函数，然后在go执行后，由于go外层其实相当于有一个循环，所以整个系统其实是由一个行为执行顺序的（需要符合逻辑）；这里是：移动-吃草-繁殖-死亡-重新长草
  reproduce
  check-death
  regrow-grass
end
```

添加的例程如下：

```lisp
to reproduce
  ask turtles
  [
    if energy > 50
    [
      set energy energy - 50
      hatch 1  ; hatch是netLogo的一个原语（孵化）
      [ set energy 50 ]  ; 形如：hatch number [ commands ]
    ]
  ]
end

to check-death
  ask turtles
  [ 
    if energy <= 0 [ die ]  ; die也是NetLogo的一个原语
  ]
end

to regrow-grass
  ask patches
  [
    if random 100 < 3  ; 即grass重新长出来的概率为3/100
    [ set pcolor green ]
  ]
end
```



##### Plotting

先实现Procedures部分：

```lisp
to do-plots
  set-current-plot "Totals"  ; 指定当前操作的plot组件
  set-current-plot-pen "turtles"  ; 指定使用哪支笔
  ;; plot命令会在图像上增加新点，横坐标加一，纵坐标为plot命令中给定的值
  plot count turtles
  set-current-plot-pen "grass"
  plot count patches with [ pcolor = green ]
end
```

**注意**：这里的代码是以前旧版本的NetLogo，新版本5以后都是直接用Reset-ticks和tick来画图的，然后组件的设置中设置画笔：

<img src="D:\AppData\Learning\Coding\NetlogoLearning\note.assets\image-20210626175417999.png" alt="image-20210626175417999" style="zoom:67%;" />

```lisp
to setup
  ...
  reset-ticks
end

to go
  ...
  tick
end
```

https://zhuanlan.zhihu.com/p/104964353



##### 使用Tick counter

tick指令将时钟计数器加1

clear-all会将时钟计数器清零

ticks是一个报告器，返回时钟计数器当前值

***将view updates更换为使用"on ticks"***：观感更好，等时间间隔查看模型

<img src="D:\AppData\Learning\Coding\NetlogoLearning\note.assets\image-20210626175739278.png" alt="image-20210626175739278" style="zoom:67%;" />

```lisp
to go
  if ticks >= 500 [ stop ]  ; 使用stop指令，控制go停止运行
  move-turtles
  eat-grass
  reproduce
  check-death
  regrow-grass
  tick
end
```



##### 添加Slider

相当于多了一个全局变量

### 界面指南

#### 菜单栏

##### File

<img src="D:\AppData\Learning\Coding\NetlogoLearning\note.assets\image-20210626182113498.png" alt="image-20210626182113498" style="zoom:67%;" />

Save As NetLogo Web：实用，可以让别人在网页端运行

Export World：保存当前所有主体和随机状态信息（csv文件）

Export Plot：保存某个plot中的数据（也是csv文件），貌似没得import plot

Import Patch Colors：将一个图像加载到瓦片

Import Patch Colors RGB：使用RBG颜色加载瓦片

Import Drawing：将一个图像加载到画图层



##### Edit

ctrl+; 注释/取消注释

ctrl+E 跳到定义

ctrl+U 显示Usage

<img src="D:\AppData\Learning\Coding\NetlogoLearning\note.assets\image-20210626183447280.png" alt="image-20210626183447280" style="zoom:67%;" />



##### Tools

<img src="D:\AppData\Learning\Coding\NetlogoLearning\note.assets\image-20210626183620185.png" alt="image-20210626183620185" style="zoom:67%;" />

Halt：强行停止（可能不符合预期）

一堆Monitors

Turtle Shapes Editor：画海龟图形

#### 界面页

添加的组件还挺多，可以试试看

其他的基本都熟悉了



##### 关于3D视图

可以用左下角的视角工具来调整

有些图形有3D图形的映射，有些没有（但是侧视图倒也不是一条线）

![image-20210626184922200](D:\AppData\Learning\Coding\NetlogoLearning\note.assets\image-20210626184922200.png)



##### 信息页Info

用来介绍模型、解释如何使用、说明要探索的事情、可能的扩展等等



##### Includes菜单

如果在Code界面加上"__includes"关键词，会出现includes菜单（.nlogo和.nls文件）



### 编程指南

#### Agents

瓦片：总数由min-pxcor, max-pxcor...四个坐标决定（边界）

海龟：也有坐标，但是可以是小数

链：两端是海龟，两个端点之间的最短路径

世界：可以改变，默认为一个环面（torus）



#### Procedures

命令command：主体执行的行动，如create、die、jump、inspect、clear（动词）

报告器reporter：计算并返回结果（名词）

原语primitive：NetLogo内置的命令

***命令和报告器的区别是，reporter只用于返回一个计算结果，并不用于被执行***

```lisp
;;; 带输入的例程
to draw-polygon [num-sides len] ; 输入参数定义
  pen-down
  repeat num-sides
    [fd len
	 rt 360 / num-sides]
end
;;; 调用
ask turtles [draw-polygon 8 who]  ; 这个who就是turtle的who number
```



#### Varaiables

全局变量：globals [score]

海龟变量：turtles-own [energy speed]

瓦片变量：patches-own [friction]

链变量：links-own [strength]

***变量的读取权限***

海龟变量可以读取所处位置的瓦片变量

```lisp
ask turtles [set pcolor red]
```

其他时候一个主体需要使用其他主体的变量时，使用of（没有修改权限，可以show）

```lisp
show [color] of turtle 5  ; 打印海龟当前颜色
show [xcor + ycor] of turtle 5
```

***设置局部变量local variables***

```lisp
to swap-colors [turtle1 turtle2]  ; 交换颜色
  let temp [color] of turtle1
  ask turtle1
  [set color [color] of turtle2]
  ask turtle2
  [set color temp]
end
```



#### Color

- NetLogo的颜色设计有点意思，一共14个色调，可以通过改变亮度和改变饱和度获取不同颜色
- 如果传入的数字不在0-140之间，NetLogo将会增加或者减去140直到在区间内（wrap-color）
- 只有小数点第一位有意义



![Screenshot](D:\AppData\Learning\Coding\NetlogoLearning\note.assets\colors.jpg)

<img src="D:\AppData\Learning\Coding\NetlogoLearning\note.assets\image-20210626230127441.png" alt="image-20210626230127441" style="zoom:67%;" />

> - Some of the colors have names. (You can use these names in your code.)
> - Every named color except black and white has a number ending in 5.
> - On either side of each named color are darker and lighter shades of the color.
> - 0 is pure black. 9.9 is pure white.
> - 10, 20, and so on are all so dark they are very nearly black.
> - 19.9, 29.9 and so on are all so light they are very nearly white.

***相关原语***

wrap-color：区间转换

scale-color：数值转换为颜色

shade-of?：是否处于同一色调



#### RGB Color

```lisp
set pcolor [255 0 0]
```

approximate-hsb和approximate-rgb可以实现两种颜色表示方式的映射

***运行Code Example***后可以发现，调低saturation其实就是当前hsb数值增大（变亮），调低brightness就是当前hsb数值减小（变暗）

<img src="D:\AppData\Learning\Coding\NetlogoLearning\note.assets\image-20210626230850118.png" alt="image-20210626230850118" style="zoom:67%;" />

```lisp
;;; HSB又称HSV，表示一种颜色模式：在HSB模式中，H(hues)表示色相，S(saturation)表示饱和度，B（brightness）表示亮度HSB模式对应的媒介是人眼。
;;; HSB模式中S和B呈现的数值越高，饱和度明度越高，页面色彩强烈艳丽，对视觉刺激是迅速的，醒目的效果，但不益于长时间的观看。

globals [
  hsb-as-rgb
  hsb-color
  rgb-color
]

to go
  set hsb-as-rgb hsb hue saturation brightness
  set hsb-color approximate-hsb hue saturation brightness
  ask quadrant -1  1 [ set pcolor hsb-as-rgb ]
  ask quadrant  1  1 [ set pcolor hsb-color ]
  set rgb-color approximate-rgb rgb-red rgb-green rgb-blue
  ask quadrant -1 -1 [ set pcolor (list rgb-red rgb-green rgb-blue) ]
  ask quadrant  1 -1 [ set pcolor rgb-color ]
  display
end

to-report quadrant [x y]  ;; inputs are 1 or -1
  report patches with [patch-quadrant = list x y]
end

to-report patch-quadrant  ;; patch procedure
  report list ifelse-value pxcor < world-width / 2 [-1] [1]
              ifelse-value pycor < world-width / 2 [-1] [1]
end


; Public Domain:
; To the extent possible under law, Uri Wilensky has waived all
; copyright and related or neighboring rights to this model.
```



#### Ask

可以向agent发出命令（不能ask观察者）

海龟和瓦片的对象获取方式已经明确，link原语通过使用两端海龟的who number获取。

***patch at原语***：相对offset

```lisp
ask turtle 0
  [ask patch-at 1 0
    [set pcolor red]
  ]
```



#### Agentsets

- 主题集合只能同时包含一种类型的主体
- 集合内部的元素没有任何特定的顺序，总是随机排列
- 报告器reporter可以以agentsets作为输入

```lisp
other turtles  ; all other turtles
other turtles-here  ; all other turtles on this patch
turtles with [color = red]  ; all red turtles
turtles-here with [color = red]
patches with [pcolor > 0]  ; right side of view
turtles in-radius 3  ; all turtles less than 3 patches away
patches at-points [[1 0] [0 1] [-1 0] [0 -1]]  ; four directions neighbour
neighbors4  ; shortstand for neighbor patches
turtles with 
  [(xcor > 0) and (ycor > 0)
   and (pcolor = green)
  ]   ; first quadrant and green
turtles-on neighbors4  ; turtles neighbors
[my-links] of turtle 0  ; all links connected to turtle 0
```

主体集合的命令

```lisp
ask
any?  ; 是否为空
all?
ask one-of tuetles [set color green]
ask one-of patches [sprout 1]  ; 随机一个瓦片生出一个海龟
ask max-one-of turtles [sum assets] [die]  ; 杀死assets最大的turtle
...
; 可以查看词典和模型库
```



#### Breeds

允许定义不同种类（breeds）的海龟或者链，即不同turtle对象或者不同的行为

```lisp
breed [wolves wolf]
breed [sheep a-sheep]  ; 后定义的显示在上层
```



...内容很多，以后再来探索吧



## 6.28

### 模型研究

#### Giant Component

```lisp
turtles-own
[
  ;; this is used to mark turtles we have already visited
  explored?
]

globals
[
  component-size          ;; number of turtles explored so far in the current component
  giant-component-size    ;; number of turtles in the giant component
  
  giant-start-node        ;; node from where we started exploring the giant component
]

;;;;;;;;;;;;;;;;;;;;;;;;
;;; Setup Procedures ;;;
;;;;;;;;;;;;;;;;;;;;;;;;

to setup
  clear-all
  set-default-shape turtles "person"
  make-turtles
  ;; at this stage, all the components will be of size 1,
  ;; since there are no edges yet
  find-all-components
  color-giant-component
  reset-ticks
end

to make-turtles
  create-turtles num-nodes [ set size 3 ]
  layout-circle turtles max-pxcor - 1
end

;;;;;;;;;;;;;;;;;;;;;;
;;; Main Procedure ;;;
;;;;;;;;;;;;;;;;;;;;;;

to go
  ;; stop if the below condition is true, as then we have a fully connected network (every two nodes are connected)
  if ( (2 * count links ) >= ( (count turtles) * (count turtles - 1) ) ) [
    display
    user-message "Network is fully connected. No more edges can be added."
    stop
  ]
  add-edge
  find-all-components
  color-giant-component
  ask links [ set color [color] of end1 ]  ;; recolor all edges
  ;; layout the turtles with a spring layout, but stop laying out when all nodes are in the giant component
  if not all? turtles [ color = red ] [ layout ]
  tick
end

;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;; Network Exploration ;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;

;; to find all the connected components in the network, their sizes and starting turtles
to find-all-components
  ask turtles [ set explored? false ]
  ;; keep exploring till all turtles get explored
  loop
  [
    ;; pick a node that has not yet been explored
    let start one-of turtles with [ not explored? ]
    if start = nobody [ stop ]
    ;; reset the number of turtles found to 0
    ;; this variable is updated each time we explore an
    ;; unexplored node.
    set component-size 0
    ;; at this stage, we recolor everything to light gray
    ask start [ explore (gray + 2) ]
    ;; the explore procedure updates the component-size variable.
    ;; so check, have we found a new giant component?
    if component-size > giant-component-size
    [
      set giant-component-size component-size
      set giant-start-node start
    ]
  ]
end

;; Finds all turtles reachable from this node (and recolors them)
to explore [new-color]  ;; node procedure
  if explored? [ stop ]
  set explored? true
  set component-size component-size + 1
  ;; color the node
  set color new-color
  ask link-neighbors [ explore new-color ]  ; 凡是linked，都涂成相同的颜色
end

;; color the giant component red
to color-giant-component
  ask turtles [ set explored? false ]
  ask giant-start-node [ explore red ]
end

;;;;;;;;;;;;;;;;;;;;;;;
;;; Edge Operations ;;;
;;;;;;;;;;;;;;;;;;;;;;;

;; pick a random missing edge and create it
to add-edge
  let node1 one-of turtles
  let node2 one-of turtles
  ask node1 [
    ifelse link-neighbor? node2 or node1 = node2
    ;; if there's already an edge there, then go back
    ;; and pick new turtles
    [ add-edge ]
    ;; else, go ahead and make it
    [ create-link-with node2 ]
  ]
end

;;;;;;;;;;;;;;
;;; Layout ;;;
;;;;;;;;;;;;;;
to layout
  if not layout? [ stop ]
  ;; the number 10 here is arbitrary; more repetitions slows down the
  ;; model, but too few gives poor layouts
  repeat 10 [
    do-layout
    display  ;; so we get smooth animation
  ]
end

to do-layout
  layout-spring (turtles with [any? link-neighbors]) links 0.4 6 1
end


; Copyright 2005 Uri Wilensky.
; See Info tab for full copyright and license.
```

**想法记录：**

* debug方式，show语句打印在command center
* **set-defult-shape turtles "person"**
* 原语相当多，这是不太友好的一点，不过有的东西可以不用原语实现（麻烦一点），所以多写多看吧
* **loop [ orders ]**，使用stop可以跳出循环

> This agent exits immediately from the enclosing procedure, ask, or ask-like construct (e.g. crt, hatch, sprout). **Only the enclosing procedure or construct stops, not all execution for the agent.**

```lisp
loop  ; 常用结构
[
  if situation [stop]
]
```

* 深度优先搜索

```lisp
;; 这个方法用在了三个地方（每一轮搜索之前explore?都是false）
;; 首先，把所有的turtles都变成gray + 2的颜色
;; 然后，explore本身会调用，进行深度优先的递归搜索（需要遍历全部，所以广度也一样，深度代码好写）
;; 最后，在color-giant-component将最大的component涂成红色
;; Finds all turtles reachable from this node (and recolors them)
to explore [new-color]  ;; node procedure
  if explored? [ stop ]
  set explored? true
  set component-size component-size + 1
  ;; color the node
  set color new-color
  ; show link-neighbors
  ask link-neighbors [ explore new-color ]
end
```

* links，之前的教程没有提到太多，以下是Dcitionary的一个例子

```lisp
crt 2
ask turtle 0
[ create-link-to turtle 1 ]
ask links
[ show end1 ] ;; shows turtle 0
```

```lisp
;; pick a random missing edge and create it
to add-edge
  let node1 one-of turtles
  let node2 one-of turtles
  ask node1 [
    ifelse link-neighbor? node2 or node1 = node2
    ;; if there's already an edge there, then go back
    ;; and pick new turtles
    [ add-edge ]
    ;; else, go ahead and make it
    [ create-link-with node2 ]
  ]
end
```

* layout是摆放方式，可以调整turtles和links的摆放方式（有两种agent的layout）

#### HIV

```lisp
globals [
  infection-chance  ;; The chance out of 100 that an infected person will pass on
                    ;;   infection during one week of couplehood.
  symptoms-show     ;; How long a person will be infected before symptoms occur
                    ;;   which may cause the person to get tested.
  slider-check-1    ;; Temporary variables for slider values, so that if sliders
  slider-check-2    ;;   are changed on the fly, the model will notice and
  slider-check-3    ;;   change people's tendencies appropriately.
  slider-check-4
]

turtles-own [
  infected?          ;; If true, the person is infected.  It may be known or unknown.
  known?             ;; If true, the infection is known (and infected? must also be true).
  infection-length   ;; How long the person has been infected.
  coupled?           ;; If true, the person is in a sexually active couple.
  couple-length      ;; How long the person has been in a couple.
  ;; the next four values are controlled by sliders
  commitment         ;; How long the person will stay in a couple-relationship.
  coupling-tendency  ;; How likely the person is to join a couple.
  condom-use         ;; The percent chance a person uses protection.
  test-frequency     ;; Number of times a person will get tested per year.
  partner            ;; The person that is our current partner in a couple.
]

;;;
;;; SETUP PROCEDURES
;;;

to setup
  clear-all
  setup-globals
  setup-people
  reset-ticks
end

to setup-globals
  set infection-chance 50    ;; if you have unprotected sex with an infected partner,
                             ;; you have a 50% chance of being infected
  set symptoms-show 200.0    ;; symptoms show up 200 weeks after infection
  set slider-check-1 average-commitment
  set slider-check-2 average-coupling-tendency
  set slider-check-3 average-condom-use
  set slider-check-4 average-test-frequency
end

;; Create carrying-capacity number of people half are righty and half are lefty
;;   and some are sick.  Also assigns colors to people with the ASSIGN-COLORS routine.

to setup-people
  create-turtles initial-people
    [ setxy random-xcor random-ycor
      set known? false
      set coupled? false
      set partner nobody
      ifelse random 2 = 0
        [ set shape "person righty" ]
        [ set shape "person lefty" ]
      ;; 2.5% of the people start out infected, but they don't know it
      set infected? (who < initial-people * 0.025)
      if infected?
        [ set infection-length random-float symptoms-show ]
      assign-commitment
      assign-coupling-tendency
      assign-condom-use
      assign-test-frequency
      assign-color ]
end

;; Different people are displayed in 3 different colors depending on health
;; green is not infected
;; blue is infected but doesn't know it
;; red is infected and knows it

to assign-color  ;; turtle procedure
  ifelse not infected?
    [ set color green ]
    [ ifelse known?
      [ set color red ]
      [ set color blue ] ]
end

;; The following four procedures assign core turtle variables.  They use
;; the helper procedure RANDOM-NEAR so that the turtle variables have an
;; approximately "normal" distribution around the average values set by
;; the sliders.

to assign-commitment  ;; turtle procedure
  set commitment random-near average-commitment
end

to assign-coupling-tendency  ;; turtle procedure
  set coupling-tendency random-near average-coupling-tendency
end

to assign-condom-use  ;; turtle procedure
  set condom-use random-near average-condom-use
end

to assign-test-frequency  ;; turtle procedure
  set test-frequency random-near average-test-frequency
end

to-report random-near [center]  ;; turtle procedure
  let result 0
  repeat 40
    [ set result (result + random-float center) ]
  report result / 20
end

;;;
;;; GO PROCEDURES
;;;

to go
  if all? turtles [known?]
    [ stop ]
  check-sliders
  ask turtles
    [ if infected?
        [ set infection-length infection-length + 1 ]
      if coupled?
        [ set couple-length couple-length + 1 ] ]
  ask turtles
    [ if not coupled?
        [ move ] ]
  ;; Righties are always the ones to initiate mating.  This is purely
  ;; arbitrary choice which makes the coding easier.
  ask turtles
    [ if not coupled? and shape = "person righty" and (random-float 10.0 < coupling-tendency)
        [ couple ] ]
  ask turtles [ uncouple ]
  ask turtles [ infect ]
  ask turtles [ test ]
  ask turtles [ assign-color ]
  tick
end

;; Each tick a check is made to see if sliders have been changed.
;; If one has been, the corresponding turtle variable is adjusted

to check-sliders
  if (slider-check-1 != average-commitment)
    [ ask turtles [ assign-commitment ]
      set slider-check-1 average-commitment ]
  if (slider-check-2 != average-coupling-tendency)
    [ ask turtles [ assign-coupling-tendency ]
      set slider-check-2 average-coupling-tendency ]
  if (slider-check-3 != average-condom-use)
    [ ask turtles [ assign-condom-use ]
      set slider-check-3 average-condom-use ]
  if (slider-check-4 != average-test-frequency )
    [ ask turtles [ assign-test-frequency ]
      set slider-check-4 average-test-frequency ]
end

;; People move about at random.

to move  ;; turtle procedure
  rt random-float 360
  fd 1
end

;; People have a chance to couple depending on their tendency to have sex and
;; if they meet.  To better show that coupling has occurred, the patches below
;; the couple turn gray.

to couple  ;; turtle procedure -- righties only!
  let potential-partner one-of (turtles-at -1 0)
                          with [not coupled? and shape = "person lefty"]
  if potential-partner != nobody
    [ if random-float 10.0 < [coupling-tendency] of potential-partner
      [ set partner potential-partner
        set coupled? true
        ask partner [ set coupled? true ]
        ask partner [ set partner myself ]
        move-to patch-here ;; move to center of patch
        ask potential-partner [move-to patch-here] ;; partner moves to center of patch
        set pcolor gray - 3
        ask (patch-at -1 0) [ set pcolor gray - 3 ] ] ]
end

;; If two peoples are together for longer than either person's commitment variable
;; allows, the couple breaks up.

to uncouple  ;; turtle procedure
  if coupled? and (shape = "person righty")
    [ if (couple-length > commitment) or
         ([couple-length] of partner) > ([commitment] of partner)
        [ set coupled? false
          set couple-length 0
          ask partner [ set couple-length 0 ]
          set pcolor black
          ask (patch-at -1 0) [ set pcolor black ]
          ask partner [ set partner nobody ]
          ask partner [ set coupled? false ]
          set partner nobody ] ]
end

;; Infection can occur if either person is infected, but the infection is unknown.
;; This model assumes that people with known infections will continue to couple,
;; but will automatically practice safe sex, regardless of their condom-use tendency.
;; Note also that for condom use to occur, both people must want to use one.  If
;; either person chooses not to use a condom, infection is possible.  Changing the
;; primitive to AND in the third line will make it such that if either person
;; wants to use a condom, infection will not occur.

to infect  ;; turtle procedure
  if coupled? and infected? and not known?
    [ if random-float 10 > condom-use or
         random-float 10 > ([condom-use] of partner)
        [ if random-float 100 < infection-chance
            [ ask partner [ set infected? true ] ] ] ]
end

;; People have a tendency to check out their health status based on a slider value.
;; This tendency is checked against a random number in this procedure. However, after being infected for
;; some amount of time called SYMPTOMS-SHOW, there is a 5% chance that the person will
;; become ill and go to a doctor and be tested even without the tendency to check.

to test  ;; turtle procedure
  if random-float 52 < test-frequency
    [ if infected?
        [ set known? true ] ]
  if infection-length > symptoms-show
    [ if random-float 100 < 5
        [ set known? true ] ]
end

;;;
;;; MONITOR PROCEDURES
;;;

to-report %infected
  ifelse any? turtles
    [ report (count turtles with [infected?] / count turtles) * 100 ]
    [ report 0 ]
end


; Copyright 1997 Uri Wilensky.
; See Info tab for full copyright and license.
```















