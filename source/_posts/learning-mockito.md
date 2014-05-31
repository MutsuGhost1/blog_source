title: Mockito Basic
date: 2014-05-18 08:48:44
tags: Software Testing
---

* ## Learning Mockito with examples ##
    1.  **[Verify some behavior (ignore any return value)](#1)**
    2.  **[Verify some behavior with stubbing the return value](#2)**
    3.  **[Verify unstubbing behavior](#3)**
    4.  **[Argument matcher](#4)**
    5.  **[Verify exact number of invocations](#5)**
    6.  **[Verification in order](#6)**
    7.  **[Finding redundant invocations](#7)**
    8.  **[Stubbing consecutive calls](#8)**
    9.  **[Stubbing with callbacks](#9)**
    10. **[doReturn()|doThrow()|doAnswer()|doNoting()|doCallRealMethod() family of methods](#10)**
    11. **[reset mock](#11)**
    12. **[Capturing Argument](#12)**
    13. **[Changing default return values of unstubbed invocations](#13)**
    14. **[Inject mock/spy object into tested target](#14)**
    15. **[Verify with timeout](#15)**

<!--more-->
----------
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

----------
<a name=1></a>
### verify somw behavior (ignore any return value) ###

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

----------
<a name=2></a>
### verify some behavior with stubbing the return value ###

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

**Output:**
    
    test02:firstfirst
    test02:firstfirst

----------
<a name=3></a>
### verify unstubbing behavior ###

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

**Output:**

    test03:null

----------
<a name=4></a>
### argument matching using default matcher ###

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

----------
### argument matching using customized matcher ###

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

----------
<a name=5></a>
### verify exact number of invocation ###

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


----------
<a name=6></a>
### verification in order, using single mock ###

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

----------
### verification in order, using multiple mock ###

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

----------
<a name=7></a>
### finding redundant invocations ###

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

----------
<a name=8></a>
### stubbing consecutive calls (iterator-style stubbing) ###

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

----------
<a name=9></a>
### stubbing with callbacks ###

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

----------
<a name=10></a>
### doReturn using mock ###

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

----------
### doReturn using spy ###

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

----------
### doThrow ###

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

----------
### doAnswer ###

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

**Output:**

    test15: answer callback
    test15: answer callback

----------
### doNothing ###

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

----------
### doRealCall ###

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

----------
<a name=11></a>
### reset mock ###

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


----------
<a name=12></a>
### capturing arguments##

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

**Output:**

    test19:[Hello World, two, three]
    test19:three

----------
<a name=13></a>
### change the default value of unstubbed invocations (using RETURNS\_SMART\_NULLS)###

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

----------
### change the default value of unstubbed invocations (using CALLS\_REAL\_METHODS)###

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

**Output:**

    test24: mock.m1()=PartialMockObject:m1
    test24: mock.m2()=PartialMockObject:m3
    test24: mock.m3()=m3 is stubbed
    test24: spy.m1()=PartialMockObject:m1
    test24: spy.m2()=PartialMockObject:m3
    test24: spy.m3()=m3 is stubbed

----------
<a name=14></a>
### inject mock/spy into tested target (using constructor) ###

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

----------
### inject mock/spy into tested target (using setter) ###

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

----------
### inject mock/spy into tested target (using field) ###

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

----------
<a name=15></a>
### verify with timeout ###

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


----------
    
* [Reference]
    * [1] [Mockito Official Website](http://code.google.com/p/mockito/)
    * [2] [30天快速上手TDD](http://msdn.microsoft.com/zh-tw/library/dn167673.aspx)