# ساختار داخلی JUnit

![image](../../assets/image/15/img-15.1.png)

<div dir="rtl">
JUnit یکی از مشهورترین فریم‌ورک‌های Java است. از نظر فریم‌ورک‌ها، این ابزار در طراحی ساده، در تعریف دقیق، و در پیاده‌سازی شیک است.
اما کد آن چگونه به نظر می‌رسد؟ در این فصل، ما نمونه‌ای برگرفته از فریم‌ورک JUnit را نقد خواهیم کرد.

## فریم‌ورک JUnit
JUnit نویسندگان زیادی داشته است، اما آغاز آن با Kent Beck و Erich Gamma در یک پرواز به آتلانتا بوده است. Kent می‌خواست Java یاد بگیرد و Erich می‌خواست دربارهٔ فریم‌ورک تست‌نویسی Smalltalk متعلق به Kent بیشتر بداند.
«چه چیزی برای دو گیک در فضایی تنگ طبیعی‌تر است از این‌که لپ‌تاپ‌هایشان را بیرون بیاورند و شروع به کدنویسی کنند؟»
پس از سه ساعت کار در ارتفاع بالا، آن‌ها توانستند اصول پایهٔ JUnit را بنویسند.

ماژولی که به آن خواهیم پرداخت، بخش هوشمندانه‌ای از کد است که به شناسایی خطاهای مقایسهٔ رشته‌ها کمک می‌کند.
این ماژول `ComparisonCompactor` نام دارد. وقتی دو رشته متفاوت به آن داده شود، مانند `ABCDE` و `ABXDE،` تفاوت را با تولید رشته‌ای مانند `<...B[X]D...>` نمایان می‌سازد.

می‌توانم آن را بیشتر توضیح دهم، اما تست‌کیس‌ها این کار را بهتر انجام می‌دهند.
پس به لیستینگ 15-1 نگاهی بیندازید و نیازمندی‌های این ماژول را به‌خوبی درک خواهید کرد.
در حین بررسی، ساختار تست‌ها را نیز نقد کنید. آیا می‌توان آن‌ها را ساده‌تر یا واضح‌تر نوشت؟

</div>


**Listing 15-1 -- ComparisonCompactorTest.java**

