# کلاس ها

![classes image](img-10.1.png)

<div dir="rtl">
تا اینجا در این کتاب بر روی نحوه‌ی نوشتن خطوط و بلوک‌های کد به‌خوبی تمرکز کرده‌ایم. به ترکیب مناسب توابع و نحوه‌ی ارتباط آن‌ها با یکدیگر پرداخته‌ایم. اما با وجود تمام توجهی که به بیان‌گری جملات کد و توابعی که تشکیل می‌دهند کرده‌ایم، هنوز به کد تمیز نرسیده‌ایم تا زمانی که به سطوح بالاتر سازماندهی کد توجه نکنیم. بیایید درباره کلاس‌های تمیز صحبت کنیم.

## سازماندهی کلاس
طبق شیوه استاندارد جاوا، یک کلاس باید با لیستی از متغیرها آغاز شود. ثابت‌های عمومی و استاتیک، در صورت وجود، باید اول بیایند. سپس متغیرهای استاتیک خصوصی و بعد از آن متغیرهای نمونه خصوصی قرار می‌گیرند. به‌ندرت دلیل خوبی برای داشتن یک متغیر عمومی وجود دارد.

توابع عمومی باید پس از لیست متغیرها قرار بگیرند. ما ترجیح می‌دهیم توابع کمکی خصوصی که توسط یک تابع عمومی فراخوانی می‌شوند، بلافاصله بعد از خود آن تابع عمومی قرار بگیرند. این کار به پیروی از قانون کاهش مرحله کمک می‌کند و باعث می‌شود برنامه مانند یک مقاله روزنامه خوانده شود.

### کپسوله‌سازی
ما دوست داریم متغیرها و توابع کمکی‌مان خصوصی باشند، اما به شدت روی آن پافشاری نمی‌کنیم. گاهی اوقات نیاز داریم که یک متغیر یا تابع کمکی را محافظت‌شده (protected) کنیم تا از طریق یک تست قابل دسترسی باشد. برای ما، تست‌ها اهمیت بالایی دارند. اگر یک تست در همان بسته نیاز به فراخوانی یک تابع یا دسترسی به یک متغیر داشته باشد، آن را محافظت‌شده یا با دامنه بسته (package scope) قرار می‌دهیم. با این حال، ابتدا به دنبال راهی برای حفظ حریم خصوصی خواهیم بود. کاهش کپسوله‌سازی همیشه آخرین گزینه است.

## کلاس‌ها باید کوچک باشند!
اولین قانون درباره کلاس‌ها این است که باید کوچک باشند. دومین قانون این است که باید از آن هم کوچکتر باشند. نه، ما قصد نداریم متن دقیقی را که در فصل توابع گفتیم، تکرار کنیم. اما همانند توابع، کوچک بودن، قانون اصلی در طراحی کلاس‌ها است. سوال فوری ما همیشه این است: "چقدر کوچک؟"

برای توابع، اندازه را با شمردن خطوط فیزیکی اندازه‌گیری می‌کردیم. برای کلاس‌ها از معیار متفاوتی استفاده می‌کنیم: ما مسئولیت‌ها را شمارش می‌کنیم.

در فهرست ۱۰-۱، کلاسی به نام SuperDashboard معرفی شده است که حدود ۷۰ متد عمومی را در معرض دید قرار می‌دهد. بیشتر توسعه‌دهندگان با این نظر موافقند که این کلاس از نظر اندازه کمی بیش از حد بزرگ است. برخی توسعه‌دهندگان ممکن است SuperDashboard را به عنوان یک "کلاس خدا" (God class) خطاب کنند.
</div>

Listing 10-1 -- Too Many Responsibilities

```java
public class SuperDashboard extends JFrame implements MetaDataUser
public String getCustomizerLanguagePath()
public void setSystemConfigPath(String systemConfigPath)
public String getSystemConfigDocument()
public void setSystemConfigDocument(String systemConfigDocument)
public boolean getGuruState()
public boolean getNoviceState()
public boolean getOpenSourceState()
public void showObject(MetaObject object)
public void showProgress(String s)
public boolean isMetadataDirty()
public void setIsMetadataDirty(boolean isMetadataDirty)
public Component getLastFocusedComponent()
public void setLastFocused(Component lastFocused)
public void setMouseSelectState(boolean isMouseSelected)
public boolean isMouseSelected()
public LanguageManager getLanguageManager()
public Project getProject()
public Project getFirstProject()
public Project getLastProject()
public String getNewProjectName()
public void setComponentSizes(Dimension dim)
public String getCurrentDir()
public void setCurrentDir(String newDir)
public void updateStatus(int dotPos, int markPos)
public Class[] getDataBaseClasses()
public MetadataFeeder getMetadataFeeder()
public void addProject(Project project)
public boolean setCurrentProject(Project project)
public boolean removeProject(Project project)
public MetaProjectHeader getProgramMetadata()
public void resetDashboard()
public Project loadProject(String fileName, String projectName)
public void setCanSaveMetadata(boolean canSave)
public MetaObject getSelectedObject()
public void deselectObjects()
public void setProject(Project project)
public void editorAction(String actionName, ActionEvent event)
public void setMode(int mode)
public FileManager getFileManager()
public void setFileManager(FileManager fileManager)
public ConfigManager getConfigManager()
public void setConfigManager(ConfigManager configManager)
public ClassLoader getClassLoader()
public void setClassLoader(ClassLoader classLoader)
public Properties getProps()
public String getUserHome()
public String getBaseDir()
public int getMajorVersionNumber()
public int getMinorVersionNumber()
public int getBuildNumber()
public MetaObject pasting(
  MetaObject target, MetaObject pasted, MetaProject project)
public void processMenuItems(MetaObject metaObject)
public void processMenuSeparators(MetaObject metaObject)
public void processTabPages(MetaObject metaObject)
public void processPlacement(MetaObject object)
public void processCreateLayout(MetaObject object)
public void updateDisplayLayer(MetaObject object, int layerIndex)
public void propertyEditedRepaint(MetaObject object)
public void processDeleteObject(MetaObject object)
public boolean getAttachedToDesigner()
public void processProjectChangedState(boolean hasProjectChanged)
public void processObjectNameChanged(MetaObject object)
public void runProject()
public void setAçowDragging(boolean allowDragging)
public boolean allowDragging()
public boolean isCustomizing()
public void setTitle(String title)
public IdeMenuBar getIdeMenuBar()
public void showHelper(MetaObject metaObject, String propertyName)
// ... many non-public methods follow ...
}
```

