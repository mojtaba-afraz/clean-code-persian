# بازسازی SerialDate

![image](../../assets/image/16/img-16.1.png)

<div dir="rtl">

اگر به آدرس http://www.jfree.org/jcommon/index.php بروید، کتابخانه `JCommon` را خواهید یافت. در اعماق آن کتابخانه، یک پکیج به نام `org.jfree.date` وجود دارد. درون این پکیج، کلاسی به نام `SerialDate` وجود دارد. ما قصد داریم این کلاس را بررسی کنیم.

نویسنده `SerialDate` دیوید گیلبرت است. دیوید به وضوح یک برنامه‌نویس با تجربه و ماهر است. همانطور که خواهیم دید، او در کد خود درجه بالایی از حرفه‌ای بودن و انضباط را نشان می‌دهد. برای تمام مقاصد و اهداف، این "کد خوب" است. و من قصد دارم آن را تکه‌تکه کنم.

این فعالیت از روی بدخواهی نیست. همچنین فکر نمی‌کنم که من از دیوید خیلی بهتر باشم که به نوعی حق قضاوت در مورد کد او را داشته باشم. در واقع، اگر شما کدهایی از من پیدا کنید، مطمئناً می‌توانید چیزهای زیادی برای انتقاد پیدا کنید.

نه، این فعالیت نه از روی دشمنی است و نه از روی تکبر. آنچه من در حال انجام آن هستم چیزی بیشتر و کمتر از یک بازبینی حرفه‌ای نیست. چیزی است که باید همه ما با آن راحت باشیم. و چیزی است که باید آن را وقتی برای ما انجام می‌شود، خوش‌آمد بگوییم. تنها از طریق نقدهایی مانند این است که یاد خواهیم گرفت. پزشکان این کار را می‌کنند. خلبان‌ها این کار را می‌کنند. و وکلا این کار را می‌کنند. و ما برنامه‌نویسان هم باید یاد بگیریم که چگونه این کار را انجام دهیم.

یک نکته دیگر در مورد دیوید گیلبرت: دیوید بیشتر از یک برنامه‌نویس خوب است. دیوید شجاعت و حسن نیت داشت که کد خود را به صورت رایگان به جامعه عرضه کند. او آن را در معرض دید عموم قرار داد و استفاده و نقد عمومی را دعوت کرد. این کار بسیار خوب انجام شد!

کلاس `SerialDate` (لیست B-1، صفحه 349) کلاسی است که تاریخ را در جاوا نمایش می‌دهد. چرا باید کلاسی که تاریخ را نشان می‌دهد داشته باشیم، در حالی که جاوا قبلاً `java.util.Date` و `java.util.Calendar` و دیگران را دارد؟ نویسنده این کلاس را در پاسخ به دردی که من اغلب خودم تجربه کرده‌ام نوشت. نظر در `Javadoc` اولیه او (خط 67) این موضوع را به خوبی توضیح می‌دهد. ما می‌توانیم در مورد نیت او بحث کنیم، اما من قطعاً با این مشکل روبه‌رو شده‌ام و از کلاسی که مربوط به تاریخ‌ها است، به جای زمان‌ها، استقبال می‌کنم.

اول، کاری کن که کار کند
در یک کلاسی به نام `SerialDateTests` (لیست B-2، صفحه 366) برخی از تست‌های واحد وجود دارد. همه تست‌ها موفق هستند. متاسفانه یک بررسی سریع از تست‌ها نشان می‌دهد که همه چیز را تست نمی‌کنند `[T1]`. به عنوان مثال، انجام جستجوی "`Find Usages`" برای متد `MonthCodeToQuarter` (خط 334) نشان می‌دهد که از آن استفاده نمی‌شود [F4]. بنابراین، تست‌های واحد آن را تست نمی‌کنند.

بنابراین من `Clover` را راه‌اندازی کردم تا ببینم تست‌های واحد چه چیزی را پوشش می‌دهند و چه چیزی را نمی‌دهند. `Clover` گزارش داد که تست‌های واحد تنها 91 از 185 دستور اجرایی در `SerialDate` (~50 درصد) را اجرا کرده‌اند `[T2]`. نقشه پوشش مانند یک لحاف پچی است که بخش‌های بزرگی از کد اجرایی نشده در سراسر کلاس پراکنده است.

هدف من این بود که کاملاً این کلاس را درک کنم و همچنین آن را بازسازی کنم. من نمی‌توانستم این کار را بدون پوشش تست بسیار بیشتر انجام دهم. بنابراین من مجموعه‌ای از تست‌های واحد کاملاً مستقل نوشتم (لیست B-4، صفحه 374).

وقتی این تست‌ها را نگاه می‌کنید، متوجه خواهید شد که بسیاری از آن‌ها کامنت شده‌اند. این تست‌ها پاس نمی‌کنند. آن‌ها رفتارهایی را نمایان می‌کنند که من فکر می‌کنم `SerialDate` باید داشته باشد. بنابراین وقتی من `SerialDate` را بازسازی می‌کنم، سعی خواهم کرد این تست‌ها را هم پاس کنم.

حتی با وجود برخی از تست‌های کامنت شده، `Clover` گزارش می‌دهد که تست‌های واحد جدید 170 (92 درصد) از 185 دستور اجرایی را اجرا می‌کنند. این خیلی خوب است و فکر می‌کنم می‌توانیم این عدد را بیشتر کنیم.

چند تست اول که کامنت شده‌اند (خطوط 23-63) کمی خودخواهی از طرف من بود. برنامه برای عبور از این تست‌ها طراحی نشده بود، اما رفتار آن‌ها به وضوح برای من آشکار بود `[G2]`. مطمئن نیستم چرا متد `testWeekdayCodeToString` در ابتدا نوشته شده بود، اما چون وجود دارد، به نظر می‌رسید که نباید حساس به حروف باشد. نوشتن این تست‌ها کار کوچکی بود `[T3]`. درست کردن آن‌ها حتی راحت‌تر بود؛ من فقط خطوط 259 و 263 را تغییر دادم تا از `equalsIgnoreCase` استفاده کنم.

تست‌های خطوط 32 و 45 را کامنت گذاشتم چون برای من مشخص نیست که آیا باید اختصارات "`tues`" و "`thurs`" پشتیبانی شوند.

تست‌های خطوط 153 و 154 پاس نمی‌کنند. واضح است که باید پاس کنند `[G2]`. ما به راحتی می‌توانیم این مورد را اصلاح کنیم، و همچنین تست‌های خطوط 163 تا 213 را با انجام تغییرات زیر در تابع `stringToMonthCode`.

</div>

```java
if ((result < 1) || (result > 12)) {
    result = -1;
    for (int i = 0; i < monthNames.length; i++) {
        if (s.equalsIgnoreCase(shortMonthNames[i])) {
            result = i + 1;
            break;
        }
        if (s.equalsIgnoreCase(monthNames[i])) {
            result = i + 1;
            break;
        }
    }
}
```

