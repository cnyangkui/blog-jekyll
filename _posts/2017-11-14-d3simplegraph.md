---
layout: post
title: D3简单图绘制学习总结
key: 10005
tags: D3js 前端
date: 2017-11-14 11:00:00 +08:00
---

参考[D3.js入门系列](http://www.ourd3js.com/wordpress/396/)

对简单图绘制的重点内容进行了总结，下述简单图绘制步骤相似，总结如下：

> 1.使用布局，转换数据格式
>
> 2.如需绘制复杂path，需要创建对应的路径生成器
>
> 3.依次绘制各个元素，如需绘制path可能需要调用路径生成器

布局内容总结：

> D3总共提供了12个布局：饼状图（Pie）、力导向图（Force）、弦图（Chord）、树状图（Tree）、集群图（Cluster）、捆图（Bundle）
> 、打包图（Pack）、直方图（Histogram）、分区图（Partition）、堆栈图（Stack）、矩阵树图（Treemap）、层级图（Hierarchy）。
>
> 12 个布局中，层级图（Hierarchy）不能直接使用。集群图、打包图、分区图、树状图、矩阵树图是由层级图扩展来的。如此一来，能够
> 使用的布局是11个（有5个是由层级图扩展而来）。这些布局的作用都是将某种数据转换成另一种数据，而转换后的数据是利于可视化的。

本节内容将描述饼状图、力导向图、弦图、集群图、树状图、打包图、地图的绘制过程。

<!--more-->

## 饼状图

#### 1.数据格式
{% highlight javascript %}

var dataset = [ 30 , 10 , 43 , 55 , 13 ];

{% endhighlight %}

#### 2.使用pie布局,转换数据

{% highlight javascript %}

var pie = d3.layout.pie();
var piedata = pie(data);

{% endhighlight %}

#### 3.绘制

弧生成器计算路径（svg的path）

{% highlight javascript %}

var arc = d3.svg.arc()  //弧生成器
	.innerRadius(innerRadius)   //设置内半径
	.outerRadius(outerRadius);  //设置外半径

{% endhighlight %}

绘制路径path，需要调用弧生成器

{% highlight javascript %}

.attr("d", function(d){
    return arc(d);   //调用弧生成器，得到路径值
})

{% endhighlight %}

或者

{% highlight javascript %}

.attr("d", arc)

{% endhighlight %}

绘制文本text，计算路径中心位置，放入文本值

{% highlight javascript %}

.attr("transform",function(d){
    return "translate(" + arc.centroid(d) + ")";
})

{% endhighlight %}

## 力导向图

#### 1.数据格式

{% highlight javascript %}

var nodes = [ { name: "桂林" }, { name: "广州" },
	{ name: "厦门" }, { name: "杭州" },
    { name: "上海" }, { name: "青岛" },
    { name: "天津" } ];

var edges = [ { source : 0 , target: 1 } , { source : 0 , target: 2 } ,
	{ source : 0 , target: 3 } , { source : 1 , target: 4 } ,
	{ source : 1 , target: 5 } , { source : 1 , target: 6 } ];

{% endhighlight %}

#### 2.force布局，转换数据

{% highlight javascript %}

var force = d3.layout.force()
      .nodes(nodes) //指定节点数组
      .links(edges) //指定连线数组
      .size([width,height]) //指定作用域范围
      .linkDistance(150) //指定连线长度
      .charge([-400]); //相互之间的作用

{% endhighlight %}

然后，使力学作用生效：

{% highlight javascript %}

force.start();    //开始作用

{% endhighlight %}

#### 3.绘制

* line,连线
* circle,节点
* text,文字

绘制节点时，使节点拖动

{% highlight javascript %}

.call(force.drag);  //使得节点能够拖动

{% endhighlight %}

力导向图布局force有一个事件tick，每进行到一个时刻，都要调用它，更新的内容就写在它的监听器里就好。

{% highlight javascript %}

force.on("tick", function(){ //对于每一个时间间隔
    //更新连线坐标
    svg_edges.attr("x1",function(d){ return d.source.x; })
        .attr("y1",function(d){ return d.source.y; })
        .attr("x2",function(d){ return d.target.x; })
        .attr("y2",function(d){ return d.target.y; });

    //更新节点坐标
    svg_nodes.attr("cx",function(d){ return d.x; })
        .attr("cy",function(d){ return d.y; });

    //更新文字坐标
    svg_texts.attr("x", function(d){ return d.x; })
       .attr("y", function(d){ return d.y; });
 });

{% endhighlight %}

## 弦图

#### 1.数据格式

{% highlight javascript %}

var city_name = [ "北京" , "上海" , "广州" , "深圳" , "香港"  ];
    
var population = [
	[ 1000,  3045, 4567, 1234, 3714 ],
    [ 3214,  2000, 2060, 124, 3234 ],
    [ 8761,  6545, 3000, 8045, 647 ],
    [ 3211,  1067, 3214, 4000, 1006 ],
    [ 2146,  1034, 6745, 4764, 5000 ]
];

{% endhighlight %}

#### 2.chord布局

{% highlight javascript %}

var chord_layout = d3.layout.chord()
	.padding(0.03) //节点之间的间隔
  	.sortSubgroups(d3.descending) //排序
  	.matrix(population); //输入矩阵

{% endhighlight %}

数据转换

{% highlight javascript %}

var groups = chord_layout.groups();
var chords = chord_layout.chords();

{% endhighlight %}

#### 3.绘制

首先绘制外部圆环，使用arc弧生成器

{% highlight javascript %}

var outer_arc = d3.svg.arc()
	.innerRadius(innerRadius)
	.outerRadius(outerRadius);

{% endhighlight %}

绘制path，使用数据为groups

{% highlight javascript %}

.attr("d", outer_arc)	//调用arc路径生成器

{% endhighlight %}

绘制text,使用数据为groups。首先旋转适当角度，然后向上移动外半径个长度，135-225度范围的文字倒置

{% highlight javascript %}

.each( function(d,i) { 
   d.angle = (d.startAngle + d.endAngle) / 2; 
   d.name = city_name[i];
})
.attr("transform", function(d) {
   return "rotate(" + ( d.angle * 180 / Math.PI ) + ")" +
   	"translate(0,"+ -1.0*(outerRadius+10) +")" +
   	(( d.angle > Math.PI*3/4 && d.angle < Math.PI*5/4 ) ? "rotate(180)" : "");
})

{% endhighlight %}

然后，绘制弦，使用chord生成器生成路径

{% highlight javascript %}

var inner_chord = d3.svg.chord()
	.radius(innerRadius);

{% endhighlight %}

绘制path,使用数据为chords

{% highlight javascript %}

.attr("d", inner_chord)	//调用chord路径生成器

{% endhighlight %}

## 集群图

#### 1.数据格式

{% highlight javascript %}

{
	"name":"中国",
	"children":
	[
	    { 
	      "name":"浙江" , 
	      "children":
	      [
	            {"name":"杭州" },
	            {"name":"宁波" },
	            {"name":"温州" },
	            {"name":"绍兴" }
	      ] 
	    },
	    { 
	        "name":"广西" , 
	        "children":
	        [
	            {"name":"桂林"},
	            {"name":"南宁"},
	            {"name":"柳州"},
	            {"name":"防城港"}
	        ] 
	    },
	    ......
	]
}

{% endhighlight %}

#### 2.定义集群图布局，转换数据

{% highlight javascript %}

var cluster = d3.layout.cluster()
	.size([width, height - 200]);	//设定尺寸

{% endhighlight %}

使用数据

{% highlight javascript %}

d3.json("city.json", function(error, root) {
  var nodes = cluster.nodes(root);
  var links = cluster.links(nodes);
}

{% endhighlight %}

#### 3.绘制

首先，创建对角线生成器

{% highlight javascript %}

var diagonal = d3.svg.diagonal()
	.projection(function(d) { return [d.y, d.x]; });

{% endhighlight %}

绘制连线，使用生成器

{% highlight javascript %}

.attr("d", diagonal)   //使用对角线生成器

{% endhighlight %}

然后绘制节点circle


## 树状图

和集群图写法基本相同，使用布局不同。

{% highlight javascript %}

var tree = d3.layout.tree()
	.size([width, height-200])
	.separation(function(a, b) { return (a.parent == b.parent ? 1 : 2); });

{% endhighlight %}

## 打包图

数据格式与树状图相同，布局如下：

{% highlight javascript %}

var pack = d3.layout.pack()
	.size([ width, height ])
	.radius(20);

{% endhighlight %}

数据转换写法也类似。

然后分别绘制circle和text。


## 地图

#### 1.投影，转换经纬度数据，使能够在二维平面上显示

{% highlight javascript %}

var projection = d3.geo.mercator()	//投影函数
	.center([107, 31])	//地图中心位置，经纬度
	.scale(850)	//缩放比例
    .translate([width/2, height/2]);

{% endhighlight %}

#### 2.创建地图路径生成器

{% highlight javascript %}

var path = d3.geo.path()
	.projection(projection);

{% endhighlight %}

#### 3.绘制

{% highlight javascript %}

.data(root.features)	//数据绑定
...
.attr("d", path)	//调用路径生成器

{% endhighlight %}