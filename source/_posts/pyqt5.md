---
title: pyqt5
date: 2020-10-23 10:29:10
categories: 
	- 编程
tags: 
	- PYTHON
	- 编程
---

# 1. Pyqt5 概念

## 1.1 入门+环境配置

简单入门：[pyqt5 入门教程](https://www.jianshu.com/p/ef71566ff8bb)，[教程2](https://zhuanlan.zhihu.com/p/28012981)，[gitbook中文教程](https://maicss.gitbooks.io/pyqt5/content/hello_world.html)

Python 入门：[python知识](https://tulingxueyuan.gitbooks.io/python/content/)， [python基础知识](https://bop.mol.uno/12.data_structures.html)

> pip3 install pyqt5

## 1.2 pyqt5 概念

GUI 编程提供的窗口类有两个，QWeight 和 QMainWindow，分别表示小组件和主窗口，其中主窗口包含了一些组件比如目录和状态栏。

组件的叠加：在组件初始化时，可以传个父组件的参数 parent 表示该控件放在什么地方。

布局思路：其提供了三个常用的布局设计器，QHBoxLayout、QVBoxLayout、QGridLayout，水平布局、垂直布局、网格布局。这些布局思路是根据设定的数字来控制间距或位置。

> Qhbox.addWeight( btn, 1)   # btn 占1/n 高度
>
> QGrid.addWeight( btn, 0, 0, 1, 2 ) # btn 从 (0,0) 开始占1行2列

这些布局的特点是动态化，可以根据窗口大小进行自动调节。

建议 大的网格布局中放入细小的水平或垂直布局。

还有一个重要概念就是信号与插槽，一个控件完成某个操作后发送信号给另一个控件的插槽方法。

## 1.3 关于布局问题

常见的四种布局各有不同，QHBoxLayout 让元素按水平排列，QVBoxLayout 让元素按垂直排列，QGridLayout 让元素按网络排列，QFrameLayout 让元素按表格排列。

```python
self.img_info = QFrameLayout()
self.img_info.addRow(CONST_VAL['MEAN_TXT'], self.img_mean_lbl)
self.img_info.addRow(CONST_VAL['CLEAR_TXT'], QLineEdit("00000"))
self.img_info.addRow(CONST_VAL['BALANCE_TXT'], QLineEdit("00000"))
```

使用表格布局可以让元素按照固定的行距和样式排列，不用设置参数，很适合用于数据展示和登录或个人信息等编辑。

当使用 网格布局时，需要在一个方格中放入其他布局，如果直接放入是不行的，这里用到一个强大的组件来转换 QLabel。

QLabel 不仅可以显示文字，还可以显示图片，当作容器放入其他控件和新布局。

```python
self.right_lbl = QLabel()
self.img_info_hbox = QFormLayout(self.right_lbl)
self.grid.addWidget(self.right_lbl, 1, 3, 3, 1)
```

通过声明构造方法中形参设置父控件，来把 form 布局到 label 上。

布局最麻烦的是间距的设定，只要是整齐排放的控件都很好看，就怕间距不均称。难点在布局控件间距自适应。需要进一步研究容器，考虑把控件放在容器中。

## 1.4 关于信号和插槽

信号表示发送方和接收方的一个约定，一般用ed结尾表示发生了的事件。插槽就是当信号来到时放在什么地方响应，比如一个方法等。

一般我们是接收方，qt 是发送方给我们提供了控件中很多信号，如

> self.img_btn = QPushButton()
>
> self.img_btn.clicked.connect(self.img_open_method_slot)

这里是接收方的代码，我们通过鼠标点击信号 clicked 链接到一个插槽，就是一个方法，用于打开图片。

那么，我们如果作为发送方，如何自定义信号呢？其实很简单。

```python
class MyWeight(QWeights):
  mouseReleased = pyqtSignal(list)
  def mouseReleaseEvent(self, event):
    xxx
    self.mouseReleased.emit([1,2,3])
```

这样就定义了一个信号  mouseReleased，并在鼠标抬起后，信号发送出去。这里信号带了一个 list 参数，需要在发送时填入参数，并在接收方的插槽方法中设定形参来接受。

## 1.5 Qt 与 opencv 相互转换

pyqt 中提供了两种图像格式一个是 qimage，一个是 qpixmap。

与 opencv 转换比较麻烦需要控制矩阵格式和图片的层次等，可以通过 pillow 的 PIL 来完成图片的转换。

qpixmap 转为 opencv：

```python
img_pil = ImageQt.fromqpixmap(self.pixmap())  # qpixmap to image
img_cv = cvtColor(asarray(img_pil), COLOR_RGB2BGR) # image to cv2
```

Image 转为 opencv：

```python
img_pil = ImageQt.fromqimage(self.qimage())
img_cv = cv.cvtColor(np.asarray(img_pil), cv.COLOR_RGB2BGR) 
```



# 2. 问题

在 MAC 下同时安装 PyQt 和 opencv 会报错，提示安装了两个 qt 组件。在网上搜索没有解决方法，尝试先安装 opencv 后安装 qt ，可以成功，但后面再安装其他组件又不行了。

但是在 WIN 下却正常运行，没有什么问题。无奈只能在 WIN 下编程了。

待解决。。。