<div dir="rtl">
اما اگر SuperDashboard فقط شامل متدهایی باشد که در فهرست ۱۰-۲ نشان داده شده‌اند، چه؟
</div>

Listing 10-2 -- Small Enough?

```java
public class SuperDashboard extends JFrame implements MetaDataUser
 public Component getLastFocusedComponent() 
 public void setLastFocused(Component lastFocused) 
 public int getMajorVersionNumber() 
 public int getMinorVersionNumber() 
 public int getBuildNumber() 
}
```

<div dir="rtl">
پنج متد زیاد نیست، درست است؟
در این حالت، با وجود تعداد کم متدها، SuperDashboard دارای مسئولیت‌های زیادی است. نام یک کلاس باید توصیف‌کننده‌ی مسئولیت‌هایی باشد که آن کلاس بر عهده دارد. در واقع، نام‌گذاری احتمالاً اولین راهی است که به ما کمک می‌کند اندازه‌ی کلاس را تعیین کنیم. اگر نتوانیم نامی مختصر برای یک کلاس پیدا کنیم، احتمالاً آن کلاس بیش از حد بزرگ است. هر چه نام کلاس مبهم‌تر باشد، احتمالاً مسئولیت‌های بیشتری دارد. برای مثال، نام کلاس‌هایی که شامل واژه‌های مبهمی مثل Processor، Manager یا Super هستند، معمولاً به تجمع نامطلوب مسئولیت‌ها اشاره دارد.

ما همچنین باید بتوانیم توصیف کوتاهی از کلاس را در حدود ۲۵ کلمه بنویسیم، بدون اینکه از واژه‌های “if”، “and”، “or” یا “but” استفاده کنیم. چگونه می‌توانیم SuperDashboard را توصیف کنیم؟ SuperDashboard به مؤلفه‌ای که آخرین بار بر روی آن فوکوس شده دسترسی می‌دهد و همچنین به ما اجازه می‌دهد نسخه و Build Number را پیگیری کنیم." اولین "and" نشانه‌ای است که SuperDashboard مسئولیت‌های زیادی دارد.

## اصل مسئولیت واحد
اصل مسئولیت واحد (Single Responsibility Principle - SRP) بیان می‌کند که یک کلاس یا ماژول باید یک، و فقط یک، دلیل برای تغییر داشته باشد. این اصل هم تعریفی از مسئولیت به ما می‌دهد و هم راهنمایی برای اندازه کلاس. کلاس‌ها باید یک مسئولیت داشته باشند—یک دلیل برای تغییر.

کلاس به‌ظاهر کوچک SuperDashboard در فهرست ۱۰-۲ دو دلیل برای تغییر دارد.

اول، این کلاس اطلاعات نسخه را پیگیری می‌کند که به نظر می‌رسد هر بار که نرم‌افزار منتشر می‌شود، نیاز به به‌روزرسانی دارد. دوم، این کلاس مؤلفه‌های جاوا سوئینگ را مدیریت می‌کند (که خود یک زیرکلاس از JFrame است، نمای سوئینگ از یک پنجره GUI سطح بالا). بدون شک، اگر کدی از سوئینگ را تغییر دهیم، می‌خواهیم شماره نسخه را به‌روزرسانی کنیم، اما برعکس این موضوع الزامی نیست: ممکن است بر اساس تغییرات دیگر کد در سیستم، اطلاعات نسخه را تغییر دهیم.

تلاش برای شناسایی مسئولیت‌ها (دلایل تغییر) معمولاً به ما کمک می‌کند تا انتزاعات بهتری در کد خود شناسایی و ایجاد کنیم. ما می‌توانیم به راحتی تمام سه متدی که در SuperDashboard به اطلاعات نسخه مربوط می‌شوند را استخراج کرده و در یک کلاس جداگانه به نام Version قرار دهیم (به فهرست ۱۰-۳ مراجعه کنید). کلاس Version ساختاری است که پتانسیل بالایی برای استفاده مجدد در سایر برنامه‌ها دارد!
</div>

Listing 10-3 -- A single-responsibility class