```java
package junit.tests.framework;
import junit.framework.ComparisonCompactor;
import junit.framework.TestCase;
public class ComparisonCompactorTest extends TestCase {
    public void testMessage() {
        String failure = new ComparisonCompactor(0, "b", "c").compact("a");
        assertTrue("a expected:<[b]> but was:<[c]>".equals(failure));
    }
    public void testStartSame() {
        String failure = new ComparisonCompactor(1, "ba", "bc").compact(null);
        assertEquals("expected:<b[a]> but was:<b[c]>", failure);
    }
    public void testEndSame() {
        String failure = new ComparisonCompactor(1, "ab", "cb").compact(null);
        assertEquals("expected:<[a]b> but was:<[c]b>", failure);
    }
    public void testSame() {
        String failure = new ComparisonCompactor(1, "ab", "ab").compact(null);
        assertEquals("expected:<ab> but was:<ab>", failure);
    }
    public void testNoContextStartAndEndSame() {
        String failure = new ComparisonCompactor(0, "abc", "adc").compact(null);
        assertEquals("expected:<...[b]...> but was:<...[d]...>", failure);
    }
    public void testStartAndEndContext() {
        String failure = new ComparisonCompactor(1, "abc", "adc").compact(null);
        assertEquals("expected:<a[b]c> but was:<a[d]c>", failure);
    }
    public void testStartAndEndContextWithEllipses() {
        String failure =
            new ComparisonCompactor(1, "abcde", "abfde").compact(null);
        assertEquals("expected:<...b[c]d...> but was:<...b[f]d...>", failure);
    }
    public void testComparisonErrorStartSameComplete() {
        String failure = new ComparisonCompactor(2, "ab", "abc").compact(null);
        assertEquals("expected:<ab[]> but was:<ab[c]>", failure);
    }
    public void testComparisonErrorEndSameComplete() {
        String failure = new ComparisonCompactor(0, "bc", "abc").compact(null);
        assertEquals("expected:<[]...> but was:<[a]...>", failure);
    }
    public void testComparisonErrorEndSameCompleteContext() {
        String failure = new ComparisonCompactor(2, "bc", "abc").compact(null);
        assertEquals("expected:<[]bc> but was:<[a]bc>", failure);
    }
    public void testComparisonErrorOverlapingMatches() {
        String failure =
            new ComparisonCompactor(0, "abc", "abbc").compact(null);
        assertEquals("expected:<...[]...> but was:<...[b]...>", failure);
    }
    public void testComparisonErrorOverlapingMatchesContext() {
        String failure =
            new ComparisonCompactor(2, "abc", "abbc").compact(null);
        assertEquals("expected:<ab[]c> but was:<ab[b]c>", failure);
    }
    public void testComparisonErrorOverlapingMatches2() {
        String failure =
            new ComparisonCompactor(0, "abcdde", "abcde").compact(null);
        assertEquals("expected:<...[d]...> but was:<...[]...>", failure);
    }
    public void testComparisonErrorOverlapingMatches2Context() {
        String failure =
            new ComparisonCompactor(2, "abcdde", "abcde").compact(null);
        assertEquals("expected:<...cd[d]e> but was:<...cd[]e>", failure);
    }
    public void testComparisonErrorWithActualNull() {
        String failure = new ComparisonCompactor(0, "a", null).compact(null);
        assertEquals("expected:<a> but was:<null>", failure);
    }
    public void testComparisonErrorWithActualNullContext() {
        String failure = new ComparisonCompactor(2, "a", null).compact(null);
        assertEquals("expected:<a> but was:<null>", failure);
    }
    public void testComparisonErrorWithExpectedNull() {
        String failure = new ComparisonCompactor(0, null, "a").compact(null);
        assertEquals("expected:<null> but was:<a>", failure);
    }
    public void testComparisonErrorWithExpectedNullContext() {
        String failure = new ComparisonCompactor(2, null, "a").compact(null);
        assertEquals("expected:<null> but was:<a>", failure);
    }
    public void testBug609972() {
        String failure =
            new ComparisonCompactor(10, "S&P500", "0").compact(null);
        assertEquals("expected:<[S&P50]0> but was:<[]0>", failure);
    }
}
```

<div dir="rtl">

من یک تحلیل پوشش کد (Code Coverage) روی ماژول `ComparisonCompactor` با استفاده از این تست‌ها انجام دادم.
کد به‌طور کامل پوشش داده شده است؛ هر خط کد، هر دستور `if` و هر حلقه `for` توسط تست‌ها اجرا می‌شود.
این موضوع به من اطمینان زیادی می‌دهد که کد به‌درستی کار می‌کند و احترام زیادی برای مهارت نویسندگان آن ایجاد می‌کند.

کد مربوط به `ComparisonCompactor` در لیستینگ 15-2 قرار دارد.
کمی وقت بگذارید و این کد را مرور کنید. فکر می‌کنم آن را خوش‌ساخت، نسبتاً گویا، و از نظر ساختار ساده خواهید یافت.
وقتی مرور آن را تمام کردید، با هم وارد جزئیات دقیق‌تر خواهیم شد.

</div>


**Listing 15-2 -- ComparisonCompactor.java (Original)**