<div dir="rtl">

تست کامنت‌گذاری‌شده در خط ۳۱۸ یک باگ را در متد `getFollowingDayOfWeek` (خط ۶۷۲) آشکار می‌کند. ۲۵ دسامبر ۲۰۰۴، روز شنبه بود. شنبه‌ی بعدی، اول ژانویه ۲۰۰۵ بود. با این حال، وقتی تست را اجرا می‌کنیم، می‌بینیم که `getFollowingDayOfWeek` تاریخ ۲۵ دسامبر را به‌عنوان شنبه‌ای که بعد از ۲۵ دسامبر می‌آید، برمی‌گرداند. به‌وضوح این اشتباه است [G3]، [T1]. مشکل را در خط ۶۸۵ می‌بینیم. این یک خطای معمول در شرایط مرزی است [T5]. باید به صورت زیر نوشته شود:

</div>

```java
if (baseDOW >= targetWeekday) {
```

<div dir="rtl">

جالب است که این تابع قبلاً یک‌بار اصلاح شده بود. تاریخچه تغییرات (خط ۴۳) نشان می‌دهد که «باگ‌هایی» در متدهای `getPreviousDayOfWeek،` `getFollowingDayOfWeek` و `getNearestDayOfWeek` رفع شده‌اند [T6].

تست واحد `testGetNearestDayOfWeek` (خط ۳۲۹)، که متد `getNearestDayOfWeek` (خط ۷۰۵) را تست می‌کند، در ابتدا به این اندازه طولانی و جامع نبود. من تست‌های زیادی به آن اضافه کردم چون تست‌های اولیه‌ام همگی موفق نبودند [T6]. می‌توانید الگوی خطا را با نگاه به تست‌هایی که کامنت شده‌اند ببینید. این الگو افشاگر است [T7]. نشان می‌دهد که الگوریتم وقتی روز نزدیک‌تر در آینده باشد، شکست می‌خورد. مشخصاً مشکلی مربوط به شرایط مرزی وجود دارد [T5].

الگوی پوشش تست که `Clover` گزارش داده نیز جالب است [T8]. خط ۷۱۹ هرگز اجرا نمی‌شود! این بدان معناست که عبارت `if` در خط ۷۱۸ همیشه مقدار `false` دارد. و واقعاً با نگاهی به کد مشخص می‌شود که همین‌طور است. متغیر `adjust` همیشه منفی است و بنابراین نمی‌تواند بزرگ‌تر یا مساوی ۴ باشد. بنابراین این الگوریتم اشتباه است. الگوریتم صحیح به صورت زیر است:

</div>

```java
int delta = targetDOW - base.getDayOfWeek();
int positiveDelta = delta + 7;
int adjust = positiveDelta % 7;
if (adjust > 3)
    adjust -= 7;
return SerialDate.addDays(adjust, base);
```

<div dir="rtl">

در نهایت، تست‌های خط ۴۱۷ و خط ۴۲۹ به‌سادگی با پرتاب یک `IllegalArgumentException` به‌جای برگرداندن یک رشته خطا از `weekInMonthToString` و `relativeToString،` قابل گذر کردن هستند.

با این تغییرات، همه تست‌های واحد پاس می‌شوند، و باور دارم که اکنون `SerialDate` به درستی کار می‌کند. بنابراین حالا وقت آن است که آن را «درست» کنیم.

## سپس آن را درست کن

ما از بالا به پایین در کلاس `SerialDate` حرکت خواهیم کرد و در طول مسیر آن را بهبود خواهیم داد. اگرچه شما این را در بحث نمی‌بینید، اما من تمام تست‌های واحد `JCommon،` شامل تست واحد بهبودیافته‌ی خودم برای `SerialDate` را بعد از هر تغییر اجرا می‌کنم. پس مطمئن باشید که هر تغییری که در اینجا می‌بینید، با کل `JCommon` کار می‌کند.

از خط ۱ شروع می‌کنیم. ما یک صفحه کامل از کامنت‌ها داریم که شامل اطلاعات مجوز، کپی‌رایت، نویسندگان و تاریخچه تغییرات است. من می‌پذیرم که بعضی مسائل قانونی باید رعایت شوند، پس کپی‌رایت و مجوزها باید باقی بمانند. اما تاریخچه تغییرات باقی‌مانده‌ای از دهه ۱۹۶۰ است. اکنون ابزارهای کنترل نسخه‌ای داریم که این کار را برایمان انجام می‌دهند. این تاریخچه باید حذف شود [C1].

لیست `import` از خط ۶۱ می‌تواند با استفاده از `java.text.*` و `java.util.*` کوتاه‌تر شود [J1].

من از قالب‌بندی `HTML` در `Javadoc` (خط ۶۷) رنج می‌برم. داشتن یک فایل سورس با بیش از یک زبان درون آن ناراحت‌کننده است. این کامنت شامل چهار زبان است: Java، انگلیسی، `Javadoc،` و HTML [G1]. با وجود این‌همه زبان، حفظ انسجام کار دشوار می‌شود. به عنوان مثال، مکان‌نمایی خوب خط ۷۱ و ۷۲ در زمان تولید `Javadoc` از بین می‌رود، و در عین حال، چه کسی می‌خواهد `<ul>` و `<li>` را در سورس‌کد ببیند؟ یک استراتژی بهتر شاید این باشد که کل کامنت را درون `<pre>` قرار دهیم تا قالب‌بندی موجود در سورس درون `Javadoc` نیز حفظ شود.

خط ۸۶، اعلان کلاس است. چرا نام این کلاس `SerialDate` است؟ اهمیت واژه `«serial»` چیست؟ آیا چون این کلاس از Serializable ارث‌بری کرده؟ به نظر نمی‌رسد.

شما را منتظر نمی‌گذارم. من می‌دانم (یا دست‌کم فکر می‌کنم می‌دانم) چرا از واژه «serial» استفاده شده. سرنخ در ثابت‌های `SERIAL_LOWER_BOUND` و `SERIAL_UPPER_BOUND` در خط ۹۸ و ۱۰۱ است. سرنخ بهتر، کامنتی است که از خط ۸۳۰ شروع می‌شود. این کلاس `«SerialDate»` نام گرفته زیرا با استفاده از یک «شماره سریال» پیاده‌سازی شده که در واقع تعداد روزهای گذشته از ۳۰ دسامبر ۱۸۹۹ است.

من دو مشکل با این دارم. اول اینکه واژه «شماره سریال» واقعاً درست نیست. این شاید ایراد کوچکی باشد، اما این نمایش بیشتر یک آفست نسبی است تا شماره سریال. واژه «شماره سریال» بیشتر در مورد شناسه محصولات استفاده می‌شود تا تاریخ‌ها. بنابراین این نام برای من توصیفی نیست [N1]. یک واژه توصیفی‌تر می‌تواند `«ordinal»` باشد.

