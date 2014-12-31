title: Mocks Aren't Stubs
date: 2014-06-03 20:45:49
categories: Software
tags: [Software Testing, Mock]
---

<a name=1></a>
## Regular Tests and Tests with mock objects

在探討 Mock 與 Stub 為何不等價之前,
我們先來比較, 使用 Real Object 測試與使用 Mock Object 測試, 兩者間的差異.
本文採用 Mockito 範例, 有別於原文使用的 JMock/EasyMock.
<!--more-->

**使用 Real Object 來做測試的 Case**

{% codeblock %}
    public void testOrderIsFilledIfEnoughInWareHouse(){
        // Arrange
        Order order = new Order(TALISKER, 50);
        Warehouse warehouse = new Warehouse();
        warehouse.add(TALISKER, 50);
        // Act
        order.fill(warehouse);
        // Assert
        assertTrue(order.isFilled());
        assertTrue(0 == warehouse.getInventory(TALISKER));
    }

    public void testOrderDoesNotRemoveIfNotEnoughInWareHouse(){
        // Arrange
        Order order = new Order(TALISKER, 51);
        Warehouse warehouse = new Warehouse();
        warehouse.add(TALISKER, 50);
        // Act
        order.fill(warehouse);
        // Assert
        assertFalse(order.isFilled());
        assertTrue(50 == warehouse.getInventory(TALISKER));
    }
{% endcodeblock %}

一般來講, 使用 Real Object 來做測試, 通常驗證正確性的方式,
都是透過 **State Verification** 完成. 有時候不見得會有這樣的 Method, 可以把 State 取出來做驗證.

**使用 Mock Object 來做測試的 Case**

{% codeblock %}
    public void testOrderIsFilledIfEnoughInWareHouseUsingMock(){
        // Arrange
        Order order = new Order(TALISKER, 50);
        Warehouse warehouse = mock(Warehouse.class);
        when(warehouse.hasInventory(TALISKER, 50)).thenReturn(true);
        // Act
        order.fill(warehouse);
        // Assert
        // User still can verify the SUT, 
        // and it also can check the call flow ... (behavior verification)
        assertTrue(order.isFilled());
        verify(warehouse).hasInventory(TALISKER, 50);
    }

    public void testOrderDoesNotRemoveIfNotEnoughInWareHouseUsingMock(){
        // Arrange
        Order order = new Order(TALISKER, 51);
        Warehouse warehouse = mock(Warehouse.class);
        when(warehouse.hasInventory(TALISKER, 51)).thenReturn(false);
        // Act
        order.fill(warehouse);
        // Assert
        // User still can verify the SUT, 
        // and it also can check the call flow ... (behavior verification)
        assertFalse(order.isFilled());
        verify(warehouse).hasInventory(TALISKER, 51);
    }
{% endcodeblock %}

一般來講, 使用 Mock Object 來做測試, 通常驗證正確性的方式,
都是透過 **Behavior Verification** 完成. 這樣的做法, 牽涉到 Object Under Test 的內部邏輯,
似乎與實作內容有點綁死.

- - -
<a name=2></a>
## The Difference Between Mocks And Stubs

*  作者將用來取代 Test Target 所相依的物件, 給了一個特定的術語, 叫做 Test Double (測試替身)
*  根據不同的特性, 又可以將 Double 區分為下列幾種:
    1. Dummy
        * objects are passed around but never actually used.
          Usually they are just used to fill parameter lists.
    2. Stubs
        * Provide **canned answers to calls made during the test**,
          usually not responding at all to anything outside what's
          programmed in for the test.
          Stubs may also record inforation about calls, such as an email
          gateway stub that remembers the messages it sent, or maybe only
          how many messages it send.
    3. Fake
        * Objects actually have working implementations,
          but usually take some shortcut which makes them
          not suitable for production (an in memory database is a good example).
    4. Mocks
        * Are what we are talking about there:
          **objects pre-programmed with expectations which form a specification
          of the calls they are exptected to receive.**<br>

以下定義一些 Stub 相關的介面與實作

{% codeblock %}
    // Common Interfaces
    public interface MailService {
        public void send(String str);
    }
    // Stubs
    public class MailServiceStub implements MailService {
        private List<Message> mMessages = new ArrayList<Message>();
        @Override
        public void send(String str) {
            mMessages.add(new Message(str));
        }
    
        public int numberSent(){
            return mMessages.size();
        }
    }
{% endcodeblock %}

以下為使用 Stub 來做測式的測項:

{% codeblock %}
    // 與 Object Under Test 相依的物件, 不見得要使用 Real Object, 也可使用 Stub
    public void testOrderSendsMailIfUnfilledUsingStub(){
        // Arrange
        Order order = new Order(TALISKER, 51);
        MailServiceStub mailer = new MailServiceStub();
        Warehouse warehouse = new Warehouse();
        warehouse.add(TALISKER, 50);
        // Act
        order.setMailer(mailer);
        order.fill(warehouse);
        // Assert
        // It may add a new method to do state verfication
        assertEquals(1, mailer.numberSent());
    }
{% endcodeblock %}