```java
public class Version {
 public int getMajorVersionNumber() 
 public int getMinorVersionNumber() 
 public int getBuildNumber() 
}
```
<div dir="rtl">
SRP یکی از مفاهیم مهم در طراحی شی‌گرا است. همچنین یکی از ساده‌ترین مفاهیم برای درک و پیروی از آن به شمار می‌آید. با این حال، به طرز عجیبی، SRP اغلب به بدترین شکل در طراحی کلاس‌ها مورد سوءاستفاده قرار می‌گیرد. ما به‌طور منظم با کلاس‌هایی مواجه می‌شویم که کارهای بسیار زیادی انجام می‌دهند. چرا؟

به کار انداختن نرم‌افزار و تمیز نگه‌داشتن آن دو فعالیت بسیار متفاوت هستند. بیشتر ما ظرفیت محدودی برای پردازش اطلاعات داریم، بنابراین بیشتر بر روی کار کردن کد تمرکز می‌کنیم تا سازماندهی و پاکیزگی آن. این موضوع کاملاً قابل درک است. حفظ جدایی بین نگرانی‌ها در فعالیت‌های برنامه‌نویسی ما به همان اندازه در برنامه‌های ما اهمیت دارد.

مشکل این است که بسیاری از ما فکر می‌کنیم که وقتی برنامه کار می‌کند، کار تمام شده است. ما از تغییر به سمت نگرانی‌های دیگر، مانند سازماندهی و پاکیزگی، غافل می‌شویم. به جای اینکه به سراغ مشکلات بعدی برویم، باید به عقب برگردیم و کلاس‌های بیش از حد شلوغ را به واحدهای جداگانه با مسئولیت‌های واحد تقسیم کنیم.

در عین حال، بسیاری از توسعه‌دهندگان نگران این هستند که تعداد زیادی کلاس کوچک و تک‌منظوره، درک تصویر کلی را دشوارتر کند. آن‌ها نگران هستند که باید از کلاسی به کلاس دیگر بروند تا بفهمند یک بخش بزرگ‌تر چگونه انجام می‌شود.

با این حال، یک سیستم با کلاس‌های کوچک بیشتر از یک سیستم با چند کلاس بزرگ اجزای متحرک ندارد. همان اندازه که باید در سیستم با کلاس‌های بزرگ بیاموزیم، در سیستم با کلاس‌های کوچک نیز به یادگیری نیاز داریم. بنابراین سوال این است: آیا می‌خواهید ابزارهای شما در جعبه‌ابزارهایی با کشوهای کوچک و مشخصی که شامل اجزای خوب تعریف شده و با برچسب هستند، سازمان‌دهی شوند؟ یا اینکه چند کشو داشته باشید که همه چیز را در آن‌ها ریخته‌اید؟

هر سیستم بزرگ شامل مقدار زیادی منطق و پیچیدگی خواهد بود. هدف اصلی در مدیریت چنین پیچیدگی‌ای این است که آن را به گونه‌ای سازمان‌دهی کنیم که یک توسعه‌دهنده بداند کجا باید به دنبال موارد بگردد و فقط نیاز داشته باشد که در هر لحظه پیچیدگی مستقیماً مرتبط را درک کند. در مقابل، سیستم‌هایی با کلاس‌های بزرگ و چندمنظوره همیشه ما را با این واقعیت مواجه می‌کنند که باید از میان چیزهای زیادی که در حال حاضر نیازی به دانستن آن‌ها نداریم، عبور کنیم.

برای تأکید بر نکات قبلی: ما می‌خواهیم سیستم‌های ما شامل بسیاری کلاس‌های کوچک باشد، نه چند کلاس بزرگ. هر کلاس کوچک یک مسئولیت واحد را در بر می‌گیرد، یک دلیل برای تغییر دارد و با چند کلاس دیگر همکاری می‌کند تا رفتارهای مورد نظر سیستم را به دست آورد.

## همبستگی (Cohesion)
کلاس‌ها باید تعداد کمی متغیر نمونه داشته باشند. هر یک از متدهای یک کلاس باید یکی یا چندتا از این متغیرها را دستکاری کند. به‌طور کلی، هر چه تعداد متغیرهایی که یک متد دستکاری می‌کند بیشتر باشد، آن متد همبستگی بیشتری با کلاس خود خواهد داشت. یک کلاس که در آن هر متغیر توسط هر متد استفاده می‌شود، حداکثر همبستگی را دارد.

به‌طور کلی، ایجاد چنین کلاس‌های حداکثری همبسته نه تنها توصیه نمی‌شود بلکه غیرممکن است؛ با این حال، ما می‌خواهیم همبستگی بالا باشد. وقتی همبستگی بالا است، به این معناست که متدها و متغیرهای کلاس وابستگی متقابل دارند و به‌عنوان یک کل منطقی با هم پیوند دارند.

به پیاده‌سازی یک پشته (Stack) در فهرست ۱۰-۴ توجه کنید. این یک کلاس بسیار همبسته است. از میان سه متد، تنها متد size() است که از هر دو متغیر استفاده نمی‌کند.
</div>

Listing 10-4  -- Stack.java A cohesive class.

```java
public class Stack {
  private int topOfStack = 0;
  List <Integer> elements = new LinkedList <Integer> ();
  public int size() {
    return topOfStack;
  }
  public void push(int element) {
    topOfStack++;
    elements.add(element);
  }
  public int pop() throws PoppedWhenEmpty {
    if (topOfStack == 0)
      throw new PoppedWhenEmpty();
    int element = elements.get(--topOfStack);
    elements.remove(topOfStack);
    return element;
  }
}
```