مشکل دوم مهم‌تر است. نام `SerialDate` به نوعی به پیاده‌سازی اشاره دارد. این کلاس یک کلاس انتزاعی است. نیازی به اشاره به جزئیات پیاده‌سازی نیست. در واقع، دلایل خوبی برای مخفی کردن پیاده‌سازی وجود دارد! بنابراین من این نام را در سطح انتزاع اشتباه می‌دانم [N2]. به نظر من، نام این کلاس باید به‌سادگی `Date` باشد.

متأسفانه، در حال حاضر کلاس‌های زیادی در کتابخانه `Java` با نام `Date` وجود دارند، بنابراین این احتمالاً نام مناسبی نیست. از آنجا که این کلاس درباره روزها است، نه زمان، نام `Day` را نیز در نظر گرفتم، اما این نام نیز در جاهای دیگر زیاد استفاده شده. در نهایت، نام `DayDate` را به عنوان بهترین سازش انتخاب کردم.

از اینجا به بعد در این بحث از واژه `DayDate` استفاده می‌کنم. این به شما بستگی دارد که به خاطر بسپارید که لیستینگ‌هایی که مشاهده می‌کنید هنوز از `SerialDate` استفاده می‌کنند.

من درک می‌کنم که چرا `DayDate` از `Comparable` و `Serializable` ارث‌بری کرده. اما چرا از `MonthConstants` ارث‌بری کرده؟ کلاس `MonthConstants` (لیست B-3، صفحه ۳۷۲) فقط شامل یک‌سری ثابت `static final` است که ماه‌ها را تعریف می‌کنند. ارث‌بری از کلاس‌هایی که فقط شامل ثابت هستند یک ترفند قدیمی بین برنامه‌نویسان `Java` بوده تا مجبور نباشند از عباراتی مثل `MonthConstants`.`January` استفاده کنند، اما این کار ایده بدی است [J2]. `MonthConstants` واقعاً باید یک `enum` باشد.

</div>

```java
public abstract class DayDate implements Comparable, Serializable
{
    public static enum Month {
        JANUARY(1),
        FEBRUARY(2),
        MARCH(3),
        APRIL(4),
        MAY(5),
        JUNE(6),
        JULY(7),
        AUGUST(8),
        SEPTEMBER(9),
        OCTOBER(10),
        NOVEMBER(11),
        DECEMBER(12);
        Month(int index) { this.index = index; }
        public static Month make(int monthIndex)
        {
            for (Month m : Month.values()) {
                if (m.index == monthIndex)
                    return m;
            }
            throw new IllegalArgumentException("Invalid month index " +
                                               monthIndex);
        }
        public final int index;
    }
}
```


<div dir="rtl">

تبدیل کلاس `MonthConstants` به یک `enum` باعث تغییرات زیادی در کلاس `DayDate` و تمام استفاده‌کنندگان آن شد. انجام این تغییرات تقریباً یک ساعت زمان برد. با این حال، حالا هر تابعی که قبلاً یک عدد صحیح (`int`) برای ماه می‌پذیرفت، اکنون یک مقدار از نوع `Month` (`enum`) دریافت می‌کند. این یعنی می‌توانیم متد `isValidMonthCode` (خط ۳۲۶) و تمام بررسی‌های خطای مربوط به کد ماه، مانند آنچه در `monthCodeToQuarter` (خط ۳۵۶) وجود دارد، را حذف کنیم [G5].

در ادامه، به خط ۹۱ می‌رسیم: `serialVersionUID`. این متغیر برای کنترل فرایند `serialization` استفاده می‌شود. اگر آن را تغییر دهیم، هر آبجکت `DayDate` که با نسخه‌ی قدیمی نرم‌افزار نوشته شده باشد، دیگر قابل خواندن نخواهد بود و خطای `InvalidClassException` ایجاد خواهد شد. اگر این متغیر را تعریف نکنیم، کامپایلر به طور خودکار یک مقدار برای آن تولید می‌کند، و این مقدار هر بار که تغییری در ماژول داده شود، متفاوت خواهد بود. می‌دانم که تمام منابع توصیه می‌کنند کنترل این متغیر به‌صورت دستی انجام شود، اما به نظر من، کنترل خودکار `serialization` ایمن‌تر است [G4]. در نهایت، من ترجیح می‌دهم با یک `InvalidClassException` مواجه شوم تا با رفتارهای عجیب و غریبی که ممکن است در اثر فراموش کردن به‌روزرسانی `serialVersionUID` رخ دهد. بنابراین فعلاً این متغیر را حذف می‌کنم.²

کامنتی که در خط ۹۳ قرار دارد، تکراری است. کامنت‌های تکراری فقط مکان‌هایی برای جمع‌آوری دروغ‌ها و اطلاعات اشتباه هستند [C2]. بنابراین این کامنت و امثال آن را حذف می‌کنم.

کامنت‌های خط ۹۷ و ۱۰۰ درباره «شماره‌های سریال» صحبت می‌کنند که پیش‌تر درباره‌شان بحث شد [C1]. متغیرهایی که این کامنت‌ها توصیف می‌کنند در واقع تاریخ‌های حداقل و حداکثری هستند که `DayDate` می‌تواند آن‌ها را نمایش دهد. می‌توان این موضوع را با دقت بیشتری بیان کرد [N1].

</div>

```java
public static final int EARLIEST_DATE_ORDINAL = 2; // 1/1/1900
public static final int LATEST_DATE_ORDINAL = 2958465; // 12/31/9999
```

<div dir="rtl">

برای من مشخص نیست که چرا مقدار `EARLIEST_DATE_ORDINAL` برابر با ۲ است و نه ۰. در کامنت خط ۸۲۹ اشاره‌ای شده که این مسئله به نحوه‌ی نمایش تاریخ‌ها در `Microsoft Excel` مربوط می‌شود. توضیح دقیق‌تری از این موضوع در کلاسی مشتق‌شده از `DayDate` به نام `SpreadsheetDate` (فهرست B-5، صفحه ۳۸۲) آمده است. کامنت خط ۷۱ در آن کلاس مسئله را به‌خوبی شرح می‌دهد.

مشکل من اینجاست که این مسئله مربوط به پیاده‌سازی `SpreadsheetDate` است و ربطی به `DayDate` ندارد. بنابراین نتیجه می‌گیرم که متغیرهای `EARLIEST_DATE_ORDINAL` و `LATEST_DATE_ORDINAL` نباید در `DayDate` تعریف شوند و باید به کلاس `SpreadsheetDate` منتقل شوند [G6].

در واقع، بررسی کد نشان می‌دهد که این متغیرها تنها در داخل `SpreadsheetDate` استفاده می‌شوند. هیچ‌چیز در `DayDate` یا سایر کلاس‌های فریم‌ورک `JCommon` از آن‌ها استفاده نمی‌کند. پس آن‌ها را به `SpreadsheetDate` منتقل می‌کنم.

