羅國榮 2019/10/23 10:00:00

4 27984

**前言**

JAVA在 1.5版本新增列舉(enum)這項新語法，列舉的出現雖然並不是驚天動地的事情，而它卻已經默默地改變撰寫程式的方式。由於列舉的簡單易用，使得程式開發人員常常只用到淺層語法就戛然而止，以至於更深層的語法較未有著墨者，著實可惜其帶來的好處。

本篇文章將深入淺出介紹列舉的各項特性以及各種用法，讓列舉的好用之處凸顯出來，尤其列舉可以實作(implement)介面(interface)、覆寫(override)抽象方法，更是深具潛力的特性，值得程式開發人員去挖掘。

**列舉(Enum (enumerated)) 用法**

**用法1. 列舉可做為「常數」的集合**

以前，**常數**通常是用static, final來宣告，放入類別(class)內視為集合，例如:

```java
public class OldGrade {
    public static final int A = 1;
    public static final int B = 2;
    …
    public static final int INCOMPLETE = 6;
}
```

現在，則是用列舉來宣告，列如：

```java
public enum Grade {
    A, B, C, D, F, INCOMPLETE
}
```

 **※ 使用列舉可以避免誤用錯誤的常數**

**先用"舊"語法定義三個類別──**

```java
public class OldGrade {
    public static final int A = 1;
    //以下略……
}

public class OldClass {
    public static final int English = 1;
    //以下略……
}

public class OldStudent {
    public void assignGrade(int grade) {
        //以下略……
    }
}
```

**再用"新"語法定義三個類別──**

```java
public enum NewGrade {
    A, B, C, D, F, INCOMPLETE
}

public enum NewClass {
    ENGLISH, MATH
}

public class NewStudent {
    public void assignGrade(NewGrade grade) {
         //以下略……
    }
}
```

**以前**，"舊"語法的assignGrade方法的輸入參數只能宣告為int 型別，如此一來OldGrade.A與OldClass.ENGLISH都可以傳入assignGrade方法，雖然傳入的數值都是1，但是邏輯上卻是南轅北轍，而這樣的錯誤在執行時期前是難以被檢核出來，發生錯誤後會花費很多時間在除蟲(debug)。

```java
student1.assignGrade(OldGrade.A); //正確傳入
student1.assignGrade(OldClass.ENGLISH); //錯誤傳入，難以檢查
```

**現在**，"新"語法的assignGrade方法可以宣告為NewGrade型別，只能傳入NewGrade列舉，若是傳入NewClass列舉，在編譯時期就可以檢出錯誤，輕鬆省下除蟲時間。

```java
student1.assignGrade(NewGrade.A); //正確傳入
student1.assignGrade(NewClass.ENGLISH); //錯誤傳入，容易檢查
```

這也是建議改用列舉最重要的原因。

**用法2. 列舉可搭配switch使用**

在JAVA 1.7擴充switch的功能後，便可以使用列舉做為引數，使得程式碼更為簡潔、易讀，更容易理解程式碼。

```java
public String getGradeDesc(NewGrade grade) {
    String desc = "……";

    switch (grade) {
        case A:
            desc = "……";
            break;
        case B:
            desc = "……";
            break;
        //以下略……
        default:
            break;
    }
    return desc;
}
```

**用法3. 列舉可以在類別內(inline)定義**

列舉通常會單獨寫在一個java檔內，若是列舉專屬於某類別使用，將其定義在某個類別內的寫法如下例，讓列舉與類別的相關性更清楚明白。

例如:

```java
public class Downloader {
    public enum DownloadStatus {INITIALIZING, IN_PROGRESS, COMPLETE};
    //以下略，class 本身程式碼
}
```

**用法4. 列舉可以有建構子(constructor)，可建構複雜的列舉**

建構較為複雜的列舉，可以在實務上更加貼近需求，此用法是個很重要的用法，值得學習。

舉例如下：