<div dir="rtl">
حفظ همبستگی و نتایج آن در ایجاد کلاس‌های کوچک
استراتژی نگه‌داشتن توابع کوچک و کوتاه کردن لیست پارامترها می‌تواند گاهی منجر به افزایش تعداد متغیرهای نمونه‌ای شود که توسط یک زیرمجموعه از متدها استفاده می‌شوند. وقتی این اتفاق می‌افتد، تقریباً همیشه به این معناست که حداقل یک کلاس دیگر در تلاش است تا از کلاس بزرگ‌تر خارج شود. شما باید سعی کنید متغیرها و متدها را به دو یا چند کلاس جداگانه تقسیم کنید به‌طوری‌که کلاس‌های جدید همبستگی بیشتری داشته باشند.

## حفظ همبستگی باعث ایجاد کلاس‌های کوچک‌تر می‌شود
فقط عمل تقسیم توابع بزرگ به توابع کوچک‌تر، باعث افزایش تعداد کلاس‌ها می‌شود. فرض کنید یک تابع بزرگ با تعداد زیادی متغیر در آن اعلام شده است. اگر بخواهید یک بخش کوچک از آن تابع را به یک تابع جداگانه استخراج کنید، آیا باید همه چهار متغیر اعلام شده در تابع را به عنوان آرگومان به تابع جدید منتقل کنید؟

خیر! اگر آن چهار متغیر را به متغیرهای نمونه کلاس ارتقا دهید، می‌توانید کد را بدون انتقال هیچ متغیری استخراج کنید. این کار باعث می‌شود که تابع را به قطعات کوچک‌تر تقسیم کنید.

متأسفانه، این همچنین به این معناست که کلاس‌های ما همبستگی خود را از دست می‌دهند، زیرا متغیرهای نمونه بیشتری را جمع‌آوری می‌کنند که فقط برای اجازه دادن به اشتراک‌گذاری چند تابع وجود دارند. اما صبر کنید! اگر چند تابع وجود داشته باشند که بخواهند برخی از متغیرها را به اشتراک بگذارند، آیا این به معنای آن نیست که آن‌ها به خودی خود یک کلاس هستند؟ قطعاً همین‌طور است. وقتی کلاس‌ها همبستگی خود را از دست می‌دهند، آن‌ها را تقسیم کنید!

بنابراین، شکستن یک تابع بزرگ به توابع کوچک‌تر اغلب به ما این فرصت را می‌دهد که چند کلاس کوچک‌تر را نیز جدا کنیم. این به برنامه ما سازماندهی بهتری می‌دهد و ساختاری شفاف‌تر فراهم می‌آورد.

به عنوان نمونه‌ای از آنچه که منظورم است، بیایید از یک مثال قدیمی استفاده کنیم که از کتاب فوق‌العاده Knuth با عنوان Literate Programming گرفته شده است. فهرست ۱۰-۵ یک ترجمه به زبان جاوا از برنامه PrintPrimes نوشته Knuth را نشان می‌دهد. برای اینکه به Knuth انصاف بدهیم، این برنامه همان‌طور که او نوشته است نیست، بلکه به‌عنوان خروجی ابزار WEB او ارائه شده است. من از آن استفاده می‌کنم زیرا نقطه شروع خوبی برای شکستن یک تابع بزرگ به توابع و کلاس‌های کوچک‌تر است.
</div>

Listing 10-5 -- PrintPrimes.java

```java
package literatePrimes;
public class PrintPrimes {
  public static void main(String[] args) {
    final int M = 1000;
    final int RR = 50;
    final int CC = 4;
    final int WW = 10;
    final int ORDMAX = 30;
    int P[] = new int[M + 1];
    int PAGENUMBER;
    int PAGEOFFSET;
    int ROWOFFSET;
    int C;
    int J;
    int K;
    boolean JPRIME;
    int ORD;
    int SQUARE;
    int N;
    int MULT[] = new int[ORDMAX + 1];
    J = 1;
    K = 1;
    P[1] = 2;
    ORD = 2;
    SQUARE = 9;
    while (K < M) {
      do {
        J = J + 2;
        if (J == SQUARE) {
          ORD = ORD + 1;
          SQUARE = P[ORD] * P[ORD];
          MULT[ORD - 1] = J;
        }
        N = 2;
        JPRIME = true;
        while (N < ORD && JPRIME) {
          while (MULT[N] < J)
            MULT[N] = MULT[N] + P[N] + P[N];
          if (MULT[N] == J)
            JPRIME = false;
          N = N + 1;
        }
      } while (!JPRIME);
      K = K + 1;
      P[K] = J;
    } {
      PAGENUMBER = 1;
      PAGEOFFSET = 1;
      while (PAGEOFFSET <= M) {
        System.out.println("The First " + M +
          " Prime Numbers --- Page " + PAGENUMBER);
        System.out.println("");
        for (ROWOFFSET = PAGEOFFSET; ROWOFFSET < PAGEOFFSET + RR; ROWOFFSET++) {
          for (C = 0; C < CC; C++)
            if (ROWOFFSET + C * RR <= M)
              System.out.format("%10d", P[ROWOFFSET + C * RR]);
          System.out.println("");
        }
        System.out.println("\f");
        PAGENUMBER = PAGENUMBER + 1;
        PAGEOFFSET = PAGEOFFSET + RR * CC;
      }
    }
  }
}
```

