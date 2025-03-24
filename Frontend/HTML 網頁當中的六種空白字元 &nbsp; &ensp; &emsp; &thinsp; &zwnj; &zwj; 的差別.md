在HTML當中，空格 (space) 不僅僅就是空格這麼簡單的事，它其實有六種不同樣的空格。如果你有編輯網頁或在網路上寫文章的經驗，肯定會發現當你連續輸入好幾個空格的時候，它永遠都只會有一個空格，甚至是被當作”無內容”來解析。這是因為瀏覽器在遇到空格的時候，如果它認為這是一串字串，那麼空格就會被當作空格但不會累計，不管你輸入再多個空格它還是只會有一個。

空格在 HTML 當中被視為是特殊符號的一種，在網頁當中如果想要正確的加入空格，就必須用 HTML 的語法輸入。HTML 提供了六種不同種類的空格，其中最大的差異在於**寬度**上的不同。

## 1. &nbsp; 不換行空格

No-Break Space，不換行空格。這是最常見的空格，也是一般我們鍵盤按下空白鑑會出現的字元。大部分來說我們只會使用這樣的空白，輸入多次 &nbsp; 空格也會累計。&nbsp; 的空格**寬度會受字體影響**。

## 2. &ensp; 半形空格

En Space，半形空格。en 是一個半型字元寬度的單位，寬度為 em 的一半。例如當 16px 大小的字體，半形就是 8px 的寬度。就定義上來說，一個 en 就是小寫字母 n 的寬度，大約是半個中文字寬。

## 3. &emsp; 全形空格

Em Space，全形空格。和半形空格同概念，em 是一個全形字元的寬度，如果字體大小是 16px，一個 em 就是 16px，大約等於一個中文字寬。

## 4. &thinsp; 窄空格

窄空格，顧名思義就是寬度較窄的空格，大約是 1/6 個 em 寬。

## 5. ‌&zwnj;

全名是 Zero Width Non Joiner，這通常是用於比較複雜的文字或特殊符號當中。首先零寬度表示這個空白不會被顯示出來，它用於分隔兩個字元排列在一起而可能發生的**“連字”**。也就是說，它是一個分隔元素切開兩個字元，讓它們各自產生自己相對應的文字。

## 6. ‍&zwj;

全名是 Zero Width Joiner。與 ZWNJ 相同的是它也是零寬度的空格，並不會被顯示在頁面上。但相反的是，它會觸發前後兩字元的連字效果。這通常是用於較複雜的語言，例如阿拉伯語或印地語等

## 單純只是想要輸入空白？ &nbsp; 就對了！

**大部分的狀況我們還是只會用到最一般的 &nbsp; 空白字元**，例如想要連續輸入好幾個空格的時候。或是想要放一個空白的元素，例如 <p> </p> 與 <p>&nbsp;</p> 就會被瀏覽器解析為不同的狀況。前者等同於無內容的元素，不會在頁面上顯示任何東西；後者則是一個段落元素中包含一個空白字元，所以我們可以在頁面上看到一個段落的存在，同時裡面還包裹著一個空白。除了第一種空白之外，其他五種的空格也要看瀏覽器有沒有支援，且寬度也可能有些許影響，不太常用。

![example for special characters](media/example_for_special_characters.jpg)

這篇文章的標題列，實際在編輯器當中長的樣子

除了空白之外，還有很多其他的特殊字元被定義在 Unicode 和 HTML當中，例如 & 這個 and 符號它在 HTML 當中也被定義為 &amp;。像我在寫這篇文章的時候也遇到很多的麻煩，常常需要轉換到原始碼編輯器去直接編輯原始碼，否者都會出現一些不可預期的結果。

Copyright announcement:  
the featured image: Photo by [Patrick Fore](https://unsplash.com/@patrickian4?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on [Unsplash](https://unsplash.com/search/photos/blank-space-type?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)


[Front-end 網頁前端](https://note.charlestw.com/category/tech/front-end-%e7%b6%b2%e9%a0%81%e5%89%8d%e7%ab%af/)[HTML](https://note.charlestw.com/category/tech/programming/html/)[Programming](https://note.charlestw.com/category/tech/programming/)[Tech 科技](https://note.charlestw.com/category/tech/)