متغیرهای بعدی، `MINIMUM_YEAR_SUPPORTED` و `MAXIMUM_YEAR_SUPPORTED` (خطوط ۱۰۴ و ۱۰۷)، کمی مسئله‌سازتر هستند. به‌وضوح اگر `DayDate` یک کلاس انتزاعی است که قرار نیست چیزی از پیاده‌سازی را افشا کند، نباید درباره‌ی سال‌های پشتیبانی‌شده اطلاعاتی ارائه دهد. باز هم وسوسه می‌شوم این متغیرها را به `SpreadsheetDate` منتقل کنم [G6]. اما جست‌وجویی سریع در میان استفاده‌کنندگان این متغیرها نشان می‌دهد که کلاس دیگری نیز از آن‌ها استفاده می‌کند: `RelativeDayOfWeekRule` (فهرست B-6، صفحه ۳۹۰). استفاده‌ی آن را در خطوط ۱۷۷ و ۱۷۸ در تابع `getDate` می‌بینیم، جایی که از آن‌ها برای اعتبارسنجی سال ورودی استفاده می‌شود.

معضل این است که یک استفاده‌کننده از یک کلاس انتزاعی نیاز به دانستن جزئیاتی از پیاده‌سازی آن دارد. ما باید این اطلاعات را فراهم کنیم بدون اینکه کلاس `DayDate` را آلوده کنیم.

در حالت عادی، این نوع اطلاعات پیاده‌سازی را از یک نمونه‌ی مشتق‌شده دریافت می‌کنیم. اما تابع `getDate` نمونه‌ای از `DayDate` دریافت نمی‌کند، هرچند یکی را برمی‌گرداند. بنابراین باید آن را در جایی بسازد. خطوط ۱۸۷ تا ۲۰۵ این نکته را روشن می‌کنند. نمونه‌ی `DayDate` توسط یکی از سه تابع `getPreviousDayOfWeek،` `getNearestDayOfWeek` یا `getFollowingDayOfWeek` ساخته می‌شود. اگر به کد `DayDate` برگردیم، می‌بینیم این توابع (خطوط ۶۳۸ تا ۷۲۴) تاریخی را بازمی‌گردانند که توسط `addDays` (خط ۵۷۱) ساخته می‌شود، و آن هم از `createInstance` (خط ۸۰۸) استفاده می‌کند که در نهایت یک `SpreadsheetDate` ایجاد می‌کند! [G7]

در کل، ایده‌ی بدی است که یک کلاس پایه از مشتقات خودش اطلاع داشته باشد. برای حل این مسئله، باید از الگوی طراحی `Abstract Factory` استفاده کنیم و یک `DayDateFactory` بسازیم. این کارخانه می‌تواند نمونه‌های `DayDate` موردنیاز را ایجاد کرده و اطلاعاتی مانند حداقل و حداکثر تاریخ‌های پشتیبانی‌شده را نیز ارائه دهد.

</div>

```java
public abstract class DayDateFactory
{
    private static DayDateFactory factory = new SpreadsheetDateFactory();
    public static void setInstance(DayDateFactory factory)
    {
        DayDateFactory.factory = factory;
    }
    protected abstract DayDate _makeDate(int ordinal);
    protected abstract DayDate _makeDate(int day,
                                         DayDate.Month month,
                                         int year);
    protected abstract DayDate _makeDate(int day, int month, int year);
    protected abstract DayDate _makeDate(java.util.Date date);
    protected abstract int _getMinimumYear();
    protected abstract int _getMaximumYear();
    public static DayDate makeDate(int ordinal)
    {
        return factory._makeDate(ordinal);
    }
    public static DayDate makeDate(int day, DayDate.Month month, int year)
    {
        return factory._makeDate(day, month, year);
    }
    public static DayDate makeDate(int day, int month, int year)
    {
        return factory._makeDate(day, month, year);
    }
    public static DayDate makeDate(java.util.Date date)
    {
        return factory._makeDate(date);
    }
    public static int getMinimumYear() { return factory._getMinimumYear(); }
    public static int getMaximumYear() { return factory._getMaximumYear(); }
}
```


<div dir="rtl">

این کلاس کارخانه‌ای جایگزین متدهای `createInstance` با متدهایی به نام `makeDate` می‌شود که نام‌هایی بسیار بهتر و توصیفی‌تر هستند [N1]. به‌صورت پیش‌فرض از `SpreadsheetDateFactory` استفاده می‌شود، اما می‌توان در هر زمانی آن را تغییر داد تا از کارخانه‌ی متفاوتی استفاده کند. متدهای ایستا که به متدهای انتزاعی واگذار می‌شوند، ترکیبی از الگوهای طراحی `Singleton،` `Decorator،` و `Abstract Factory` را به‌کار می‌گیرند که من آن‌ها را مفید یافته‌ام.

کلاس `SpreadsheetDateFactory` به این شکل است.

</div>

```java
public class SpreadsheetDateFactory extends DayDateFactory
{
    public DayDate _makeDate(int ordinal)
    {
        return new SpreadsheetDate(ordinal);
    }
    public DayDate _makeDate(int day, DayDate.Month month, int year)
    {
        return new SpreadsheetDate(day, month, year);
    }
    public DayDate _makeDate(int day, int month, int year)
    {
        return new SpreadsheetDate(day, month, year);
    }
    public DayDate _makeDate(Date date)
    {
        final GregorianCalendar calendar = new GregorianCalendar();
        calendar.setTime(date);
        return new SpreadsheetDate(
          calendar.get(Calendar.DATE),
          DayDate.Month.make(calendar.get(Calendar.MONTH) + 1),
          calendar.get(Calendar.YEAR));
    }
    protected int _getMinimumYear()
    {
        return SpreadsheetDate.MINIMUM_YEAR_SUPPORTED;
    }
    protected int _getMaximumYear()
    {
        return SpreadsheetDate.MAXIMUM_YEAR_SUPPORTED;
    }
}
```

<div dir="rtl">

همان‌طور که می‌بینید، من متغیرهای `MINIMUM_YEAR_SUPPORTED` و `MAXIMUM_YEAR_SUPPORTED` را به `SpreadsheetDate` منتقل کرده‌ام، جایی که به آن تعلق دارند [G6].

مسئله‌ی بعدی در `DayDate` مربوط به ثابت‌های مربوط به روزهای هفته است که از خط 109 شروع می‌شوند. این ثابت‌ها واقعاً باید به یک `enum` دیگر تبدیل شوند [J3]. ما قبلاً با این الگو مواجه شده‌ایم، بنابراین آن را اینجا تکرار نمی‌کنم—می‌توانید آن را در فهرست نهایی ببینید.

سپس به مجموعه‌ای از جدول‌ها می‌رسیم که با `LAST_DAY_OF_MONTH` در خط 140 آغاز می‌شوند. اولین مسئله‌ی من با این جدول‌ها، کامنت‌هایی است که آن‌ها را توصیف کرده‌اند—این کامنت‌ها زائد هستند [C3]؛ نام جدول‌ها به‌تنهایی گویا و کافی است. بنابراین آن‌ها را حذف می‌کنم.