<div dir="rtl">
این برنامه که به‌صورت یک تابع واحد نوشته شده، بسیار نامنظم است. ساختار آن به شدت تو در تو است، شامل تعداد زیادی متغیر نامناسب می‌باشد و ساختاری به شدت وابسته دارد. حداقل باید این تابع بزرگ به چند تابع کوچک‌تر تقسیم شود.

فهرست‌های ۱۰-۶ تا ۱۰-۸ نتایج تقسیم کد موجود در فهرست ۱۰-۵ به کلاس‌ها و توابع کوچک‌تر را نشان می‌دهند و همچنین نام‌های معناداری برای آن کلاس‌ها، توابع و متغیرها انتخاب شده است.
</div>

Listing 10-6 -- PrimePrinter.java (refactored)

```java
package literatePrimes;
public class PrimePrinter {
  public static void main(String[] args) {
    final int NUMBER_OF_PRIMES = 1000;
    int[] primes = PrimeGenerator.generate(NUMBER_OF_PRIMES);
    final int ROWS_PER_PAGE = 50;
    final int COLUMNS_PER_PAGE = 4;
    RowColumnPagePrinter tablePrinter =
      new RowColumnPagePrinter(ROWS_PER_PAGE,
        COLUMNS_PER_PAGE,
        "The First " + NUMBER_OF_PRIMES +
        " Prime Numbers");
    tablePrinter.print(primes);
  }
}
```

Listing 10-7 -- RowColumnPagePrinter.java

```java
package literatePrimes;
import java.io.PrintStream;
public class RowColumnPagePrinter {
  private int rowsPerPage;
  private int columnsPerPage;
  private int numbersPerPage;
  private String pageHeader;
  private PrintStream printStream;
  public RowColumnPagePrinter(int rowsPerPage,
    int columnsPerPage,
    String pageHeader) {
    this.rowsPerPage = rowsPerPage;
    this.columnsPerPage = columnsPerPage;
    this.pageHeader = pageHeader;
    numbersPerPage = rowsPerPage * columnsPerPage;
    printStream = System.out;
  }
  public void print(int data[]) {
    int pageNumber = 1;
    for (int firstIndexOnPage = 0; firstIndexOnPage < data.length; firstIndexOnPage += numbersPerPage) {
      int lastIndexOnPage =
        Math.min(firstIndexOnPage + numbersPerPage - 1,
          data.length - 1);
      printPageHeader(pageHeader, pageNumber);
      printPage(firstIndexOnPage, lastIndexOnPage, data);
      printStream.println("\f");
      pageNumber++;
    }
  }
  private void printPage(int firstIndexOnPage,
    int lastIndexOnPage,
    int[] data) {
    int firstIndexOfLastRowOnPage =
      firstIndexOnPage + rowsPerPage - 1;
    for (int firstIndexInRow = firstIndexOnPage; firstIndexInRow <= firstIndexOfLastRowOnPage; firstIndexInRow++) {
      printRow(firstIndexInRow, lastIndexOnPage, data);
      printStream.println("");
    }
  }
  private void printRow(int firstIndexInRow,
    int lastIndexOnPage,
    int[] data) {
    for (int column = 0; column < columnsPerPage; column++) {
      int index = firstIndexInRow + column * rowsPerPage;
      if (index <= lastIndexOnPage)
        printStream.format("%10d", data[index]);
    }
  }
  private void printPageHeader(String pageHeader,
    int pageNumber) {
    printStream.println(pageHeader + " --- Page " + pageNumber);
    printStream.println("");
  }
  public void setOutput(PrintStream printStream) {
    this.printStream = printStream;
  }
}
```

Listing 10-8 -- PrimeGenerator.java

```java
package literatePrimes;
import java.util.ArrayList;
public class PrimeGenerator {
  private static int[] primes;
  private static ArrayList < Integer > multiplesOfPrimeFactors;
  protected static int[] generate(int n) {
    primes = new int[n];
    multiplesOfPrimeFactors = new ArrayList < Integer > ();
    set2AsFirstPrime();
    checkOddNumbersForSubsequentPrimes();
    return primes;
  }
  private static void set2AsFirstPrime() {
    primes[0] = 2;
    multiplesOfPrimeFactors.add(2);
  }
  private static void checkOddNumbersForSubsequentPrimes() {
    int primeIndex = 1;
    for (int candidate = 3; primeIndex < primes.length; candidate += 2) {
      if (isPrime(candidate))
        primes[primeIndex++] = candidate;
    }
  }
  private static boolean isPrime(int candidate) {
    if (isLeastRelevantMultipleOfNextLargerPrimeFactor(candidate)) {
      multiplesOfPrimeFactors.add(candidate);
      return false;
    }
    return isNotMultipleOfAnyPreviousPrimeFactor(candidate);
  }
  private static boolean
  isLeastRelevantMultipleOfNextLargerPrimeFactor(int candidate) {
    int nextLargerPrimeFactor = primes[multiplesOfPrimeFactors.size()];
    int leastRelevantMultiple = nextLargerPrimeFactor * nextLargerPrimeFactor;
    return candidate == leastRelevantMultiple;
  }
  private static boolean
  isNotMultipleOfAnyPreviousPrimeFactor(int candidate) {
    for (int n = 1; n < multiplesOfPrimeFactors.size(); n++) {
      if (isMultipleOfNthPrimeFactor(candidate, n))
        return false;
    }
    return true;
  }
  private static boolean
  isMultipleOfNthPrimeFactor(int candidate, int n) {
    return
    candidate == smallestOddNthMultipleNotLessThanCandidate(candidate, n);
  }
  private static int
  smallestOddNthMultipleNotLessThanCandidate(int candidate, int n) {
    int multiple = multiplesOfPrimeFactors.get(n);
    while (multiple < candidate)
      multiple += 2 * primes[n];
    multiplesOfPrimeFactors.set(n, multiple);
    return multiple;
  }
}
```

