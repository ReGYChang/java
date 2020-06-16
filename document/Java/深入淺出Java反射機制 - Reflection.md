# 究竟反射是為何物
> 反射機制的核心在於允許在運行中的Java程式獲取自身訊息並操作類和物件的內部方法

先來看看一般我們所熟悉的操作物件的方法:
```Java
//直接初始化並呼叫方法
Regy regy = new Regy(); 
Garra garra = new Garra(); 
regy.setGirlFriend(garra);
```

如果使用JDK的Reflect API來進行調用:
```Java
//獲取Class物件
Class regyClass = Class.forName("com.demo.reflect.Regy");
Class garraClass = Class.forName("com.demo.reflect.Garra");

//通過Constructor物件newInstance()方法創建類別實體
Constructor regyConstructor = regyClass.getConstructor();
Constructor garraConstructor = garraClass.getConstructor();
Object regyObject = regyConstructor.newInstance();
Object garraObject = garraConstructor.newInstance();

//獲取反射方法並調用
Method method = regyClass.getMethod("setGirlFriend", Garra.class);
method.invoke(regyObject, garraObject);
```

在上面兩段程式碼中，所做的事情是完全一樣的。都是先初始化Regy跟Garra，然後調用Regy的方法setGirlFriend()

- 基於第一種方式實體化物件，物件的類型都是在編譯期就確定下來，隨後透過ClassLoader加載進JVM，此謂靜態加載
- 而第二種方法透過反射，在JVM運行期時才動態加載類別或調用方法/判斷屬性，而在編譯期時不確定運行的物件為何，此謂動態加載

## 為何而反?
> 通常學反射會有個很大的障礙：那既然可以透過new()來實體化物件，又何必大費周章地使用反射機制?

**最大的原因在於 : 解耦**

試想，如果我們今天需要阿花來幫我們打掃家裡，於是乎：
```Java
//使用new()召喚阿花
Housekeeper 阿花 = new Housekeeper();
//叫阿花打掃
阿花.cleanHouse();
```
這裡使用第一種方法手動new()方法把初始化類別寫死讓JVM去跑;但如果今天阿花請假，我們就要再去找一個阿姨阿水，重新修改source code，把阿花改成阿水。聰明如你，當你系統裡的阿花成百上千，依賴關係交錯複雜，這樣改絕對風險極高。而且也違反了開閉原則 (Open–closed principle)。

再用個簡單的場景說明一下：

```Java
Class.forName("com.mysql.jdbc.Driver");

//獲取資料庫連線物件-Connetcion
connection = DriverManager.getConnection("jdbc:mysql://localhost:0415/regy", "root", "root");

//獲取執行sql語句的statement物件
statement = connection.createStatement();

//執行sql語句，取得結果集
resultSet = statement.executeQuery("SELECT * FROM users");
```
剛開始寫資料庫連線可能會這樣寫，但連線的ip或port number都是寫死的，如果今天需要換一個資料源，就要把所有連線資料庫程式碼的地方找出來重寫->重新compile再丟到JVM上跑，這是非常不科學的，內容耦合太高。於是一些聰明的人想到了另一種辦法:

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

如此一來，如果要更改資料庫配置，直接從配置檔裡面改就好，不需要跳下去改source code，大大降低內容耦合性。

```Java
反射其實時常圍繞在我們身邊
不論是IDE裡的提示字功能、Spring IoC、還是動態代理其核心都是通過反射去實現
所以弄懂反射原理很大程度上可以幫助我們學習各種框架或設計模式，也有助於拓展程式設計思維
```

# 淺談JVM
> 反射原理之所以艱澀難以理解，因為關係到一些語言特性與JVM結構。當然JVM可以花非常多的時間去深入探討，但在這裡要先簡單探討一下JVM的類加載機制

## JVM如何建構實體
如果有讀過Java的編譯過程，大概就知道我們所編寫的Java檔要運行需要經過compiler編譯成.class的字節碼文件，再經由ClassLoader載入JVM運行。

先來看看我們的JVM內存架構：
<!-- JVM內存架構圖 -->
今天我們當寫了一行程式：
```Java
HandsomeGuy regy = new HandsomeGuy();
```
執行javac、java後整個流程是這樣子的：
<!-- 實體物件建構圖 -->知呼@Java反射
如此一來，我們的HandsomeGuy實體物件就生成了

## ClassLoader
前面提到，java source code經過Compile後，透過ClassLoader加載到JVM創建Class物件，才能使用這個類。所以ClassLoader究竟為何物?

- 火星版：通過一個class的Fully Qualified Name來描述該class的byte streams放到Java JVM外部實現，以便讓應用程式決定如何獲取所需要的類。實現這個動作的物件即為ClassLoader
- 地球版：**實現將class文件加載進JVM記憶體功能的物件**
 