به‌نظر می‌رسد دلیل خوبی وجود ندارد که این جدول `private` نباشد [G8]، چرا که یک تابع استاتیک به نام `lastDayOfMonth` وجود دارد که همین داده‌ها را ارائه می‌دهد.

جدول بعدی، `AGGREGATE_DAYS_TO_END_OF_MONTH` کمی مرموز است، چراکه در هیچ‌جای فریم‌ورک `JCommon` استفاده نمی‌شود [G9]. بنابراین آن را حذف کردم.

همین موضوع برای `LEAP_YEAR_AGGREGATE_DAYS_TO_END_OF_MONTH` نیز صدق می‌کند.

جدول بعدی، `AGGREGATE_DAYS_TO_END_OF_PRECEDING_MONTH` تنها در `SpreadsheetDate` (خطوط 434 و 473) استفاده می‌شود. این موضوع این سؤال را پیش می‌آورد که آیا این جدول باید به `SpreadsheetDate` منتقل شود؟ استدلال مخالف این است که جدول برای هیچ پیاده‌سازی خاصی نیست [G6]. اما از آنجا که هیچ پیاده‌سازی دیگری برای `DayDate` وجود ندارد، بهتر است جدول نزدیک جایی قرار گیرد که واقعاً از آن استفاده می‌شود [G10].

آنچه باعث شد تصمیمم نهایی شود این بود که برای حفظ یکپارچگی و انسجام [G11]، باید جدول را `private` کنیم و در صورت نیاز از طریق تابعی مانند `julianDateOfLastDayOfMonth` آن را در دسترس قرار دهیم. ولی ظاهراً هیچ‌کس به چنین تابعی نیاز ندارد. افزون بر آن، در صورت نیاز پیاده‌سازی جدید، انتقال دوباره جدول به `DayDate` کار ساده‌ای است. بنابراین جدول را منتقل کردم.

همین منطق برای جدول `LEAP_YEAR_AGGREGATE_DAYS_TO_END_OF_MONTH` نیز صدق می‌کند.

در ادامه، با سه دسته از ثابت‌ها روبرو می‌شویم که می‌توان آن‌ها را به `enum` تبدیل کرد (خطوط 162 تا 205). اولین دسته، هفته‌ای را در ماه انتخاب می‌کند. من آن را به `enum` ‌ای به نام `WeekInMonth` تبدیل کردم.

</div>

```java
public enum WeekInMonth
{
    FIRST(1),
    SECOND(2),
    THIRD(3),
    FOURTH(4),
    LAST(0);
    public final int index;
    WeekInMonth(int index) { this.index = index; }
}
```

<div dir="rtl">

دومین مجموعه از ثابت‌ها (خطوط 187–177) کمی مبهم‌تر هستند. ثابت‌های `INCLUDE_NONE،` `INCLUDE_FIRST`، `INCLUDE_SECOND` و `INCLUDE_BOTH` برای تعیین این موضوع استفاده می‌شوند که آیا نقاط انتهایی یک بازه زمانی باید در آن بازه لحاظ شوند یا نه. از دیدگاه ریاضی، این حالت‌ها با اصطلاحاتی مانند “بازه باز”، “بازه نیمه‌باز” و “بازه بسته” توصیف می‌شوند. به نظرم استفاده از نام‌گذاری ریاضی روشن‌تر است [N3]، بنابراین آن‌ها را به یک `enum` به نام `DateInterval` با مقادیر `CLOSED`, `CLOSED_LEFT`، `CLOSED_RIGHT` و `OPEN` تبدیل کردم.

سومین مجموعه از ثابت‌ها (خطوط 205–187) مشخص می‌کنند که در جستجوی یک روز خاص از هفته، باید آخرین، بعدی یا نزدیک‌ترین نمونه آن انتخاب شود. انتخاب نام مناسب برای این مورد کار ساده‌ای نیست. در نهایت، نام `WeekdayRange` را انتخاب کردم که شامل مقادیر `LAST`، `NEXT` و `NEAREST` می‌شود.

ممکن است با نام‌هایی که انتخاب کرده‌ام موافق نباشید. این نام‌ها برای من معنا دارند، ولی شاید برای شما نداشته باشند. نکته‌ی اصلی این است که این مقادیر اکنون در قالبی هستند که تغییر دادنشان راحت است [J3]. دیگر به صورت اعداد صحیح (`int`) منتقل نمی‌شوند، بلکه به صورت نمادهای معنادار ارسال می‌شوند. حالا می‌توانم با استفاده از قابلیت «تغییر نام» در IDE، به‌راحتی نام یا نوع آن‌ها را عوض کنم، بدون نگرانی از اینکه در کجای کد یک -1 یا 2 جا مانده یا اینکه یک پارامتر `int` به‌درستی توصیف نشده باشد.

فیلد `description` در خط 208 به نظر نمی‌رسد که مورد استفاده قرار گرفته باشد. بنابراین همراه با متدهای `getter` و `setter` آن حذفش کردم [G9].

همچنین سازنده‌ی پیش‌فرض و ناکارآمد در خط 213 را نیز حذف کردم [G12]؛ چون کامپایلر به‌صورت خودکار آن را برای ما تولید خواهد کرد.

می‌توانیم از متد `isValidWeekdayCode` (خطوط 216 تا 238) عبور کنیم چون آن را هنگام ساخت `enum` مربوط به روز هفته حذف کردیم.

اکنون به متد `stringToWeekdayCode` (خطوط 242 تا 270) می‌رسیم. `Javadocهایی` که چیزی بیش از امضای متد به ما نمی‌دهند، فقط مزاحم هستند [C3], [G12]. تنها ارزش این `Javadoc` توضیح مقدار بازگشتی -1 بود. اما حالا که از `enum Day` استفاده می‌کنیم، این توضیح دیگر درست نیست [C2]. این متد اکنون در صورت بروز خطا، `IllegalArgumentException` پرتاب می‌کند. بنابراین `Javadoc` را حذف کردم.

تمام کلیدواژه‌های `final` در آرگومان‌ها و تعریف متغیرها را نیز حذف کردم. تا جایی که بررسی کردم، آن‌ها ارزش چندانی نداشتند ولی باعث شلوغی کد شده بودند [G12]. حذف `final` برخلاف برخی توصیه‌های رایج است. برای مثال، رابرت سیمونز تأکید دارد که باید "`final` را همه‌جا در کدت پخش کنی." ولی من موافق نیستم. معتقدم `final` در موارد معدودی مثل تعریف ثابت‌ها مفید است، اما در غیر این صورت بیشتر مزاحم است تا مفید. شاید چون نوع خطاهایی که `final` ممکن است مانع آن‌ها شود، از طریق تست‌های واحدی که می‌نویسم، از قبل شناسایی شده‌اند.

