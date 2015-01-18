title: Refactoring (1-1)
date: 2015-01-18 15:21:48
categories: Software
tags: [Refactoring, Study Group]
---

#### 前言 ####

在開始介紹 Refactoring 相關議之前, 我們將在本章先介紹一個例子,
了解這個例子之後, 將在下一章節針對這個例子做 Refactoring
<!--more-->

----------

#### 一個影片出租程式的例子 ####

> 這是一個影片出租店用的程式, 計算每一位顧客的消費金額並列印報表(statement).
> 操作者告訴程式: 顧客租了哪些影片, 租期有多長,程式便根據租賃時間和影片類型算出費用.
> 影片分為三類: 普通片,兒童片和新片.
> 除了計算費用, 還要為常客計算點數, 作為回饋.
> 點數會隨著**租片總類是否為新片**而有所不同.

根據上述的需求, 可以將相關的業務, 以 UML class diagram 來表示:
![](/images/refactoring/refactoring_fig_1_1.PNG)

以下為最初版本的代碼, 是個可以正確執行無誤的版本:

    // Movie.java
    public class Movie {
        public static final int CHILDRENS = 2;
        public static final int REGULAR = 0;
        public static final int NEW_RELEASE = 1;
    
        private String _title; // title of the movie
        private int _priceCode;// type of the movie
    
        public Movie(String title, int priceCode){
            _title = title;
            _priceCode = priceCode;
        }
    
        public int getPriceCode() {
            return _priceCode;
        }
    
        public void setPriceCode(int arg) {
            _priceCode = arg;
        }
    
        public String getTitle() {
            return _title;
        }
    }

<br/>

    // Rental.java
    class Rental {
        private Movie _movie;
        private int _daysRented;
    
        public Rental(Movie movie, int daysRented) {
            _movie = movie;
            _daysRented = daysRented;
        }
    
        public int getDaysRented() {
            return _daysRented;
        }
    
        public Movie getMovie() {
            return _movie;
        }
    }

<br/>

    // Customer.java
    class Customer {
        private String _name; // the name of the customer
        private Vector _rentals = new Vector(); // a customer can have many rentals
    
        public Customer(String name) {
            _name = name;
        }
    
        public void addRental(Rental arg) {
            _rentals.addElement(arg);
        }
    
        public String getName() {
            return _name;
        }
    
        public String statement() {
            double totalAmount = 0;
            int frequentRenterPoints = 0;
            Enumeration rentals = _rentals.elements();
            String result = "Rental Record for " + getName() + "\n";
    
            while(rentals.hasMoreElements()){
                double thisAmount = 0;
                Rental each = (Rental) rentals.nextElement();
    
                //determine amounts for each line
                switch(each.getMovie().getPriceCode()){
                    case Movie.REGULAR:
                        thisAmount += 2;
                        if(each.getDaysRented()>2)
                            thisAmount += (each.getDaysRented()-2)*1.5;
                    break;
    
                    case Movie.NEW_RELEASE:
                        thisAmount += each.getDaysRented()*3;
                    break;
    
                    case Movie.CHILDRENS:
                        thisAmount += 1.5;
                        if(each.getDaysRented()>3)
                            thisAmount += (each.getDaysRented()-3)*1.5;
                    break;
                }
    
                // add frequent rental points
                frequentRenterPoints ++;
                // add bonus for a two day new release rental
                if ((each.getMovie().getPriceCode() == Movie.NEW_RELEASE) &&
                 each.getDaysRented() > 1)
                    frequentRenterPoints ++;
    
                result += "\t" + each.getMovie().getTitle() + "\t" +
                String.valueOf(thisAmount) + "\n";
                totalAmount += thisAmount;
            }
    
            // add footer lines
            result += "Amount owed is " + String.valueOf(totalAmount) + "\n";
            result += "You earned " + String.valueOf(frequentRenterPoints) +
              " frequent renter points";
            return result;
        }
    }

執行特定客戶的列表(statement), 其 Sequence Diagram 如下:
![](/images/refactoring/refactoring_fig_1_2.PNG)

在準備做 Refactoring 之前, 請確保下列幾件事情:

1. **準備好 Test Suite, 用來驗證每次 Refactoring 的結果, 確保不讓任何 Test Case Fail**
2. **確保 Test Suite 在做 Refactoring 之前是完全 Pass 的**
3. **請單獨執行 Refactoring, 不要與開發新功能一起同時進行**