<div dir="rtl">
اولین چیزی که ممکن است متوجه شوید این است که برنامه به طور قابل توجهی طولانی‌تر شده است. این برنامه از کمی بیش از یک صفحه به نزدیک به سه صفحه طول کشیده است. دلایل متعددی برای این رشد وجود دارد. اول، برنامه بازنویسی‌شده از نام‌های متغیر طولانی‌تر و توصیفی‌تری استفاده می‌کند. دوم، برنامه بازنویسی‌شده از اعلان‌های تابع و کلاس به‌عنوان راهی برای افزودن توضیحات به کد استفاده می‌کند. سوم، از فضا و تکنیک‌های فرمت‌بندی برای حفظ خوانایی برنامه استفاده شده است.

توجه کنید که برنامه به سه مسئولیت اصلی تقسیم شده است. برنامه اصلی در کلاس PrimePrinter به‌طور مستقل قرار دارد. مسئولیت این کلاس مدیریت محیط اجرای برنامه است. این کلاس در صورت تغییر روش فراخوانی تغییر خواهد کرد. به‌عنوان مثال، اگر این برنامه به یک سرویس SOAP تبدیل شود، این کلاس است که تحت تأثیر قرار می‌گیرد.

کلاس RowColumnPagePrinter همه چیز را در مورد نحوه فرمت کردن یک لیست از اعداد به صفحات با تعداد مشخصی از ردیف‌ها و ستون‌ها می‌داند. اگر نیاز به تغییر فرمت خروجی باشد، این کلاس است که تحت تأثیر قرار می‌گیرد.

کلاس PrimeGenerator می‌داند که چگونه یک لیست از اعداد اول تولید کند. توجه کنید که این کلاس برای نمونه‌سازی به‌عنوان یک شیء طراحی نشده است. این کلاس تنها یک فضای مفید است که می‌توان در آن متغیرها را اعلام و پنهان کرد. اگر الگوریتم محاسبه اعداد اول تغییر کند، این کلاس تغییر خواهد کرد.

این یک بازنویسی نبود! ما از ابتدا شروع نکردیم و برنامه را دوباره نوشتیم. در واقع، اگر به‌دقت به دو برنامه مختلف نگاه کنید، متوجه می‌شوید که آنها از همان الگوریتم و مکانیسم‌ها برای انجام کار خود استفاده می‌کنند.

این تغییر با نوشتن یک مجموعه آزمایشی که رفتار دقیق برنامه اول را تأیید می‌کرد، انجام شد. سپس تغییرات کوچک و متعددی یکی پس از دیگری انجام شد. پس از هر تغییر، برنامه اجرا شد تا اطمینان حاصل شود که رفتار آن تغییر نکرده است. یک قدم کوچک پس از دیگری، برنامه اول تمیز و به برنامه دوم تبدیل شد.

## سازماندهی برای تغییر

برای اکثر سیستم‌ها، تغییر مداوم است. هر تغییری ما را در معرض این خطر قرار می‌دهد که بخش‌های دیگر سیستم دیگر به درستی کار نکنند. در یک سیستم تمیز، ما کلاس‌های خود را به‌گونه‌ای سازمان‌دهی می‌کنیم که خطر تغییر را کاهش دهیم.

کلاس Sql در فهرست 10-9 برای تولید رشته‌های SQL به‌طور صحیح با توجه به متادیتای مناسب استفاده می‌شود. این کلاس در حال توسعه است و هنوز از قابلیت‌هایی مانند عبارات به‌روزرسانی SQL پشتیبانی نمی‌کند. هنگامی که زمان آن فرا برسد که کلاس Sql از یک عبارت به‌روزرسانی پشتیبانی کند، باید این کلاس را "باز کنیم" تا تغییرات لازم را اعمال کنیم. مشکل با باز کردن یک کلاس این است که خطر را افزایش می‌دهد. هرگونه تغییر در کلاس می‌تواند به شکستن سایر کدهای موجود در آن کلاس منجر شود. بنابراین، باید آن را به‌طور کامل دوباره آزمایش کرد.
</div>

Listing 10-9 -- A class that must be opened for change

```java
public class Sql {
  public Sql(String table, Column[] columns)
  public String create()
  public String insert(Object[] fields)
  public String selectAll()
  public String findByKey(String keyColumn, String keyValue)
  public String select(Column column, String pattern)
  public String select(Criteria criteria)
  public String preparedInsert()
  private String columnList(Column[] columns)
  private String valuesList(Object[] fields, final Column[] columns)
  private String selectWithCriteria(String criteria)
  private String placeholderList(Column[] columns)
}
```

