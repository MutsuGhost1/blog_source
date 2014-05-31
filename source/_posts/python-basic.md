title: Python Basic
date: 2013-09-29 20:45:42
categories: Software
tags: [Python, Study Group]
---
* [Python's Folder Structure](#python_folder_structure "Python's Folder Structure")
* [關於 *yield*](#yield_about "關於 yield")
    *  [yield 是為何而生 ?](#yield_history "yield 是為何而生 ?")
    *  [yield 的各種用法](#yield_usage "yield 的各種用法")
        *  [yield 的基本運作原理](#yield_fundamental "的基本運作原理")
        *  [send(expression) - pass a value to generator](#yield_send "send(expression)")
        *  [throw(type[,value[,traceback]]) - raise exception inside generator](#yield_throw "throw")
        *  [close() - terminate the generator](#yield_close "close()")
        *  [generator expression - 快速產生一個 generator object](#yield_generator_exp "generator expression")
    *  [Reference](#yield_reference)
* Python Scope 概念
* [其他](#others "其他")
    * [List Comprehesion](http://www.python.org/dev/peps/pep-0202/ "PEP202") 
    * [Lambda expression](#lambda_exp "Lambda expression")
    * [Built-in Funtion map](#built_in_map "Built-in Funtion map")
    * [Built-in Funtion reduce](#built_in_reduce "Built-in Funtion reduce")
    * [Built-in Funtion filter](#built_in_filter "Built-in Funtion filter")
    * [Built-in Funtion zip](#built_in_zip "Built-in Funtion zip")
<!--more-->
----------

<a name=python_folder_structure></a>
# *Python's Folder Structure* #
在開始使用 Python 之前, 必須先在系統裝[安裝 Python 的執行環境](http://www.python.org/getit/), 安裝完後會發現, 預設的資料夾結構如下:

![](http://MutsuGhost1.github.io/image/python_install1.PNG)


* 在 PythonXX --> Lib 目錄中, 可以看到 Python Built-in Module 的 Source Code  
* 在 PythonXX --> Lib --> site-packages 目錄中, 可以看到安裝的 3rd Party Package

![](http://MutsuGhost1.github.io/image/python_install2.PNG)
  
----------

<a name=yield_about></a>
# 關於 *yield* 用法 #
<a name=yield_history></a>
## yield 是為何而生 ? ##

在 [PEP255](http://www.python.org/dev/peps/pep-0255 "PEP255") 中,有一段話是這麼描述的  

> *When a producer function has a hard enough job that it requires
> maintaining state between values produced, most programming languages
> offer no pleasant and efficient solution beyond adding a callback
> function to the producer's argument list, to be called with each value produced*  

當一個複雜的 producer function, 為了在每次被呼叫時, 正確的 produce 出資料, 必須在 producer function  
內實作一些複雜的 states 使得資料可以被正確產生.  

[以 Fibonacci Series 為例](http://en.wikipedia.org/wiki/Fibonacci_number "Fibonacci Series in Wiki")

    # fib0.py produce 出 n 個 number 的 fibonacci series
    def fib0(end):
        a, b = 0, 1
        for i in range(end):
            a, b = b, a + b
            print "i=", i, " a=", a
    
    def main():
        fib0(10)

    if __name__ == '__main__':
        main()

Output:  

    i= 0 a= 1
    i= 1 a= 1
    i= 2 a= 2
    i= 3 a= 3
    i= 4 a= 5
    i= 5 a= 8
    i= 6 a= 13
    i= 7 a= 21
    i= 8 a= 34
    i= 9 a= 55
  
上述 **fib0.py** 的做法, 只能一次 produce 出所有的 numbers, 若要做成每次呼叫才 produce 出 number,  
必須由 programmer 定義額外的 states 來做處理, 類似的做法如下:
    
    # fib1.py produce 出 n 個 number 的 fibonacci series
    # global variable to keep state
    fib_state_cur = 0
    fib_state_end = 0
    fib_state_a = 0
    fib_state_b = 1
    def fib1(end=-1):
        # use global variable to keep state
        global fib_state_cur, fib_state_end, fib_state_a, fib_state_b
    
        # reset state for each new call
        if 0 < end:
            fib_state_cur = 0
            fib_state_end = end
            fib_state_a = 0
            fib_state_b = 1
    
        # restore the state for each call
        start = fib_state_cur
        end = fib_state_end
        a, b = fib_state_a, fib_state_b
    
        for i in range(start, end):
            a, b = b, a + b
            # keep the state
            fib_state_cur = i+1
            fib_state_a, fib_state_b = a, b
            return a
    
    def main():
        for i in range(10):
            if 0 == i:
                # first time to initialize the number of items
                print "i=", i, "a=", fib1(10)
            else:
                # get the next number
                print "i=", i, "a=", fib1()
    
    if __name__ == '__main__':
        main()
    
Output:  

    i= 0 a= 1
    i= 1 a= 1
    i= 2 a= 2
    i= 3 a= 3
    i= 4 a= 5
    i= 5 a= 8
    i= 6 a= 13
    i= 7 a= 21
    i= 8 a= 34
    i= 9 a= 55


大多數的程式語言, 針對這樣的問題在語法上並沒有直接的支援, 因此除了上述的做法外, 大多數折衷的做法都是提供  
callback function 給producer function 當作參數, 當每次資料被 produce 出來時, 透過 callback function 來通知 consumer.  

為了對類似的需求, 有更好的支援, [PEP255](http://www.python.org/dev/peps/pep-0255 "PEP255") 被提出且實現於 Python 2.3 中, 在初版 [PEP255](http://www.python.org/dev/peps/pep-0255 "PEP255") 的定義中, yield 是一個 statement.

透過 yield 的支援, 可將 **fib1.py** 簡化為 **fib2.py**, 如下:
    
    def fib2(end):
        a, b = 0, 1
        for i in range(end):
            a, b = b, a + b
            yield a
    
    def main():
        i = 0
        for a in fib2(10):
            print "i=", i, "a=", a
            i+=1  

Output:  

    i= 0 a= 1
    i= 1 a= 1
    i= 2 a= 2
    i= 3 a= 3
    i= 4 a= 5
    i= 5 a= 8
    i= 6 a= 13
    i= 7 a= 21
    i= 8 a= 34
    i= 9 a= 55

到了 Python 2.5, 針對 yield 有更進一步的 enhancement [PEP342](http://www.python.org/dev/peps/pep-0342/ "PEP342"), 使其能夠較完善的處理 [Coroutine](http://en.wikipedia.org/wiki/Coroutine#Implementations_for_Python "Coroutine in Wiki") 這類的問題.  

----------
<a name=yield_usage></a>
## yield 的各種用法 ##
由於 [PEP342](http://www.python.org/dev/peps/pep-0342/ "PEP342") 是針對 [PEP255](http://www.python.org/dev/peps/pep-0255 "PEP255") 的 enhancement, 因此以下直接針對 [PEP342](http://www.python.org/dev/peps/pep-0342/ "PEP342") 的各項用法, 一一做個說明.  
[PEP255](http://www.python.org/dev/peps/pep-0255 "PEP255") 中, yield 是一個 statement.  
[PEP342](http://www.python.org/dev/peps/pep-0342/ "PEP342") 中, yield 是一個 expression, 因此將 yield expression 的回傳值忽略掉當成 yield statement 來討論既可.  

<a name=yield_fundamental></a>
### yield 的基本運作原理 ###
  
首先, 先看看 Python 如何看待一個包含有 yield 語法的 function [1]:

> A function which returns an iterator. It looks like a normal function except that it contains yield statements  
> for producing a series a values usable in a for-loop or that can be retrieved one at a time with the next() function.  
> Each yield temporarily suspends processing, remembering the location execution state  
> (including local variables and pending try-statements).  
> When the generator resumes, it picks-up where it left-off  (in contrast to functions which start fresh on every invocation).  

上面這句話, 我們可以拆成幾個部份來理解:  

1. 當一個 function 內包含了 yield 語法, 這個 function 就稱為 generator.  
2. 這個 function (generator) 被呼叫時, 回傳一個 iterator (generator object).  
3. iterator 內執行的代碼, 既為 function 內定義的代碼.  
   唯一的差異是, 每次呼叫 next() 都會在 yield expression 處 suspend (儲存整個 context 的變數狀態)   
   並且 return 其 yield 的 value 給 next() 的 caller
4. 每次呼叫 next(), 都會從上一次結束的地方 resume (恢復整個 context 的變數狀態), 並繼續執行到下個 yield 處
5. 若呼叫 next() 時, 代碼中已無 yield expression 可 suspend, 則產生 StopIterator Exception  

下面的代碼, 驗證了上述的說法:
    
    # generator_demo.py  

    # 1. define a function (generator)
    def gen():
        x = 100              # 第一次呼叫 next() 將於此開始執行
        yield x              # 第一次 next() 呼叫後, 將 suspend 於此, 並回傳 x
        x += 20              # 第二次呼叫 next() 將於此開始執行, 保有上次執行個變數內容的 context
        yield x              # 第二次 next() 呼叫後, 將 suspend 於此, 並回傳 x
                             # 第三次呼叫 next() 將觸發 StopIterator Exception
    
    def main():
        print type(gen)      # 1. gen   型別為 <type 'function'>, 提供產生 generator(iterator) object
        print type(gen())    # 2. gen() 型別為 <type 'generator'>
        gen_var = gen()      
        print gen_var        # gen_var 為一個 <generator object gen at 0x00DE8288>
        print type(gen_var)  # gen_var 型別為 <type 'generator'>
        print gen_var.next() # 3. 100
        print gen_var.next() # 4. 120
        print gen_var.next() # 5. raise StopIteratorException
    
    if __name__ == '__main__':
        main() 

Output:  
    
    <type 'function'>
    <type 'generator'>
    <type 'generator'>
    <generator object gen at 0x00DE82B0>
    100
    120
    Traceback (most recent call last):
      File "<module1>", line 29, in <module>
      File "<module1>", line 26, in main
    StopIteration
    
歸納上述幾項重點:  

*  **yield 基本上是搭配在 function 內部使用的**
*  使用 yield 語法的 function, 又稱之為 generator
*  generator 所產生的 generator object 是一個 iterator.
*  generator 每次執行的 next() 代碼, 即為 function 內部的代碼, 差別在於
    * next() 執行到 yield 處即 suspend, 並回傳 yield 的 value 給 next() caller  
    * 每次執行 next() 將會從上次 suspend 處, 繼續 resume, 且保有上次 context 內的變數狀態
* 若呼叫 next() 時, 代碼中已無 yield expression 可 suspend, 則產生 StopIterator Exception

<a name=yield_send></a>
## send(expression) - pass a value to generator ##
理解完 [yield 的基本運作原理](#yield_fundamental "的基本運作原理"), 就很好理解 send 這個 method 的行為.  
send(expression) 和 next() 一樣, 是 trigger iterator 的執行, 直到遇到下一個 yield expression.  
差異在於:  

* send(expression) 是 [PEP342](http://www.python.org/dev/peps/pep-0342/ "Coroutines via Enhanced Generators") 提出, 在 Python 2.5 實現, 用來指定 yield expression 所 evaluate 的結果  
* 這也是為何在 Python 2.5 中, yield 從 statement 變更為 expression 的原因之一
* 在 Python 2.3, yield 還是一個 statement  

下列代碼說明了, send(expression) 如何指令 yield expression 的結果.

    # generator_send_demo.py

    def gen2():
        x = 100                     # 呼叫 next() 將於此開始執行
        y = yield x                 # next() 呼叫後, 將 suspend 於此, 並回傳 x
                                    # 呼叫 send(10) 將於此開始執行, 保有上次執行個變數內容的 context,  
                                    # 將 send 指定的值 assign 給 y
        x += y                      # send(10) 呼叫後, 執行此 statement, 
        y = yield x                 # send(10) 呼叫後, 將 suspend 於此, 並回傳 x
                                    # 呼叫 send(20) 將於此開始執行, 保有上次執行個變數內容的 context,   
                                    # 將 send 指定的值 assign 給 y
        x += y                      # send(20) 呼叫後, 執行 statement,
        yield x                     # send(20) 呼叫後, 將 suspend 於此, 並回傳 x
    
    def main():
        gen_var2 = gen2()           # 產生一個 generator object 儲存於 gen_var2
        print gen_var2.next()       # the same result with calling gen_var2.send(None)
        print gen_var2.send(10)     # 執行 gen_var2.send()
        print gen_var2.send(20)     # 執行 gen_var2.send()
        print gen_var2.send(30)     # the same reslut with calling gen_var2.next(), 
                                    # raise StopIterator Exception

    if __name__ == '__main__':
        main() 

Output:  

    100
    110
    130
    Traceback (most recent call last):
      File "<module1>", line 49, in <module>
      File "<module1>", line 35, in main
    StopIteration

這邊補充幾個重點:  

* 可將 y = yield x 看成右半部跟左半部兩個動作, suspend 的時候執行右半部, resume 的時候才執行左半部的 assign.  
* 透過 generator object, 第一次呼 iterate iterator 時, 只能使用 next() or send(None), 兩者等價 

<a name=yield_throw></a>
## throw(type[,value[,traceback]]) - raise exception inside generator ##
[PEP342](http://www.python.org/dev/peps/pep-0342/ "Coroutines via Enhanced Generators") 提到, 想在 generator 中, suspend 在 iterator 的點, 產生 exception, 可以透過 throw 來達成.  

參考下列代碼:  

    # generator_throw_demo.py

    def gen3():
        x = 100                              # 呼叫 gen_var3.send(None) 將由此開始執行
        try:
            y = yield x                      # gen_var3.send(None) 呼叫後, 將 suspend 於此, 並回傳 x
                                             # throw GeneratorExit 發生於此
            x += y
            y = yield x
            x += y
            yield x
    except:                                  # 處理所有型態的 exception, 包含 GeneratorExit Exception
        pass                                 # pass GeneratorExit Exception 的處理
                                             # 由於沒有 yield expression 了, 
                                             # 因此 raise StopIterator Exception
    
    def main():
        gen_var3 = gen3()                    # 產生一個 generator object 儲存於 gen_var3
        print gen_var3.send(None)            # the same result with calling gen_var2.next()
        print gen_var3.throw(GeneratorExit)  # 在 generator 內部產生一個 GeneratorExit Exception
    
    if __name__ == '__main__':
        main()

Output:  

    100
    Traceback (most recent call last):
      File "<module1>", line 47, in <module>
      File "<module1>", line 33, in main
    StopIteration

此外, 由外部 trigger generator 內的 exception 後, 仍然可在內部的 exception handler 搭配 yield 繼續做處理.  

參考下列代碼:  

    # generator_throw_demo2.py

    def gen3():
        x = 100                               # 第一次 gen_var3.send(None) 將由此開始執行
        try:
            y = yield x                       # 第一次 gen_var3.send(None) 呼叫後, 將 suspend 於此,  
                                              # 並回傳 x
                                              # throw GeneratorExit 發生於此
            x += y
            y = yield x
            x += y
            yield x
    except:                                   # 處理所有型態的 exception, 包含 GeneratorExit Exception
        x +=10                                # throw GeneratorExit 發生後, 執行此 statement 
        yield x                               # throw GeneratorExit 發生後, 將 suspend 於此, 並回傳 x
                                              # 第二次 gen_var3.send(None) 由此開始執行
        x +=20                                # 第二次 gen_var3.send(None) 發生後, 執行此 statement
        yield x                               # 第二次 gen_var3.send(None) 發生後, 將 suspend 於此, 
                                              # 並回傳 x
    
    def main():
        gen_var3 = gen3()                     # 產生一個 generator object 儲存於 gen_var3
        print gen_var3.send(None)             # the same result with calling gen_var2.next()
        print gen_var3.throw(GeneratorExit)   # 在 generator 內部產生一個 GeneratorExit Exception
        print gen_var3.send(None)             # the same result with calling gen_var2.next()
        print gen_var3.send(None)             # the same result with calling gen_var2.next()

Output:  

    100
    110
    130
    Traceback (most recent call last):
      File "<module1>", line 51, in <module>
      File "<module1>", line 37, in main
    StopIteration

<a name=yield_close></a>
## close() - terminate the generator ##
[PEP342](http://www.python.org/dev/peps/pep-0342/ "Coroutines via Enhanced Generators") 提到, 當 generator object 不再使用時, 可以呼叫 close() method, 之後如果再呼叫 next() method 則會 raise StopIterator Exception.  

參考下面代碼:  

    # generator_close_demo.py

    def gen2():
        x = 100
        y = yield x
        x += y
        y = yield x
        x += y
        yield x
    
    def main():
        gen_var2 = gen2()
        print gen_var2.send(None)
        gen_var2.close()
        print gen_var2.send(10)     # raise StopIterator Exception, if next() is called after close()
        print gen_var2.send(20)
    
    if __name__ == '__main__':
        main()

Output:

    100
    Traceback (most recent call last):
      File "<module1>", line 46, in <module>
      File "<module1>", line 31, in main
    StopIteration

<a name=yield_generator_exp></a>
## generator expression - 快速產生一個 generator object ##
除了透過定義包含 yield expression 的 function 外, 可以透過 Python 語法快速的產生 generator object.
引述 [Python Glossary](http://docs.python.org/2.7/glossary.html#term-generator "generator expression") 的定義:

> An expression that returns an iterator. It looks like a normal expression followed by a for expression defining a loop variable, range, and an optional if expression

範例如下:

    # generator_expression_demo.py

    def main():
        gen_exp = ((i+1)*(i+1) for i in range(10))  # 產生一個 generator object
    
        print type(gen_exp)
        print gen_exp                               # 和 generator 產生出來的 object 印出來的資訊不太一樣
    
        for x in gen_exp:
            print x
    
    if __name__ == '__main__':
        main()

Output:
    
    <type 'generator'>
    <generator object <genexpr> at 0x00DE82B0>
    1
    4
    9
    16
    25
    36
    49
    64
    81
    100

這裡有一點要注意, generator expression 產生出來的 object 與 generator 產生出來的 object print 出來的結果有些不同.  

  * print generator expression 產生的 object 會印出 &lt;generator object **&lt;genexpr&gt;** at 0x00DE82B0>
  * print generator 產生的 object 會印出 &lt;generator object **gen** at 0x00DE82B0&gt;

<a name=yield_reference></a>
## Reference: ##

 * [1] [Definition for generator in python](http://docs.python.org/2.7/glossary.html#term-generator)
 * [2] [limodou的学习记录](http://blog.donews.com/limodou/archive/2006/09/04/1028747.aspx)
 * [3] [python yield 研究](http://dhcmrlchtdj.github.io/sia/post/2012-11-20/python_yield.html)
 * [4] [PEP255](http://www.python.org/dev/peps/pep-0255/ "Simple Generators")
 * [5] [PEP289](http://www.python.org/dev/peps/pep-0289/ "Generator Expressions")
 * [6] [PEP342](http://www.python.org/dev/peps/pep-0342/ "Coroutines via Enhanced Generators")
 * [7] [Python 深入理解yield](http://www.jb51.net/article/15717.htm)

----------
<a name=others></a>
# *其它* #

<a name=lambda_exp></a>
## Lambda Expression ##
Python 中使用 Lambda Expression 的語法可以一個建立 Annonymous Function 

Ex:

    f = lambda x: x ** x
    print(f(1))
    print(f(2))
    print(f(3))
    print(f(4))
    print(f(5))
     
    g = lambda x, y: x * y
    print(g(1, 6))
    print(g(2, 7))
    print(g(3, 8))
    print(g(4, 9))
    print(g(5, 10))

Output:
    
    1
    4
    27
    256
    3125
    6
    14
    24
    36
    50

<a name=built_in_map></a>
## Built-in Function map ##
在探討 map function 之前, 讓我們先回憶一下國中時候學過的數學函數,  
假設函數 f(x,y,..,z) 輸入一組 x1, y1, ..., z1 參數後, 可得到一個結果 r1,  
現在我們準備 n 組資料 [(x1, y1, ..., z1), (x2, y2, ..., z2), ..., (xn, yn, ..., zn)] 
依序輸入給函數 f(x,y,...,z) 後,  
可得到 n 個結果 [r1,r2, ..., rn], 如下圖所示:

![](http://MutsuGhost1.github.io/image/python_map_function.PNG)

事實上 map function 就是執行類似這樣的概念. map function 有兩組 input 參數:  
1. function (對應到上述的函數 f)  
2. 輸入 function 的參數, 參數個數必須與 function 參數匹配

下列代碼展示 map function 的使用方法:
    
    def f(a,b,c):
        return a*b*c
    
    def main():
        lista = [1,2,3,4,5,6]
        listb = [1,2,3,4,5,6]
        listc = [1,2,3,4,5,6]
        print map(f, lista, listb, listc)

Output:  

    [1, 8, 27, 64, 125, 216]

有一點要注意的事情, 當參數是一個 sequence 時, **各個 sequence 的參數不一致, 將取最長的為主, 不足補 None**.  
最後, 附上 [Python 文件](http://docs.python.org/2/library/functions.html#map)上的說明:  

*map(function, iterable, ...)*  

>Apply function to every item of iterable and return a list of the results.
If additional iterable arguments are passed, function must take that many arguments and is applied to the items from all iterables in parallel. **If one iterable is shorter than another it is assumed to be extended with None items**. **If function is None, the identity function is assumed**; if there are multiple arguments, map() returns a list consisting of tuples containing the corresponding items from all iterables (a kind of transpose operation). The iterable arguments may be a sequence or any iterable object; **the result is always a list**.

<a name=built_in_reduce></a>
## Built-in Function reduce ##

> Apply function of two arguments cumulatively to the items of iterable, from left to right, so as to reduce the iterable to a single value

Reduce 的 Pseudo Code, 類似下面這個範例:

    def reduce(function, iterable, initializer=None):
        it = iter(iterable)
        
        if initializer is None:
        try:
            initializer = next(it)
        except StopIteration:
            raise TypeError('reduce() of empty sequence with no initial value')
        
        accum_value = initializer
        for x in it:
            accum_value = function(accum_value, x)
        return accum_value

Ex:  

* reduce(lambda x, y: x+y, [1, 2, 3, 4, 5]) calculates ((((1+2)+3)+4)+5)

<a name=built_in_filter></a>
## Built-in Function filter ##

> Construct a list from those elements of iterable for which function returns true. iterable may be either a sequence, a container which supports iteration, or an iterator. If iterable is a string or a tuple, the result also has that type; otherwise it is always a list. I**f function is None, the identity function is assumed**, that is, all elements of iterable that are false are removed.

Ex:  

* **filter(function, iterable)** is equivalent to **[item for item in iterable if function(item)]**

<a name=built_in_zip></a>
## Built-in Function zip ##

zip function 做的事情, 如下圖:

![](http://MutsuGhost1.github.io/image/python_zip_function.PNG)

Ex:
    
    >>> a = [1,2,3]
    >>> b = [4,5,6]
    >>> c = [4,5,6,7,8]      # 以短的為主
    >>> zipped = zip(a,b)
    [(1, 4), (2, 5), (3, 6)]
    >>> zip(a,c)
    [(1, 4), (2, 5), (3, 6)]
    >>> zip(*zipped)         # unzip 回原來模樣
    [(1, 2, 3), (4, 5, 6)]