```java
package junit.framework;
public class ComparisonCompactor {
    private static final String ELLIPSIS = "...";
    private static final String DELTA_END = "]";
    private static final String DELTA_START = "[";
    private int fContextLength;
    private String fExpected;
    private String fActual;
    private int fPrefix;
    private int fSuffix;
    public ComparisonCompactor(
        int contextLength, String expected, String actual) {
        fContextLength = contextLength;
        fExpected = expected;
        fActual = actual;
    }
    public String compact(String message) {
        if (fExpected == null || fActual == null || areStringsEqual())
            return Assert.format(message, fExpected, fActual);
        findCommonPrefix();
        findCommonSuffix();
        String expected = compactString(fExpected);
        String actual = compactString(fActual);
        return Assert.format(message, expected, actual);
    }
    private String compactString(String source) {
        String result = DELTA_START
            + source.substring(fPrefix, source.length() - fSuffix + 1)
            + DELTA_END;
        if (fPrefix > 0)
            result = computeCommonPrefix() + result;
        if (fSuffix > 0)
            result = result + computeCommonSuffix();
        return result;
    }
    private void findCommonPrefix() {
        fPrefix = 0;
        int end = Math.min(fExpected.length(), fActual.length());
        for (; fPrefix < end; fPrefix++) {
            if (fExpected.charAt(fPrefix) != fActual.charAt(fPrefix))
                break;
        }
    }
    private void findCommonSuffix() {
        int expectedSuffix = fExpected.length() - 1;
        int actualSuffix = fActual.length() - 1;
        for (; actualSuffix >= fPrefix && expectedSuffix >= fPrefix;
             actualSuffix--, expectedSuffix--) {
            if (fExpected.charAt(expectedSuffix)
                != fActual.charAt(actualSuffix))
                break;
        }
        fSuffix = fExpected.length() - expectedSuffix;
    }
    private String computeCommonPrefix() {
        return (fPrefix > fContextLength ? ELLIPSIS : "")
            + fExpected.substring(
                Math.max(0, fPrefix - fContextLength), fPrefix);
    }
    private String computeCommonSuffix() {
        int end = Math.min(fExpected.length() - fSuffix + 1 + fContextLength,
            fExpected.length());
        return fExpected.substring(fExpected.length() - fSuffix + 1, end)
            + (fExpected.length() - fSuffix + 1
                        < fExpected.length() - fContextLength
                    ? ELLIPSIS
                    : "");
    }
    private boolean areStringsEqual() {
        return fExpected.equals(fActual);
    }
}
```

<div dir="rtl">

ممکن است چند ایراد به این ماژول داشته باشید.
برخی عبارات طولانی هستند و تعدادی `+1` عجیب و چیزهایی از این دست وجود دارد.
اما در کل، این ماژول نسبتاً خوب است.
در هر صورت، ممکن بود شبیه لیستینگ 15-3 باشد.

</div>

**Listing 15-3 -- ComparisonCompator.java (defactored)**

```java
package junit.framework;
public class ComparisonCompactor {
    private int ctxt;
    private String s1;
    private String s2;
    private int pfx;
    private int sfx;
    public ComparisonCompactor(int ctxt, String s1, String s2) {
        this.ctxt = ctxt;
        this.s1 = s1;
        this.s2 = s2;
    }
    public String compact(String msg) {
        if (s1 == null || s2 == null || s1.equals(s2))
            return Assert.format(msg, s1, s2);
        pfx = 0;
        for (; pfx < Math.min(s1.length(), s2.length()); pfx++) {
            if (s1.charAt(pfx) != s2.charAt(pfx))
                break;
        }
        int sfx1 = s1.length() - 1;
        int sfx2 = s2.length() - 1;
        for (; sfx2 >= pfx && sfx1 >= pfx; sfx2--, sfx1--) {
            if (s1.charAt(sfx1) != s2.charAt(sfx2))
                break;
        }
        sfx = s1.length() - sfx1;
        String cmp1 = compactString(s1);
        String cmp2 = compactString(s2);
        return Assert.format(msg, cmp1, cmp2);
    }
    private String compactString(String s) {
        String result = "[" + s.substring(pfx, s.length() - sfx + 1) + "]";
        if (pfx > 0)
            result = (pfx > ctxt ? "..." : "")
                + s1.substring(Math.max(0, pfx - ctxt), pfx) + result;
        if (sfx > 0) {
            int end = Math.min(s1.length() - sfx + 1 + ctxt, s1.length());
            result = result
                + (s1.substring(s1.length() - sfx + 1, end)
                    + (s1.length() - sfx + 1 < s1.length() - ctxt ? "..."
                                                                  : ""));
        }
        return result;
    }
}
```

<div dir="rtl">