```java
public enum Grade {
    // 優先定義列舉實例，傳入二個參數
    A(9, "優異"),
    B(8, "佳"),
    C(7, "良好"),
    D(6, "普通"),
    F(5, "略差"),
    INCOMPLETE(4, "多努力");  // ’;’分號為必要，不可少

    // 新屬性需寫在列舉實例之後
    private int score;
    private String description;

    // 建構子預設為 private，可寫可不寫；不能定義為public
    // 建構子有二個參數
    Grade(int score, String desc) {
        this.score = score;
        this.description = desc;
    }

    public int getScore() {
        return score;
    }

    public String getDescription() {
        return description;
    }
}
```

**依據上例，有幾件事項要說明一下：**

1.     列舉實例必須最先定義。

2.     最後一個列舉實例必須加上分號(;)。

3.     新屬性不能在列舉實例之前宣告，需在列舉實例之後宣告。

4.     建構子有二個參數，所以每個列舉實例都要傳入二個參數。

5.     建構子是隱含性的private，private修飾詞可寫可不寫，不能變更為public。

**用法示範：**

```java
1. 取得Grade.A的分數：
int score = Grade.A.getScore(); //9
或 
Grade gradeInstance = Grade.A;
int score = gradeInstance.getScore(); //9

2. 取得Grade.A的說明：
String desc = Grade.A.getDescription(); //等級C
或
Grade gradeInstance = Grade.C;
String desc = gradeInstance.getDescription(); //等級C
```

**用法5. 列舉常用的方法(method)**

以下的說明使用「用法4」的程式範例，請參考。

