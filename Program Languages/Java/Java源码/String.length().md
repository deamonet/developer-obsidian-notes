### Method Detail

- 
    public int length()
    
    Returns the length of this string. The length is equal to the number of [Unicode code units](https://docs.oracle.com/javase/8/docs/api/java/lang/Character.html#unicode) in the string.
    
    Specified by:
    
    [length](https://docs.oracle.com/javase/8/docs/api/java/lang/CharSequence.html#length--) in interface [CharSequence](https://docs.oracle.com/javase/8/docs/api/java/lang/CharSequence.html "interface in java.lang")
    
    Returns:
    
    the length of the sequence of characters represented by this object.

这里的意思是说：该方法返回的字符串的长度，具体是 unicode 编码单位的数量。

### Unicode Character Representations

> The `char` data type (and therefore the value that a `Character` object encapsulates) are based on the original Unicode specification, which defined characters as fixed-width 16-bit entities. The Unicode Standard has since been changed to allow for characters whose representation requires more than 16 bits. The range of legal _code point_s is now U+0000 to U+10FFFF, known as _Unicode scalar value_. (Refer to the [_definition_](http://www.unicode.org/reports/tr27/#notation) of the U+_n_ notation in the Unicode Standard.) 
> 
> The set of characters from U+0000 to U+FFFF is sometimes referred to as the _Basic Multilingual Plane (BMP)_. Characters whose code points are greater than U+FFFF are called _supplementary character_s. The Java platform uses the UTF-16 representation in `char` arrays and in the `String` and `StringBuffer` classes. In this representation, supplementary characters are represented as a pair of `char` values, the first from the _high-surrogates_ range, (\uD800-\uDBFF), the second from the _low-surrogates_ range (\uDC00-\uDFFF).
> 
> A `char` value, therefore, represents Basic Multilingual Plane (BMP) code points, including the surrogate code points, or code units of the UTF-16 encoding. An `int` value represents all Unicode code points, including supplementary code points. The lower (least significant) 21 bits of `int` are used to represent Unicode code points and the upper (most significant) 11 bits must be zero. Unless otherwise specified, the behavior with respect to supplementary characters and surrogate `char` values is as follows:

-   The methods that only accept a `char` value cannot support supplementary characters. They treat `char` values from the surrogate ranges as undefined characters. For example, `Character.isLetter('\uD840')` returns `false`, even though this specific value if followed by any low-surrogate value in a string would represent a letter.
-   The methods that accept an `int` value support all Unicode characters, including supplementary characters. For example, `Character.isLetter(0x2F81A)` returns `true` because the code point value represents a letter (a CJK ideograph).

In the Java SE API documentation, _Unicode code point_ is used for character values in the range between U+0000 and U+10FFFF, and _Unicode code unit_ is used for 16-bit `char` values that are code units of the _UTF-16_ encoding. For more information on Unicode terminology, refer to the [Unicode Glossary](http://www.unicode.org/glossary/).

Since: 1.0
See Also: [Serialized Form](https://docs.oracle.com/javase/8/docs/api/serialized-form.html#java.lang.Character)

简单来说 Java 中的 char 类型是根据早期 unicode 针对每一个字符采用 16 bit 表示的特点而设计的，而后期 unicode 扩展到更多位，char 类型就只能把超过16 bit 的字符用两个 char 类型来表示，这样计数就会多一个。