### 延遲加載
JVM運行並不會一次性加載全部的淚，會根據所需逐步加載，即所謂延遲加載。程式在Runtime過程中會逐漸遇到許多尚未加載的新類，這時就會調用ClassLoader來加載這些類。加載完成後會創建Class物件並保存在記憶體，下次使用到相同類時就不需要重複加載。

### ClassLoader結構
<!-- ClassLoader結構圖 -->
- BootstrapClassLoader：啟動類ClassLoader，它用來加载<JAVA_HOME>/jre/lib/re.jar
- ExtClassLoader：拓展類ClassLoader，它用来加载<JAVA_HOME>/jre/lib/ext路徑以及java.ext.dirs環境變數指定的classpath下的類
- AppClassLoader：應用程式類ClassLoader，它主要應用程式ClassPath下的類（包含jar包中的類）。它是jav應用程式默認的ClassLoader
- 自訂義ClassLoader：根據自訂義需求，自由訂製類加載邏輯，繼承AppClassLoader，僅Override findClass（）即將繼續遵守**The parent-delegation model(雙親委託模型)**
  
### ClassLoader工作流程
前面提到雙親委託模型，其實就是當一個ClassLoader去加載類時先丟給父類ClassLoader加載，如果父類加載不了再嘗試自己加載。
<!-- 工作流程圖 知呼@class對象-->