**1.** **public final int compareTo([E](https://docs.oracle.com/javase/8/docs/api/java/lang/Enum.html "type parameter in Enum")**    **o)** – 與指定的列舉比較，該方法返回一個負整數、零或正整數，據此以表示小於、等於或大於指定的列舉。回傳值的計算方式為該列舉的ordinal()減掉指定列舉的ordinal()。

用法：  

```java
int retValue =  Grade.INCOMPLETE.compareTo(Grade.D); //回傳值為正整數2(5-3=2)，表示大於Grade.D
```

**_2._ public final boolean equals(****[Object](https://docs.oracle.com/javase/8/docs/api/java/lang/Object.html "class in java.lang")** **other)** –與指定的列舉比較，該方法返回一個布林值，據此以表示是不相等(false)或相等(true)於指定的列舉。

※除了可以使用equals()比較，也可以使用"**==**"比較。

用法：  

```java
(1) Boolean retValue = Grade.INCOMPLETE.equals(Grade.D); //false
(2) Boolean retValue = Grade.INCOMPLETE == Grade.D; // false
```

**_3._** **public final [String](https://docs.oracle.com/javase/8/docs/api/java/lang/String.html "class in java.lang")** **name()**–返回列舉實例的名稱。

用法：  

```java
String retValue = Grade.INCOMPLETE.name(); // INCOMPLETE
```

**4.** **public final int ordinal()**–返回列舉實例的序號，從零開始編號。

用法：  

```java
int retValue = Grade.INCOMPLETE.ordinal(); // 5
```

**5.** **public [String](https://docs.oracle.com/javase/8/docs/api/java/lang/String.html "class in java.lang")** **toString()**–返回列舉實例的名稱。

用法：  

```java
String retValue = Grade.INCOMPLETE.toString(); // INCOMPLETE
```

※本方法(method)可以覆寫(Override)，客製化返回的字串。

例:  

```java
@Override  
public String toString() {  
    return this.score+"_"+this.description;  
}  
```

**6.** **public static E valueOf(String name)** –返回具有指定名稱的列舉實例。

※本方法(method)是**靜態**(static)方法。

用法：

```java
Grade gradeInstance = Grade.valueOf("INCOMPLETE"); //可取得Grade.INCOMPLETE，好用之處就是用字串就可以取得相對應的列舉
```

**7. public static E[] values()** –返回所有列舉實例。

※本方法(method)是**靜態**(static)方法。

用法：

```java
Grade[] grades = Grade.values(); //陣列長度為6，陣列內容為[Grade.A, Grade.B, Grade.C, Grade.D, Grade.F, Grade.INCOMPLETE]
```

以上用法可參考下列網址: 

[http://tw.gitbook.net/java/lang/enum_compareto.html](http://tw.gitbook.net/java/lang/enum_compareto.html)

[http://tw.gitbook.net/java/lang/enum_equals.html](http://tw.gitbook.net/java/lang/enum_equals.html)

[http://tw.gitbook.net/java/lang/enum_name.html](http://tw.gitbook.net/java/lang/enum_name.html)

[http://tw.gitbook.net/java/lang/enum_ordinal.html](http://tw.gitbook.net/java/lang/enum_ordinal.html)

[http://tw.gitbook.net/java/lang/enum_tostring.html](http://tw.gitbook.net/java/lang/enum_tostring.html)

[http://tw.gitbook.net/java/lang/enum_valueof.html](http://tw.gitbook.net/java/lang/enum_valueof.html)

**用法6. 列舉可以實作(implement)介面(interface)**

※    **列舉(enum)是一種特殊類別(class)，擁有類別的某些特性，例如:可以實作(implement)介面(interface)。 善用此特性，程式開發人員能讓列舉更具應用價值。**

假設有個情境，有少數的學科有自己的評分等級，卻也希望所有學科有統一的方法可以調用，此時**用法6**就是個絕佳的方案。

例：

定義介面—

```java
public interface GradeAction {
   public Integer getScore();
   public String getDescription();
}
```

定義列舉MathGrade—

```java
public enum MathGrade implements GradeAction {
    A, B, C, D;

    @Override
    public Integer getScore() {
        Integer score;
        switch (this) {
            case A:
                score = 90;
                break;
            case B:
                score = 80;
                break;
            case C:
                score = 70;
                break;
            case D:
                score = 60;
                break;
            default:
                score = 0;
                break;
        }
        return score;
    }

    @Override
    public String getDescription() {
        switch (this) {
            case A:
                return "優異";
            case B:
                return "佳";
            case C:
                return "良好";
            case D:
                return "普通";
            default:
                return "--";
        }
    }
}
```

定義列舉EnglishGrade—

```java
public enum EnglishGrade implements GradeAction {
    A, B, C;

    @Override
    public Integer getScore() {
         //以下略...
    }

    @Override
    public String getDescription() {
        //以下略...
    }
}
```

上述列舉程式碼的switch展示了二種寫法。

**用法7. 列舉可以定義抽象方法並覆寫(override)**

※    **列舉(enum)是一種特殊類別(class)，擁有類別的某些特性，例如:**定義抽象方法並覆寫(override)**。 善用此特性，程式開發人員能讓列舉更具應用價值。**

假設有個情境，每個等級的評分有各自邏輯，此時**用法7**就是個絕佳的方案。

例：

定義列舉—

```java
public enum CommonGrade {
    A {
        @Override
        public Integer getScore() {
            return 90;
        }

        @Override
        public String getDescription() {
            return "優異";
        }
    },
    B {
        @Override
        public Integer getScore() {
            return null;
        }

        @Override
        public String getDescription() {
            return null;
        }
    },
    C {
        @Override
        public Integer getScore() {
            return null;
        }

        @Override
        public String getDescription() {
            return null;
        }
    },
    D {
        @Override
        public Integer getScore() {
            return null;
        }

        @Override
        public String getDescription() {
            return null;
        }
    };

    public abstract Integer getScore();

    public abstract String getDescription();
}
```

**結語**

希望透過本文的拋磚引玉，能夠讓大家更深入了解列舉(enum)，將列舉的特性發揮得更好。上述的例子只是淺顯的展露列舉的特性，相信各位讀者能實現出更佳的應用。