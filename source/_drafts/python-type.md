---
title: 1.1. 基本数据类型
category: Python
tags: Python基础
cover: img/python.png
---
> **`Python`中的数据类型有： 整型、浮点型、字符串类型、列表类型、字典类型、布尔值类型**
## 数字类型
```python
"""整型int"""
age = 18
print(type(age))
# <class 'int'>


"""浮点型float"""
price = 2.5
print(type(price))
# <class 'float'>


"""整形和浮点型相加"""
a = 1
b = 1.5
c = a + b
print(c, type(c))
# 2.5 <class 'float'>
```

## 字符串类型
```python
"""字符串str"""
a = 'xxx'
b = "xxx"
c = '''xxx'''
quotes = """醉后不知天在水，满船清梦压星河。"""
print(type(quotes))
# <class 'str'>


"""字符串中含引号"""
name = 'chengmo'
print(name)
# chengmo

name = '\'chengmo\''
print(name)
# 'chengmo'

name = "'chengmo'"
print(name)
# 'chengmo'

name = """'chengmo'"""
print(name)
# 'chengmo'

name = "\"chengmo\""
print(name)
# "chengmo"

name = '"chengmo"'
print(name)
# "chengmo"

name = '''"chengmo"'''
print(name)
# "chengmo"

"""字符串相加"""
a = 'my name is '
b = 'chengmo'
print(a + b)
# my name is chengmo
# 字符串只能和字符串相加
# 字符串相加效率极低，不推荐使用

"""字符串相乘"""
name = 'chengmo'
print(name * 5)
# chengmochengmochengmochengmochengmo
# 字符串相乘相当于连续拼接了5遍
```

## 列表类型
```python
"""列表list"""
hobbies = ['游泳', '游戏']
print(type(hobbies))
# <class 'list'>

"""取值"""
l = ['米饭', '西红柿鸡蛋汤', ['麻婆豆腐', '红烧肉']]
print(l[0])
print(l[1])
print(l[2])
print(l[2][0])
# 米饭
# 西红柿鸡蛋汤
# ['麻婆豆腐', '红烧肉']
# 麻婆豆腐

# 列表根据索引进行取值，索引从0开始
```

## 字典类型
```python
"""字典dict"""
dic = {
    'name': 'chengmo',
    'age': 18,
    'height': 175,
    'weight': 70
}
print(type(dic))
# <class 'dict'>

"""取值"""
print(dic['name'])
print(dic['height'])
# chengmo
# 175

# 字典根据键值对的键进行取值
```

## 布尔类型
```python
"""布尔类型bool"""
a = True
b = False
print(type(a))
# <class 'bool'>
print(type(b))
# <class 'bool'>

a = 1
b = 0
c = None
print(type(c))
# <class 'NoneType'>
# None是python中一种特殊的类型，None也表示False

# 0 None '' [] {}都代表False
# 除上述几种，其他都的代表True
```

## 列表和字典的嵌套
```python
"""list 和 dict 的嵌套"""
person = [
    {'name': 'chengmo0', 'age': 18, 'height': 190, 'hobbies': ['游戏', '游泳']},
    {'name': 'chengmo1', 'age': 28, 'height': 180, 'hobbies': ['游戏']},
    {'name': 'chengmo2', 'age': 38, 'height': 175, 'hobbies': ['游戏', '游泳', '睡觉']}
]
print(person[0]['hobbies'][0])
# 游戏
print(person[2]['hobbies'][2])
# 睡觉
```

## 可变与不可变类型
- **可变类型**
   值改变的情况下，`id` 不变，说明改的是原值（也就是内存地址不变，变量还是指向原来的内存地址）
```python
# 列表、字典
hobbies = ['游戏', '游泳', '睡觉']
print(id(hobbies))
# 2217765248384
hobbies[0] = '学习'
print(hobbies)
# ['学习', '游泳', '睡觉']
print(id(hobbies))
# 2217765248384

# 可以看出在改变值的情况下，id没有发生改变
```
- **不可变类型**
   值改变的情况下，`id` 变了（也就是申请了新的内存地址，变量指向了新的内存地址）
```python
# 整型、浮点型、字符串类型、布尔值类型
name = 'xiaoxiaomo'
print(id(name))
# 2217765503536
name = 'chengmo'
print(id(name))
# 2217765503088

# 可以看出在改变值的情况下，id发生了改变
```
### 补充
> 字典类型数据：{key: value} 中的key必须为不可变类型，但一般均为str类型

## 直接引用、间接引用、循环引用
```python
"""直接引用和间接引用"""
name = 'chengmo'
# name 对chengmo进行了直接引用
list = ['a', 'b', 'c', name]
# list 对chengmo进行了间接引用
# 此时, name 的引用计数为2
name = 'chengmo23'
print(list[3])
print(name)
# chengmo
# chengmo23

# 字符串'chengmo'的引用计数由2减为1

"""循环引用"""
list1 = ['a', 'b']
list2 = ['x', 'y']
list1.append(list2)
list2.append(list1)
# 循环引用会导致内存泄漏
```
![引用](img/reference.png)
## 垃圾回收机制
> 引用计数 - 当引用计数为0时，对象被垃圾回收机制回收
>
> 
> 标记清除 - 当堆区的值没有被栈区的变量直接或间接引用时，就会被标记清除
>
> 
> 分代回收 - 对标记清除执行的时机进行优化
```python
a = 100
b = a
c = a
# 100 分别被 a、b、c 引用，引用计数为：3
```
![引用计数增加](img/reference-add.png)
```python
a = 100
b = a
c = a
del a
del b
# 100的引用计数变成了1
# 若内存空间引用计数变为0后，就会被进行回收
```
![引用计数减少](img/reference-sub.png)