با اینکه نویسندگان این ماژول را در وضعیت بسیار خوبی باقی گذاشته‌اند،
**Boy Scout Rule** به ما می‌گوید که باید آن را تمیزتر از زمانی که آن را یافتیم، ترک کنیم.
پس، چگونه می‌توانیم کد اصلی در لیستینگ 15-2 را بهبود دهیم؟

اولین چیزی که برایم خوشایند نیست، پیشوند `f` برای متغیرهای عضو است `[N6]`.
امروزه محیط‌های توسعه این نوع کدگذاری دامنه را غیرضروری کرده‌اند.
بنابراین بیایید تمام `f` ها را حذف کنیم.
</div>

```java
private int contextLength;
private String expected;
private String actual;
private int prefix;
private int suffix;
```

<div dir="rtl">

مورد بعدی، یک شرط بدون انکپسوله‌شده در ابتدای تابع `compact` است `[G28]`.

</div>

```java
public String compact(String message) {
    if (expected == null || actual == null || areStringsEqual())
        return Assert.format(message, expected, actual);
    findCommonPrefix();
    findCommonSuffix();
    String expected = compactString(this.expected);
    String actual = compactString(this.actual);
    return Assert.format(message, expected, actual); 
}
```

<div dir="rtl">

این شرط باید انکپسوله شود تا هدف ما واضح‌تر گردد. پس بیایید یک متد استخراج کنیم که توضیح‌دهنده باشد.

</div>

```java
public String compact(String message) {
    if (shouldNotCompact())
        return Assert.format(message, expected, actual);
    findCommonPrefix();
    findCommonSuffix();
    String expected = compactString(this.expected);
    String actual = compactString(this.actual);
    return Assert.format(message, expected, actual);
}

private boolean shouldNotCompact() {
    return expected == null || actual == null || areStringsEqual();
}
```

<div dir="rtl">

من زیاد از استفاده از `this.expected` و `this.actual` در تابع `compact` خوشم نمی‌آید.

این مورد وقتی اتفاق افتاد که نام `fExpected` به `expected` تغییر یافت.
چرا در این تابع متغیرهایی با همان نام متغیرهای عضو وجود دارد؟
آیا آن‌ها نمایانگر چیز دیگری نیستند؟ `[N4]` باید نام‌ها را واضح‌تر کنیم.

</div>

```java
String compactExpected = compactString(expected);
String compactActual = compactString(actual);
```

<div dir="rtl">

جملات منفی کمی سخت‌تر از جملات مثبت قابل‌درک‌اند `[G29]`.
پس بیایید شرط را برعکس کرده و معنای آن را تغییر دهیم.

</div>

```java
public String compact(String message) {
    if (canBeCompacted()) {
        findCommonPrefix();
        findCommonSuffix();
        String compactExpected = compactString(expected);
        String compactActual = compactString(actual);
        return Assert.format(message, compactExpected, compactActual);
    } else {
        return Assert.format(message, expected, actual);
    }
}

private boolean canBeCompacted() {
    return expected != null && actual != null && !areStringsEqual();
}

```

<div dir="rtl">

نام این تابع کمی عجیب است `[N7]`.
اگرچه رشته‌ها را فشرده می‌کند، اما ممکن است اصلاً فشرده‌سازی انجام ندهد اگر `canBeCompacted` مقدار `false` بازگرداند.
پس نام تابع `compact` اثر جانبی بررسی خطا را پنهان می‌کند.
همچنین توجه کنید که این تابع یک پیام قالب‌بندی‌شده بازمی‌گرداند، نه فقط رشته‌های فشرده‌شده.

پس نام تابع باید واقعاً `formatCompactedComparison` باشد.
این نام با توجه به آرگومان آن، خوانایی را بسیار بیشتر می‌کند:

</div>

```java
public String formatCompactedComparison(String message)
```

<div dir="rtl">

بدنه‌ی if جایی است که فشرده‌سازی واقعی رشته‌های `expected` و `actual` انجام می‌شود.
باید آن را به متدی با نام `compactExpectedAndActual` استخراج کنیم.

با این حال، می‌خواهیم تابع `formatCompactedComparison` تمام فرمت‌بندی را انجام دهد.
تابع `compact...` نباید کاری جز فشرده‌سازی انجام دهد `[G30]`.
پس بیایید آن را به این شکل تقسیم کنیم:

</div>

```java
private String compactExpected;
private String compactActual;

public String formatCompactedComparison(String message) {
    if (canBeCompacted()) {
        compactExpectedAndActual();
        return Assert.format(message, compactExpected, compactActual);
    } else {
        return Assert.format(message, expected, actual);
    }
}

private void compactExpectedAndActual() {
    findCommonPrefix();
    findCommonSuffix();
    compactExpected = compactString(expected);
    compactActual = compactString(actual);
}
```

<div dir="rtl">

توجه کنید که مجبور شدیم متغیرهای `compactExpected` و `compactActual` را به متغیرهای عضو ارتقا دهیم.
از اینکه دو خط آخر تابع جدید مقدار بازمی‌گردانند ولی دو خط اول این‌گونه نیستند، خوشم نمی‌آید.
آن‌ها از قرارداد یکسانی استفاده نمی‌کنند `[G11]`.
پس باید `findCommonPrefix` و `findCommonSuffix` را تغییر دهیم تا مقدار `prefix` و `suffix` را بازگردانند.

</div>

```java
private void compactExpectedAndActual() {
    prefixIndex = findCommonPrefix();
    suffixIndex = findCommonSuffix();
    compactExpected = compactString(expected);
    compactActual = compactString(actual);
}

private int findCommonPrefix() {
    int prefixIndex = 0;
    int end = Math.min(expected.length(), actual.length());
    for (; prefixIndex < end; prefixIndex++) {
        if (expected.charAt(prefixIndex) != actual.charAt(prefixIndex))
            break;
    }
    return prefixIndex;
}

private int findCommonSuffix() {
    int expectedSuffix = expected.length() - 1;
    int actualSuffix = actual.length() - 1;
    for (; actualSuffix >= prefixIndex && expectedSuffix >= prefixIndex;
         actualSuffix--, expectedSuffix--) {
        if (expected.charAt(expectedSuffix) != actual.charAt(actualSuffix))
            break;
    }
    return expected.length() - expectedSuffix;
}
```

<div dir="rtl">

باید نام متغیرهای عضو را نیز کمی دقیق‌تر کنیم `[N1]`؛
در نهایت، هر دو متغیر، اندیس هستند.

بررسی دقیق `findCommonSuffix` یک وابستگی زمانی پنهان را نشان می‌دهد `[G31]`؛
این تابع به این وابسته است که `prefixIndex` قبلاً توسط `findCommonPrefix` محاسبه شده باشد.
اگر این دو تابع به ترتیب اشتباهی فراخوانی شوند، دیباگ کردن بسیار دشوار خواهد شد.
پس برای نمایان کردن این وابستگی زمانی، بیایید `findCommonSuffix` را طوری تغییر دهیم که `prefixIndex` را به‌عنوان آرگومان دریافت کند.

</div>

```java
private void compactExpectedAndActual() {
    prefixIndex = findCommonPrefix();
    suffixIndex = findCommonSuffix(prefixIndex);
    compactExpected = compactString(expected);
    compactActual = compactString(actual);
}

private int findCommonSuffix(int prefixIndex) {
    int expectedSuffix = expected.length() - 1;
    int actualSuffix = actual.length() - 1;
    for (; actualSuffix >= prefixIndex && expectedSuffix >= prefixIndex; 
         actualSuffix--, expectedSuffix--) {
        if (expected.charAt(expectedSuffix) != actual.charAt(actualSuffix))
            break;
    }
    return expected.length() - expectedSuffix;
}
```

<div dir="rtl">

از این موضوع واقعاً خوشم نمی‌آید.
ارسال آرگومان `prefixIndex` کمی دلبخواهی است `[G32]`.
این کار ترتیب فراخوانی را تضمین می‌کند اما دلیلی برای نیاز به این ترتیب ارائه نمی‌دهد.
برنامه‌نویس دیگری ممکن است آنچه را انجام داده‌ایم، بازگرداند چون هیچ نشانه‌ای از ضروری بودن آن پارامتر وجود ندارد.
پس بیایید مسیر متفاوتی را انتخاب کنیم.

