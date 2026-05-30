# Manim 思维导图插件

## 简介

Manim 作为一个优秀的动画引擎,始终缺少思维导图的支持，这便是该项目的初衷。

我自己作为一个数学内容的创作者（B站博主“**究尽数学**”）和 Manim 的使用者，深知内容创作本身就是需要花费心思、精心打磨的。为了让其他创作者更好的专注于内容，特意开源思维导图插件。希望大家积极使用和优化项目。

作为第一款思维导图插件(截至项目提交,没有找到同类 Manim 插件)，开发经历了大概三个阶段。另外,追加了时序图的实现。

### 第一版：简单递归

后序遍历：将同级的兄弟节点，放在Group容器中，通过Group的arrange方法排列、对齐；然后将这些兄弟节点的父节点，放在它们的前(上)面，并居中；再将父节点与它的子节点作为一个整体，视为一个新的节点，与其他兄弟节点重复前面的操作，直至排到根节点，结束递归，完成布局。

该实现简单粗暴，但布局方法和Group中对象*高度耦合*，丧失了灵活性，严重限制了动画的实现。

### 第二版：Reingold-Tilford算法

在使用了第一个版本一段时间之后，为了**更丰富、更灵活**的动画效果，查找了关于‘树的布局算法’。最先进入视野的是Reingold和Tilford的算法，算法提出了5个布局原则，一定程度上，第一版采用的就是该算法。但是具体实现是通过节点类，该类记录节点的尺寸，当前节点相对于父节点的位置(priem)，以及以当前节点为根的子树的偏移量(mod)。然后通过两次遍历，完成树的布局。第一次后序遍历：计算节点的位置priem和mod，而且要消除子树间发遮挡；第二次前序遍历：第一次遍历的过程中，只是计算，并不真实的应用于作为节点的VMobject对像，最终的位置确定，在此完成。

基于该算法，实现了节点类，也因此实现了低耦合，丰富了动画效果。但是，我在代码实现中，为避免子树的遮挡，将子树包在一个框中作为一个整体，然后与左邻居作碰撞检测，进行分离。实现虽然简单，但不会满足紧致的布局原则。这是我留下的一个坑，也是一个心病。

### 第三版：Walker算法及其改进

为了提高Reingold-Tilford算法中子树碰撞检测的效率,Walker改进了算法,又有人改进了Walker的算法，其具体的算法,可参考论文：《Improving Walker's Algorithm to Run in Linear Time》。

这次的实现，严格参考了论文的算法：虽然在Manim动画中，限于屏幕的大小，一般思维导图节点不会太多，子树遮挡的性能提升其实影响不大；除了提高算法效率，更主要的是布局紧密；而且布局算法和作为节点内容的VMobject是完全分离的，如果对我动画部分的实现不满意，完全可以基于布局算法，开发自己的动画(就问你够不够灵活)。

## 使用方法

我开源的就是最好的第三版，用法也是关于第三版的。通过 pip 进行安装：

```
pip install manim-mindmap
```

在 Manim 项目中,导入插件：

```python
from manim_mindmap import *
```

导入的类如下：
+ MindMap:一个思维导图类
+ Node:思维导图节点类
+ NodeStyle：思维导图的布局参数类(布局方向、节点的间距、节点的样式等)
+ 动画类：LayoutAnimation
    + RemoveNode：在思维导图中移除节点
    + InsertNode：向思维导图中添加节点
    + ScaleNode：放缩思维导图中的节点
    + AlterNode：替换节点中的内容

具体的使用方法，请参考后续的代码。

### 动画类 InsertNode：插入节点

场景中已经显示一个思维导图，向其中添加一个或多个节点，并呈现添加的动画效果：

```python
self.camera.frame.set_width(25).move_to(RIGHT)
root = Node(Tex('球体积').to_edge(LEFT))

A1 = Node(Tex('公元前3世纪'))
A2 = Node(Tex('公元3世纪'))
A3 = Node(Tex('公元5世纪'))
A4 = Node(Tex('公元17世纪'))
A5 = Node(Tex('公元18世纪'))
# 首次创建
self.play(
    InsertNode(self,{root:[A1,A2,A3,A4,A5]}),
    run_time = 2
)
self.wait()

A11 = Node(Tex('阿基米德平衡法'))

A21 = Node(Tex('《九章算术》'))
A22 = Node(Tex('刘徽：牟合方盖'))

A221 = Node(Tex('球与牟合方盖的关系'))
A222 = Node(Tex('牟合方盖体积？'))

A31 = Node(Tex('祖暅：开立圆术'))
A32 = Node(Tex('祖冲之：球体积'))

A41 = Node(Tex('开普勒'))
A42 = Node(Tex('卡瓦列里原理'))

A51 = Node(Tex('松永良弼：会玉术'))

# 插入节点
for dic in [{A2:[A21,A22]},{A22:[A221,A222]},{A3:[A31,A32]},{A4:[A41,A42]}]:
    self.play(
        InsertNode(self,dic)
    )
self.play(
    InsertNode(self,{A1:[A11],A5:[A51]}),
)
```