<div dir="rtl">
کلاس Sql باید تغییر کند زمانی که نوع جدیدی از عبارت را اضافه می‌کنیم. همچنین باید تغییر کند زمانی که جزئیات یک نوع عبارت خاص را تغییر می‌دهیم به عنوان مثال، اگر بخواهیم عملکرد select را برای پشتیبانی از subselect‌ها اصلاح کنیم. این دو دلیل برای تغییر به این معناست که کلاس Sql اصل مسئولیت واحد (SRP) را نقض می‌کند.

می‌توانیم این نقض SRP را از یک منظر سازمانی ساده شناسایی کنیم. طرح متدهای کلاس Sql نشان می‌دهد که متدهای خصوصی، مانند selectWithCriteria، به نظر می‌رسد که تنها به عبارات select مربوط باشند. رفتار متدهای خصوصی که تنها به یک زیرمجموعه کوچک از کلاس مربوط می‌شوند، می‌تواند به‌عنوان یک قاعده مفید برای شناسایی مناطق بالقوه برای بهبود عمل کند. با این حال، محرک اصلی برای اقدام باید خود تغییر سیستم باشد. اگر کلاس Sql به‌طور منطقی کامل در نظر گرفته شود، نیازی به جداسازی مسئولیت‌ها نداریم. اگر برای آینده قابل پیش‌بینی به عملکرد update نیاز نداشته باشیم، باید کلاس Sql را به حال خود بگذاریم. اما به محض اینکه خود را در حال باز کردن یک کلاس یافتیم، باید در نظر داشته باشیم که طراحی‌مان را اصلاح کنیم.

چه می‌شود اگر یک راه‌حل مشابه آنچه در فهرست 10-10 آمده است، در نظر بگیریم؟ هر متد عمومی که در کلاس Sql قبلی از فهرست 10-9 تعریف شده بود، به نسخه‌های فرعی خود از کلاس Sql جدا شده است. توجه کنید که متدهای خصوصی، مانند valuesList، مستقیماً به جایی که نیاز دارند منتقل می‌شوند. رفتار خصوصی مشترک به دو کلاس کاربردی، یعنی Where و ColumnList، جداسازی شده است.
</div>

Listing 10-10 -- A set of closed classes

```java
abstract public class Sql {
  public Sql(String table, Column[] columns)
  abstract public String generate();
}
public class CreateSql extends Sql {
  public CreateSql(String table, Column[] columns)
  @Override public String generate()
}
public class SelectSql extends Sql {
  public SelectSql(String table, Column[] columns)
  @Override public String generate()
}
public class InsertSql extends Sql {
  public InsertSql(String table, Column[] columns, Object[] fields)
  @Override public String generate()
  private String valuesList(Object[] fields, final Column[] columns)
}
public class SelectWithCriteriaSql extends Sql {
  public SelectWithCriteriaSql(
    String table, Column[] columns, Criteria criteria)
  @Override public String generate()
}
public class SelectWithMatchSql extends Sql {
  public SelectWithMatchSql(
    String table, Column[] columns, Column column, String pattern)
  @Override public String generate()
}
public class FindByKeySql extends Sql
public FindByKeySql(
  String table, Column[] columns, String keyColumn, String keyValue)
@Override public String generate()
}
public class PreparedInsertSql extends Sql {
  public PreparedInsertSql(String table, Column[] columns)
  @Override public String generate() {
    private String placeholderList(Column[] columns)
  }
  public class Where {
    public Where(String criteria)
    public String generate()
  }
  public class ColumnList {
    public ColumnList(Column[] columns)
    public String generate()
  }
```

<div dir="rtl">
کد در هر کلاس به شدت ساده می‌شود. زمان لازم برای درک هر کلاس به حداقل می‌رسد و خطر اینکه یک تابع دیگری را خراب کند، به طرز چشمگیری کاهش می‌یابد. از نظر تست، اثبات تمام بخش‌های منطقی این راه‌حل آسان‌تر می‌شود، زیرا کلاس‌ها به طور مستقل از یکدیگر هستند.

به‌همین ترتیب، زمانی که زمان افزودن عبارات به‌روزرسانی فرا برسد، هیچ‌یک از کلاس‌های موجود نیازی به تغییر ندارند! ما منطق ساخت عبارات به‌روزرسانی را در یک زیرکلاس جدید از Sql به نام UpdateSql کدنویسی می‌کنیم. هیچ کد دیگری در سیستم به خاطر این تغییر خراب نخواهد شد.

منطق بازسازی‌شده Sql ما بهترین وضعیت را نمایندگی می‌کند. این منطق از اصل مسئولیت واحد (SRP) پشتیبانی می‌کند. همچنین از یک اصل کلیدی دیگر در طراحی کلاس‌های شیءگرا به نام اصل باز-بسته (Open-Closed Principle یا OCP) نیز پشتیبانی می‌کند: کلاس‌ها باید برای گسترش باز باشند اما برای تغییر بسته. کلاس Sql بازسازی‌شده ما به‌گونه‌ای طراحی شده است که اجازه اضافه کردن عملکرد جدید از طریق زیرکلاس‌ها را می‌دهد، اما می‌توانیم این تغییر را در حالی انجام دهیم که سایر کلاس‌ها بدون تغییر باقی بمانند. به سادگی کلاس UpdateSql را در محل خود قرار می‌دهیم.