</div>

```java
private void compactExpectedAndActual() {
    findCommonPrefixAndSuffix();
    compactExpected = compactString(expected);
    compactActual = compactString(actual);
}

private void findCommonPrefixAndSuffix() {
    findCommonPrefix();
    int expectedSuffix = expected.length() - 1;
    int actualSuffix = actual.length() - 1;
    for (; actualSuffix >= prefixIndex && expectedSuffix >= prefixIndex;
         actualSuffix--, expectedSuffix--) {
        if (expected.charAt(expectedSuffix) != actual.charAt(actualSuffix))
            break;
    }
    suffixIndex = expected.length() - expectedSuffix;
}

private void findCommonPrefix() {
    prefixIndex = 0;
    int end = Math.min(expected.length(), actual.length());
    for (; prefixIndex < end; prefixIndex++)
        if (expected.charAt(prefixIndex) != actual.charAt(prefixIndex))
            break;
}
```

<div dir="rtl">

ما `findCommonPrefix` و `findCommonSuffix` را به حالت قبلی بازگرداندیم،
نام `findCommonSuffix` را به `findCommonPrefixAndSuffix` تغییر دادیم و آن را طوری نوشتیم که قبل از انجام هر کار دیگری `findCommonPrefix` را فراخوانی کند.
این کار ماهیت زمانی بین این دو تابع را به‌شکل خیلی واضح‌تری نشان می‌دهد نسبت به راه‌حل قبلی.

همچنین مشخص می‌کند که `findCommonPrefixAndSuffix` چقدر زشت است.
بیایید آن را تمیزتر کنیم.

</div>

```java
private void findCommonPrefixAndSuffix() {
    findCommonPrefix();
    int suffixLength = 1;
    for (; !suffixOverlapsPrefix(suffixLength); suffixLength++) {
        if (charFromEnd(expected, suffixLength) != 
            charFromEnd(actual, suffixLength))
            break;
    }
    suffixIndex = suffixLength;
}

private char charFromEnd(String s, int i) {
    return s.charAt(s.length() - i);
}

private boolean suffixOverlapsPrefix(int suffixLength) {
    return actual.length() - suffixLength < prefixIndex ||
           expected.length() - suffixLength < prefixIndex;
}
```

<div dir="rtl">

این بسیار بهتر است.
نشان می‌دهد که `suffixIndex` در واقع طول پسوند است و نام‌گذاری خوبی ندارد.
همین موضوع در مورد `prefixIndex` نیز صدق می‌کند، هرچند در آنجا «اندیس» و «طول» مترادف هستند.

با این حال، استفاده از `length` از نظر مفهومی منسجم‌تر است.
مشکل این است که متغیر `suffixIndex` از صفر شروع نمی‌شود؛ مقدار آن از یک شروع می‌شود و بنابراین یک طول واقعی نیست.
این همان دلیلی است که باعث شده در تابع `computeCommonSuffix` از آن +1ها استفاده شود `[G33]`.
پس بیایید آن را اصلاح کنیم.

نتیجه در لیستینگ 15-4 آمده است.

</div>

```java
public class ComparisonCompactor {
    ... private int suffixLength;
    ... private void findCommonPrefixAndSuffix() {
        findCommonPrefix();
        suffixLength = 0;
        for (; !suffixOverlapsPrefix(suffixLength); suffixLength++) {
            if (charFromEnd(expected, suffixLength)
                != charFromEnd(actual, suffixLength))
                break;
        }
    }
    private char charFromEnd(String s, int i) {
        return s.charAt(s.length() - i - 1);
    }
    private boolean suffixOverlapsPrefix(int suffixLength) {
        return actual.length() - suffixLength <= prefixLength
            || expected.length() - suffixLength <= prefixLength;
    }
    ...
    
    private String compactString(String source) {
        String result = DELTA_START
            + source.substring(prefixLength, source.length() - suffixLength)
            + DELTA_END;
        if (prefixLength > 0)
            result = computeCommonPrefix() + result;
        if (suffixLength > 0)
            result = result + computeCommonSuffix();
        return result;
    }
    ...
    
    private String computeCommonSuffix() {
        int end = Math.min(expected.length() - suffixLength + contextLength,
            expected.length());
        return expected.substring(expected.length() - suffixLength, end)
            + (expected.length() - suffixLength
                        < expected.length() - contextLength
                    ? ELLIPSIS
                    : "");
    }
}
```