参数说明：父节点作为字典的键，表示被插入的节点；对应的值，为节点列表，表示待插入的节点。
需要注意：node_style 作为思维导图全局外观的参数，在修改时也会呈现动画效果。比如节点外框的颜色、连接线的粗细，布局的方向等等

### 动画类 ScaleNode：放缩节点

场景中已经显示一个思维导图，放缩一个或多个节点，并呈现添加的动画效果(思维导图沿用上面示例)：

```python
# 调整节点大小 + 修改节点样式
node_style = NodeStyle(
    node_style=[
        {'color':RED,'stroke_width':20,'stroke_opacity':0.5},
        {'color':YELLOW,'stroke_width':12,'stroke_opacity':0.5},
        {'color':BLUE,'stroke_width':6,'stroke_opacity':0.5},
        {'color':GREEN,'stroke_width':3,'stroke_opacity':0.5},
    ],
    line_style=[
        {'color':RED,'stroke_width':20,'stroke_opacity':0.5},
        {'color':YELLOW,'stroke_width':12,'stroke_opacity':0.5},
        {'color':BLUE,'stroke_width':6,'stroke_opacity':0.5},
        {'color':GREEN,'stroke_width':3,'stroke_opacity':0.5},
    ]
)
self.play(
    ScaleNode(self,{A31:1.5,A32:1.5,A51:0.8},node_style = node_style),
    run_time = 2
)
```

参数说明：节点作为被放大、缩小的节点，值对应节点放缩的数值。

### 动画类 AlterNode：修改或替换节点

场景中已经显示一个思维导图，替换节点中的 VMobject ，并呈现添加的动画效果(思维导图沿用上面示例)：

```python
self.play(
    AlterNode(self,{root:Tex(r'球\\体\\积')},node_style = node_style),
)
```

参数说明：待修改的的节点作为键，替换用的VMobject作为值。

### 动画类 RemoveNode：移除节点

场景中已经显示一个思维导图，移除节点，并呈现添加的动画效果(思维导图沿用上面示例)：

```python
self.play(
    RemoveNode(self,[A11,A51],node_style = node_style),
)
self.wait()

self.play(
    RemoveNode(self,root,node_style = node_style),
)
```

参数说明：待移除的节点可以是一个，也可以是多个。注意，移除一个节点时，意味着移除以该节点为根的子树；所以，如果该节点是根节点，将移除整个思维导图。

### 动画类：LayoutAnimation

动画类RemoveNode、InsertNode、ScaleNode和AlterNode，都继承于LayoutAnimation。这四个动画，如果操作的是同一个思维导图，在一个play中，不能同时使用；如下的使用方式是错的：

```python
self.play(
    RemoveNode(self,node1),
    InsertNode(self,node2)
)
```

如果需要同时添加和移除等操作，可以使用LayoutAnimation。使用方法如下：

```python
self.camera.frame.set_width(25).move_to(RIGHT)
root = Node(Tex('球体积').to_edge(LEFT))

A1 = Node(Tex('公元前3世纪'))
A2 = Node(Tex('公元3世纪'))
A3 = Node(Tex('公元5世纪'))
A4 = Node(Tex('公元17世纪'))
A5 = Node(Tex('公元18世纪'))

for A in [A1, A2, A3, A4, A5]:
    root.add_child(A)

A11 = Node(Tex('阿基米德平衡法'))
A1.add_child(A11)

A21 = Node(Tex('《九章算术》'))
A22 = Node(Tex('刘徽：牟合方盖'))
A2.add_child(A21)
A2.add_child(A22)

# 首次创建
self.play(
    LayoutAnimation(self,root)
)

A221 = Node(Tex('球与牟合方盖的关系'))
A222 = Node(Tex('牟合方盖体积？'))
A22.add_child(A221)
A22.add_child(A222)
# 插入节点
self.play(
    LayoutAnimation(self,root)
)
self.wait()

A31 = Node(Tex('祖暅：开立圆术'))
A32 = Node(Tex('祖冲之：球体积'))
A3.add_child(A31)
A3.add_child(A32)
# 插入节点
self.play(
    LayoutAnimation(self,root)
)
self.wait()

A41 = Node(Tex('开普勒'))
A42 = Node(Tex('卡瓦列里原理'))
A4.add_child(A41)
A4.add_child(A42)

A51 = Node(Tex('松永良弼：会玉术'))
A5.add_child(A51)
# 插入节点
self.play(
    LayoutAnimation(self,root)
)
self.wait()

# 放缩节点
A31.scale(1.5)
A51.scale(0.8)
self.play(
    LayoutAnimation(self,root)
)
self.wait()

# 修改节点内容
root.alter_content(Tex(r'球\\体\\积',color = RED,font_size = 60))
self.play(
    LayoutAnimation(self,root)
)
self.wait()

# 修改节点、连线样式
node_style = NodeStyle(
    node_style=[
        {'color':RED,'stroke_width':20,'stroke_opacity':0.5},
        {'color':YELLOW,'stroke_width':12,'stroke_opacity':0.5},
        {'color':BLUE,'stroke_width':6,'stroke_opacity':0.5},
        {'color':GREEN,'stroke_width':3,'stroke_opacity':0.5},
    ],
    line_style=[
        {'color':RED,'stroke_width':20,'stroke_opacity':0.5},
        {'color':YELLOW,'stroke_width':12,'stroke_opacity':0.5},
        {'color':BLUE,'stroke_width':6,'stroke_opacity':0.5},
        {'color':GREEN,'stroke_width':3,'stroke_opacity':0.5},
    ]
)
self.play(
    LayoutAnimation(self,root,node_style = node_style)
)
self.wait()

# 修改布局
layout_config = LayoutConfig(
    direction = LEFT
)
self.play(
    LayoutAnimation(self,root,layout_config = layout_config,node_style = node_style),
    self.camera.frame.animate.shift(12*LEFT)
)
```

