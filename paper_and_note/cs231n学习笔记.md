## python
- 1. python的基础类型有：整型、浮点型、布尔型和字符串
  - 整形和其它语言的计算方式大致一样，区别：**表示次方；没有++和--
  - 布尔型直接用：and、or和not
  - 字符型：两个字符串可以用+来拼接字符串；重要的方法：replace替换字符、strip去除字符、split将字符串分开
- 2.容器有列表、字典、集合和元组
  - 列表（list）：
  
    一个列表中可以存放不同基础类型的变量；
    
    两个列表可以用+号拼接；重要的方法：append在后面插进一个元素、insert在任意位置插入和pop末尾弹出一个元素
  
    列表解析式：用循环定义列表：[i**2 for i in x if i >0]
  
  - 字典（dict）：
  
    字典存放的是（关键字，值）对，由关键字可以索引到对应的值。
    
    重要的方法：get(key,default)、items()返回关键字-值对、keys返回所有关键字、values返回所有值
    
    字典解析式：用循环定义字典：{i:i**2 for i in x if i>0}
  
  - 集合（set）：
  
    集合中的元素没有顺序且不重复
    
    重要的方法：issubset检验是否是子集、issuperset检验是否是父集、union和集、difference差集、symmetric_difference对称差集
    
    集合解析式：{i for i in range(5)}
- 函数：

  用关键字def来定义函数，def后接函数名和参数列表。定义完函数名和参数后，函数主体需要缩进。
  在函数主体之前，最好加上对函数的描述，包括：总体的概述、参数的解释、输出变量的解释和如何使用函数等。
  
  参数列表中：*arguments是用来接收元组的，元组的长度任意。**keywords是用来接收字典的，字典长度不限
  
- 类（class）：

  类用关键字class来定义，用def __init__()来定义类的初始化方法。
  
  类变量，类变量是类的实例化对象所共有的，在类初始化之前定义。
  
  多重继承：
  ```python
  class DerivedClassName(Base1, Base2, Base3):
    <statement-1>
    .
    .
    .
    <statement-N>
  ```
方法的重写，如果基类的方法无法满足需求，就可以在子类中重写基类的方法。