ما می‌خواهیم سیستم‌های خود را به گونه‌ای ساختاربندی کنیم که در هنگام به‌روزرسانی با ویژگی‌های جدید یا تغییر یافته، به کمترین میزان ممکن دست بزنیم. در یک سیستم ایده‌آل، ویژگی‌های جدید را با گسترش سیستم ادغام می‌کنیم، نه با تغییر کدهای موجود.

## جداسازی از تغییر
نیازها تغییر می‌کنند، بنابراین کد نیز تغییر خواهد کرد. ما در درس OO 101 یاد گرفتیم که کلاس‌های عینی وجود دارند که جزئیات پیاده‌سازی (کد) را شامل می‌شوند و کلاس‌های انتزاعی که تنها مفاهیم را نمایندگی می‌کنند. یک کلاس مشتری که به جزئیات عینی وابسته است، در خطر است زمانی که آن جزئیات تغییر می‌کنند. ما می‌توانیم واسط‌ها و کلاس‌های انتزاعی را معرفی کنیم تا به جداسازی تأثیر آن جزئیات کمک کنیم.

وابستگی به جزئیات عینی چالش‌هایی برای تست سیستم ما ایجاد می‌کند. اگر ما یک کلاس Portfolio بسازیم و آن به یک API خارجی به نام TokyoStockExchange وابسته باشد تا ارزش Portfolio را تعیین کند، موارد آزمایشی ما تحت تأثیر نوسانات چنین جستجویی قرار می‌گیرد. نوشتن یک تست دشوار است زمانی که هر پنج دقیقه یک پاسخ متفاوت دریافت می‌کنیم!

به‌جای طراحی Portfolio به گونه‌ای که به‌طور مستقیم به TokyoStockExchange وابسته باشد، یک واسط به نام StockExchange ایجاد می‌کنیم که یک متد واحد را اعلام می‌کند:
</div>

```java
public interface StockExchange {
 Money currentPrice(String symbol);
}
```

<div dir="rtl">
ما TokyoStockExchange را طوری طراحی می‌کنیم که این واسط را پیاده‌سازی کند. همچنین اطمینان حاصل می‌کنیم که سازنده کلاس Portfolio یک مرجع از StockExchange را به‌عنوان آرگومان دریافت می‌کند:
</div>

```java
public Portfolio {
 private StockExchange exchange;
 public Portfolio(StockExchange exchange) {
 this.exchange = exchange;
 }
 // ...
}
```

<div dir="rtl">
اکنون تست ما می‌تواند یک پیاده‌سازی قابل تست از واسط StockExchange ایجاد کند که رفتار TokyoStockExchange را شبیه‌سازی می‌کند. این پیاده‌سازی تست، مقدار فعلی را برای هر نماد مورد استفاده در تست ثابت می‌کند. اگر تست ما خرید پنج سهم از مایکروسافت را برای پرتفوی ما نشان دهد، پیاده‌سازی تست را طوری کدنویسی می‌کنیم که همیشه 100 دلار به‌ازای هر سهم مایکروسافت برگرداند. پیاده‌سازی تست ما از واسط StockExchange به یک جستجوی ساده در جدول کاهش می‌یابد. سپس می‌توانیم تستی بنویسیم که انتظار دارد ارزش کلی Portfolio ما 500 دلار باشد.
</div>

```java
public class PortfolioTest {
  private FixedStockExchangeStub exchange;
  private Portfolio portfolio;
  @Before
  protected void setUp() throws Exception {
    exchange = new FixedStockExchangeStub();
    exchange.fix("MSFT", 100);
    portfolio = new Portfolio(exchange);
  }
  @Test
  public void GivenFiveMSFTTotalShouldBe500() throws Exception {
    portfolio.add(5, "MSFT");
    Assert.assertEquals(500, portfolio.value());
  }
}
```

<div dir="rtl">
اگر یک سیستم به اندازه کافی از هم جدا باشد که به این صورت تست شود، همچنین انعطاف‌پذیری بیشتری خواهد داشت و امکان استفاده مجدد بیشتری را فراهم می‌آورد. عدم وابستگی به این معناست که عناصر سیستم ما بهتر از یکدیگر و از تغییرات جدا هستند. این جداسازی فهم هر عنصر سیستم را آسان‌تر می‌کند.

با حداقل کردن وابستگی به این شکل، کلاس‌های ما به یک اصل طراحی کلاس دیگر به نام اصل وارونگی وابستگی (Dependency Inversion Principle یا DIP) پایبند می‌شوند. به‌طور اساسی، DIP می‌گوید که کلاس‌های ما باید به انتزاعات وابسته باشند، نه به جزئیات عینی.

به‌جای اینکه به جزئیات پیاده‌سازی کلاس TokyoStockExchange وابسته باشد، اکنون کلاس Portfolio به واسط StockExchange وابسته است. واسط StockExchange مفهوم انتزاعی درخواست قیمت فعلی یک نماد را نمایندگی می‌کند. این انتزاع تمام جزئیات خاص به‌دست آوردن چنین قیمتی، از جمله منبع به‌دست آوردن آن قیمت را جدا می‌کند.
</div>

* [فصل قبل](../9_Unit_Tests/9_Unit_Tests.md)