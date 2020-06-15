# 究竟反射是為何物
> 反射機制的核心在於允許在運行中的Java程式獲取自身訊息並操作類和物件的內部方法

一般我們所熟悉的操作物件的方法:
```Java
//直接初始化並呼叫方法
Regy regy = new Regy(); 
Garra garra = new Garra(); 
regy.setGirlFriend(garra);
```

如果使用JDK的Reflect API來進行調用:
```Java
Class regyClass = Class.forName("com.demo.reflect.Regy");
Class garraClass = Class.forName("com.demo.reflect.Garra");

Constructor regyConstructor = regyClass.getConstructor();
Constructor garraConstructor = garraClass.getConstructor();

Object regyObject = regyConstructor.newInstance();
Object garraObject = garraConstructor.newInstance();

Method method = regyClass.getMethod("setGirlFriend", Garra.class);
method.invoke(regyObject, garraObject);
```

在上面兩段程式碼中，所做的事情是完全一樣的。都是先初始化Regy跟Garra，然後調用Regy的方法setGirlFriend()

- 基於第一種方式實體化物件，物件的類型都是在編譯期就確定下來，隨後透過ClassLoader加載進JVM，此謂靜態加載
- 而第二種方法透過反射，在JVM運行期時才動態加載類別或調用方法/判斷屬性，而在編譯期時不確定運行的物件為何，此謂動態加載
  
那既然可以透過new()來實體化物件，又何必大費周章地使用反射機制?

**最大的原因在於 : 解耦**

試想，使用第一種方法手動new()方法把初始化類別寫死讓JVM去跑;但如果我因為需求變動需要抽換其他的類別去載入，就要重新修改source code，違反了開閉原則 (Open–closed principle)。用個簡單的例子說明一下:

```Java
Class.forName("com.mysql.jdbc.Driver");

//獲取資料庫連線物件-Connetcion
connection = DriverManager.getConnection("jdbc:mysql://localhost:0415/regy", "root", "root");

//獲取執行sql語句的statement物件
statement = connection.createStatement();

//執行sql語句，取得結果集
resultSet = statement.executeQuery("SELECT * FROM users");
```
剛開始寫資料庫連線可能會這樣寫，但連線的ip或port number都是寫死的，如果今天需要換一個資料源，就要把所有連線資料庫程式碼的地方找出來重寫->重新compile才能丟到JVM上跑，這是非常不科學的，內容耦合太高。於是一些聰明的人想到了另一種辦法:

```Java
//獲取配置文件inputStream
InputStream inputStream = UtilsDemo.class.getClassLoader().getResourceAsStream("db.properties");

Properties properties = new Properties();
properties.load(inputStream);

//獲取配置文件內容
driver = properties.getProperty("driver");
url = properties.getProperty("url");
username = properties.getProperty("username");
password = properties.getProperty("password");

//加載驅動類別
Class.forName(driver);
```

如此一來，如果要更改資料庫配置，直接從配置檔裡面改就好，不需要跳進去改source code，大大降低內容耦合性。

```
反射其實時常圍繞在我們身邊
不論是IDE裡的提示字功能、Spring IoC、還是動態代理其核心都是通過反射去實現
所以弄懂反射原理很大程度上可以幫助我們學習各種框架或設計模式，也有助於拓展開發思維
```

# 淺談JVM
> 反射原理之所以艱澀難以理解，因為關係到一些語言特性與JVM結構。當然JVM可以花非常多的時間去深入探討，但在這裡要先簡單探討一下JVM的類加載機制。

## JVM如何建構實體
## .class
## ClassLoader