<div dir="rtl">

ما علامت‌های `+1` را در تابع `computeCommonSuffix` با یک -1 در تابع `charFromEnd` جایگزین کردیم، جایی که کاملاً منطقی است، و همچنین از دو عملگر <= در تابع `suffixOverlapsPrefix` استفاده کردیم، که در آنجا هم کاملاً منطقی هستند. این تغییر به ما اجازه داد تا نام متغیر `suffixIndex` را به `suffixLength` تغییر دهیم، که خوانایی کد را به طرز چشمگیری افزایش داد.

با این حال، یک مشکل وجود دارد. هنگامی که داشتم علامت‌های `+1` را حذف می‌کردم، به خط زیر در تابع `compactString` برخورد کردم:

</div>

```java
if (suffixLength > 0)
```

<div dir="rtl">

نگاهی به آن در لیست 15-4 بیندازید. از نظر منطقی، حالا که مقدار `suffixLength` یک واحد کمتر از مقدار قبلی‌اش است، باید عملگر > را به >= تغییر دهم. اما این کار بی‌معناست. حالا منطقی است! این بدان معناست که قبلاً بی‌معنا بوده و احتمالاً یک باگ بوده است.

البته نه دقیقاً یک باگ. با بررسی بیشتر می‌بینیم که این دستور شرطی اکنون از اضافه شدن یک پسوند با طول صفر جلوگیری می‌کند. قبل از اینکه این تغییر را ایجاد کنیم، آن دستور شرطی بی‌اثر بود، چون مقدار `suffixIndex` هیچ‌گاه کمتر از یک نمی‌توانست باشد!

این مسئله هر دو دستور شرطی موجود در تابع `compactString` را زیر سؤال می‌برد! به نظر می‌رسد که هر دوی آن‌ها را می‌توان حذف کرد. پس بیایید آن‌ها را کامنت کنیم و تست‌ها را اجرا کنیم. همه‌ی تست‌ها پاس شدند! حالا بیایید تابع `compactString` را بازسازی کنیم تا دستورات شرطی اضافی را حذف کرده و تابع را بسیار ساده‌تر کنیم `[G9]`.

</div>

```java
private String compactString(String source) {
 return
 computeCommonPrefix() +
 DELTA_START +
 source.substring(prefixLength, source.length() - suffixLength) +
 DELTA_END +
 computeCommonSuffix();
}
```

<div dir="rtl">

این خیلی بهتر است! حالا می‌بینیم که تابع `compactString` به سادگی در حال کنار هم گذاشتن قطعه‌هاست. احتمالاً می‌توانیم این را حتی واضح‌تر کنیم. در واقع، پاکسازی‌های کوچکی زیادی هست که می‌توان انجام داد. اما به جای اینکه شما را درگیر بقیه تغییرات کنم، فقط نتیجه نهایی را در لیست 15-5 نشان می‌دهم.

</div>

**Listing 15-5 -- ComparisonCompactor.java (final)**

