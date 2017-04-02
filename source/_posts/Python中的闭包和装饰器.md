---
title: Python中的闭包和装饰器
date: 2017-04-02 21:22:16
categories: Python
tags: 
	- Python
---

根据廖雪峰博客上的python教程总结。

#### 闭包 ####

python中的闭包大概是以下形式：

```python
def func(k):
    #########
    def g():
        return k*k
    return g
```

由上述函数我们可以看到 func 函数返回的是一个函数实例，而此时函数并没有执行。当我们调用返回的实例时，即：

<!-- more -->



```python
>>> f = func(2)
>>> f()
4
```

函数才会执行，并且输出值为 4（也就是 2 的平方）。按照个人的理解，此时返回的函数实例 f 由于并没有执行，局部变量仍然保存在实例中，并没有释放，当我们调用该实例时函数才会执行，并将保存的局部变量作为参数。

而当我们这样调用时：

```python
def func():
    #########
	fn = []
    for item in range(0, 4):
        def g():
            return item * item
        fn.append(g)
    return fn
	
    

>>> f1, f2, f3, f4 = func()
>>> f1()
9
>>> f2()
9
```

并且

```python
def F():
	fn = []
    for item in range(0, 4):
        def f():
        	def g():
            	return item * item
            return g
        fn.append(f())
    return fn
```

两种情况输出一样

我们会发现 f1、f2、f3、f4 实例的执行结果都保持为最后 item = 3 的状态，由此我们可以了解到，每个闭包实例保存的都是**上层（局部变量所在的）函数执行完毕**时局部变量的状态。由此，我们可以加一层嵌套来使四个实例输出不同的值： 

```python
def F():
	fn = []
    for item in range(0, 4):
        def f(i):
        	def g():
            	return i * i
            return g
        fn.append(f(item))
    return fn



>>> f1()
0
>>> f2()
1
```

当然，使用匿名函数 lambda 可以缩短代码。

#### 装饰器 ####

装饰器的使用方式为：

```python
@decorator
def func():
    #######
```

它等效于

```python
func = decorator(func)
```

可以很容易地猜想到，decorator 的返回值是一个函数。装饰器可以让我们在不改变原函数的情况下为函数增加功能。比如说：

```python
@decorate
def func():
    print("the function I am using")
    
def decorate(func):
    @functools.wraps(func)
    def wrapper(*args, **kw):
        print("%s" % func.__name__)
        return func(*args, **kw)
    return wrapper
```

其中，decorate 函数作为一个装饰器接受一个函数作为参数，并输出一个函数。而之后我们调用 func 函数就等效于调用 wrapper 函数了。如果我们需要让 decorate 函数可以接收其他参数时，我们可以再增加一层嵌套：

```python
@log('look !')
def func():
    print("the function I am using")

def log(text)
    def decorate(func):
        @functools.wraps(func)
        def wrapper(*args, **kw):
            print("%s" % (text, func.__name__))
            return func(*args, **kw)
        return wrapper
    return decorate
```

此时等效于：

```python
func() = log('look !')(func)
```

其中，因为此时函数的 \_\_name\_\_ 值发生了改变，第 7 行的代码用来将  \_\_name\_\_ 从 "wrapper" 恢复成 "func"。防止某些依赖函数签名的代码执行出错。