title: Python OOP Exception
date: 2013-09-27 14:55:04
categories: Software
tags: [Python, Study Group]
---
* Python OOP
* Constructor/Destructor
* Class/Instance Members
* Access Control in Python OOP
* Inheritance
* Polymorphism
* Operator Overloading
* Function Overloading
* Object Types in Python
* Check relationship between class and instance

<!--more-->
----------

# Python OOP #

Python 語言, 本身也支援 **O**bject **O**riented **P**rogramming 這樣的 Programming Pradigm.  
語法上, 是採取類似 C++ 與 Modula-3 的語法.  

一個最簡單的 class definition 如下：
    
    class oldStyle:
        pass

語法上, class definition 可以寫成:

    class DerivedClassName(BaseClass1[,BaseClass2[,…]]):
        statement1
        …
        statementN

其中的 statement1 ... statementN 可以是:

* Data memeber definition
* Member function definition
* Memeber class definition

當 import module 時, python interpret 到一個 class definition 時,  
它會將 class 內相關的 definition 都定義在此 class 所建立的 namespace 內.  

**New Style Class v.s. Old Style Class**  

  * Python 2.2 之前, type 和 class 是不同的
      * type (list, tuple, dict ... etc)
      * class
  * Python 2.2 Unifying type and class
      * The obvious change is built-in types can be used to as base class
      * 沒有明確繼承其他 class 的 class definition, 稱為 Old Style Class
      * Python 3 後, 沒有明確繼承 class 將會視為 default 繼承 object
  * Reference: [http://www.python.org/download/releases/2.2.3/descrintro/](http://www.python.org/download/releases/2.2.3/descrintro/ "說明 Python 2.2 後, 整合 type and class 的文章")  

Ex:  

    # define a old style class in python 2.x
    class oldStyleClass:
    pass
    # define a new style class in python 2.x
    class newStyleClass (object):
    pass
    
----------

# Constructor/Destructor #

先前提到, 定一個最簡單的 new style class, 如下:

    #define a simplest class named "Employee"
    class Employee (object):
        pass

假設我們要新增一個功能, 來替 Employee 來紀錄, 目前 instance of Employee 的有效個數.    
可以透過新增 Constructor & Destructor 加上一個 class variable 來實現, 具體如下:

    class Employee (object):
        empCount = 0

        def __init__(self):
            Employee.empCount += 1

        def __del__(self):
            Employee.empCount -= 1

也許有人會問, 在有 GC 機制的語言中, Destructor 存在的目的是?  
通常 Destructor 的存在, 就是讓 Object 能提早釋放占用的資源.   

----------

# Class/Instance Memebers #

接下來, 我們想替 Class 加上額外的 Memebers, 包含了:

* Instance Variable
    * 用來表示 instance of Employee 的姓名, 命名為 name, 預設為 "anonymous" 
    * 用來記錄各個 instance of Employee 的基本薪資, 命名為 base_salary, 預設為 1000
* Static Method
    * 定義一個 static method 能夠印出目前 # of instance of Employee, 命名為 EmployeeCount 
* Instance Method
    * 定義一個 instance method, 能夠取得目前 instance of Employee 的總薪資, 命名為 getSalary 

具體代碼如下:

    class Employee (object):
        empCount = 0

        def __init__(self, name="anonymous", base_salary=1000):
            Employee.empCount = 0
            self.name = name
            self.base_salary = base_salary

        def __del__(self):
            Employee.empCount = 0

        @staticmethod
        def EmployeeCount():
            print "Employee Count:" + str(Employee.empCount)

        def getSalary(self):
            return self.base_salary

從上列代碼, 可以歸納出以下幾件事:

* 定義在 class 內部, 非 method 內部的變數, 視為 class variable
* 定義在 class 內部的 method, 預設都是 instance method, 第一個參數都是 reference to the instance, 習慣上命名為 self
* 通常 instance variable 都直接定義在 class 定義裡, instance method body 中, 習慣上都是以 self. 當作 prefix  
* 在 class 內部定義 static method, 請在 method 的宣告前面使用 @staticmethod, 來告訴 python interpreter 這個 method 是一個 static method, 不帶 self 參數


接下來將展示此 class 將如何被使用:  

    class Employee (object):
        empCount = 0

        def __init__(self, name="anonymous", base_salary=1000):
            Employee.empCount = 0
            self.name = name
            self.base_salary = base_salary

        def __del__(self):
            Employee.empCount = 0

        @staticmethod
        def EmployeeCount():
            print "Employee Count:" + str(Employee.empCount)

        def getSalary(self):
            return self.base_salary

        # the method will be called when call str(new Employee())
        def __str__(self):
            return "Employee name:" + self.name + " base_salary:" + self.base_salary

    def main():
       employee1 = Employee("Peter")
       print employee1
       print “Salary=” + str(employee1.getSalary())
       Employee.EmployeeCount()
       del employee1
       Employee.EmployeeCount()

    if "__main__" == __name__:
        main()

Output:

    Employee: name=Peter base_salary=1000
    Salary=1000
    EmployeeCount: Employee.empCount = 1
    EmployeeCount: Employee.empCount = 0

----------

# Access Control in Python OOP #

在 Pyhon 中, 並沒有提供 Modifier 來明確的來控制 Attribute 的存取權限.  
相反的, Python 是透過一種 Naming Convention 來告訴 User, 這個 Attribute 是 Private 的.  
雖然如此, User 想要強制存取此 Private Attribute, 還是可以達成的.

Ex:

    class Employee (object):
       __empCount = 0
    
    def main():
        # the name of attribute is changed as "_Employee" + "__empCount"
        print Employee._Employee__empCount

    if "__main__" == __name__:
        main()

Output:

    0

* Python 中, 每個 Attribute 都是 Public 的
* 以 "__" 開頭的 Attribute, 會被做 Name Mangling, 改名為 "_" + ClassName + AttributeName
* 以 "__" 開頭的 Attribute, 只是在習慣上被視為 Private 的, 但是仍然可強制存取 
    
----------

# Inheritance #

如果我們想要加入一個特別的 Employee, 稱為 Manager, 可以考慮用繼承的方式達成. 
Manager 除了本身是一個 (is a) Emplyoee 外, 還必須滿足:

* Manager 繼承了 Employee 的所有 attributes
* Manager 多了 bonus 的 attribute
* Manager 的 getSalary 結果, 必須是 base_salary + bonus 的結果


Ex: **invoke the method in the super class**

    # demo how to invoke the method in the super class

    class Manager (Employee):
         
        def __init__(self, name="anonymous", base_salary=1200, bonus = 200):
            super(Manager, self).__init__(name, base_salary)
            self.bonus = bonus

        def __del__(self):
            super(Manager, self).__del__()


Ex: **Override the method in the super class**

    class Manager (Employee):
         
        def __init__(self, name="anonymous", base_salary=1200, bonus = 200):
            super(Manager, self).__init__(name, base_salary)
            self.bonus = bonus

        def __del__(self):
            super(Manager, self).__del__()

        def getSalary(self):
            return super(Manager, self).getSalary() + self.bonus

        def __str__(self):
            return "Manager name:" + self.name + " base_salary:" + str(self.base_salary) + \
                   " bonus:" + str(self.bonus)

    def main():
       manager1 = Manager("Peter")
       print manager1
       print “Salary=” + str(manager1.getSalary())
       del manager1

    if "__main__" == __name__:
        main()

Output:

    Manager: name=John base_salary=1200 bonus=200
    Salary=1400

----------

# Polymorphism #

由於 Python 是一個 Weakly Type 的語言, 因此在變數的宣告時, 並不會明確指出特定的型別.  
因此, 只要此變數所指向的 Object, 符合操作所需要的各項 Attribute, 則 Program 即可正常運作.  

Ex:

    def main():
        list_employees = [Manager(“John”), Employee("Peter")]
        for employee in list_employees:
            print employee.__str__()

    if __name__ == '__main__':
        main()
    
Output:
    
    Manager: name=John base_salary=1200 bonus=200
    Employee: name=Peter base_salary=1000


**事實上, Python 這樣的做法提供了更彈性的使用方式. 在撰寫特定演算法的時候,  
只需要考慮 Object 本身能提供哪些 Attributes, 不需要考慮 Object 的繼承體系.**

----------

# Operator Overloading #

Python 本身定義了一組 Built-in Operators, 讓使用者可以 Overloading 其行為.

Ex:

    class Vector (object):
       def __init__(self, x=0, y=0):
           self.x = x
           self.y = y
       def __add__(self, rhs):
           return Vector(self.x + rhs.x, self.y + rhs.y)
       def __str__(self):
           return "Vector: (" + str(self.x) + ", " + str(self.y) + ")"

       class Point (object):
       def __init__(self, x=0, y=0):
           self.x = x
           self.y = y
       def __add__(self, rhs):
           return Point(self.x + rhs.x, self.y + rhs.y)
       def __str__(self):
           return "Point: (" + str(self.x) + ", " + str(self.y) + ")"
    
    def main():
       vec1 = Vector(10, 20)
       vec2 = Vector(90, 80)
       print vec1 + vec2

       p1 = Point(3,   10)
       p2 = Point(-3, -10)
       print p1 + p2
    
       # the evaluation order is ((vec1 + p1) + p2)
       print vec1 + p1 + p2
       # the evaluation order is ((p2 + vec1) + vec2)
       print p2 + vec1 + vec2
    
Output:

    Vector: (100, 100)
    Point: (0, 0)
    Vector: (10, 20)
    Point: (97, 90)

如同之前提到的, Python 中, 只要 Object 有能力 (有 Attributes) 就可以正常執行.  
上面這個例子顯示, 既使是 vec1 + p1 或 p2 + vec1, 這樣的操作都是合法的.  
因為 Python 只知道, 把 vec1 + p1 換成 vec1.__add__(p1), 只要 p1 提供足夠的能力, 能完成 __add__ 既可.  
此外, Python 中 Operator 的 Evaluation Order 都是從左到右, 不能被改變.

Reference:

* [http://docs.python.org/2/library/operator.html](http://docs.python.org/2/library/operator.html "Operator Overloading")
* [http://docs.python.org/2/reference/expressions.html#evaluation-order](http://docs.python.org/2/reference/expressions.html#evaluation-order "Operator Evaluation Order")

----------

# Function Overloading #

Python 中, 並沒有提供 Function Overloading 的機制. 一般而言, 會透過下列方式來達成:  

* Default Arguments
* Variable Length Arguments With List
* Variable Length Arguments With Dict 

Ex: **Default Arguments**

    class Employee (object):
       empCount = 0

       def __init__(self, name="", base_salary=1000):
           self.base_salary = base_salary
           self.name = name
           Employee.empCount += 1

       def __del__(self):
           Employee.empCount -= 1

       @staticmethod
       def EmployeeCount():
           print "Employee.empCount = " + str(Employee.empCount)

       def getSalary(self):
           return self.base_salary

       def __str__(self):
           return "Employee: name=" + self.name + \
                  " base_salary=" + str(self.base_salary)
    
    def main():
       employee1 = Employee("Peter")
       print employee1
       print “Salary=” + str(employee1.getSalary())
       Employee.EmployeeCount()
       del employee1
       Employee.EmployeeCount()

    if __name__ == '__main__':
        main()

Output:

    Employee: name=Peter base_salary=1000
    Salary=1000
    EmployeeCount: Employee.empCount = 1
    EmployeeCount: Employee.empCount = 0


Ex: **Variable Length Arguments With Tuple**
    
    class Employee (object):
       empCount = 0
    
       def __init__(self, *args):
           self.name  = “”
           self.salary = 1000
           if 1 <= len(args): self.name  = args[0]
           if 2 <= len(args): self.salary = args[1]
           Employee.__empCount += 1
    
       def __del__(self):
           Employee.empCount -= 1
    
       @staticmethod
       def EmployeeCount():
           print "Employee.empCount = " + str(Employee.empCount)
    
       def getSalary(self):
           return self.base_salary
    
       def __str__(self):
           return "Employee: name=" + self.name + \
                  " base_salary=" + str(self.base_salary)
    
    def main():
       employee1 = Employee("Peter")
       print employee1
       print “Salary=” + str(employee1.getSalary())
       Employee.EmployeeCount()
       del employee1
       Employee.EmployeeCount()

    if __name__ == '__main__':
       main()
    
Output:

    Employee: name=Peter base_salary=1000
    Salary=1000
    EmployeeCount: Employee.empCount = 1
    EmployeeCount: Employee.empCount = 0


Ex: **Variable Length Arguments With Dict**

    class Employee (object):
       empCount = 0
    
       def __init__(self, **args):
           self.name  = args.get(“name”, “”)
           self.salary = args.get(“salary”, 1000)
           Employee.__empCount += 1
    
       def __del__(self):
           Employee.empCount -= 1
    
       @staticmethod
       def EmployeeCount():
           print "Employee.empCount = " + str(Employee.empCount)
    
       def getSalary(self):
           return self.base_salary
    
       def __str__(self):
           return "Employee: name=" + self.name + \
                  " base_salary=" + str(self.base_salary)

    def main():
       employee1 = Employee("Peter")
       print employee1
       print “Salary=” + str(employee1.getSalary())
       Employee.EmployeeCount()
       del employee1
       Employee.EmployeeCount()
    
    if __name__ == '__main__':
       main()

Output:

    Employee: name=Peter base_salary=1000
    Salary=1000
    EmployeeCount: Employee.empCount = 1
    EmployeeCount: Employee.empCount = 0


此外, 要注意的一點是, 如果你強制寫了兩個相同 Signature 的 Method, 後者會覆蓋前者.  

Ex:


Output:

----------

# Object Types in Python #



在 Python 中, 所有物件的型別可以分為下列 3 種:

1. Class Object  
 * Instantiation
 * Attribute Reference
2. Instance Object
 * Attribute Reference
3. Method Object
 * 

----------

# Check relationship between class and instance #