```java
package junit.framework;
public class ComparisonCompactor {
    private static final String ELLIPSIS = "...";
    private static final String DELTA_END = "]";
    private static final String DELTA_START = "[";
    private int contextLength;
    private String expected;
    private String actual;
    private int prefixLength;
    private int suffixLength;
    public ComparisonCompactor(
        int contextLength, String expected, String actual) {
        this.contextLength = contextLength;
        this.expected = expected;
        this.actual = actual;
    }
    public String formatCompactedComparison(String message) {
        String compactExpected = expected;
        String compactActual = actual;
        if (shouldBeCompacted()) {
            findCommonPrefixAndSuffix();
            compactExpected = compact(expected);
            compactActual = compact(actual);
        }
        return Assert.format(message, compactExpected, compactActual);
    }
    private boolean shouldBeCompacted() {
        return !shouldNotBeCompacted();
    }
    private boolean shouldNotBeCompacted() {
        return expected == null || actual == null || expected.equals(actual);
    }
    private void findCommonPrefixAndSuffix() {
        findCommonPrefix();
        suffixLength = 0;
        for (; !suffixOverlapsPrefix(); suffixLength++) {
            if (charFromEnd(expected, suffixLength)
                != charFromEnd(actual, suffixLength))
                break;
        }
    }
    private char charFromEnd(String s, int i) {
        return s.charAt(s.length() - i - 1);
    }
    private boolean suffixOverlapsPrefix() {
        return actual.length() - suffixLength <= prefixLength
            || expected.length() - suffixLength <= prefixLength;
    }
    private void findCommonPrefix() {
        prefixLength = 0;
        int end = Math.min(expected.length(), actual.length());
        for (; prefixLength < end; prefixLength++)
            if (expected.charAt(prefixLength) != actual.charAt(prefixLength))
                break;
    }
    private String compact(String s) {
        return new StringBuilder()
            .append(startingEllipsis())
            .append(startingContext())
            .append(DELTA_START)
            .append(delta(s))
            .append(DELTA_END)
            .append(endingContext())
            .append(endingEllipsis())
            .toString();
    }
    private String startingEllipsis() {
        return prefixLength > contextLength ? ELLIPSIS : "";
    }
    private String startingContext() {
        int contextStart = Math.max(0, prefixLength - contextLength);
        int contextEnd = prefixLength;
        return expected.substring(contextStart, contextEnd);
    }
    private String delta(String s) {
        int deltaStart = prefixLength;
        int deltaEnd = s.length() - suffixLength;
        return s.substring(deltaStart, deltaEnd);
    }
    private String endingContext() {
        int contextStart = expected.length() - suffixLength;
        int contextEnd =
            Math.min(contextStart + contextLength, expected.length());
        return expected.substring(contextStart, contextEnd);
    }
    private String endingEllipsis() {
        return (suffixLength > contextLength ? ELLIPSIS : "");
    }
}
```

<div dir="rtl">

این در واقع خیلی زیباست. ماژول به گروهی از توابع تحلیلی و گروه دیگری از توابع ترکیبی تقسیم شده است. این توابع به‌صورت توپولوژیکی مرتب شده‌اند به‌طوری که تعریف هر تابع درست بعد از استفاده آن قرار می‌گیرد. تمام توابع تحلیلی اول آمده‌اند و تمام توابع ترکیبی آخر.

اگر با دقت نگاه کنید، متوجه خواهید شد که چند تصمیمی که قبلاً در این فصل گرفتم را معکوس کرده‌ام. به عنوان مثال، بعضی از متدهای استخراج‌شده را دوباره در تابع `formatCompactedComparison` گنجاندم و حس عبارت `shouldNotBeCompacted` را تغییر دادم. این کاملاً طبیعی است. اغلب یک بازسازی منجر به بازسازی دیگری می‌شود که منجر به برگرداندن تغییرات اول می‌شود. بازسازی یک فرایند تکراری است که پر از آزمایش و خطاست و به‌طور اجتناب‌ناپذیری به چیزی می‌رسد که احساس می‌کنیم شایسته یک حرفه‌ای است.

## نتیجه‌گیری
و به این ترتیب ما قانون پیش‌خدمت اسکات را رعایت کرده‌ایم. این ماژول را کمی تمیزتر از آنچه که پیدا کردیم، رها کرده‌ایم. نه اینکه قبلاً تمیز نبوده باشد. نویسندگان کار فوق‌العاده‌ای با آن انجام داده بودند. اما هیچ ماژولی از بهبود مصون نیست و هرکدام از ما مسئولیت داریم که کد را کمی بهتر از آنچه که پیدا کرده‌ایم، ترک کنیم.

</div>

* [فصل قبل](../14_Successive_Refinement/Successive_Refinement.md)
* [فصل بعد](../16_Refactoring_SerialDate/Refactoring_SerialDate.md)
