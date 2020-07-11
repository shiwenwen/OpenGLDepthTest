# （五）OpenGL深度测试

在[OpenGL正背面剔除](https://www.yuque.com/shiwenwen-qfo44/nrzz49/mflvgp)中我们使用 **正背面剔除** 解决了油画算法中存在的弊端，但是还存在着新的问题，就是当两个正面重合时，OpenGL并不知道改哪个面在前，哪个面在后，从而导致渲染错误的情况。<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/674886/1594436653906-57c29cf5-ecc0-4468-9a04-964676136a44.png#align=left&display=inline&height=729&margin=%5Bobject%20Object%5D&name=image.png&originHeight=729&originWidth=1770&size=136677&status=done&style=none&width=1770)<br />我们可以通过开启 **深度测试** 来解决这个问题。
<a name="bBQ5o"></a>
## 了解深度

- 什么是深度？
   - 像素点的深度其实就该像素点在3D世界中距离摄像机的距离，Z轴距离。
- 什么深度缓冲（缓存）区？
   - 和帧缓冲区一样，就是一块内存区域（显存中）。专门存储每个像素点（绘制在屏幕上的）的深度值，和素点一一对应，每个像素点在深度缓冲区对应存储一个深度值。
   - 如果观察者在Z轴的正方向，则Z值越大越靠近观察者。
   - 如果观察者在Z轴的负方向，则Z值越小越靠近观察者。
- 为什么要使用深度缓冲区？
   - 在不使用深度测试的时候,如果我们先绘制一个距离比较近的物体,再绘制距离较远的物体。则距离远的物体因为后绘制,会把距离近的物体覆盖掉。有了深度缓冲区后，绘制物体的顺序就不那么重要了。 实际上，只要存在深度缓冲区，OpenGL都会把像素的深度值写⼊到缓冲区中。除非调⽤ `glDepthMask(GL_FALSE)` 来禁止写⼊入。



<a name="GclR6"></a>
## 深度测试
深度缓冲区( `DepthBuffer` )和颜⾊缓存区( `ColorBuffer` )是对应的。颜色缓存区存储像素的颜⾊信息，而深度缓冲区存储像素的深度信息。在决定是否绘制⼀个物体表面时，⾸先要将表面对应的像素的深度值与当前深度缓冲区中的值进行比较。如果⼤于深度缓冲区中的值，则丢弃这部分。否则利⽤这个像素对应的深度值和颜⾊色值，分别更更新深度缓冲区和颜⾊缓存区。这个过程称为**深度测试**。<br />
* ![image.png](https://cdn.nlark.com/yuque/0/2020/png/674886/1594440492685-4cb6c5bb-1e2e-408b-a82d-04cbe87b721c.png#align=left&display=inline&height=1049&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1049&originWidth=1427&size=103691&status=done&style=shadow&width=1427)<br />
* ![image.png](https://cdn.nlark.com/yuque/0/2020/png/674886/1594440547336-d7148d71-fd40-4dcc-8d68-62a7e531bb6b.png#align=left&display=inline&height=1099&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1099&originWidth=1427&size=91305&status=done&style=shadow&width=1427)
<a name="kOg83"></a>
### 深度计算


- 深度值⼀般由16位，24位或者32位值表示，通常是24位。位数越高的话，深度的精确度越好。深度值的范围在 `[0,1]` 之间，值越⼩表示越靠近观察者，值越⼤表示远离观察者。
- 深度缓冲主要是通过计算深度值来⽐较⼤小，在深度缓冲区中包含深度值介于0.0和1.0之间， 从观察者看到其内容与场景中的所有对象的 `z` 值进⾏了⽐较。这些视图空间中的 `z` 值可以在投影平头截体的近平面和远平面之间的任何值。我们因此需要一些方法来转换这些视图空间 `z` 值到 `[0，1]` 的范围内,下面的 (线性) 方程把 `z` 值转换为 `0.0` 和 `1.0` 之间的值 :
   - ![image.png](https://cdn.nlark.com/yuque/0/2020/png/674886/1594440900156-1f5ba918-79b1-4b4b-a679-26477ecf0815.png#align=left&display=inline&height=92&margin=%5Bobject%20Object%5D&name=image.png&originHeight=366&originWidth=1167&size=110451&status=done&style=none&width=292)
   - `far` 和 `near` 是提供到投影矩阵设置可⻅见视图截锥的远近值。
<a name="CG2cz"></a>
## 在OpenGL中使用深度测试

- 在绘制场景前，清除深度缓冲区和颜色缓冲区：
```cpp
glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
glClear(GL_COLOR_BUFFER_BIT|GL_DEPTH_BUFFER_BIT);
```

- 开启深度测试：

`glEnable(GL_DEPTH_TEST);`

- 清除深度缓冲区默认值为1.0（1.0表示最⼤大的深度值，深度值的范围为(0,1)之间。值越小表示越靠近观察者,值越⼤表示越远离观察者）。



- 除了深度测试默认的比较规则，我们还可以通过 `glDepthFunc(GLEnum mode)` 来指定测试的判断方式：
| 函数 | 说明 |
| --- | --- |
| GL_ALWAYS | 总是通过测试，不管深度值多少，都认为测试通过，替换。 |
| GL_NEVER | 总是不通过测试，与GL_ALWAYS相反 |
| GL_LESS | 在当前深度值 < 存储的深度值时通过 |
| GL_EQUAL | 在当前深度值 = 存储的深度值时通过 |
| GL_LEQUAL | 在当前深度值 <= 存储的深度值时通过 |
| GL_GREATER | 在当前深度值 > 存储的深度值时通过 |
| GL_NOTEQUTAL | 在当前深度值不等于存储的深度值时通过 |
| GLGEQUAL | 在当前深度值 >= 存储的深度值时通过 |

- 我们还可以通过 `glDepthMask(GLBool value)` 开启/关闭深度缓冲区的写入；


<br />回到我们最开始的问题，当我们使用深度测试后，之前的渲染问题就不存在了。<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/674886/1594442323770-fe11cb85-16a8-4ed3-a4bd-15c3565752ff.png#align=left&display=inline&height=475&margin=%5Bobject%20Object%5D&name=image.png&originHeight=475&originWidth=688&size=36921&status=done&style=none&width=688)<br />

<a name="zsKOB"></a>
## 深度测试ZFighting闪烁问题
> 虽然我们解决了上图甜甜圈的绘制问题，但是深度测试也同样存在着问题： `ZFighting` 

开启深度测试后，OpenGL就不会再去绘制模型被遮挡的部分，这样实现的显示更更加真实。但是，由于深度缓冲区是用浮点数存储深度值的，那么就存在浮点数的精度问题，精度的限制对于深度相差⾮常小的情况下，OpenGL 就可能出现不能正确判断两者的深度值的情况。会导致深度测试的结果不可预测，显示出来的现象是交错闪烁的，前面2个画面交错出现。<br />![image.png](https://cdn.nlark.com/yuque/0/2020/png/674886/1594442699261-af6e4684-4b17-42f9-8897-b61c7d31b804.png#align=left&display=inline&height=608&margin=%5Bobject%20Object%5D&name=image.png&originHeight=608&originWidth=1276&size=92445&status=done&style=none&width=1276)
<a name="zUMz9"></a>
### ZFighting问题解决
既然是因为靠的太近，无法区分图层先后。就那么此时就可以在2个图层之间加入一个微妙的间隔，如果手动添加，既复杂也不精确。此时OpenGL提供了一个解决方案： **多边形偏移** 。

1. 启动多边形偏移 `Polygon Offset` :
   - 解决方案：让深度值之间产⽣生间隔。如果2个图形之间有间隔，就意味着就不会产生⼲涉。可以理解为在执行深度测试前将⽴⽅体的深度值做一些细微的增加，于是就能将重叠的2个图形深度值有所区分。
```cpp
/**
启用 Polygon Offset
参数：
GL_POLYGON_OFFSET_POINT 对应光栅化模式：GL_POINT
GL_POLYGON_OFFSET_LINE 对应光栅化模式：GL_LINE
GL_POLYGON_OFFSET_FILL 对应光栅化模式：GL_FILL
*/
gLEnable(GL_POLYGON_OFFSET_FILL);
```

2. 指定偏移量
   - 通过 `glPolygonOffset` 来指定 `glPolygonOffset` 需要2个参数: `factor` , `units` 。
```cpp
void glPolygonOffset(Glfloat factor, Glfloat units);
```

   - 每个Fragment 的深度值都会增加如下所示的偏移量:

`Offset = ( m * factor ) + ( r * units)` <br />`m` : 多边形的深度的斜率的最⼤值，一个多边形越是与近裁剪面平行 `m` 就越接近于0。<br />`r` : 能产生于窗⼝坐标系的深度值中可分辨的差异最⼩值。 `r` 具体是由OpenGL平台指定的 ⼀个常量。

   - 一个大于0的 `Offset` 会把模型推到离你(摄像机)更远的位置，相应的⼀个小于0的 `Offset` 会把模型拉近。
   - 一般⽽言，只需要将 `-1.0` 和 `-1` 这样简单赋值给 `glPolygonOffset` 基本可以满⾜需求。
3. 关闭 `Polygon Offset` 

使用完毕后，我们要记得关闭 `Polygon Offset` : `glDisable(GL_POLYGON_OFFSET_FILL)` 。防止对其它物体的绘制产生影响。
<a name="HrXRx"></a>
### ZFlighting问题预防
ZFlighting问题在绘制时也可以尽可能的去避免：

- 不要将两个物体靠的太近，避免渲染时三角形叠在一起。这种⽅式要求对场景中物体插⼊⼀个少量的偏移，那么就可能避免ZFighting现象。当然手动去插⼊这个⼩的偏移是要付出代价的。
- 尽可能将近裁剪面设置得离观察者远一些。在近裁剪平面附近，深度的精确度是很高的，因此尽可能让近裁剪面远一些的话，会使整个裁剪范围内的精确度变高一些。但是这种方式会使离观察者较近的物体被裁减掉，因此需要调试好裁剪面参数。
- 使⽤更⾼位数的深度缓冲区，通常使用的深度缓冲区是24位的，现在有⼀些硬件使用的是32位的缓冲区，使精确度得到提⾼。