以下為使用 Mock 來做測式的測項:

{% codeblock %}
    public void testOrderSendsMailIfUnfilledUsingMock(){
        // Arrange
        Order order = new Order(TALISKER, 51);
        MailService mailer = mock(MailService.class);
        Warehouse warehouse = mock(Warehouse.class);        
        when(warehouse.hasInventory(anyString(), anyInt())).thenReturn(false);
        // Act
        order.setMailer(mailer);
        order.fill(warehouse);
        // Assert
        verify(warehouse).hasInventory(anyString(), anyInt());
        verify(mailer).send(anyString());
    }
{% endcodeblock %}

* ## Conclusion
    * **Mock objects always use behavior verification.**
    *  **A stub can go either way (behavior or state verification)**.
      Meszaros refers to **stubs that use behavior verfication as a Test Spy**.
    * 本質上, Mock 本來就與 Stub 相異了.
        * Mock 使用 behavior verification, 流程都符合規則, 才算通過
        * Stub 使用 state verification, 結果對, 就算通過 

- - -
<a name=3></a>
## Classic and Mockist Testing ##

上述針對測試時, 使用的 Real Object 與 Mock Object 之間做了一些比較.
作者把這樣的差異, 畫分為兩種不同的風格:

1.  The classical TDD style
    * Use real objects if possible and a double if it's awkwar to use the real thing.
    * Use a real warehouse and a double for the mail service.
2.  A mockist TDD practitioner
    * Always use a mock for any object with interesting behavior.
    * In this case, both the warehouse and the mail service.

- - -
<a name=4></a>
## Choosing Between the Differences ##

雖然文章的 Title 說明了 Mock Aren't Stubs,
但這篇文章有很大的篇幅在說明 classical TDD 與 mockist TDD 之間的優劣,
透過不同的角度來說兩種不同的 sytle 做比較以及如何影響 coding 內容.

1. Driving TDD
   * Classical
       * Mockist 能做到的, 它也能做到.
         甚至更好 ... (這一條還沒領悟到)
   * Mockist
       * 擁護此種 Style 的使用者們深信, 透過這樣的方式來執行 TDD, 
         可以將與 Object Under Test 互相溝通的 Interface 切得更好.  
2. Fixture Setup (在 Test Case 執行前後, 做資源的初始化以及回收)
   * Classical
       * 被批評每個測項的初始化資料, 有可能不一樣, 因此初始化複雜, 緩慢.
         舉例而言, 測項中如果要使用 Stubs, 有可能必須準備不同實做的 Stubs, 來驗證不同狀況.
         此外, 還有一個缺點, 就是初始化的相關資料一變動, 測項就有可能被影響, 進而導致 Fail!
   * Mockist
       * 通常只將 Collaborators 相對應的 Mock Object 做初始化.  
3. Test Isolation
   * Classical
       * 一旦測項 Fail, 有可能不容易找出 Root Cause.
         因為測試時, 牽扯到的 Collaborators 也有可能出錯, 因為都是使用 Real Objects.
         但是, Classical 的擁護者反駁, 通常問題都可以很容易找出來.
         否則就是測項的粒度切的不夠細.
       * 本質上, classic xunit tests 不只是 unit test 而已, 也是一種 mini-integration test.
   * Mockist
       * 一旦測項 Fail, 很明確的就是 Object Under Test 出了問題.
         PS: 當然, 這個前提是 Mock Object 回傳的值, 都有被正確指定. 
4. Coupling Test Implementations
   * Classical
       * 不需要管 Object Under Test 的內部實作為何, 因此只要 Objects 之間相戶溝通的介面保持一致.
         即使實作方式改變, 也不影響原本測項結果. 
   * Mockist
       * 與 Object Under Test 的內部實作有緊密的關係, 一但內部實作改變, 有可能造成測項 Fail!
5. Design Style
   * Classical
       * 有可能為了做 State Verification, 進而開出一些不必要的 Interface. 
   * Mockist
       * 盡量避免使用 Method Chain, Ex: getThis().getThat().gtTheOther()
       * 傾向避免讓 method return 結果, 而改用丟入參數收集將原本要 return 的結果.
         舉例來講: 要收集各個物件的狀態, 會用一個 Buffer 物件丟到各個物件的 method, 去收集各物件的狀態. 

最後, 有一點值得提出來提醒大家, **自動測試除了 unit test 外, 最好也包含一些跨整個系統的 acceptance tests.**
這才能確保測試是完整的.

- - -
<a name=5></a>
## Should I be a Classicist or Mockist 
作者本身為 Classicist, 最主要的原因是因為 Mock 的做法, 會有 Coupling Test Implementation 的 Concern.
但他認為, 每個人觀點不同, 哪種方法對你最直覺好用, 這種方法對你就是好方法.

- - -
* [Reference]
    * [1] [Mock Aren't Stubs](http://martinfowler.com/articles/mocksArentStubs.html)
    * [2] [相關代碼](https://github.com/MutsuGhost1/Code/tree/master/mockito/MockArentStubs)