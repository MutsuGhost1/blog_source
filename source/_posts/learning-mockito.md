title: Mockito Basic
date: 2014-05-18 08:48:44
categories: Software
tags: [Software Testing, Mock]
---
<a name=table></a>
* ## Basic Concepts ##
    * 撰寫 Unit Test 的 3A 原則:
        1. Arrange: 
            * Set up th object to be tested.
            * We may need to surround the object with collaborators. <br> For testing purposes, those collaborators minght be test objects (mocks, fakes, etc...) or the real ting.
        2. Act:
            * Act on the object. (invoke the tested method with parameters.)
        3. Assert:
            * Make claims about the object, its collaborators, its parameters and possibly global state.
            * In other words, verify the test result.
<!--more-->
- - -
<a name=1></a>
## Verify somw behavior (ignore any return value)
* 如果被依賴的 method, 不需要特定的 return value, 或是沒有 return value, 則直接使用 default stubbing 的行為.
* 使用 verify + times 來驗證被依賴的 method 是否有被呼叫到(包含呼叫時參數為何 ... etc), 且呼叫了幾次. 

{% codeblock %}
@SuppressWarnings("unchecked")
public void test01(){

    // Arrange
    // Create the mock object, even it's an interface
    List<String> mockedList = (List<String>)mock(List.class);

    // Act
    // Action on the object
    mockedList.add("one");
    mockedList.clear();
    
    // Assert
    // Verify the result
    verify(mockedList).add("one");
    verify(mockedList).clear();
    
    // Act
    // Action on the object
    mockedList.add("one");
    mockedList.clear();
    
    // Assert
    // Verify the result.
    // The interactions should be accumulated unless the mock object is reset
    verify(mockedList, times(2)).add("one");
    verify(mockedList, times(2)).clear();
}
{% endcodeblock %}
[回目錄](#table)
- - -
<a name=2></a>
## Verify some behavior with stubbing the return value
* 如果被依賴的 method, 需要特定的 return value, 則必須改變 default stubbing 的行為
* 透過 when(...).thenReturn(...) 來改變 default stubbing 的行為

{% codeblock %}
@SuppressWarnings("unchecked")
public void test02(){
    // Arrange
    // Create the mock object, it can be a concrete class
    LinkedList<String> mockedList = (LinkedList<String>) mock(LinkedList.class);

    // Act
    // Stubbing it.
    // Make it return what you want
    when(mockedList.get(0)).thenReturn("first");
    // The latter stubbing will override the former one
    when(mockedList.get(0)).thenReturn("firstfirst");
    System.out.println("test02:" + mockedList.get(0));
    System.out.println("test02:" + mockedList.get(0));

    // Assert
    verify(mockedList, times(2)).get(0);
}
{% endcodeblock %}
**Output:**
    
    test02:firstfirst
    test02:firstfirst

[回目錄](#table)
- - -
<a name=3></a>
## Verify unstubbing behavior
* Default stubbing 的行為, 回傳型別 Object 對應到 null, primitive type 的 int 對應到 0 ... etc

{% codeblock %}
@SuppressWarnings("unchecked")
public void test03(){
    // Arrange
    // Create the mock object, it can be a concrete class
    LinkedList<String> mockedList = (LinkedList<String>) mock(LinkedList.class);

    // Act
    // The default return value for object is null.
    // For primitive type, the default value for int is 0.
    //                   , the default value for boolean is false.
    System.out.println("test03:" + mockedList.get(0));

    // Assert
    // times(1) is the default value, if you don't specify it
    verify(mockedList, times(1)).get(0);
}
{% endcodeblock %}
**Output:**

    test03:null

[回目錄](#table)
- - -
<a name=4></a>
## Argument matching using default matcher  
* 在替換 default stubbing 行為時, 針對參數的部份, 可利用 argument matching 的方式, 來作更大範圍的 matching
* 在驗證被依賴 method 被呼叫幾次時, 針對參數的部份, 可利用 argument matching 的方式, 來作更大範圍的 matching
* anyInt() 是一個系統已經寫好的 argument matcher, 表示參數為任何 int

{% codeblock %}
// Argument Matching Using Default Matcher
@SuppressWarnings("unchecked")
public void test04(){
    // Arrange
    // Create the mock object, even it's an interface
    List<String> mockedList = (List<String>) mock(List.class);
    when(mockedList.get(anyInt())).thenReturn("stubbing string");
    
    // Act
    for(int i=0 ; i<10; i++){
        System.out.println("["+ i + "]:" + mockedList.get(i));
    }

    // Assert
    verify(mockedList, times(10)).get(anyInt());
}
{% endcodeblock %}
**Output:**
    
    [0]:stubbing string
    [1]:stubbing string
    [2]:stubbing string
    [3]:stubbing string
    [4]:stubbing string
    [5]:stubbing string
    [6]:stubbing string
    [7]:stubbing string
    [8]:stubbing string
    [9]:stubbing string

[回目錄](#table)
- - -
## Argument matching using customized matcher  
* 在替換 default stubbing 行為時, 針對參數的部份, 可利用 argument matching 的方式, 來作更大範圍的 matching
* 在驗證被依賴 method 被呼叫幾次時, 針對參數的部份, 可利用 argument matching 的方式, 來作更大範圍的 matching
* listOfTwoElements() 是一個使用者自己寫的 argument matcher, 表示參數必須為 size() == 2 的 List
* 只要有一個參數是使用 argument matcher, 所有參數都必須使用 argument matcher

{% codeblock %}
@SuppressWarnings("rawtypes")
private class IsListOfTwoElements extends ArgumentMatcher<List>{
    @Override
    public boolean matches(Object argument) {
        return 2 == ((List)argument).size();
    }
    
}

@SuppressWarnings("rawtypes")
private List listOfTwoElements(){
    return argThat(new IsListOfTwoElements());
}

// Argument Matching Using Custom Matcher
@SuppressWarnings("unchecked")
public void test05(){
    // Arrange
    // Create the mock object, even it's an interface
    List<String> mockedList = (List<String>) mock(List.class);
    when(mockedList.addAll(listOfTwoElements())).thenReturn(true);

    // Act
    mockedList.addAll(Arrays.asList("one", "two"));

    // Assert
    verify(mockedList).addAll(listOfTwoElements());
}
{% endcodeblock %}
[回目錄](#table)
- - -
<a name=5></a>
## Verify exact number of invocation
* 檢查被依賴的 method, 被呼叫了幾次, 至少被呼叫了幾次, 沒有被呼叫 ... 等

{% codeblock %}
@SuppressWarnings("unchecked")
public void test06(){
    // Arrange
    // Create the mock object, even it's an interface
    List<String> mockedList = (List<String>) mock(List.class);

    // Act
    mockedList.add("one");
    mockedList.add("two");
    mockedList.add("two");
    mockedList.add("three");
    mockedList.add("three");
    mockedList.add("three");

    // Assert
    verify(mockedList).add("one");
    verify(mockedList, times(1)).add("one");
    verify(mockedList, times(2)).add("two");
    verify(mockedList, times(3)).add("three");
    
    verify(mockedList, never()).add("none");
    verify(mockedList, atLeastOnce()).add("one");
    verify(mockedList, atLeast(2)).add("two");
    verify(mockedList, atMost(5)).add("three");
}
{% endcodeblock %}
[回目錄](#table)
- - -
<a name=6></a>
## Verification in order, using single mock
* 使用 inOrder 來檢視被依賴的 methods 之間被呼叫的順序
* 此例針對一個 object 的 methods 來作驗證 

{% codeblock %}
public void test07(){
    // Arrange
    // Create the mock object, even it's an interface
    List<String> singleMock = (List<String>) mock(List.class);

    // Act
    singleMock.add("one");
    singleMock.add("two");

    // Assert
    InOrder inOrder = inOrder(singleMock);
    inOrder.verify(singleMock).add("one");
    inOrder.verify(singleMock).add("two");
}
{% endcodeblock %}
[回目錄](#table)
- - -
## Verification in order, using multiple mock
* 使用 inOrder 來檢視被依賴的 methods 之間被呼叫的順序
* 此例針對多個 objects 的 methods 來作驗證 

{% codeblock %}
@SuppressWarnings("unchecked")
public void test08(){
    // Arrange
    // Create the mock object, even it's an interface
    List<String> firstMock = (List<String>) mock(List.class);
    List<String> secondMock = (List<String>) mock(List.class);

    // Act
    firstMock.add("one");
    secondMock.add("two");
    firstMock.add("three");
    secondMock.add("four");

    // Assert
    // Verification in order is flexible - you don't have to
    //   verify all interactions one-by-one but only those you're
    //   interested in testing in order
    InOrder inOrder = inOrder(firstMock, secondMock);
    inOrder.verify(firstMock).add("one");
    inOrder.verify(secondMock).add("two");
    // it still passes, even marks it as comment
    inOrder.verify(firstMock).add("three");
    inOrder.verify(secondMock).add("four");
}
{% endcodeblock %}
[回目錄](#table)
- - -
<a name=7></a>
## Finding redundant invocations
* 透過 verifyNoMoreInteraction(...) 驗證, 是否被依賴的 object 中, 所有被呼叫的 methods 都被 verify 過了

{% codeblock %}
public void test09(){
    // Arrange
    // Create the mock object, even it's an interface
    List<String> mockedList = (List<String>) mock(List.class);

    // Act
    mockedList.add("one");
    mockedList.add("two");

    // Assert
    verify(mockedList).add("one");
    verify(mockedList).add("two"); // mark it to fail
    
    verifyNoMoreInteractions(mockedList);
}
{% endcodeblock %}
[回目錄](#table)
- - -
<a name=8></a>
## Stubbing consecutive calls (iterator-style stubbing)
* 一般來說, 最後一次的 stubbing 會蓋過之前的行為
* 如果你想要連續幾次的呼叫, 回傳的值都不一樣, 可以考慮 stubbing consecutive calls

{% codeblock %}
@SuppressWarnings("unchecked")
public void test10(){
    // Arrange
    // Create the mock object, even it's an interface
    List<String> mockedList = (List<String>) mock(List.class);

    // Act
    when(mockedList.get(0)).thenReturn("0").thenReturn("1");
    // the usage above is equal to the below
    // when(mockedList.get(0)).thenReturn("0","1");

    // Assert
    assertTrue("0".equals(mockedList.get(0)));
    assertTrue("1".equals(mockedList.get(0)));
}
{% endcodeblock %}
[回目錄](#table)
- - -
<a name=9></a>
## Stubbing with callbacks
* 當呼叫到某個被依賴的 method 時, 你想要執行一段特定的代碼, 可以考慮 when(...).thenAnswer(...)
* 這種方式通常可以用來模擬 callback 行為

{% codeblock %}
// stubbing with callback
//   it can be used to simulate the response callback
public void test11(){
    // Arrange
    Util mock = mock(Util.class);

    // Act
    when(mock.asyncCall()).thenAnswer(new Answer(){
        @Override
        public Object answer(InvocationOnMock invocation) throws Throwable {
            // do the response call
            System.out.println("test11: answer callback");
            return Boolean.valueOf(true);
        }
    });

    // Assert
    assertTrue(mock.asyncCall());
    verify(mock).asyncCall();
}
{% endcodeblock %}
[回目錄](#table)
- - -
<a name=10></a>
## doReturn using mock
* 想替換掉 default stubbing, 另一種選擇方式是 doReturn(...).when(...).someMethod(...)
* 這種方式特別適合 spy, 因為 spy 預設行為是呼叫 real method, 有可能造成問題

{% codeblock %}
// doReturn using mock
public void test12(){
    // Arrange
    // Create the mock object, even it's an interface
    List<String> mockedList = (List<String>) mock(List.class);

    // Act
    // When you use mock, it's equal to when(mockedList.get(0)).thenReturn("0")
    doReturn("one").when(mockedList).get(0);
    // when(mockedList.get(0)).thenReturn("0");
    
    // Assert
    assertTrue("one".equals(mockedList.get(0)));
    verify(mockedList).get(0);
}
{% endcodeblock %}
[回目錄](#table)
- - -
## doReturn using spy  
* 想替換掉 default stubbing, 另一種選擇方式是 doReturn(...).when(...).someMethod(...)
* 這種方式特別適合 spy, 因為 spy 預設行為是呼叫 real method, 有可能造成問題

{% codeblock %}
// doReturn using spy
public void test13(){
    // Arrange
    // Create the mock object, even it's an interface
    List<String> spyList = (List<String>) spy(new LinkedList<String>());

    // Act
    // When you use spy, it's not equal to when(spyList.get(0)).thenReturn("0")
    doReturn("one").when(spyList).get(0);
    // when(spyList.get(0)).thenReturn("0");
    
    // Assert
    assertTrue("one".equals(spyList.get(0)));
    verify(spyList).get(0);
}
{% endcodeblock %}
[回目錄](#table)
- - -
## doThrow
* 讓 method 被呼叫時, 丟出特定的 Exception

{% codeblock %}
// doThrow
@SuppressWarnings("unchecked")
public void test14(){
    // Arrange
    // Create the mock object, even it's an interface
    List<String> mockedList = (List<String>) mock(List.class);
    
    // it's equal to
    // doThrow(new RuntimeException()).when(mockedList).toString();
    doThrow(RuntimeException.class).when(mockedList).toString();
    
    try{
        System.out.println(mockedList);
    }catch(RuntimeException e){
        /// pass
    }
}
{% endcodeblock %}
[回目錄](#table)
- - -
## doAnswer
* when(...).thenAnswer(...) 的另一種替代方案
* 這種方式通常可以用來模擬 callback 行為

{% codeblock %}
// rewrite case 10 using doAnswer
@SuppressWarnings("rawtypes")
public void test15(){
    // Arrange
    Util mock = mock(Util.class);

    // Act
    doAnswer(new Answer(){
        @Override
        public Object answer(InvocationOnMock invocation) throws Throwable {
            // do the response call
            System.out.println("test15: answer callback");
            return Boolean.valueOf(true);
        }
    }).when(mock).asyncCall();

    // Assert
    assertTrue(mock.asyncCall());
    verify(mock).asyncCall();
}
{% endcodeblock %}
**Output:**

    test15: answer callback
    test15: answer callback

[回目錄](#table)
- - -
## doNothing
* 使用 stubbing consecutive calls 時, 讓第一次 method call 不作任何事情

{% codeblock %}
// doNothing
// it's rarely to use doNothing, the following is an example
@SuppressWarnings("unchecked")
public void test16(){
    // Arrange
    // Create the mock object, even it's an interface
    List<String> mockedList = (List<String>) mock(List.class);
    doNothing().doThrow(new RuntimeException()).when(mockedList).clear();

    // Act
    // does nothing the first time:
    mockedList.clear();
    
    try{
        // throws RuntimeException the next time:
        mockedList.clear();
    }catch(RuntimeException e){
        
    }
}
{% endcodeblock %}
[回目錄](#table)
- - -
## doRealCall
* 替換掉 default stubbing 行為, 讓 mock object 呼叫其真正的 method
* 必須為 concrete class 

{% codeblock %}
private class Util2{
    public String A(){
        return "A";
    }

    private String B(){
        return "B";
    }
    
    public String AB(){
        return A() + B();
    }
}

// doRealCall
// mock object can also be used to do partial mock
public void test17(){
    Util2 mockUtil2 = mock(Util2.class);
    assertTrue(null == mockUtil2.A());

    when(mockUtil2.A()).thenCallRealMethod();
    assertTrue("A".equals(mockUtil2.A()));
    
    // mark any one to fail
    when(mockUtil2.B()).thenCallRealMethod();
    when(mockUtil2.AB()).thenCallRealMethod();
    assertTrue("AB".equals(mockUtil2.AB()));
}
{% endcodeblock %}
[回目錄](#table)
- - -
<a name=11></a>
## Reset mock
* 將 mock object 設置為初始狀態

{% codeblock %}
// reset mock
@SuppressWarnings("unchecked")
public void test18(){
    // Arrange
    // Create the mock object, even it's an interface
    List<String> mockedList = (List<String>) mock(List.class);
    
    // Act
    mockedList.get(0);
    reset(mockedList);

    // Assert
    verify(mockedList, never()).get(0);
}
{% endcodeblock %}
[回目錄](#table)
- - -
<a name=12></a>
## Capturing arguments
* 將 method invocation 時的參數 capture 下來, 用於後續的比對

{% codeblock %}
// capturing arguments for further assertions
@SuppressWarnings("unchecked")
public void test19(){
    // Arrange
    // Create the mock object, even it's an interface
    List<String> mockedList = (List<String>) mock(List.class);
    ArgumentCaptor<String> argument = ArgumentCaptor.forClass(String.class);
    
    // Act
    mockedList.add("Hello World");
    mockedList.add("two");
    mockedList.add("three");
    
    // Assert
    verify(mockedList, times(3)).add(argument.capture());
    
    System.out.println("test19:" + argument.getAllValues());
    // print the argument of last call
    System.out.println("test19:" + argument.getValue());
}
{% endcodeblock %}
**Output:**

    test19:[Hello World, two, three]
    test19:three

[回目錄](#table)
- - -
<a name=13></a>
## Change the default value of unstubbed invocations (using RETURNS\_SMART\_NULLS)
* 將所有 default stubbing 行為中, 回傳 null 物件的方式, 改為傳一個特殊物件, 一旦被呼叫, 將引發 SmartNullPointerException
* 可以讓 Developer 很清楚知道, 這個 NullPointerException 是因為忘了 Mock 導致的

{% codeblock %}
// change the default value of unstubbed invocations
//   use RETURNS_SMART_NULLS to know which null pointer exception
//   is caused by unstubing
@SuppressWarnings("rawtypes")
public void test20(){
    // Arrange
    // Create the mock object, even it's an interface
    List mockedList =  mock(List.class, RETURNS_SMART_NULLS);
    ListIterator iterator = mockedList.listIterator();
    
    try{
        System.out.println("test20:" + iterator.nextIndex());
    }catch(SmartNullPointerException e){
        
    }
}
{% endcodeblock %}
[回目錄](#table)
- - -
## Change the default value of unstubbed invocations (using CALLS\_REAL\_METHODS)
* 將所有 default stubbing 的行為, 都改成 real method calls
* 等於 spy ???

{% codeblock %}
public class PartialMockObject {
    public String m1(){
        return "PartialMockObject:" + "m1";
    }
    
    public String m2(){
        return p() + "m3";
    }
    
    public String m3(){
        return pp() + "m3";
    }
    
    public String p(){
        return "PartialMockObject:";
    }
    
    private String pp(){
        return p();
    }
}

// change the default value of unstubbed invocations
//   use CALLS_REAL_METHODS to simulate partial mock
public void test24(){
    // Arrange
    // Create the mock object, even it's an interface
    PartialMockObject mock =  mock(PartialMockObject.class, CALLS_REAL_METHODS);
    PartialMockObject spy = spy(new PartialMockObject());
    
    when(mock.m3()).thenReturn("m3 is stubbed");
    System.out.println("test24: mock.m1()=" + mock.m1());
    System.out.println("test24: mock.m2()=" + mock.m2());
    System.out.println("test24: mock.m3()=" + mock.m3());

    when(spy.m3()).thenReturn("m3 is stubbed");
    System.out.println("test24: spy.m1()=" + spy.m1());
    System.out.println("test24: spy.m2()=" + spy.m2());
    System.out.println("test24: spy.m3()=" + spy.m3());
}
{% endcodeblock %}
**Output:**

    test24: mock.m1()=PartialMockObject:m1
    test24: mock.m2()=PartialMockObject:m3
    test24: mock.m3()=m3 is stubbed
    test24: spy.m1()=PartialMockObject:m1
    test24: spy.m2()=PartialMockObject:m3
    test24: spy.m3()=m3 is stubbed

[回目錄](#table)
- - -
<a name=14></a>
## Inject mock/spy into tested target (using constructor)
* 針對 SUT 中, 特定的 fields 做 Dependency Injection
* 只要 Constructor 參數型別一致既可

{% codeblock %}
public class SUT {
    List<String> mList;
    
    public SUT(List<String> list){
        System.out.println("SUT's constructor");
        System.out.println("SUT's mList:" + mList);
        mList = list;
    }

    public String get(int index){
        return mList.get(index);
    }

    public boolean put(String obj){
        return mList.add(obj);
    }
    
    @Override
    public boolean equals(Object obj) {
        System.out.println("SUT'equals: (SUT)obj).mList =" + ((SUT)obj).mList);
        System.out.println("SUT'equals: mList =" + mList);
        return ((SUT)obj).mList == mList;
    }
}
{% endcodeblock %}
**Output:**

    test21: start
    SUT's constructor
    SUT's mList:null
    SUT2's constructor
    SUT3's constructor
    SUT's constructor
    SUT's mList:null
    SUT'equals: (SUT)obj).mList =mList
    SUT'equals: mList =mList
    test21: end

[回目錄](#table)
- - -
## Inject mock/spy into tested target (using setter)  
* 針對 SUT 中, 特定的 fields 做 Dependency Injection
* 只要 setter 參數型別一致既可

{% codeblock %}
public class SUT2 {
    List<String> mList2;
    
    public SUT2(){
        System.out.println("SUT2's constructor");
    }
    
    public String get(int index){
        return mList2.get(index);
    }
    
    public boolean put(String obj){
        return mList2.add(obj);
    }
    
    public void setList(List<String> list){
        System.out.println("SUT2's setList");
        mList2 = list;
    }

    @Override
    public boolean equals(Object obj) {
        System.out.println("SUT2's equals: (SUT2)obj).mList2 =" + ((SUT2)obj).mList2);
        System.out.println("SUT2's equals: mList2 =" + mList2);
        return ((SUT2)obj).mList2 == mList2;
    }
}

// dependency injection using the setter
@Mock private List<String> mList2;
@InjectMocks private SUT2 mSUT2;
public void test22(){
    // Arrange
    System.out.println("test22: start");
    MockitoAnnotations.initMocks(this);
    // Act
    // Assert
    assertNotNull(mList2);
    assertNotNull(mSUT2);
    
    SUT2 sut2 = new SUT2();
    sut2.setList(mList2);
    assertTrue(mSUT2.equals(sut2));
    System.out.println("test22: end");
}
{% endcodeblock %}
**Output:**

    test22: start
    SUT's constructor
    SUT's mList:null
    SUT2's constructor
    SUT3's constructor
    SUT2's constructor
    SUT2's setList
    SUT2's equals: (SUT2)obj).mList2 =mList2
    SUT2's equals: mList2 =mList2
    test22: end

[回目錄](#table)
- - -
## Inject mock/spy into tested target (using field)
* 針對 SUT 中, 特定的 fields 做 Dependency Injection
* 只要欄位型別/名稱一致即可

{% codeblock %}
public class SUT3 {
    List<String> mList3;
    
    public SUT3(){
        System.out.println("SUT3's constructor");
    }
    
    public String get(int index){
        return mList3.get(index);
    }
    
    public boolean put(String obj){
        return mList3.add(obj);
    }

    @Override
    public boolean equals(Object obj) {
        System.out.println("SUT3's equals: (SUT3)obj).mList3 =" + ((SUT3)obj).mList3);
        System.out.println("SUT3's equals: mList3 =" + mList3);
        return ((SUT3)obj).mList3 == mList3;
    }
}

// dependency injection using field
@Mock(name="mList3") private List<String> mLIST3;
@InjectMocks private SUT3 mSUT3;
public void test23(){
    // Arrange
    System.out.println("test23: start");
    MockitoAnnotations.initMocks(this);
    // Act
    // Assert
    assertNotNull(mLIST3);
    assertNotNull(mSUT3);
    
    SUT3 sut3 = new SUT3();
    sut3.mList3 = mLIST3;
    assertTrue(mSUT3.equals(sut3));
    System.out.println("test23: end");
}
{% endcodeblock %}
**Output:**
    
    test23: start
    SUT's constructor
    SUT's mList:null
    SUT2's constructor
    SUT3's constructor
    SUT3's constructor
    SUT3's equals: (SUT3)obj).mList3 =mList3
    SUT3's equals: mList3 =mList3
    test23: end

[回目錄](#table)
- - -
<a name=15></a>
## Verify with timeout
* 指定 verify 的 timeout 時間, 這特別適合於等待 async 的結果
* 一旦結果成立, verify 就會結束, 不會 blocking 到時間結束才驗證結果

{% codeblock %}
@SuppressWarnings("unchecked")
public void test25(){
    // Arrange
    // Create the mock object, even it's an interface
    final List<String> mockedList = (List<String>) mock(List.class);

    // Act
    // When you use mock, it's equal to when(mockedList.get(0)).thenReturn("0")
    Thread t1 = new Thread(new Runnable() {
        @Override
        public void run() {
            try{
                Thread.sleep(2500);
                // when(mockedList.get(0)).thenReturn("0");
                doReturn("one").when(mockedList).get(0);
                assertTrue("one".equals(mockedList.get(0)));
            } catch(InterruptedException e){
                
            }
        }
    });
    t1.start();

    // Assert
    long start, end, elapsed;
    start = System.currentTimeMillis();
    // if the condition is satisfied, it won't be blocked
    verify(mockedList, timeout(3000)).get(0);
    end = System.currentTimeMillis();
    elapsed = end - start;
    assertTrue(elapsed > 2000);
}
{% endcodeblock %}
[回目錄](#table)
- - -
    
* [Reference]
    * [1] [Mockito Official Website](http://code.google.com/p/mockito/)
    * [2] [30天快速上手TDD](http://msdn.microsoft.com/zh-tw/library/dn167673.aspx)
    * [3] [相關代碼](https://github.com/MutsuGhost1/Code/tree/master/mockito/Mockito)