有興趣了解更多ClassLoader細節可以參考知呼@请叫我程序猿大人 - [好怕怕的类加载器
](https://zhuanlan.zhihu.com/p/54693308)

## Class類
大家應該都知道，Java的世界中，萬物皆物件。在某種意義上，Java中物件又分為兩種：實體物件與Class物件

- 類別是一類事物的敘述，包含了事物的屬性及行為
- 而Class類別就是用來描述類別的資訊，其本身也是一種類別

> 先說結論，**每一個Java類別都伴隨著一個Class物件**，物件實體化的本質，就是透過Class物件來創造類別實體。可以將Class類別看成是一般類別的更高層抽象，描述一般類別如何構成，包含的資訊等等

### Class類與反射
一般我們使用new()方法來創建實體，我們要先定義好要創造物件的類別，進而產生實體：
```Java
HandsomeGuy regy = new HandsomeGuy();
```

但透過反射，意味我們透過實體去反射類別，並獲取類別方法與屬性。而這裡的實體，指的就是Class物件。再回顧一下一開始的程式碼，反射操作的第一步即是獲取類別所屬的Class物件
```Java
//獲取Class物件
Class regyClass = Class.forName("com.demo.reflect.Regy");
Class garraClass = Class.forName("com.demo.reflect.Garra");
```

### Class物件的誕生
前面提到，當JVM將.class Byte Code加載到記憶體時，如果是第一此加載這個類別，會同時創建一個對應的Class物件。(**一個Java類別伴隨一個Class物件**)

當使用new()創建類別實體的時候，JVM會先去類別所對應的Class物件獲取該類的內容，再來創建類別實體

**! 只有Runtime才能通過Class物件獲取類別資訊 !**

### 解剖Class物件
> 看到這裡應該會非常好奇，那既然是描述一般類別內容的類，內部結構應該要怎麼設計呢?

假設現在有個很帥的Java類:
```Java
public class HandsomeGuy implements Handsome {
    @Notnull
    private String name;

    public HandsomeGuy(){
        
    }

    public HandsomeGuy(String name){
        this.name = name;
    }

    public String getName(){
        return name;
    }

    public void setName(){
        this.name = name;
    }
}
```
這個類應該包含了下列資訊：
- 權限修飾符
- 類名
- 寫了三個類來分別對應
- interface
- annotation
- **Field**
- **Constructor**
- **Method**

其中Field、Constructor和Method對類來說最為重要。為了更詳細描述這些內容，JDK還單獨寫了三個類來分別對應，可通過Class物件提供的方法來獲取相對應物件
<!-- Class get方法圖 -->

# 反射基本運用
繞了一大圈，最後就來聊聊基本的反射功能的使用與實現。

## 判斷是否為某個類別實體
一般我們用 instanceof 操作符來判斷是否為某個類的實體
```Java
public static void main(String[] args) {
    HandsomeGuy regy = new HandsomeGuy();
    if (regy instanceof HandsomeGuy)
      System.out.println("regy 是 HandsomeGuy (類的實體)");
}
```
同時也可以借助反射中 Class 物件的 isInstance() 方法來判斷，它是一个 native 方法：
```Java
public native boolean isInstance(Object obj);
```

## 獲取Class物件
有三種方法可以獲取Class物件：
- 使用Class類forName靜態方法
```Java
Class regyClass = Class.forName("com.demo.reflect.Regy");
```
- 直接獲取某個類別的Class
```Java
Class<HandsomeGuy> handsomeGuy_Class = HandsomeGuy.class;
```
- 調用某個物件的getClass()方法獲得
```Java
Class<?> handsomeGuy_Class = regy.getClass();
```

## 獲取Constructor物件
主要通過Class類的getConstructor獲得對應的Constructor物件，參數為建構參數的Class物件
```Java
getConstructor(Class<?>... parameterTypes)

//取得 HandsomeGuy Class物件
Class<HandsomeGuy> handsomeGuy_class = HandsomeGuy.class;
//取得帶一個String類別參數的Constructor物件
Constructor constructor = handsomeGuy_class.getConstructor(String.class);
```

## 創建實體
通過反射創建物件主要有兩種方式
- 使用Class物件的newInstance()方法來創建對應的類別的實體
```Java
Class<HandsomeGuy> handsomeGuy_class = HandsomeGuy.class;
HandsomeGuy regy = handsomeGuy_class.newInstance();
```
- 通過Class物件獲取指定Constructor物件，再調用Constructor物件的newInstance()方法來創建實體。此方法可以指定Constructor類的實體
```Java
//取得 HandsomeGuy Class物件
Class<HandsomeGuy> handsomeGuy_class = HandsomeGuy.class;
//取得帶一個String類別參數的Constructor物件
Constructor constructor = handsomeGuy_class.getConstructor(String.class);
//根據Constructor建構 HandsomeGuy 類別的實體
HandsomeGuy regy = constructor.newInstance("XD");
```

## 獲取方法
獲取某個Class物件的方法集合，主要有以下幾種方法：
- getDeclaredMethods()，返回class或interface所有方法，包括public、private、protected，但不包括繼承方法
```Java
public Method[] getDeclaredMethods() throws SecurityException
```
- getMethods()：返回class所有public methods，包括繼承方法
```Java
public Method[] getMethods() throws SecurityException
```
- getMethod()：返回特定方法，一個參數為方法名稱，後面的參數為方法參數所對應的Class物件
```Java
public Method getMethod(String name, Class<?>... parameterTypes)
```

舉個例子：
```Java
public class demo {
	public static void test() throws IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
	    //創建HandsomeGuy實體
        Class<HandsomeGuy> handsomeGuy_class = HandsomeGuy.class;
        Constructor constructor = handsomeGuy_class.getConstructor(String.class);
        HandsomeGuy regy = constructor.newInstance("XD");

        //獲取HandsomeGuy類所有方法，不包含繼承方法
        Method[] declaredMethods = regy.getDeclaredMethods();
        //獲取HandsomeGuy類所有public方法，包含繼承方法
        Method[] methods = regy.getMethods();
        //獲取HandsomeGuy類 kiss方法
        Method method = regy.getMethod("kiss", String.class);
	}
}

@Getter
@Setter
class HandsomeGuy {
    private String name;

    public HandsomeGuy(String name){
        this.name = name;
    }

    public void kiss(String girl) {
        System.out.println(name + "kiss" + girl);
    }
}
```

## 獲取Filed物件
主要有這幾種方法，細節不再贅述
- getFiled：訪問 public 成員變數
- getDeclaredField：訪問所有已宣告的變數，但不能得到父類的成員變數
使用方法參照 **Method**

## 調用方法
當我們從類中獲取一個方法，我們就可以使用invoke()方法來調用
```Java
public Object invoke(Object obj, Object... args)
        throws IllegalAccessException, IllegalArgumentException,
           InvocationTargetException
```

以下用一個例子說明：
```Java
public class demo {
	public static void test() throws IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
	    //創建HandsomeGuy實體
        Class<HandsomeGuy> handsomeGuy_class = HandsomeGuy.class;
        Constructor constructor = handsomeGuy_class.getConstructor(String.class);
        HandsomeGuy regy = constructor.newInstance("XD");

        //獲取HandsomeGuy類 kiss方法
        Method method = regy.getMethod("kiss", String.class);

        Object result = method.invoke(regy,"garra")
	}
}

@Getter
@Setter
class HandsomeGuy {
    private String name;

    public HandsomeGuy(String name){
        this.name = name;
    }

    public void kiss(String girl) {
        System.out.println(name + "kiss" + girl);
    }
}
```

# 反射注意事項
- 由於反射會額外消耗系統資源，如不需要動態創建實體，則不需要使用反射
- 反射調用方法可以忽視權限檢查，可能破壞封裝性而導致安全問題(常被拿來做壞壞的事XD)

```lua
由於全文由全手打，且並未經過編譯檢查，以上例子均是為了方便理解。
如有不規範或錯誤再麻煩指出以作修正，最後感謝閱讀，能讀到最後的都不容易啊!
覺得對你有幫助可以點個讚支持，也歡迎留言分享觀點。
```