通过各个节点的`add_child`、`remove_child`、`scale`和`alter_content`方法，实现插入、移除、放缩和修改节点。然后，将根节点传入LayoutAnimation，会将所有的动画效果收集返回。

如果不想通过节点类 Node 组织思维导图，可以考虑 MindMap 类。

### MindMap 类

通过 MindMap 类,解析思维导图结构数据，形成节点的树结构，完成布局，其中“思维导图数据结构”如下：

```python
mindmap = {
    'node':r'球体积',
    'text':'用于TTS讲解的文本',
    'child':[
        {
            'node':r'公元前3世纪',#或者为VMobject、Mobject对象
            'child':[
                {
                    'node':r'阿基米德平衡法',
                }
            ]
        },
        {
            'node':r'公元3世纪',
            'child':[
                {
                    'node':r'《九章算术》',
                },
                {
                    'node':r'刘徽：牟合方盖',
                    'child':[
                        {
                            'node':r'球与牟合方盖的关系',
                        },
                        {
                            'node':r'牟合方盖体积？',
                        }
                    ]
                }
            ]
        },
        {
            'node':r'公元5世纪',
            'child':[
                {
                    'node':r'祖暅：开立圆术',
                }
            ]
        },
        {
            'node':r'公元17世纪',
            'child':[
                {
                    'node':r'开普勒',
                },
                {
                    'node':r'卡瓦列里原理',
                }
            ]
        },
        {
            'node':r'公元18世纪',
            'child':[
                {
                    'node':r'松永良弼：会玉术',
                }
            ]
        }
    ]
}
```

将上面结构的参数 `mindmap` 传给 MindMap 类，然后可以遍历 MindMap 实例的各个节点

```python
mind = MindMap(mindmap)
mind.scale_to_fit_width(12)
for node in mind.dfs_walker():
    if node.connector:
        self.play(
            Create(node.connector),
            run_time = 0.5
        )
    self.play(
        Create(node.surr_rect),
        Write(node.vmobject)
    )
self.wait()
```

MindMap 类还实现了子节点，后代节点和子树的方法：

```python
children = mind.get_children(ID = (0,1))
self.play(
    *[
        Wiggle(child) for child in children
    ]
)
self.wait()

descendants = mind.get_descendants(ID = (0,1))
self.play(
    *[
        Wiggle(descendant) for descendant in descendants
    ]
)
```

其中，参数 ID 是一个元组，对应一个节点：
+ 根节点是：`(0,)`
+ 根节点的第一个、第二个子节点分别是：`(0,0),(0,1)`

### TimeLine 类

时序图的实现：TimeLine 类与 MindMap 类用法相同。

在前面的 `LayoutAnimation` 和插入节点 `InsertNode`等动画中，通过 `layout_type` 参数指定为时序图的布局

```python
self.play(
    LayoutAnimation(
        self,
        root,
        layout_type = LayoutType.TimeLine,
        node_style = node_style
    )
)
```

布局参数 `LayoutConfig` 说明：
```python
# 时序图布局参数
self.play(
    LayoutAnimation(
        self,
        root,
        layout_type = LayoutType.TimeLine,
        layout_config = LayoutConfig(
            node_spacing = 0.5,
            level_spacing = 0.5,
            sides = (UP,DOWN), 
        ),
        node_style = node_style
    )
)
# 默认的思维导图布局参数
self.play(
    LayoutAnimation(
        self,
        root,
        layout_config = LayoutConfig(
            direction = RIGHT,
            node_spacing = 0.5,
            level_spacing = 0.5, 
        ),
        node_style = node_style
    )
)
```
+ sides: 指定时序图的二级子树的生长方向只能是UP或DOWN，或(UP,DOWN)和(DOWN,UP)。不指定的话，默认为(UP,DOWN)
+ direction：思维导图的布局方向，时序图的布局方向只能是RIGHT,不需指定
+ node_spacing：节点之间的间距
+ level_spacing：层级之间的间距

## 开发计划

+ 添加其他的布局算法
    + 鱼骨图