دو عبارت شرطی تکراری [G5] درون حلقه `for` (خط 259 و 263) نیز خوشایند نبودند، بنابراین آن‌ها را با استفاده از عملگر منطقی `||` با هم ترکیب کردم. همچنین از `enum Day` برای هدایت حلقه استفاده کرده و چند تغییر ظاهری دیگر نیز اعمال کردم.

به نظرم رسید که این متد واقعاً به کلاس `DayDate` تعلق ندارد. در واقع این متد باید تابع `parse` مربوط به `Day` باشد. بنابراین آن را به `enum Day` منتقل کردم. اما با این کار `enum Day` بسیار بزرگ شد. از آنجا که مفهوم `Day` به `DayDate` وابسته نیست، در نهایت `Day` را از `DayDate` جدا کرده و در فایل سورس مخصوص خودش قرار دادم [G13].

متد بعدی، `weekdayCodeToString` (خطوط 272 تا 286)، نیز به `Day` منتقل شد و نام آن را `toString` گذاشتم.

</div>

```java
public enum Day
{
    MONDAY(Calendar.MONDAY),
    TUESDAY(Calendar.TUESDAY),
    WEDNESDAY(Calendar.WEDNESDAY),
    s THURSDAY(Calendar.THURSDAY),
    FRIDAY(Calendar.FRIDAY),
    SATURDAY(Calendar.SATURDAY),
    SUNDAY(Calendar.SUNDAY);
    public final int index;
    private static DateFormatSymbols dateSymbols = new DateFormatSymbols();
    Day(int day) { index = day; }
    public static Day make(int index) throws IllegalArgumentException
    {
        for (Day d : Day.values())
            if (d.index == index)
                return d;
        throw new IllegalArgumentException(
          String.format("Illegal day index: %d.", index));
    }
    public static Day parse(String s) throws IllegalArgumentException
    {
        String[] shortWeekdayNames = dateSymbols.getShortWeekdays();
        String[] weekDayNames = dateSymbols.getWeekdays();
        s = s.trim();
        for (Day day : Day.values()) {
            if (s.equalsIgnoreCase(shortWeekdayNames[day.index]) ||
                s.equalsIgnoreCase(weekDayNames[day.index])) {
                return day;
            }
        }
        throw new IllegalArgumentException(
          String.format("%s is not a valid weekday string", s));
    }
    public String toString() { return dateSymbols.getWeekdays()[index]; }
}
```

<div dir="rtl">

دو متد `getMonths` (خطوط 316–288) وجود دارد. اولی، دومی را فراخوانی می‌کند. دومی نیز تنها توسط اولی فراخوانی می‌شود. بنابراین این دو متد را در یکدیگر ادغام کرده و به‌طور قابل توجهی ساده‌شان کردم [G9]، [G12]، [F4]. در نهایت، نام آن را به شکلی گویا‌تر تغییر دادم [N1]:

</div>

```java
public static String[] getMonthNames() {
    return dateFormatSymbols.getMonths();
}
```

<div dir="rtl">

متد `isValidMonthCode` (خطوط 346–326) با معرفی `enum Month` دیگر بی‌فایده شده بود، بنابراین آن را حذف کردم [G9].

متد `monthCodeToQuarter` (خطوط 375–356) بوی حسادت به ویژگی (Feature Envy) می‌دهد [G14] و به نظر می‌رسد که باید در داخل `enum Month` باشد، به‌صورت متدی به نام `quarter`. بنابراین آن را با متد زیر جایگزین کردم:

</div>

```java
public int quarter() {
    return 1 + (index - 1) / 3;
}
```

<div dir="rtl">

با این تغییرات، `enum Month` به اندازه‌ای بزرگ شد که ارزشش را داشت در کلاس جداگانه‌ای قرار بگیرد. بنابراین به دلایل انسجام [G11] و سازمان‌دهی بهتر [G13]، آن را مانند `Day` از `DayDate` جدا کرده و به فایل مخصوص به خودش منتقل کردم.

دو متد بعدی `monthCodeToString` هستند (خطوط 426–377). باز هم الگوی متدی که متد مشابه دیگری را با یک پرچم (`flag`) فراخوانی می‌کند، تکرار شده است. استفاده از پرچم‌ها به عنوان آرگومان، خصوصاً زمانی که فقط برای تعیین قالب خروجی استفاده می‌شوند، معمولاً ایده خوبی نیست [G15]. بنابراین این دو متد را تغییر نام دادم، ساده‌سازی کردم و به `enum Month` منتقل کردم [N1]، [N3]، [C3]، [G14]:

</div>

```java
public String toString() {
    return dateFormatSymbols.getMonths()[index - 1];
}
public String toShortString() {
    return dateFormatSymbols.getShortMonths()[index - 1];
}
```

<div dir="rtl">

متد بعدی `stringToMonthCode` (خطوط 472–428) نیز تغییر نام داده شد، ساده شد و به `Month` منتقل گردید [N1]، [N3]، [C3]، [G14]، [G12].

متد `parse` به صورت زیر بازنویسی و ساده‌سازی شده است:

</div>

```java
public static Month parse(String s) {
    s = s.trim();
    for (Month m : Month.values())
        if (m.matches(s))
            return m;
    try {
        return make(Integer.parseInt(s));
    } catch (NumberFormatException e) {}
    throw new IllegalArgumentException("Invalid month " + s);
}

private boolean matches(String s) {
    return s.equalsIgnoreCase(toString()) ||
           s.equalsIgnoreCase(toShortString());
}
```

<div dir="rtl">

متد `isLeapYear` (خطوط 517–495) را کمی خواناتر کردم [G16]:

</div>

```java
public static boolean isLeapYear(int year) {
    boolean fourth = year % 4 == 0;
    boolean hundredth = year % 100 == 0;
    boolean fourHundredth = year % 400 == 0;
    return fourth && (!hundredth || fourHundredth);
}
```

<div dir="rtl">

متد `leapYearCount` (خطوط 536–519) واقعاً متعلق به `DayDate` نیست و فقط توسط دو متد در `SpreadsheetDate` استفاده می‌شود، بنابراین آن را به کلاس مربوطه انتقال دادم [G6].

متد `lastDayOfMonth` (خطوط 560–538) به آرایه `LAST_DAY_OF_MONTH` متکی است، که این آرایه در واقع باید در `enum Month` قرار گیرد [G17]. همچنین این متد ساده‌تر و گویا‌تر بازنویسی شد [G16]:

</div>

```java
public static int lastDayOfMonth(Month month, int year) {
    if (month == Month.FEBRUARY && isLeapYear(year))
        return month.lastDay() + 1;
    else
        return month.lastDay();
}
```

<div dir="rtl">

حالا وارد بخش‌های جالب‌تری می‌شویم.

متد `addDays` (خطوط 576–562):
1. این متد از متغیرهای داخلی `DayDate` استفاده می‌کند، پس نباید `static` باشد [G18].
2. متدی به نام `toSerial` را فراخوانی می‌کرد که بهتر است به `toOrdinal` تغییر نام یابد [N1].
3. ساختار کلی متد قابل ساده‌سازی بود:

</div>

```java
public DayDate plusDays(int days) {
    return DayDateFactory.makeDate(toOrdinal() + days);
}
```

<div dir="rtl">

متد `addMonths` (خطوط 602–578):
1. مانند `addDays` باید به متد نمونه‌ای تبدیل شود [G18].
2. الگوریتمش پیچیده بود، پس از متغیرهای موقت توضیح‌دهنده برای شفافیت استفاده شد [G19].
3. نام متد `getYYY` به `getYear` تغییر یافت [N1]:

</div>

```java
public DayDate plusMonths(int months) {
    int thisMonthAsOrdinal = 12 * getYear() + getMonth().index - 1;
    int resultMonthAsOrdinal = thisMonthAsOrdinal + months;
    int resultYear = resultMonthAsOrdinal / 12;
    Month resultMonth = Month.make(resultMonthAsOrdinal % 12 + 1);
    int lastDayOfResultMonth = lastDayOfMonth(resultMonth, resultYear);
    int resultDay = Math.min(getDayOfMonth(), lastDayOfResultMonth);
    return DayDateFactory.makeDate(resultDay, resultMonth, resultYear);
}
```

<div dir="rtl">

متد `addYears` (خطوط 626–604):
بدون تغییر خاص، فقط ساختار آن ساده و خواناتر شد:

</div>

```java
public DayDate plusYears(int years) {
    int resultYear = getYear() + years;
    int lastDayOfMonthInResultYear = lastDayOfMonth(getMonth(), resultYear);
    int resultDay = Math.min(getDayOfMonth(), lastDayOfMonthInResultYear);
    return DayDateFactory.makeDate(resultDay, getMonth(), resultYear);
}
```

<div dir="rtl">

در این بخش، یک نگرانی منطقی مطرح می‌شود: آیا نام متدی مثل `addDays` می‌تواند این تصور اشتباه را ایجاد کند که خود شیء تغییر کرده است؟ مثلاً:

</div>

```java
DayDate date = DateFactory.makeDate(5, Month.DECEMBER, 1952);
date.addDays(7); 
```

<div dir="rtl">

برای رفع این ابهام، نام‌ها به `plusDays` و `plusMonths` تغییر یافتند [N4]، تا به‌وضوح نشان دهند که شیء جدیدی ساخته می‌شود:

</div>

```java
DayDate newDate = oldDate.plusDays(5); 
```

<div dir="rtl">

متد `getPreviousDayOfWeek` (خطوط 660–628):
الگوریتم به نسبت پیچیده بود، بنابراین با تحلیل دقیق‌تر [G21] و استفاده از متغیرهای موقت توضیح‌دهنده [G19]، آن را ساده‌سازی کرده و از متد تکراری آن (خطوط 1008–997) نیز خلاص شدم [G5]. همچنین آن را به یک متد نمونه‌ای تبدیل کردم [G18].»

اگر خواستی نسخه‌ی بازنویسی‌شده‌ی `getPreviousDayOfWeek` رو هم بیارم، بگو تا اضافه‌اش کنم.

</div>

<div dir="rtl">

متد `getPreviousDayOfWeek` (خطوط 660–628) با کمی تفکر و بازنویسی، بسیار ساده و گویا شد:

</div>

```java
public DayDate getPreviousDayOfWeek(Day targetDayOfWeek) {
    int offsetToTarget = targetDayOfWeek.index - getDayOfWeek().index;
    if (offsetToTarget >= 0)
        offsetToTarget -= 7;
    return plusDays(offsetToTarget);
}
```

<div dir="rtl">

و دقیقاً همان تحلیل و نتیجه برای متد `getFollowingDayOfWeek` (خطوط 662–693) نیز اعمال شد:

</div>

```java
public DayDate getFollowingDayOfWeek(Day targetDayOfWeek) {
    int offsetToTarget = targetDayOfWeek.index - getDayOfWeek().index;
    if (offsetToTarget <= 0)
        offsetToTarget += 7;
    return plusDays(offsetToTarget);
}
```

<div dir="rtl">

در هر دو متد، از متغیر موقت توضیح‌دهنده `offsetToTarget` برای افزایش خوانایی استفاده شده است [G19]، و هر دو به متدهای نمونه‌ای تبدیل شده‌اند [G18]. ساختار شرطی بسیار خلاصه و روشن است و بازتاب دقیقی از منطق مورد نظر برای محاسبه‌ی فاصله‌ی زمانی بین روزهای هفته می‌باشد. این ساده‌سازی‌ها باعث می‌شود منطق در نگاه اول قابل‌درک باشد و احتمال بروز خطا کاهش یابد.

متد `getNearestDayOfWeek`:
متد `getNearestDayOfWeek` (خطوط 695–726) پیش‌تر اصلاح شده بود، اما تغییرات آن با الگوی جدید متدهای قبلی که اعمال شده بودند، هم‌خوانی نداشتند [G11]. بنابراین، آن را با الگوهای جدید همسان کردم و برای شفافیت الگوریتم از متغیرهای موقت توضیح‌دهنده [G19] استفاده کردم.

</div>

```java
public DayDate getNearestDayOfWeek(final Day targetDay) {
    int offsetToThisWeeksTarget = targetDay.index - getDayOfWeek().index;
    int offsetToFutureTarget = (offsetToThisWeeksTarget + 7) % 7;
    int offsetToPreviousTarget = offsetToFutureTarget - 7;
    if (offsetToFutureTarget > 3)
        return plusDays(offsetToPreviousTarget);
    else
        return plusDays(offsetToFutureTarget);
}
```

<div dir="rtl">

متد `getEndOfMonth`:
متد `getEndOfMonth` (خطوط 728–740) ابتدا به عنوان یک متد نمونه که برای یک پارامتر `DayDate` تعریف شده بود، مشکلاتی داشت [G14]. من آن را به یک متد نمونه واقعی تبدیل کرده و نام‌ها را کمی شفاف‌تر کردم.

</div>

```java
public DayDate getEndOfMonth() {
    Month month = getMonth();
    int year = getYear();
    int lastDay = lastDayOfMonth(month, year);
    return DayDateFactory.makeDate(lastDay, month, year);
}
```

<div dir="rtl">

بازسازی متد `weekInMonthToString`:
بازسازی متد `weekInMonthToString` (خطوط 742–761) جالب توجه بود. ابتدا این متد را به `WeekInMonth enum` منتقل کرده و نام آن را به `toString` تغییر دادم. سپس آن را از یک متد استاتیک به یک متد نمونه تبدیل کردم.

اما در نهایت، متوجه شدم که تنها استفاده‌کنندگان این متد، تست‌ها هستند. بنابراین، متد و تست‌های مرتبط با آن را حذف کردم. این تغییرات باعث شد که در کد فقط از نام‌های `enumerator` (`FIRST`, `SECOND`, ...) استفاده شود و تست‌ها به‌روزرسانی شوند.