以下為事先準備好的 Test Suite:

    // CustoomerTest.java
    public class CustomerTest {
        @Test
        public void testCustomer() {
            Customer c = new CustomerBuilder().build();
            assertNotNull(c);
        }

        @Test
        public void testAddRental() {
            Customer customer2 = new CustomerBuilder().withName("Sallie").build();
            Movie movie1 = new Movie("Gone with the Wind", Movie.REGULAR);
            Rental rental1 = new Rental(movie1, 3); // 3 day rental
            customer2.addRental(rental1);
        }
    
        @Test
        public void testGetName() {
            Customer c = new Customer("David");
            assertEquals("David", c.getName());
        }
    
        @Test
        public void statementForRegularMovie() {
            Movie movie1 = new Movie("Gone with the Wind", Movie.REGULAR);
            Rental rental1 = new Rental(movie1, 3); // 3 day rental
            Customer customer2 =
                new CustomerBuilder()
                .withName("Sallie")
                .withRentals(rental1)
                .build();
            String expected = "Rental Record for Sallie\n" +
                "\tGone with the Wind\t3.5\n" +
                "Amount owed is 3.5\n" +
                "You earned 1 frequent renter points";
            String statement = customer2.statement();
            assertEquals(expected, statement);
        }
    
        @Test
        public void statementForNewReleaseMovie() {
            Movie movie1 = new Movie("Star Wars", Movie.NEW_RELEASE);
            Rental rental1 = new Rental(movie1, 3); // 3 day rental
            Customer customer2 =
                new CustomerBuilder()
                .withName("Sallie")
                .withRentals(rental1)
                .build();
            String expected = "Rental Record for Sallie\n" +
                "\tStar Wars\t9.0\n" +
                "Amount owed is 9.0\n" +
                "You earned 2 frequent renter points";
            String statement = customer2.statement();
            assertEquals(expected, statement);
        }
    
        @Test
        public void statementForChildrensMovie() {
            Movie movie1 = new Movie("Madagascar", Movie.CHILDRENS);
            Rental rental1 = new Rental(movie1, 3); // 3 day rental
            Customer customer2
                = new CustomerBuilder()
                .withName("Sallie")
                .withRentals(rental1)
                .build();
            String expected = "Rental Record for Sallie\n" +
                "\tMadagascar\t1.5\n" +
                "Amount owed is 1.5\n" +
                "You earned 1 frequent renter points";
            String statement = customer2.statement();
            assertEquals(expected, statement);
        }
    
        @Test
        public void statementForManyMovies() {
            Movie movie1 = new Movie("Madagascar", Movie.CHILDRENS);
            Rental rental1 = new Rental(movie1, 6); // 6 day rental
            Movie movie2 = new Movie("Star Wars", Movie.NEW_RELEASE);
            Rental rental2 = new Rental(movie2, 2); // 2 day rental
            Movie movie3 = new Movie("Gone with the Wind", Movie.REGULAR);
            Rental rental3 = new Rental(movie3, 8); // 8 day rental
            Customer customer1
                = new CustomerBuilder()
                .withName("David")
                .withRentals(rental1, rental2, rental3)
                .build();
            String expected = "Rental Record for David\n" +
                "\tMadagascar\t6.0\n" +
                "\tStar Wars\t6.0\n" +
                "\tGone with the Wind\t11.0\n" +
                "Amount owed is 23.0\n" +
                "You earned 4 frequent renter points";
            String statement = customer1.statement();
            assertEquals(expected, statement);
        }
        //TODO make test for price breaks in code.
    }

<br/>

    public class CustomerBuilder {
    
        public static final String NAME = "Gregroire";
        private String name = NAME;
        private List<Rental> rentals = new ArrayList<Rental>();
    
        public Customer build() {
            Customer result = new Customer(name);
            for (Rental rental : rentals) {
                result.addRental(rental);
            }
            return result;
        }
    
        public CustomerBuilder withName(String name) {
            this.name = name;
            return this;
        }
    
        public CustomerBuilder withRentals(Rental... rentals) {
            Collections.addAll(this.rentals, rentals);
            return this;
        }
    }
    

下列為 Eclipse 中, 所有測項都 Pass 的結果:    
![](/images/refactoring/refactoring_fig_test_all_pass.PNG)

----------

### Note ###

1. 這邊有個疑問, 從 figure 1.1 的 UML class diagram 中顯示多個租賃可以對應到同一部電影,
   為何經過第一次 Refactoring 之後, UML class diagram 卻表示為一個租賃對應一部電影 ?
2. [Source Code Download](https://github.com/MutsuGhost1/Code/tree/master/Refactoring "Source Code")
3. 執行環境: Eclipse + junit-4.12.jar + hamcrest-core-1.3.jar 