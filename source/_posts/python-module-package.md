title: Python Module Package
date: 2013-09-27 14:54:50
categories: Software
tags: [Python, Study Group]
---

* [What's a python module ?](#what_module "What's a python module ?")
* [Why use modules ?](#why_module "Why use modules ?")
* [How to use/write modules ?](#how_module "How to use/write modules ?")
* [Module search Path ?](#module_search_path "Module search Path")
* [Reload your module after applying any change ?](#reload_module "Reload your module after applying any change") 
* [What's package ?](#what_package "What's package ?")
* [Why use packaes ?](#why_package "Why use packaes ?")
* [How to use packages ?](#how_package "How to use packages ?")
* [An example to refactor a module as a package](#package_example "An example to refactor a module as a package")

<!--more-->
- - -
<a name=what_module></a>
# What's a python module ? #

在 Python 中, 每一個檔案都是一個 Python Module, 通常以 py 做為其副檔名
  
> 定義上, python module 個包含了 python definitions 以及 python executable statements.  
> python definitions 通常是指 variable definitions, function definitions and class definitions.  

Python module 只會在第一次被 import 的時候被執行. 被 import module 會在目前執行的 global namespace 中,  
建立一個屬於此 module 的 namespace. 在預設下, 這個 module 的 namespace 會是 module name.  
任何人可以透過這個 module name 來存取 module 內的各項 attributes.  

下面的例子即為一個 Module:

    # A python module fibo.py
    
    __VERSION = "Version 1"  # variable definition
    
    def fibo(n):             # function definition
        pass

    # you can define your class here

    if "__main__" == __name__:
        pass                 # write executable statements
  
- - -
<a name=why_module></a>
# Why use python modules ?#
 
將常用的 code 整理成 module 的好處如下:

* 將相關的 code 組織在同一個 module 中, 讓 code 更容易維護, 更容易讀, 更容易 re-use
* 避免 naming conflict, 所有定義在 module 內的變數, 都屬於此 module namespace 下
 
- - -
<a name=how_module></a>
# How to use/write python modules ?#

**Syntax for using a module**

> **import** module1 [,module2[,...moduleN]]  
> **import** module [as alias]  
> **from** module **import** item1[,item2[,...itemN]]  
> **from** module **import** item [as alias]  

**一個計算 fibonacci sequence 的 python module 如下:**

    # module fibo.py
    def fib(n):
        """ show fibonacci sequence up to n """
        a, b = 0, 1
        while b < n:
            print b,
            a,b = b, a+b
    
    def fib2(n):
        """ return fibonacci sequence up to n """
        listFibo = []
        while b < n:
            listFibo.append(b)
            a,b = b, a+b
        return listFibo

**啟動 Python Interactive Console, 並切換到 module fibo.py 所在目錄下, 並執行下列指令使用 fibo module:**  

    >>> dir()
    ['__builtins__', '__doc__', '__name__', '__package__']
    >>> import fibo
    >>> dir()
    ['__builtins__', '__doc__', '__name__', '__package__', 'fibo']
    >>> fibo.fib(1000)
    1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987
    >>> fibo.fib2(100)
    [1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89]
    
**執行 import statement 時, 實際上執行了 3 件事情:**

1. Find the file of imported module (從 sys.path 去找)  

2. Compile it as byte code (if needed)  

3. Run the module's code to build the objects it defines.  

 * A variable will be created in global namespace to reference the object of the module  

**利用 from ... import ... 來簡化使用 module 內的 atrribute name**

透過 from module import item 來簡化使用 module 內的 attribute name

    >>> dir()
    ['__builtins__', '__doc__', '__name__', '__package__']
    >>> from fibo import fib, fib2
    >>> dir()
    ['__builtins__', '__doc__', '__name__', '__package__', 'fib', 'fib2']
    >>> fib(1000)
    1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987
    >>> fib2(100)
    [1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89]


**事實上, from module import item 是基於 import statement 來實現的**

**from fibo import fib, fib2** 可以看拆解成下列幾個動作:  

1. import fibo  
2. fib = fibo.fib  
3. fib2 = fibo.fib2  
4. del fibo

**每個 module 都會有一個 \_\_name\_\_ attribute, 代表這個 module 的名稱**

    >>> import fibo
    >>> dir(fibo)
    ['__builtins__', '__doc__', '__file__', '__name__', '__package__', 'fib', 'fib2', 'main']
    >>> fibo.__name__
    'fibo'

當這個 python module 被當作 script 執行時, \_\_name\_\_ 的內容將被改為 "\_\_main\_\_"  
這樣的做法可以讓每個 module 將特定的代碼只有在 module 被當作 script 執行時, 才會執行

    def fib(n):
        a,b = 0,1
        while b < n:
            print b,
            a,b = b, a+b
    
    def fib2(n):
        result = []
        a,b = 0,1
        while b < n:
            result.append(b)
            a, b = b, a+b
        return result
    
    
    def main():
        fib(1000)
        print "\n", fib2(100)
    
    if __name__ == '__main__':
        main()

在 Python Interactive Console 下執行:

    E:\Temp\Python\hw3\example>python.exe fibo.py
    1 1 2 3 5 8 13 21 34 55 89 144 233 377 610 987
    [1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89]

此時 __ name __ 將等於 "__ main __", 會跑到 main() function 去執行

**from module import \*** statement

如果 module 中無定義 \_\_all\_\_ 變數, 則將會 copy 此 module 中所有 attributes (不含\_\_開頭之 attributes)  
否則, 只會 copy \_\_all\_\_ 變數中所指定之 attributes

Exampe: (imported module 中無定義 \_\_all\_\_ 變數)

    # module fibo.py

    def fib(n):
        a,b = 0,1
        while b < n:
            print b,
            a,b = b, a+b
    
    def fib2(n):
        result = []
        a,b = 0,1
        while b < n:
            result.append(b)
            a, b = b, a+b
        return result

Output:

    >>> dir()
    ['__builtins__', '__doc__', '__name__', '__package__']
    >>> import fibo
    >>> dir()
    ['__builtins__', '__doc__', '__name__', '__package__', 'fibo']
    >>> dir(fibo)
    ['__builtins__', '__doc__', '__file__', '__name__', '__package__', 'fib', 'fib2']
    >>> del fibo
    >>> dir()
    ['__builtins__', '__doc__', '__name__', '__package__']    
    >>> from fibo import *
    >>> dir()
    ['__builtins__', '__doc__', '__name__', '__package__', 'fib', 'fib2']

Exampe: (imported module 中有定義 \_\_all\_\_ 變數)

    # module fibo.py

    __all__ = ["fib2"]

    def fib(n):
        a,b = 0,1
        while b < n:
            print b,
            a,b = b, a+b
    
    def fib2(n):
        result = []
        a,b = 0,1
        while b < n:
            result.append(b)
            a, b = b, a+b
        return result

Output:

    >>> dir()
    ['__builtins__', '__doc__', '__name__', '__package__']
    >>> from fibo import *
    >>> dir()
    ['__builtins__', '__doc__', '__name__', '__package__', 'fib2']
    
- - -
<a name=module_search_path></a>
# Module Search Path #

當你寫好自己的 Module 後, 想當成 Library 可以提供其他 Python Module 來使用時,  
必須將 Module 放到其 Search Path 中, 才能讓 Module 被正常 import 且使用.  
一般來說, Module Search Path 是以 sys.path 來表示, 由下列成員所組成:

1. Directory containing the script (目前執行之 Script 所在的目錄)
2. PYTHONPATH (系統中的環境變數, User 可將自訂的 Module Location 填入此環境變數中)
3. Installation Default Dependent (與平台系統相關的路徑, 安裝 Python 時, 將會自動決定)

當被 import 的 module 無法在 sys.path 中被搜尋到, 將會產生 ImportError Exception.  
當前執行的 script 目錄中, 若有與 Standard Library 中的 module 有相同名稱, 則會遮蔽 Standard Library 的 Module.

- - -
<a name=reload_module></a>
# Reload Your Module To Apply Any Modification #

請記得, 每個 python interpreter session 中, 只會 import 一次 module, 因此當 module 變動時, 請執行  

* reload(module)

![](/images/python_reload_function.PNG)

所有 reference 到此 module 的變數, 都將受影響

- - -
<a name=what_package></a>
# What's package ? #

Package 由一個 Directory 所構成, 裡面包含了 Python Module 檔案, 以及一個名稱為 **\_\_ini\_\_.py** 的檔案.  
透過 **\_\_ini\_\_.py** 檔案, Python Interpreter 才會將此 Directory 視為一個 Package.  

![](/images/python_package.PNG)

上圖既為一個簡單的 Package, 透過 import A.B 則可 import package A 中的 module B.

- - -
<a name=why_package></a>
# Why use package ? #

使用 Package 來組織 Python Module 的好處在於: 

* 可以降低 Module 間的 Coupling, 增加 Module 內的 Cohesion  
* 使得 Module 更易於 Maintain
* 讓 Module 更 readable  

- - -
<a name=how_package></a>
# How to use packages #

透過下列幾種方法, 可以 import package 或 package 內的 module:

* import package.module [as alias]
    * 將執行 package 內的 \_\_init\_\_.py 檔案, 做為 package 的初始化  
    * 建立一個新的 namespace for this module of package
    * 透過 package.module.item (or alias.item) 來存取此 module 中的各項 attributes 
* import package
    * 僅執行 package 內的 \_\_init\_\_.py 檔案, 做為 package 的初始化
* from package import module [as alias] 
    * 將執行 package 內的 \_\_init\_\_.py 檔案, 做為 package 的初始化
    * 從 package 中, copy 出此 module
    * 透過 module.item (or alias.item) 來存取此 module 中的各項 attributes  

假設目前有一個 sound package 其結構如下:

    + sound/                    # top level package
        __init__.py             # initialize the sound package
      + formats/                # sub-package for file format converstions
          __init__.py
          wavread.py
          wavwrite.py
          aifread.py
          aifwrite.py
          auread.py
          auwrite.py
          ...
      + effects/                # sub-package for sound effects
          __init__.py
          echo.py
          surround.py
          reverse.py
          ...
      + filters/                # sub-package for filters
          __init__.py
          equalizer.py
          vocoder.py
          karaoke.py
          ...

使用者透過 import package.module, 可以 import package 中的某個 module,  
Ex:

    import sound.effects.echo 
    sound.effects.echo.echofilter(input, output, delay = 0.7, atten = 4)
    

透過 from package import module, 使用者可以從特定 package 中, copy 出某個 module,  
Ex:

    from sound.effects
    import echo echo.echofilter(input, output, delay = 0.7, atten = 4)
    
透過 from package.module import item, 使用者可以從特定 package 中的 module, copy 某個 attribute,  
Ex:
    
    from sound.effects.echo 
    import echofilter echofilter(input, output, delay = 0.7, atten = 4)


**from package import \*** statement

在實際的應用中, 使用者可能會想要一次將 package 中的 module 都 import 進 namespace 中.  
但事實上, 這樣的做法會因為 import 某些使用者不預期的 module 而產生一些使用不預期的行為.  
正因為如此, 針對 package 的 import \* 就必須遵守下列的規則:

  * 若是 package 中的 \_\_init\_\_.py 檔案內有定義 \_\_all\_\_ 變數, 則會依據 \_\_all\_\_變數中的內容,  
  來決定要 import 哪些 sub-modules
  * 否則當\_\_init\_\_.py 檔案內沒有定義 \_\_all\_\_ 變數, 則只會執行 \_\_init\_\_.py 內的敘述

**intra package reference**  

* import module in the same package  
  The surround module, it can simply use:

Ex:  

    # surround.py
    import echo
    from echo import echofilter
    
* import module in the different package  
  Use absolute import to refer sub-modules of sibling package:

Ex:

    #vocoder.py
    from sound.effects import echo

- - -
<a name=package_example></a>
# An example to refactor a module as a package #

假設目前有一個 module phone.py, 裡面包含了 utility for Pots phone, Isdn phone and G3 phone.  
如下:

    # Phone.py
    def  Pots():
        print “I’m Pots Phone”
    
    def Isdn():
        print “I’m Isdn Phone”
    
    Def G3():
        print “I’m G3 Phone”
     
事實上, 我們可以將它規劃成 package 的形式,  
如下：  

    + Phone
        \_\_init\_\_.py
        Pots.py
        Isdn.py
        G3.py

透過在 \_\_init\_\_.py　中, 將 sub-module 中的 utility 都 copy 出來,  
如下:

    # file: __init__.py
    from Pots import Pots    # 從 Pots 這個 sub-module 中, copy Pots 這個 attribute
    from Isdn import Isdn    # 從 Isdn 這個 sub-module 中, copy Isdn 這個 attribute
    from G3 import G3        # 從 G3 這個 sub-module 中, copy G3 這個 attribute

用戶端的代碼,  
如下:

    # test.py
    import Phone
    
    Phone.Pots()
    Phone.Isdn()
    Phone.G3()

Output:

    I’m Pots Phone
    I’m Isdn Phone
    I’m G3 Phone


這樣的設計, 不僅能夠讓使用者更容易理解這些 module, 也能降低 module maintain 上的 efforts