تغییرات در متد `toSerial` و دیگر متدهای انتزاعی:
متد `toSerial` (خطوط 838–844) به `getOrdinalDay` تغییر نام داد تا با سایر متدهای مشابه هم‌خوانی داشته باشد.

</div>

```java
public int getOrdinalDay() {
    // ...
}
```

<div dir="rtl">

همچنین، متد `toDate` که برای تبدیل `DayDate` به `java.util.Date` استفاده می‌شود، به دلیل این‌که هیچ وابستگی خاصی به کلاس `SpreadsheetDate` نداشت، به سطح بالاتر انتقال داده شد.

تغییرات در متد `getDayOfWeek`:
متد `getDayOfWeek` در ابتدا وابسته به پیاده‌سازی‌های خاص بود، اما با تغییراتی که در `getDayOfWeekForOrdinalZero` (که در کلاس `DayDate` به‌طور انتزاعی تعریف شد) دادم، این وابستگی‌ها را به حداقل رساندم.

</div>

```java
public Day getDayOfWeek() {
    Day startingDay = getDayOfWeekForOrdinalZero();
    int startingOffset = startingDay.index - Day.SUNDAY.index;
    return Day.make((getOrdinalDay() + startingOffset) % 7 + 1);
}
```

<div dir="rtl">

تغییر در نام متد `compare`:
متد `compare` (خطوط 902–913) که برای مقایسه تاریخ‌ها استفاده می‌شد، نام مناسبی نداشت، بنابراین آن را به `daysSince` تغییر دادم تا معنای دقیق‌تری از عملیات مورد نظر بدهد.

</div>

```java
public int daysSince(DayDate otherDate) {
    // ...
}
```

<div dir="rtl">

متد `isInRange` و بازنویسی‌های آن:
در نهایت، متد `isInRange` (خطوط 982–995) که از یک دستور `switch` استفاده می‌کرد، بهینه‌سازی شد. دستور `switch` با انتقال هر یک از حالت‌ها به `DateInterval enum`، به‌طور کامل بازنویسی شد تا خوانایی و نگهداری کد بهبود یابد.

این تغییرات نشان‌دهنده‌ی استفاده از الگوهای طراحی بهینه و ساده‌سازی کد است که باعث شفافیت و کارایی بیشتر کد شده است. همچنین، حذف توابع اضافی و بازنویسی‌های متناسب با آن باعث می‌شود که کد قابل‌فهم‌تر و نگهداری آن راحت‌تر باشد.

</div>

```java
public enum DateInterval
{
    OPEN {
        public boolean isIn(int d, int left, int right)
        {
            return d > left && d < right;
        }
    },
    CLOSED_LEFT {
        public boolean isIn(int d, int left, int right)
        {
            return d >= left && d < right;
        }
    },
    CLOSED_RIGHT {
        public boolean isIn(int d, int left, int right)
        {
            return d > left && d <= right;
        }
    },
    CLOSED {
        public boolean isIn(int d, int left, int right)
        {
            return d >= left && d <= right;
        }
    };
    public abstract boolean isIn(int d, int left, int right);
}
```

```java
public boolean isInRange(DayDate d1, DayDate d2, DateInterval interval)
{
    int left = Math.min(d1.getOrdinalDay(), d2.getOrdinalDay());
    int right = Math.max(d1.getOrdinalDay(), d2.getOrdinalDay());
    return interval.isIn(getOrdinalDay(), left, right);
}
```

<div dir="rtl">

### بازبینی نهایی کد در کلاس DayDate:

در پایان، به بازبینی کلی کلاس `DayDate` پرداختم تا بررسی کنم که چگونه جریان کد بهتر شده است.

نظرات اولیه:
ابتدا، نظر اولیه کلاس که اکنون قدیمی و بی‌ربط بود، کوتاه و به‌روز کردم [C2].

انتقال `enums`:
سپس، تمامی enumهای باقی‌مانده را به فایل‌های جداگانه منتقل کردم تا کد منظم‌تر شود [G12].

انتقال متغیرها و متدهای استاتیک به کلاس جدید:
متغیر استاتیک `dateFormatSymbols` و سه متد استاتیک (`getMonthNames،` `isLeapYear،` `lastDayOfMonth`) را به یک کلاس جدید به نام `DateUtil` منتقل کردم [G6].

انتقال متدهای انتزاعی به بالای کلاس:
متدهای انتزاعی را به بالای کلاس منتقل کردم تا ساختار بهتری داشته باشد [G24].

تغییر نام متدهای `enum`:
متد `Month.make` را به `Month.fromInt` تغییر دادم [N1] و همین تغییر را برای سایر `enum` ها نیز اعمال کردم.

اضافه کردن `toInt()` به `enums`:
برای تمامی `enum` ها یک متد `toInt()` برای دسترسی به مقادیر آنها ایجاد کرده و فیلد `index` را خصوصی (`private`) کردم.

حذف تکرار در متدهای `plusYears` و `plusMonths`:
در متدهای `plusYears` و `plusMonths،` که تکرارهایی داشتند، با استخراج یک متد جدید به نام `correctLastDayOfMonth` توانستم کد را ساده و شفاف‌تر کنم.

حذف اعداد جادویی (`magic number`):
عدد جادویی 1 را حذف کردم و آن را با `Month.JANUARY.toInt()` یا `Day.SUNDAY.toInt()` جایگزین کردم، بسته به مورد.

تمیزکاری کد در کلاس `SpreadsheetDate`:
زمانی را صرف تمیزکاری الگوریتم‌ها در کلاس `SpreadsheetDate` کردم تا خوانایی و کارایی کد افزایش یابد.

نتیجه‌گیری:
در پایان، کد به شکلی تمیزتر و سازمان‌یافته‌تر درآمد. پوشش تست در `DayDate` کاهش یافته به `84.9` درصد، اما این کاهش ناشی از کوچک‌تر شدن کلاس است. این کاهش در پوشش تست به دلیل این است که تعدادی از خطوط بدون پوشش، به قدری جزئی و بی‌اهمیت هستند که نیاز به تست ندارند.

## نتیجه‌گیری کلی:

طبق قاعده `Boy Scout Rule`، ما کد را در وضعیتی تمیزتر از زمانی که آن را بررسی کردیم، وارد کردیم. این کار زمان‌بر بود، اما ارزشش را داشت. پوشش تست افزایش یافت، برخی اشکالات برطرف شدند، کد شفاف‌تر و کوچک‌تر شد. فردی که در آینده به این کد نگاه کند، امیدوارم که آن را راحت‌تر از ما مدیریت کند. همچنین، احتمالاً می‌تواند کد را به‌مراتب تمیزتر از آنچه که ما انجام دادیم، بهبود بخشد.

</div>

* [فصل قبل](../15_JUnit_Internals/JUnit_Internals.md)
* [فصل بعد](../17_Smells_And_Heuristics/Smells_And_Heuristics.md)
