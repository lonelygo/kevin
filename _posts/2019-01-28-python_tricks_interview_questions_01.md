---
layout: post
title: Python tricks for interview questions
subtitle: Some python tricks you should know.
bigimg: 
  -  "/img/big-imgs/hangzhou.jpeg" : "杭州，西湖，2016"
tags: [Python, 教程]
comments: true
---

闲来无事，把平时看到的一些Python tricks记录下来，大多都是基础用法，对于应付基础面试应该很好用的。

# Python tricks - 01

## list and dictionary

### Merging and sorted two list

有以下两个列表，把两个无序列表合并为一个有序列表。

``` python
a = [4, 6, 2, 11, 33, 19]
b =[1, 7, 5,13, 18, 28, 21, 99]
```

最快速的方法是使用：list.**extend()**:

``` python
a = [4, 6, 2, 11, 33, 19]
b =[1, 7, 5,13, 18, 28, 21, 99]
a.extend(b)
c = sorted(a)
print(c)
[1, 2, 4, 5, 6, 7, 11, 13, 18, 19, 21, 28, 33, 99]
```

list.**extend()** 与 list.**append()** 是不同的：

``` python
# a.append(b)
[4, 6, 2, 11, 33, 19, [1, 7, 5, 13, 18, 28, 21, 99]]  

# a.extend(b)
[4, 6, 2, 11, 33, 19, 1, 7, 5, 13, 18, 28, 21, 99]

```

### Initializing dictionary : 获取词频

``` python
s = '''Writing programs (or programming) is a very creative 
and rewarding activity.  You can write programs for 
many reasons, ranging from making your living to solving
a difficult data analysis problem to having fun to helping
someone else solve a problem.  This book assumes that 
everyone needs to know how to program, and that once 
you know how to program you will figure out what you want 
to do with your newfound skills. '''
```

使用**fromkeys()**将字典初始化为0 ,写法如下：

``` python
words = s.split()
d = {}.fromkeys(words, 0)
for w in words:
  d[w] += 1
print(d)
{'Writing': 1, 'programs': 2, '(or': 1, 'programming)': 1, 'is': 1, 'a': 3, 'very': 1, 'creative': 1, 'and': 2, 'rewarding': 1, 'activity.': 1, 'You': 1, 'can': 1, 'write': 1, 'for': 1, 'many': 1, 'reasons,': 1, 'ranging': 1, 'from': 1, 'making': 1, 'your': 2, 'living': 1, 'to': 7, 'solving': 1, 'difficult': 1, 'data': 1, 'analysis': 1, 'problem': 1, 'having': 1, 'fun': 1, 'helping': 1, 'someone': 1, 'else': 1, 'solve': 1, 'problem.': 1, 'This': 1, 'book': 1, 'assumes': 1, 'that': 2, 'everyone': 1, 'needs': 1, 'know': 2, 'how': 2, 'program,': 1, 'once': 1, 'you': 3, 'program': 1, 'will': 1, 'figure': 1, 'out': 1, 'what': 1, 'want': 1, 'do': 1, 'with': 1, 'newfound': 1, 'skills.': 1}
```

另一种初始化的方法：

``` python
d = {}
for w in s.split():
    d[w] = d.get(w, 0) + 1
print (d)
```

第三种方法，使用**collections.defaultdict(int)**进行字典初始化，便于计数。

``` python
from collections import defaultdict
d = defaultdict(int)
for w in words:
    d[w] += 1
print (d)
```

### Initializing dictionary : 用列表初始化

我们知道，字典的`key`值是有唯一性的，像下面这个字典，为了解决Key值的唯一性，只能这样构造，但是我们可以重新构造，使`value`变为列表。

``` python
cities = {'San Francisco': 'US', 'London':'UK',
        'Manchester':'UK', 'Shanghai':'China',
        'Los Angeles':'US', 'Beijing':'China'}
```

我们可以这样做：

``` python
from collections import defaultdict
# using collections.defaultdict()
dict1 = defaultdict(list)
for k,v in cities.items():
    dict1[v].append(k)
print (dict1)

# using dict.setdefault(key, default=None)
dict2 = {}
for k,v in cities.items():
       dict2.setdefault(v,[]).append(k)
print (dict2)
```

结果如下：

``` python
# using collections.defaultdict()
defaultdict(<class 'list'>, {'US': ['San Francisco', 'Los Angeles'], 'UK': ['London', 'Manchester'], 'China': ['Shanghai', 'Beijing']})
# using dict.setdefault()
{'US': ['San Francisco', 'Los Angeles'], 'UK': ['London', 'Manchester'], 'China': ['Shanghai', 'Beijing']}
```

