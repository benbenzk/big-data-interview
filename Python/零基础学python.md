# 极客时间 - Python笔记

## 第一章 Python介绍与安装

python发展历史

## 第二章 Python基础语法

### Python第一个程序

```python
#这是我的第一个python程序
import time

#在屏幕上打印出从1970年1月1日0:00 到现在过了多少秒
print(time.time())
if 10 - 9 > 0:
    #这行需要缩进，缩进一个空格
    print('10 大于 9')
```

### 基本数据类型


| 整型(int) | 浮点型(float) | 字符串(str)     | 布尔值(bool)  |
| ----------- | --------------- | ----------------- | --------------- |
| 8         | 8.8           | "8"    "python" | True    false |

使用type()判断数据类型；类型转换：int('8')，str(8), bool(8)

### 变量

![变量](imgs_零基础学python/变量.png)

变量命名规则：必须使用字母、下划线开头,数字不能开头；不能使用Python内置的关键字或保留字；变量严格区分大小写

## 第三章 序列

### 序列的概念及常用操作符

![序列](imgs_零基础学python/序列.png)

序列的基本操作

![序列的基本操作](imgs_零基础学python/序列的基本操作.png)

### 字符串

```python
#记录生肖，根据年份来判断生肖
chinese_zodiac = '猴鸡狗猪鼠牛虎兔龙蛇马羊'

print(chinese_zodiac[0:4]) #猴鸡狗猪
print(chinese_zodiac[-1]) #羊

year = 2018
print(year % 12) #2

print(chinese_zodiac[year % 12]) #狗
print('狗' not in chinese_zodiac) #False
print (chinese_zodiac + 'abcd') #猴鸡狗猪鼠牛虎兔龙蛇马羊abcd
print (chinese_zodiac * 3) #猴鸡狗猪鼠牛虎兔龙蛇马羊猴鸡狗猪鼠牛虎兔龙蛇马羊猴鸡狗猪鼠牛虎兔龙蛇马羊
```

### 元组

```python
#定义一个星座元组
zodiac_name = (u'摩羯座', u'水瓶座', u'双鱼座', u'白羊座', u'金牛座', u'双子座', u'巨蟹座', u'狮子座', u'处女座', u'天秤座', u'天蝎座', u'射手座')
zodiac_days = ((1, 20), (2, 19), (3, 21), (4, 21), (5, 21), (6, 22), (7, 23), (8, 23), (9, 23), (10, 23), (11, 23), (12, 23))
(month, day) = (2, 15)

zodiac_day = filter(lambda x: x <= (month, day), zodiac_days)
zodiac_len = len(list(zodiac_day)) % 12
print(zodiac_name[zodiac_len])
```

### 列表

```python
a_list = ['abc', 'xyz']

# 列表追加元组
a_list.append('X')
print(a_list)	#['abc', 'xyz', 'X']
# 列表删除元素
a_list.remove('abc')
print(a_list)	#['xyz', 'X']
```

**元组与列表的核心区别在于,元组是静态的,列表是动态的, 可变的**

## 第四章 条件与循环

### 条件语句if

![条件语句](imgs_零基础学python/条件语句if.png)

```python
x = 'abcd'
if x == 'abc':
    print('x的值和abc相等')
else:
    print('x和abc不想等')
  
#记录生肖，根据年份来判断生肖
chinese_zodiac = '猴鸡狗猪鼠牛虎兔龙蛇马羊'
year = int(input('请用户输入出生年份'))
if chinese_zodiac[year % 12] == '狗':
    print('狗年运势。。。')
```

### for循环和while循环

![循环语句](imgs_零基础学python/循环语句.png)

```python
import time

# 记录生肖，根据年份来判断生肖
chinese_zodiac = '猴鸡狗猪鼠牛虎兔龙蛇马羊'

year = int(input('请用户输入出生年份'))
if(chinese_zodiac[year % 12]) == '狗':
    print('狗年运势。。。')

for cz in chinese_zodiac:
    print(cz)

for i in range(9):
    print(i)

for year in range(2000, 2019):
    print("%s 年的生肖是 %s" % (year, chinese_zodiac[year % 12]))

num = 5

while True:
    num = num + 1
    if num > 10:
        break
    # if num == 10:
    #     continue
    print(num)
    time.sleep(1)
```

### for循环语句中的if嵌套

```python
zodiac_name = (u'摩羯座', u'水瓶座', u'双鱼座', u'白羊座', u'金牛座', u'双子座', u'巨蟹座', u'狮子座', u'处女座', u'天秤座', u'天蝎座', u'射手座')
zodiac_days = ((1, 20), (2, 19), (3, 21), (4, 21), (5, 21), (6, 22), (7, 23), (8, 23), (9, 23), (10, 23), (11, 23), (12, 23))

int_month = int(input("请输入月份："))
int_day = int(input("请输入日期："))

for zd_num in range(len(zodiac_days)):
    if zodiac_days[zd_num] >= (int_month, int_day):
        print(zodiac_name[zd_num])
        break
    elif int_month == 12 and int_day > 23:
        print(zodiac_name[0])
        break
```

### while循环语句中的if嵌套

```python
zodiac_name = (u'摩羯座', u'水瓶座', u'双鱼座', u'白羊座', u'金牛座', u'双子座', u'巨蟹座', u'狮子座', u'处女座', u'天秤座', u'天蝎座', u'射手座')
zodiac_days = ((1, 20), (2, 19), (3, 21), (4, 21), (5, 21), (6, 22),
              (7, 23), (8, 23), (9, 23), (10, 23), (11, 23), (12, 23))


# 用户输入月份和日期
int_month = int(input('请输入月份：'))
int_day = int(input('请输入日期'))

n = 0
while zodiac_days[n] < (int_month,int_day):
    if int_month == 12 and int_day >23:
        break
    n += 1

print(zodiac_name[n])
```

## 第五章 映射与字典

![映射的字典](imgs_零基础学python/映射的字典.png)

### 字典的定义和常用操作

```

```

### 列表推倒式与字典推倒式

```

```

## 第六章 文件和输入输出

### 文件的内建函数

### 文件的常用操作

## 第七章 错误和异常

### 异常的检测和处理
