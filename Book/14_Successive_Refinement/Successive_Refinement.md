# بهبود تدریجی

Case Study of a Command-Line Argument Parser

![image](../../assets/image/14/img-14.1.png)

<div dir="rtl">

این فصل یک مطالعهٔ موردی در زمینهٔ بهبود تدریجی است. در ابتدا ماژولی را می‌بینید که شروع خوبی داشت، اما به خوبی مقیاس‌پذیر نبود. سپس خواهید دید که این ماژول چگونه  refactor و تمیز شد.

بیشتر ما گاهی مجبور شده‌ایم که آرگومان‌های خط فرمان (command-line arguments) را پردازش کنیم. اگر ابزار مناسبی در دسترس نداشته باشیم، معمولاً آرایه‌ای از رشته‌ها را که به تابع `main` پاس داده می‌شود، پیمایش می‌کنیم. ابزارهای خوبی از منابع مختلف برای این کار وجود دارد، اما هیچ‌کدام دقیقاً آن کاری را نمی‌کنند که من می‌خواهم. بنابراین، تصمیم گرفتم ابزار خودم را بنویسم. اسمش را گذاشتم: `Args`.

استفاده از `Args` بسیار ساده است. کافی است شیئی از کلاس `Args` را با آرگومان‌های ورودی و یک رشتهٔ قالب (format string) بسازید و سپس از طریق آن شیء، مقادیر آرگومان‌ها را استخراج کنید. به مثال سادهٔ زیر توجه کنید:
</div>

Listing 14-1

```java
public static void main (String[] args)
{
    try
        {
            Args arg = new Args ("l,p#,d*", args);
            boolean logging = arg.getBoolean ('l');
            int port = arg.getInt ('p');
            String directory = arg.getString ('d');
            executeApplication (logging, port, directory);
        }
    catch (ArgsException e)
        {
            System.out.printf ("Argument error: %s\n", e.errorMessage ());
        }
}
```

<div dir="rtl">

می‌بینید که استفاده از این ابزار چقدر ساده است. فقط کافی است یک نمونه از کلاس `Args` با دو پارامتر ایجاد کنیم. پارامتر اول، یک رشتهٔ قالب (یا شِما – schema) است: `"l,p#,d*"` که سه آرگومان خط فرمان را تعریف می‌کند. اولین آرگومان -l یک آرگومان بولی (`boolean`) است. دومی -p یک آرگومان عددی (`integer`) است. سومی -d یک آرگومان رشته‌ای (`string`) است.

پارامتر دوم سازندهٔ `Args` همان آرایه‌ای از آرگومان‌های خط فرمان است که به تابع `main` پاس داده شده.

اگر سازنده بدون پرتاب (`throw`) کردن یک استثنای `ArgsException` اجرا شود، یعنی ورودی خط فرمان به‌درستی تجزیه شده و شیء Args آمادهٔ پرس‌وجو است. متدهایی مثل `getBoolean،` `getInteger` و `getString` به ما امکان می‌دهند مقدار آرگومان‌ها را بر اساس نامشان دریافت کنیم.

اگر مشکلی در رشتهٔ قالب یا خود آرگومان‌های خط فرمان وجود داشته باشد، یک استثنای (exception) از نوع `ArgsException` پرتاب می‌شود. با استفاده از متد errorMessage در این استثنا، می‌توان توضیح دقیقی از خطا دریافت کرد.

## پیاده‌سازی `Args`
Listing 14-2 پیاده‌سازی کلاس `Args` را نشان می‌دهد. لطفاً آن را با دقت مطالعه کنید. من برای سبک و ساختار آن تلاش زیادی کرده‌ام و امیدوارم ارزش الگوبرداری داشته باشد.
</div>

Listing 14-2

```java
package com.objectmentor.utilities.args;
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;

import java.util.*;
public class Args
{
    private Map<Character, ArgumentMarshaler> marshalers;
    private Set<Character> argsFound;
    private ListIterator<String> currentArgument;
    public Args (String schema, String[] args) throws ArgsException
    {
        marshalers = new HashMap<Character, ArgumentMarshaler> ();
        argsFound = new HashSet<Character> ();
        parseSchema (schema);
        parseArgumentStrings (Arrays.asList (args));
    }
    private void
    parseSchema (String schema) throws ArgsException
    {
        for (String element : schema.split (","))
            if (element.length () > 0)
                parseSchemaElement (element.trim ());
    }
    private void
    parseSchemaElement (String element) throws ArgsException
    {
        char elementId = element.charAt (0);
        String elementTail = element.substring (1);
        validateSchemaElementId (elementId);
        if (elementTail.length () == 0)
            marshalers.put (elementId, new BooleanArgumentMarshaler ());
        else if (elementTail.equals ("*"))
            marshalers.put (elementId, new StringArgumentMarshaler ());
        else if (elementTail.equals ("#"))
            marshalers.put (elementId, new IntegerArgumentMarshaler ());
        else if (elementTail.equals ("##"))
            marshalers.put (elementId, new DoubleArgumentMarshaler ());
        else if (elementTail.equals ("[*]"))
            marshalers.put (elementId, new StringArrayArgumentMarshaler ());
        else
            throw new ArgsException (INVALID_ARGUMENT_FORMAT, elementId,
                                     elementTail);
    }
    private void
    validateSchemaElementId (char elementId) throws ArgsException
    {
        if (!Character.isLetter (elementId))
            throw new ArgsException (INVALID_ARGUMENT_NAME, elementId, null);
    }
    private void
    parseArgumentStrings (List<String> argsList) throws ArgsException
    {
        for (currentArgument = argsList.listIterator ();
             currentArgument.hasNext ();)
            {
                String argString = currentArgument.next ();
                if (argString.startsWith ("-"))
                    {
                        parseArgumentCharacters (argString.substring (1));
                    }
                else
                    {
                        currentArgument.previous ();
                        break;
                    }
            }
    }
    private void
    parseArgumentCharacters (String argChars) throws ArgsException
    {
        for (int i = 0; i < argChars.length (); i++)
            parseArgumentCharacter (argChars.charAt (i));
    }
    private void
    parseArgumentCharacter (char argChar) throws ArgsException
    {
        ArgumentMarshaler m = marshalers.get (argChar);
        if (m == null)
            {
                throw new ArgsException (UNEXPECTED_ARGUMENT, argChar, null);
            }
        else
            {
                argsFound.add (argChar);
                try
                    {
                        m.set (currentArgument);
                    }
                catch (ArgsException e)
                    {
                        e.setErrorArgumentId (argChar);
                        throw e;
                    }
            }
    }
    public boolean
    has (char arg)
    {
        return argsFound.contains (arg);
    }
    public int
    nextArgument ()
    {
        return currentArgument.nextIndex ();
    }
    public boolean
    getBoolean (char arg)
    {
        return BooleanArgumentMarshaler.getValue (marshalers.get (arg));
    }
    public String
    getString (char arg)
    {
        return StringArgumentMarshaler.getValue (marshalers.get (arg));
    }
    public int
    getInt (char arg)
    {
        return IntegerArgumentMarshaler.getValue (marshalers.get (arg));
    }
    public double
    getDouble (char arg)
    {
        return DoubleArgumentMarshaler.getValue (marshalers.get (arg));
    }
    public String[]
    getStringArray (char arg)
    {
        return StringArrayArgumentMarshaler.getValue (marshalers.get (arg));
    }
}
```

<div dir="rtl">

توجه کنید که می‌توانید این کد را از بالا به پایین بخوانید، بدون اینکه مجبور شوید زیاد بین بخش‌های مختلف جابه‌جا شوید یا به جلو بپرید. تنها چیزی که شاید نیاز داشته باشید جلوتر ببینید، تعریف `ArgumentMarshaler` است که عمداً آن را حذف کرده‌ام. اگر این کد را با دقت خوانده باشید، باید متوجه شده باشید که رابط (interface) `ArgumentMarshaler` چیست و پیاده‌سازی‌های مختلف آن چه کاری انجام می‌دهند.

الان چند نمونه از آن‌ها را به شما نشان می‌دهم.
</div>

Listing 14-3 - ArgumentMarshaler.java

```java
public interface ArgumentMarshaler
{
    void set (Iterator<String> currentArgument) throws ArgsException;
}
```

Listing 14-4 - BooleanArgumentMarshaler.java

```java
public class BooleanArgumentMarshaler implements ArgumentMarshaler
{
    private boolean booleanValue = false;
    public void
    set (Iterator<String> currentArgument) throws ArgsException
    {
        booleanValue = true;
    }
    public static boolean
    getValue (ArgumentMarshaler am)
    {
        if (am != null && am instanceof BooleanArgumentMarshaler)
            return ((BooleanArgumentMarshaler)am).booleanValue;
        else
            return false;
    }
}
```

Listing 14-5 - StringArgumentMarshaler.java

```java
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;
public class StringArgumentMarshaler implements ArgumentMarshaler
{
    private String stringValue = "";
    public void
    set (Iterator<String> currentArgument) throws ArgsException
    {
        try
            {
                stringValue = currentArgument.next ();
            }
        catch (NoSuchElementException e)
            {
                throw new ArgsException (MISSING_STRING);
            }
    }
    public static String
    getValue (ArgumentMarshaler am)
    {
        if (am != null && am instanceof StringArgumentMarshaler)
            return ((StringArgumentMarshaler)am).stringValue;
        else
            return "";
    }
}
```

Listing 14-6 - IntegerArgumentMarshaler.java

```java
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;
public class IntegerArgumentMarshaler implements ArgumentMarshaler
{
    private int intValue = 0;
    public void
    set (Iterator<String> currentArgument) throws ArgsException
    {
        String parameter = null;
        try
            {
                parameter = currentArgument.next ();
                intValue = Integer.parseInt (parameter);
            }
        catch (NoSuchElementException e)
            {
                throw new ArgsException (MISSING_INTEGER);
            }
        catch (NumberFormatException e)
            {
                throw new ArgsException (INVALID_INTEGER, parameter);
            }
    }
    public static int
    getValue (ArgumentMarshaler am)
    {
        if (am != null && am instanceof IntegerArgumentMarshaler)
            return ((IntegerArgumentMarshaler)am).intValue;
        else
            return 0;
    }
}
```

<div dir="rtl">

سایر مشتق‌های `ArgumentMarshaler` هم به همین سبک برای `double` ‌ها و آرایه‌های رشته‌ای `(String[])` پیاده‌سازی شده‌اند و فقط باعث شلوغی این فصل می‌شوند. آن‌ها را به‌عنوان تمرین به شما واگذار می‌کنم.

ممکن است یک نکتهٔ دیگر ذهن شما را مشغول کرده باشد: تعریف ثابت‌های مربوط به کد خطا (error code constants). این موارد در کلاس `ArgsException` تعریف شده‌اند (Listing 14-7).
</div>

Listing 14-7 - ArgsException.java

```java
import static com.objectmentor.utilities.args.ArgsException.ErrorCode.*;
public class ArgsException extends Exception
{
    private char errorArgumentId = '\0';
    private String errorParameter = null;
    private ErrorCode errorCode = OK;
    public ArgsException () {}
    public ArgsException (String message) { super (message); }
    public ArgsException (ErrorCode errorCode) { this.errorCode = errorCode; }
    public ArgsException (ErrorCode errorCode, String errorParameter)
    {
        this.errorCode = errorCode;
        this.errorParameter = errorParameter;
    }
    public ArgsException (ErrorCode errorCode, char errorArgumentId,
                          String errorParameter)
    {
        this.errorCode = errorCode;
        this.errorParameter = errorParameter;
        this.errorArgumentId = errorArgumentId;
    }
    public char
    getErrorArgumentId ()
    {
        return errorArgumentId;
    }
    public void
    setErrorArgumentId (char errorArgumentId)
    {
        this.errorArgumentId = errorArgumentId;
    }
    public String
    getErrorParameter ()
    {
        return errorParameter;
    }
    public void
    setErrorParameter (String errorParameter)
    {
        this.errorParameter = errorParameter;
    }
    public ErrorCode
    getErrorCode ()
    {
        return errorCode;
    }
    public void
    setErrorCode (ErrorCode errorCode)
    {
        this.errorCode = errorCode;
    }
    public String
    errorMessage ()
    {
        switch (errorCode)
            {
            case OK:
                return "TILT: Should not get here.";
            case UNEXPECTED_ARGUMENT:
                return String.format ("Argument -%c unexpected.",
                                      errorArgumentId);
            case MISSING_STRING:
                return String.format (
                    "Could not find string parameter for -%c.",
                    errorArgumentId);
            case INVALID_INTEGER:
                return String.format (
                    "Argument -%c expects an integer but was '%s'.",
                    errorArgumentId, errorParameter);
            case MISSING_INTEGER:
                return String.format (
                    "Could not find integer parameter for -%c.",
                    errorArgumentId);
            case INVALID_DOUBLE:
                return String.format (
                    "Argument -%c expects a double but was '%s'.",
                    errorArgumentId, errorParameter);
            case MISSING_DOUBLE:
                return String.format (
                    "Could not find double parameter for -%c.",
                    errorArgumentId);
            case INVALID_ARGUMENT_NAME:
                return String.format ("'%c' is not a valid argument name.",
                                      errorArgumentId);
            case INVALID_ARGUMENT_FORMAT:
                return String.format ("'%s' is not a valid argument format.",
                                      errorParameter);
            }
        return "";
    }
    public enum ErrorCode
    {
        OK,
        INVALID_ARGUMENT_FORMAT,
        UNEXPECTED_ARGUMENT,
        INVALID_ARGUMENT_NAME,
        MISSING_STRING,
        MISSING_INTEGER,
        INVALID_INTEGER,
        MISSING_DOUBLE,
        INVALID_DOUBLE
    }
}
```
<div dir="rtl">

جالبه که ببینیم برای پیاده‌سازی جزئیات چنین مفهوم ساده‌ای، این‌همه کد لازم شده. یکی از دلایلش اینه که ما از زبانی استفاده می‌کنیم که به‌طور خاص پرحرفه! جاوا، به‌عنوان یک زبان ایستا از نظر نوع‌دهی (statically typed)، مجبورمون می‌کنه که برای راضی کردن سیستم نوع‌دهی، کد زیادی بنویسیم. در زبان‌هایی مثل `Ruby،` `Python` یا `Smalltalk،` این برنامه بسیار جمع‌وجورتر می‌شد.

لطفاً یک بار دیگه این کد رو با دقت بخونید. به نام‌گذاری‌ها، اندازهٔ توابع، و قالب‌بندی کد توجه ویژه‌ای داشته باشید. اگر یک برنامه‌نویس با تجربه باشید، ممکنه به بعضی از سبک‌ها یا ساختارهای استفاده‌شده انتقادهایی داشته باشید. اما در کل، امیدوارم به این نتیجه برسید که این برنامه به‌خوبی نوشته شده و ساختار تمیزی دارد.

برای مثال، باید کاملاً مشخص باشه که چطور می‌تونید یک نوع آرگومان جدید، مثل تاریخ (Date) یا عدد مختلط (Complex Number) اضافه کنید و این کار چقدر ساده خواهد بود. در واقع، برای این کار فقط باید:

یک زیرکلاس جدید از `ArgumentMarshaler` تعریف کنید،

یک تابع `getXXX` جدید بنویسید،

و یک case جدید به تابع `parseSchemaElement` اضافه کنید.
احتمالاً باید یک `ArgsException`.`ErrorCode` جدید هم اضافه بشه و یک پیام خطای جدید هم برایش تعریف بشه.

## چطور این کار رو انجام دادم؟
بذارید خیالتون رو راحت کنم: من این برنامه رو به‌صورت یکجا، از اول تا آخر، توی همین فرم نهایی ننوشتم. مهم‌تر از اون، انتظار ندارم که شما هم بتونید برنامه‌های تمیز و شسته‌رفته‌ای رو در یک‌بار نوشتن تولید کنید.
اگر از چند دهه گذشته چیزی یاد گرفته باشیم، اینه که برنامه‌نویسی بیشتر یک مهارته تا یک علم. برای نوشتن کد تمیز، باید اول کد کثیف بنویسید و بعد اون رو تمیز کنید.

این نباید براتون تعجب‌آور باشه. ما این حقیقت رو توی مدرسه یاد گرفتیم، وقتی معلم‌هامون (معمولاً بی‌نتیجه!) سعی می‌کردن ما رو قانع کنن که پیش‌نویس‌های اولیه بنویسیم.
فرآیندش، همون‌طور که می‌گفتن، این بود که باید یک پیش‌نویس بنویسیم، بعد نسخهٔ دوم، و بعد چند نسخهٔ بعدی، تا برسیم به نسخهٔ نهایی.
اون‌ها سعی داشتن بهمون بگن که نوشتن یک متن تمیز، فرایندی از پالایش گام‌به‌گام (successive refinement) است.

بیشتر برنامه‌نویس‌های تازه‌کار (مثل بیشتر دانش‌آموزها) خیلی خوب این توصیه رو رعایت نمی‌کنن. اون‌ها فکر می‌کنن هدف اصلی اینه که برنامه "کار کنه". وقتی برنامه "کار کرد"، می‌رن سراغ کار بعدی و برنامه رو همون‌جوری که بالاخره به کار افتاده، رها می‌کنن.
اما بیشتر برنامه‌نویس‌های باتجربه می‌دونن که این کار، خودکشی حرفه‌ایه.

## Args: پیش‌نویس اولیه
Listing 14-8 نسخه‌ای اولیه از کلاس `Args` رو نشون می‌ده. این نسخه "کار می‌کنه"، ولی به‌هم‌ریخته است.
</div>

Listing 14-8 - Args.java (first draft)

```java
import java.text.ParseException;
import java.util.*;
public class Args
{
    private String schema;
    private String[] args;
    private boolean valid = true;
    private Set<Character> unexpectedArguments = new TreeSet<Character> ();
    private Map<Character, Boolean> booleanArgs
        = new HashMap<Character, Boolean> ();
    private Map<Character, String> stringArgs
        = new HashMap<Character, String> ();
    private Map<Character, Integer> intArgs
        = new HashMap<Character, Integer> ();
    private Set<Character> argsFound = new HashSet<Character> ();
    private int currentArgument;
    private char errorArgumentId = '\0';
    private String errorParameter = "TILT";
    private ErrorCode errorCode = ErrorCode.OK;
    private enum ErrorCode
    {
        OK,
        MISSING_STRING,
        MISSING_INTEGER,
        INVALID_INTEGER,
        UNEXPECTED_ARGUMENT
    }
    public Args (String schema, String[] args) throws ParseException
    {
        this.schema = schema;
        this.args = args;
        valid = parse ();
    }
    private boolean
    parse () throws ParseException
    {
        if (schema.length () == 0 && args.length == 0)
            return true;
        parseSchema ();
        try
            {
                parseArguments ();
            }
        catch (ArgsException e)
            {
            }
        return valid;
    }
    private boolean
    parseSchema () throws ParseException
    {
        for (String element : schema.split (","))
            {
                if (element.length () > 0)
                    {
                        String trimmedElement = element.trim ();
                        parseSchemaElement (trimmedElement);
                    }
            }
        return true;
    }
    private void
    parseSchemaElement (String element) throws ParseException
    {
        char elementId = element.charAt (0);
        String elementTail = element.substring (1);
        validateSchemaElementId (elementId);
        if (isBooleanSchemaElement (elementTail))
            parseBooleanSchemaElement (elementId);
        else if (isStringSchemaElement (elementTail))
            parseStringSchemaElement (elementId);
        else if (isIntegerSchemaElement (elementTail))
            {
                parseIntegerSchemaElement (elementId);
            }
        else
            {
                throw new ParseException (
                    String.format ("Argument: %c has invalid format: %s.",
                                   elementId, elementTail),
                    0);
            }
    }
    private void
    validateSchemaElementId (char elementId) throws ParseException
    {
        if (!Character.isLetter (elementId))
            {
                throw new ParseException ("Bad character:" + elementId
                                              + "in Args format: " + schema,
                                          0);
            }
    }
    private void
    parseBooleanSchemaElement (char elementId)
    {
        booleanArgs.put (elementId, false);
    }
    private void
    parseIntegerSchemaElement (char elementId)
    {
        intArgs.put (elementId, 0);
    }
    private void
    parseStringSchemaElement (char elementId)
    {
        stringArgs.put (elementId, "");
    }
    private boolean
    isStringSchemaElement (String elementTail)
    {
        return elementTail.equals ("*");
    }
    private boolean
    isBooleanSchemaElement (String elementTail)
    {
        return elementTail.length () == 0;
    }
    private boolean
    isIntegerSchemaElement (String elementTail)
    {
        return elementTail.equals ("#");
    }
    private boolean
    parseArguments () throws ArgsException
    {
        for (currentArgument = 0; currentArgument < args.length;
             currentArgument++)
            {
                String arg = args[currentArgument];
                parseArgument (arg);
            }
        return true;
    }
    private void
    parseArgument (String arg) throws ArgsException
    {
        if (arg.startsWith ("-"))
            parseElements (arg);
    }
    private void
    parseElements (String arg) throws ArgsException
    {
        for (int i = 1; i < arg.length (); i++)
            parseElement (arg.charAt (i));
    }
    private void
    parseElement (char argChar) throws ArgsException
    {
        if (setArgument (argChar))
            argsFound.add (argChar);
        else
            {
                unexpectedArguments.add (argChar);
                errorCode = ErrorCode.UNEXPECTED_ARGUMENT;
                valid = false;
            }
    }
    private boolean
    setArgument (char argChar) throws ArgsException
    {
        if (isBooleanArg (argChar))
            setBooleanArg (argChar, true);
        else if (isStringArg (argChar))
            setStringArg (argChar);
        else if (isIntArg (argChar))
            setIntArg (argChar);
        else
            return false;
        return true;
    }
    private boolean
    isIntArg (char argChar)
    {
        return intArgs.containsKey (argChar);
    }
    private void
    setIntArg (char argChar) throws ArgsException
    {
        currentArgument++;
        String parameter = null;
        try
            {
                parameter = args[currentArgument];
                intArgs.put (argChar, new Integer (parameter));
            }
        catch (ArrayIndexOutOfBoundsException e)
            {
                valid = false;
                errorArgumentId = argChar;
                errorCode = ErrorCode.MISSING_INTEGER;
                throw new ArgsException ();
            }
        catch (NumberFormatException e)
            {
                valid = false;
                errorArgumentId = argChar;
                errorParameter = parameter;
                errorCode = ErrorCode.INVALID_INTEGER;
                throw new ArgsException ();
            }
    }
    private void
    setStringArg (char argChar) throws ArgsException
    {
        currentArgument++;
        try
            {
                stringArgs.put (argChar, args[currentArgument]);
            }
        catch (ArrayIndexOutOfBoundsException e)
            {
                valid = false;
                errorArgumentId = argChar;
                errorCode = ErrorCode.MISSING_STRING;
                throw new ArgsException ();
            }
    }
    private boolean
    isStringArg (char argChar)
    {
        return stringArgs.containsKey (argChar);
    }
    private void
    setBooleanArg (char argChar, boolean value)
    {
        booleanArgs.put (argChar, value);
    }
    private boolean
    isBooleanArg (char argChar)
    {
        return booleanArgs.containsKey (argChar);
    }
    public int
    cardinality ()
    {
        return argsFound.size ();
    }
    public String
    usage ()
    {
        if (schema.length () > 0)
            return "-[" + schema + "]";
        else
            return "";
    }
    public String
    errorMessage () throws Exception
    {
        switch (errorCode)
            {
            case OK:
                throw new Exception ("TILT: Should not get here.");
            case UNEXPECTED_ARGUMENT:
                return unexpectedArgumentMessage ();
            case MISSING_STRING:
                return String.format (
                    "Could not find string parameter for -%c.",
                    errorArgumentId);
            case INVALID_INTEGER:
                return String.format (
                    "Argument -%c expects an integer but was '%s'.",
                    errorArgumentId, errorParameter);
            case MISSING_INTEGER:
                return String.format (
                    "Could not find integer parameter for -%c.",
                    errorArgumentId);
            }
        return "";
    }
    private String
    unexpectedArgumentMessage ()
    {
        StringBuffer message = new StringBuffer ("Argument(s) -");
        for (char c : unexpectedArguments)
            {
                message.append (c);
            }
        message.append (" unexpected.");
        return message.toString ();
    }
    private boolean
    falseIfNull (Boolean b)
    {
        return b != null && b;
    }
    private int
    zeroIfNull (Integer i)
    {
        return i == null ? 0 : i;
    }
    private String
    blankIfNull (String s)
    {
        return s == null ? "" : s;
    }
    public String
    getString (char arg)
    {
        return blankIfNull (stringArgs.get (arg));
    }
    public int
    getInt (char arg)
    {
        return zeroIfNull (intArgs.get (arg));
    }
    public boolean
    getBoolean (char arg)
    {
        return falseIfNull (booleanArgs.get (arg));
    }
    public boolean
    has (char arg)
    {
        return argsFound.contains (arg);
    }
    public boolean
    isValid ()
    {
        return valid;
    }
    private class ArgsException extends Exception
    {
    }
}
```

<div dir="rtl">

امیدوارم واکنش اولیه شما به این حجم کد این باشه که "خوشحالم که اون رو به همین شکل رها نکرده!". اگر این احساس رو دارید، پس به یاد داشته باشید که دیگران هم همین‌طور در مورد کدی که شما به‌صورت پیش‌نویس رها می‌کنید، احساس خواهند کرد.

در واقع، شاید `first draft` شاید مهربانانه‌ترین چیزی باشه که می‌تونید در مورد این کد بگید. این کد قطعاً در حال پیشرفت است. تعداد زیاد متغیرهای نمونه (instance variables) نگران‌کننده است. رشته‌های عجیب مثل `TILT`، **HashSet**‌ ها و **TreeSet**‌ ها، و بلاک‌های try-catch-catch همه با هم جمع شده‌اند و تبدیل به یک پشتهٔ عفونت‌زای (festering pile) شده‌اند.

من قصد نداشتم که یک پشتهٔ عفونت‌زا بنویسم. در واقع، تلاش می‌کردم که همه چیز رو به‌طور نسبتاً سازمان‌دهی‌شده نگه دارم. احتمالاً می‌توانید این رو از انتخاب نام توابع و متغیرها و این حقیقت که برنامه ساختار نسبتاً خامی دارد، متوجه بشید. اما، واضح است که من اجازه دادم مشکل از کنترل من خارج بشه.

این بهم‌ریختگی به‌تدریج ساخته شد. نسخه‌های قبلی اصلاً به این اندازه بد نبودند. برای مثال، Listing 14-9 نسخه‌ای از کد رو نشون می‌ده که تنها آرگومان‌های بولی کار می‌کردند.

</div>

Listing 14-9 - Args.java (Boolean only)

```java
package com.objectmentor.utilities.getopts;
import java.util.*;
public class Args
{
    private String schema;
    private String[] args;
    private boolean valid;
    private Set<Character> unexpectedArguments = new TreeSet<Character> ();
    private Map<Character, Boolean> booleanArgs
        = new HashMap<Character, Boolean> ();
    private int numberOfArguments = 0;
    public Args (String schema, String[] args)
    {
        this.schema = schema;
        this.args = args;
        valid = parse ();
    }
    public boolean
    isValid ()
    {
        return valid;
    }
    private boolean
    parse ()
    {
        if (schema.length () == 0 && args.length == 0)
            return true;
        parseSchema ();
        parseArguments ();
        return unexpectedArguments.size () == 0;
    }
    private boolean
    parseSchema ()
    {
        for (String element : schema.split (","))
            {
                parseSchemaElement (element);
            }
        return true;
    }
    private void
    parseSchemaElement (String element)
    {
        if (element.length () == 1)
            {
                parseBooleanSchemaElement (element);
            }
    }
    private void
    parseBooleanSchemaElement (String element)
    {
        char c = element.charAt (0);
        if (Character.isLetter (c))
            {
                booleanArgs.put (c, false);
            }
    }
    private boolean
    parseArguments ()
    {
        for (String arg : args)
            parseArgument (arg);
        return true;
    }
    private void
    parseArgument (String arg)
    {
        if (arg.startsWith ("-"))
            parseElements (arg);
    }
    private void
    parseElements (String arg)
    {
        for (int i = 1; i < arg.length (); i++)
            parseElement (arg.charAt (i));
    }
    private void
    parseElement (char argChar)
    {
        if (isBoolean (argChar))
            {
                numberOfArguments++;
                setBooleanArg (argChar, true);
            }
        else
            unexpectedArguments.add (argChar);
    }
    private void
    setBooleanArg (char argChar, boolean value)
    {
        booleanArgs.put (argChar, value);
    }
    private boolean
    isBoolean (char argChar)
    {
        return booleanArgs.containsKey (argChar);
    }
    public int
    cardinality ()
    {
        return numberOfArguments;
    }
    public String
    usage ()
    {
        if (schema.length () > 0)
            return "-[" + schema + "]";
        else
            return "";
    }
    public String
    errorMessage ()
    {
        if (unexpectedArguments.size () > 0)
            {
                return unexpectedArgumentMessage ();
            }
        else
            return "";
    }
    private String
    unexpectedArgumentMessage ()
    {
        StringBuffer message = new StringBuffer ("Argument(s) -");
        for (char c : unexpectedArguments)
            {
                message.append (c);
            }
        message.append (" unexpected.");
        return message.toString ();
    }
    public boolean
    getBoolean (char arg)
    {
        return booleanArgs.get (arg);
    }
}
```

<div dir="rtl">

با اینکه می‌توانید ایرادات زیادی در این کد پیدا کنید، اما واقعاً آنقدرها هم بد نیست. کد فشرده و ساده است و فهمیدنش راحت. با این حال، در دل این کد، به‌راحتی می‌توان بذرهای پشتهٔ عفونت‌زای بعدی را مشاهده کرد. خیلی واضح است که چطور این کد به آن بهم‌ریختگی تبدیل شد.

توجه کنید که آن بهم‌ریختگی تنها دو نوع آرگومان بیشتر از این کد دارد: رشته (`String`) و عدد (`integer`). اضافه کردن تنها دو نوع آرگومان دیگر تاثیر منفی عظیمی روی کد داشت. این تغییر کد را از چیزی که می‌توانست به‌طور نسبتاً قابل نگهداری باشد، به چیزی تبدیل کرد که من انتظار دارم به زودی پر از باگ و مشکلات مختلف بشود.

من این دو نوع آرگومان را به‌طور تدریجی اضافه کردم. اول، آرگومان رشته را اضافه کردم که این نتیجه را داد:

</div>

Listing 14-10 - Args.java (Boolean and String)

```java
package com.objectmentor.utilities.getopts;
import java.text.ParseException;
import java.util.*;
public class Args
{
    private String schema;
    private String[] args;
    private boolean valid = true;
    private Set<Character> unexpectedArguments = new TreeSet<Character> ();
    private Map<Character, Boolean> booleanArgs
        = new HashMap<Character, Boolean> ();
    private Map<Character, String> stringArgs
        = new HashMap<Character, String> ();
    private Set<Character> argsFound = new HashSet<Character> ();
    private int currentArgument;
    private char errorArgument = '\0';
    enum ErrorCode
    {
        OK,
        MISSING_STRING
    }
    private ErrorCode errorCode = ErrorCode.OK;
    public Args (String schema, String[] args) throws ParseException
    {
        this.schema = schema;
        this.args = args;
        valid = parse ();
    }
    private boolean
    parse () throws ParseException
    {
        if (schema.length () == 0 && args.length == 0)
            return true;
        parseSchema ();
        parseArguments ();
        return valid;
    }
    private boolean
    parseSchema () throws ParseException
    {
        for (String element : schema.split (","))
            {
                if (element.length () > 0)
                    {
                        String trimmedElement = element.trim ();
                        parseSchemaElement (trimmedElement);
                    }
            }
        return true;
    }
    private void
    parseSchemaElement (String element) throws ParseException
    {
        char elementId = element.charAt (0);
        String elementTail = element.substring (1);
        validateSchemaElementId (elementId);
        if (isBooleanSchemaElement (elementTail))
            parseBooleanSchemaElement (elementId);
        else if (isStringSchemaElement (elementTail))
            parseStringSchemaElement (elementId);
    }
    private void
    validateSchemaElementId (char elementId) throws ParseException
    {
        if (!Character.isLetter (elementId))
            {
                throw new ParseException ("Bad character:" + elementId
                                              + "in Args format: " + schema,
                                          0);
            }
    }
    private void
    parseStringSchemaElement (char elementId)
    {
        stringArgs.put (elementId, "");
    }
    private boolean
    isStringSchemaElement (String elementTail)
    {
        return elementTail.equals ("*");
    }
    private boolean
    isBooleanSchemaElement (String elementTail)
    {
        return elementTail.length () == 0;
    }
    private void
    parseBooleanSchemaElement (char elementId)
    {
        booleanArgs.put (elementId, false);
    }
    private boolean
    parseArguments ()
    {
        for (currentArgument = 0; currentArgument < args.length;
             currentArgument++)
            {
                String arg = args[currentArgument];
                parseArgument (arg);
            }
        return true;
    }
    private void
    parseArgument (String arg)
    {
        if (arg.startsWith ("-"))
            parseElements (arg);
    }
    private void
    parseElements (String arg)
    {
        for (int i = 1; i < arg.length (); i++)
            parseElement (arg.charAt (i));
    }
    private void
    parseElement (char argChar)
    {
        if (setArgument (argChar))
            argsFound.add (argChar);
        else
            {
                unexpectedArguments.add (argChar);
                valid = false;
            }
    }
    private boolean
    setArgument (char argChar)
    {
        boolean set = true;
        if (isBoolean (argChar))
            setBooleanArg (argChar, true);
        else if (isString (argChar))
            setStringArg (argChar, "");
        else
            set = false;
        return set;
    }
    private void
    setStringArg (char argChar, String s)
    {
        currentArgument++;
        try
            {
                stringArgs.put (argChar, args[currentArgument]);
            }
        catch (ArrayIndexOutOfBoundsException e)
            {
                valid = false;
                errorArgument = argChar;
                errorCode = ErrorCode.MISSING_STRING;
            }
    }
    private boolean
    isString (char argChar)
    {
        return stringArgs.containsKey (argChar);
    }
    private void
    setBooleanArg (char argChar, boolean value)
    {
        booleanArgs.put (argChar, value);
    }
    private boolean
    isBoolean (char argChar)
    {
        return booleanArgs.containsKey (argChar);
    }
    public int
    cardinality ()
    {
        return argsFound.size ();
    }
    public String
    usage ()
    {
        if (schema.length () > 0)
            return "-[" + schema + "]";
        else
            return "";
    }
    public String
    errorMessage () throws Exception
    {
        if (unexpectedArguments.size () > 0)
            {
                return unexpectedArgumentMessage ();
            }
        else
            switch (errorCode)
                {
                case MISSING_STRING:
                    return String.format (
                        "Could not find string parameter for -%c.",
                        errorArgument);
                case OK:
                    throw new Exception ("TILT: Should not get here.");
                }
        return "";
    }
    private String
    unexpectedArgumentMessage ()
    {
        StringBuffer message = new StringBuffer ("Argument(s) -");
        for (char c : unexpectedArguments)
            {
                message.append (c);
            }
        message.append (" unexpected.");
        return message.toString ();
    }
    public boolean
    getBoolean (char arg)
    {
        return falseIfNull (booleanArgs.get (arg));
    }
    private boolean
    falseIfNull (Boolean b)
    {
        return b == null ? false : b;
    }
    public String
    getString (char arg)
    {
        return blankIfNull (stringArgs.get (arg));
    }
    private String
    blankIfNull (String s)
    {
        return s == null ? "" : s;
    }
    public boolean
    has (char arg)
    {
        return argsFound.contains (arg);
    }
    public boolean
    isValid ()
    {
        return valid;
    }
}
```

<div dir="rtl">

می‌بینید که اوضاع کم‌کم داره از کنترل خارج می‌شه. هنوز فاجعه‌بار نیست، اما بهم‌ریختگی داره کم‌کم رشد می‌کنه. فعلاً فقط یه توده‌ است، اما هنوز عفونت‌زده و فاسد نشده. اضافه شدن نوع آرگومان عددی بود که این توده رو واقعاً وارد مرحلهٔ تخمیر و فساد کرد.

## پس متوقف شدم
حداقل دو نوع آرگومان دیگه هم داشتم که می‌خواستم اضافه کنم، اما مشخص بود که اوضاع رو خیلی بدتر خواهند کرد. اگر بی‌مهابا جلو می‌رفتم، احتمالاً می‌تونستم کاری کنم که درست کار کنن، اما پشت سرم یه آشوب بزرگی باقی می‌ذاشتم که دیگه قابل اصلاح نبود. اگر قرار بود ساختار این کد واقعاً قابل نگهداری بمونه، الان وقت اصلاحش بود.

پس افزودن قابلیت‌ها رو متوقف کردم و شروع به بازآرایی (Refactoring) کردم. چون تازه آرگومان‌های رشته‌ای و عددی رو اضافه کرده بودم، خوب می‌دونستم که هر نوع آرگومان جدید به کدهای تازه‌ای در سه نقطهٔ اصلی نیاز داره:

ابتدا باید یه راهی برای تحلیل عنصر اسکیمای اون آرگومان تعریف می‌شد تا بتونم `HashMap` مخصوص اون نوع رو انتخاب کنم.

بعد، اون نوع آرگومان باید در آرگومان‌های خط فرمان تجزیه (parse) می‌شد و به نوع واقعی خودش تبدیل می‌شد.

در نهایت، یه متد `getXXX` مخصوص لازم بود که مقدار اون آرگومان رو به عنوان نوع واقعی خودش به کاربر برگردونه.

چندین نوع مختلف که همشون متدهایی مشابه دارن — این یعنی وقتشه که یه کلاس جدید تعریف کنیم. و این‌طوری بود که مفهوم `ArgumentMarshaler` شکل گرفت.

دربارهٔ رویکرد تدریجی (Incrementalism)
یکی از بدترین کارهایی که می‌تونید با یه برنامه بکنید اینه که در نام اصلاح و بهبود، تغییرات عظیم ساختاری در اون ایجاد کنید. بعضی برنامه‌ها هیچ‌وقت از این «بهبودها» جان سالم به در نمی‌برن. مشکل اینجاست که خیلی سخته برنامه‌ای رو که قبل از این تغییرات به‌درستی کار می‌کرد، دوباره دقیقاً به همون وضعیت عملکردی برسونید.

برای جلوگیری از این موضوع، من از روش توسعه مبتنی بر آزمون (`TDD`) استفاده می‌کنم. یکی از اصول اصلی در این روش اینه که سیستم همیشه باید کار کنه. به‌عبارت دیگه، در `TDD` من اجازه ندارم تغییری در سیستم بدم که باعث خراب شدنش بشه. هر تغییری که اعمال می‌کنم باید سیستم رو در همون وضعیت قبلی نگه داره.

برای رسیدن به این هدف، باید یه مجموعه تست خودکار داشته باشم که هر لحظه بتونم اجراش کنم تا مطمئن شم که رفتار سیستم تغییر نکرده. برای کلاس `Args،` من همزمان با ساختن همون پشتهٔ عفونت‌زا، یه مجموعه تست واحد (unit test) و تست پذیرش (acceptance test) ایجاد کرده بودم. تست‌های واحد رو با استفاده از JUnit در جاوا نوشته بودم. تست‌های پذیرش هم به صورت صفحات ویکی در `FitNesse` نوشته شده بودن.

هر زمان که می‌خواستم، می‌تونستم این تست‌ها رو اجرا کنم و اگر همه‌شون پاس می‌شدن، با اطمینان می‌گفتم که سیستم همون‌طوری که انتظار دارم کار می‌کنه.

پس من جلو رفتم و تعداد زیادی تغییر خیلی کوچک اعمال کردم. هر تغییر، ساختار سیستم رو به سمت مفهوم `ArgumentMarshaler` هدایت می‌کرد. و در عین حال، هر تغییر سیستم رو سالم نگه می‌داشت.

اولین کاری که کردم، اضافه کردن اسکلت اولیهٔ کلاس `ArgumentMarshaler` به انتهای همون پشتهٔ عفونت‌زا بود.
</div>

Listing 14-11 - ArgumentMarshaller appended to Args.java

```java
private class ArgumentMarshaler
{
    private boolean booleanValue = false;
    public void setBoolean(boolean value) { booleanValue = value; }
    public boolean getBoolean() { return booleanValue; }
}
private class BooleanArgumentMarshaler extends ArgumentMarshaler
{}
private class StringArgumentMarshaler extends ArgumentMarshaler
{}
private class IntegerArgumentMarshaler extends ArgumentMarshaler
{}
}
```

<div dir="rtl">

بدیهیه که این تغییر قرار نبود چیزی رو خراب کنه. پس قدم بعدی رو برداشتم و ساده‌ترین تغییری رو اعمال کردم که می‌تونستم، طوری که کمترین آسیب ممکن رو وارد کنه. من `HashMap` مربوط به آرگومان‌های بولی (`Boolean`) رو تغییر دادم تا به جای نگهداری مقدار نهایی، یک `ArgumentMarshaler` نگه‌داری کنه.

</div>

```java
private Map<Character, ArgumentMarshaler> booleanArgs = 
    new HashMap<Character, ArgumentMarshaler>();
```

<div dir="rtl">

این تغییر چند خط از کد رو خراب کرد، که من سریعاً اصلاحشون کردم.

</div>

```java
...
private void parseBooleanSchemaElement(char elementId) 
{
   booleanArgs.put(elementId, new BooleanArgumentMarshaler());
}
...

private void setBooleanArg(char argChar, boolean value) {
   booleanArgs.get(argChar).setBoolean(value);
}
...

public boolean getBoolean(char arg) {
   return falseIfNull(booleanArgs.get(arg).getBoolean());
}
```

<div dir="rtl">

دقت کنید که این تغییرات دقیقاً در همون بخش‌هایی اعمال شدن که قبلاً هم بهشون اشاره کردم: تحلیل (`parse`)، تنظیم (`set`) و دریافت (`get`) مقدار برای نوع آرگومان.
با وجود اینکه این تغییر خیلی کوچک بود، ولی متأسفانه باعث شد بعضی از تست‌ها fail بشن.

اگر به متد `getBoolean` دقت کنید، می‌بینید که اگه این تابع رو با آرگومانی مثل 'y' صدا بزنید، ولی هیچ آرگومان 'y'ای وجود نداشته باشه، اون‌وقت `booleanArgs.get('y')` مقدار `null` برمی‌گردونه و این باعث می‌شه که تابع، یه استثنای `NullPointerException` بندازه.
قبلاً برای جلوگیری از این اتفاق از تابعی به نام `falseIfNull` استفاده می‌کردم، ولی با تغییری که دادم، دیگه این تابع کارایی نداشت.

رویکرد تدریجی (Incrementalism) ایجاب می‌کرد که اول همین مشکل رو برطرف کنم، قبل از اینکه تغییرات دیگه‌ای اعمال کنم.
خوشبختانه رفع این مشکل خیلی سخت نبود. فقط کافی بود بررسی null بودن رو جابه‌جا کنم — دیگه لازم نبود بررسی کنم که مقدار بولی null هست یا نه؛ باید بررسی می‌کردم که آیا `ArgumentMarshaler` موجود هست یا نه.

اول از همه، فراخوانی تابع `falseIfNull` در `getBoolean` رو حذف کردم چون دیگه بی‌فایده بود، و بعد خود تابع `falseIfNull` رو هم کاملاً پاک کردم.
تست‌ها هنوز هم به همون شکل fail می‌شدن، و این یعنی با اطمینان می‌تونستم بگم که اشکال جدیدی ایجاد نکردم.

</div>

```java
public boolean getBoolean(char arg) 
{
    return booleanArgs.get(arg).getBoolean();
}
```

<div dir="rtl">

در مرحلهٔ بعد، تابع رو به دو خط تقسیم کردم و نمونهٔ `ArgumentMarshaler` رو داخل یک متغیر جداگانه به نام `argumentMarshaller` قرار دادم.
از اسم طولانی این متغیر خوشم نیومد؛ خیلی تکراری و شلوغ‌کننده بود و باعث می‌شد تابع ظاهر نامرتبی داشته باشه.
پس اسمش رو کوتاه کردم و گذاشتم `am` [یادداشت N5].
</div>

```java
public boolean getBoolean(char arg)
{
   Args.ArgumentMarshaler am = booleanArgs.get(arg);
   return am.getBoolean();
}
```

<div dir="rtl">

و بعدش منطق تشخیص مقدار `null` رو اضافه کردم.

</div>

```java
public boolean getBoolean(char arg) 
{
   Args.ArgumentMarshaler am = booleanArgs.get(arg);
   return am != null && am.getBoolean();
}
```

<div dir="rtl">

## آرگومان‌های رشته‌ای (String Arguments)
اضافه کردن آرگومان‌های `String` خیلی شبیه به اضافه کردن آرگومان‌های `boolean` بود.
باید `HashMap` مربوطه رو تغییر می‌دادم و مطمئن می‌شدم که توابع `parse،` `set` و `get` به‌درستی کار می‌کنن.

نباید چیز شگفت‌انگیزی در ادامه وجود داشته باشه، مگر اینکه متوجه بشید که من بخش زیادی از پیاده‌سازی `marshalling` رو در کلاس پایهٔ `ArgumentMarshaller` قرار دادم،
به‌جای اینکه این منطق رو بین کلاس‌های مشتق‌شده پخش کنم.

</div>

```java
private Map<Character, ArgumentMarshaler> stringArgs
    = new HashMap<Character, ArgumentMarshaler> ();
... 

private void parseStringSchemaElement (char elementId)
{
    stringArgs.put (elementId, new StringArgumentMarshaler ());
}
... 

private void setStringArg (char argChar) throws ArgsException
{
    currentArgument++;
    try
        {
            stringArgs.get (argChar).setString (args[currentArgument]);
        }
    catch (ArrayIndexOutOfBoundsException e)
        {
            valid = false;
            errorArgumentId = argChar;
            errorCode = ErrorCode.MISSING_STRING;
            throw new ArgsException ();
        }
}
... 

public String getString (char arg)
{
    Args.ArgumentMarshaler am = stringArgs.get (arg);
    return am == null ? "" : am.getString ();
}
...

private class ArgumentMarshaler
{
    private boolean booleanValue = false;
    private String stringValue;

    public void setBoolean (boolean value)
    {
        booleanValue = value;
    }
    public boolean getBoolean ()
    {
        return booleanValue;
    }
    public void setString (String s)
    {
        stringValue = s;
    }
    public String getString ()
    {
        return stringValue == null ? "" : stringValue;
    }
}
```

<div dir="rtl">

باز هم، این تغییرات یکی‌یکی و به‌صورت تدریجی انجام شدن، به‌طوری که تست‌ها همچنان قابل اجرا بودن—حتی اگر همه‌شون پاس نمی‌کردن.
هر وقت یکی از تست‌ها `fail` می‌شد، اول اونو اصلاح می‌کردم و دوباره به حالت پاس‌شده درمی‌آوردم، بعد سراغ تغییر بعدی می‌رفتم.

تا اینجا احتمالاً متوجه هدف کلی من شده‌اید:
می‌خواستم تمام رفتارهای فعلی `marshalling` رو وارد کلاس پایه‌ی `ArgumentMarshaler` کنم،
و بعدش به‌تدریج این رفتارها رو به کلاس‌های مشتق‌شده انتقال بدم.
این کار باعث می‌شه برنامه در طول فرایند تغییر ساختار، همچنان کار کنه و نشکنه.

گام بعدی هم مشخص بود: باید قابلیت پردازش آرگومان‌های عدد صحیح (`int`) رو به `ArgumentMarshaler` منتقل می‌کردم.
و باز هم، مثل قبل، هیچ شگفتی خاصی در این مرحله وجود نداشت.

</div>

```java
private Map<Character, ArgumentMarshaler> intArgs
    = new HashMap<Character, ArgumentMarshaler> ();

private void parseIntegerSchemaElement (char elementId)
{
    intArgs.put (elementId, new IntegerArgumentMarshaler ());
}
...

private void setIntArg (char argChar) throws ArgsException
{
    currentArgument++;
    String parameter = null;
    try
        {
            parameter = args[currentArgument];
            intArgs.get (argChar).setInteger (Integer.parseInt (parameter));
        }
    catch (ArrayIndexOutOfBoundsException e)
        {
            valid = false;
            errorArgumentId = argChar;
            errorCode = ErrorCode.MISSING_INTEGER;
            throw new ArgsException ();
        }
    catch (NumberFormatException e)
        {
            valid = false;
            errorArgumentId = argChar;
            errorParameter = parameter;
            errorCode = ErrorCode.INVALID_INTEGER;
            throw new ArgsException ();
        }
}
...

public int getInt (char arg)
{
    Args.ArgumentMarshaler am = intArgs.get (arg);
    return am == null ? 0 : am.getInteger ();
}
...

private class ArgumentMarshaler
{
    private boolean booleanValue = false;
    private String stringValue;
    private int integerValue;
    public void setBoolean (boolean value)
    {
        booleanValue = value;
    }
    public boolean getBoolean ()
    {
        return booleanValue;
    }
    public void setString (String s)
    {
        stringValue = s;
    }
    public String getString ()
    {
        return stringValue == null ? "" : stringValue;
    }
    public void setInteger (int i)
    {
        integerValue = i;
    }
    public int getInteger ()
    {
        return integerValue;
    }
}
```

<div dir="rtl">

وقتی که تمام منطق `marshalling` به کلاس پایه‌ی `ArgumentMarshaler` منتقل شد، شروع کردم به انتقال عملکردها به کلاس‌های مشتق‌شده.
اولین قدم، منتقل کردن تابع `setBoolean` به کلاس `BooleanArgumentMarshaller` بود و مطمئن شدم که این تابع به‌درستی فراخوانی می‌شه.
برای این کار، یک متد انتزاعی (`abstract`) به نام `set` ایجاد کردم.

این کار پایه‌ای بود برای اینکه هر کلاس مشتق‌شده، رفتار مخصوص به خودش رو در متد `set` پیاده‌سازی کنه.
</div>

```java
private abstract class ArgumentMarshaler {
    protected boolean booleanValue = false;
    private String stringValue;
    private int integerValue;
    public void setBoolean(boolean value) {
        booleanValue = value;
    }
    public boolean getBoolean() {
        return booleanValue;
    }
    public void setString(String s) {
    stringValue = s;
    }
    public String getString() {
        return stringValue == null ? "" : stringValue;
    }
    public void setInteger(int i) {
        integerValue = i;
    }
    public int getInteger() {
        return integerValue;
    }
    public abstract void set(String s);
}
```

<div dir="rtl">

سپس متد `set` رو در کلاس `BooleanArgumentMarshaller` پیاده‌سازی کردم:
</div>

```java
private class BooleanArgumentMarshaler extends ArgumentMarshaler {
    public void set(String s) {
        booleanValue = true;
    }
}
```

<div dir="rtl">

و در نهایت، فراخوانی تابع `setBoolean` رو با فراخوانی `set` جایگزین کردم:
</div>

```java
private void setBooleanArg(char argChar, boolean value) {
    booleanArgs.get(argChar).set("true");
}
```

<div dir="rtl">

تمام تست‌ها همچنان پاس می‌شدن، و چون این تغییر باعث شد که متد set حالا در `BooleanArgumentMarshaller` پیاده‌سازی شده باشه، متد `setBoolean` رو از کلاس پایه‌ی `ArgumentMarshaler` حذف کردم.

به این نکته دقت کن که متد انتزاعی `set` یک آرگومان از نوع `String` دریافت می‌کنه،
اما پیاده‌سازی اون در `BooleanArgumentMarshaller` اصلاً از این آرگومان استفاده نمی‌کنه.
من این آرگومان رو عمداً اضافه کردم، چون می‌دونستم که کلاس‌های `StringArgumentMarshaller` و `IntegerArgumentMarshaller` به اون نیاز خواهند داشت.

در ادامه، تصمیم گرفتم تابع `get` رو هم به `BooleanArgumentMarshaler` منتقل کنم.
انتقال توابع `get` معمولاً کمی ناخوشایند هست، چون باید مقدار بازگشتی از نوع `Object` باشه
و در این مورد، باید به `Boolean` تبدیل (`cast`) بشه:
</div>

```java
public boolean getBoolean(char arg) {
    Args.ArgumentMarshaler am = booleanArgs.get(arg);
    return am != null && (Boolean) am.get();
}
```

<div dir="rtl">

برای اینکه کد کامپایل بشه، ابتدا متد `get` رو به صورت پیش‌فرض در `ArgumentMarshaler` اضافه کردم:
</div>

```java
private abstract class ArgumentMarshaler {
    ...
    public Object get() {
        return null;
    }
}
```

<div dir="rtl">

کد کامپایل شد اما تست‌ها شکست خوردن. برای درست کردن تست‌ها،
متد `get` رو `abstract` کردم و در `BooleanArgumentMarshaler` پیاده‌سازی‌اش کردم:
</div>

```java
private abstract class ArgumentMarshaler {
    ...
    public abstract Object get();
}

private class BooleanArgumentMarshaler extends ArgumentMarshaler {
    private boolean booleanValue = false;

    public void set(String s) {
        booleanValue = true;
    }

    public Object get() {
        return booleanValue;
    }
}

```

<div dir="rtl">

و حالا دوباره همه‌ی تست‌ها پاس می‌شن.
با این کار، متدهای `set` و `get` به‌طور کامل به کلاس `BooleanArgumentMarshaler` منتقل شدن.
این اجازه رو بهم داد که:

متد قدیمی `getBoolean` رو از `ArgumentMarshaler` حذف کنم

متغیر `booleanValue` رو از کلاس پایه حذف و در `BooleanArgumentMarshaler` به صورت `private` منتقل کنم

بعد از این، دقیقاً همین الگو رو برای آرگومان‌های `String` هم پیاده‌سازی کردم:
تابع‌های `set` و `get` رو منتقل کردم، تابع‌های بلااستفاده رو حذف کردم، و متغیرها رو جابجا کردم.
</div>

```java
private void setStringArg(char argChar) throws ArgsException {
    currentArgument++;
    try {
        stringArgs.get(argChar).set(args[currentArgument]);
    } catch (ArrayIndexOutOfBoundsException e) {
        valid = false;
        errorArgumentId = argChar;
        errorCode = ErrorCode.MISSING_STRING;
        throw new ArgsException();
    }
}
... 
public String getString(char arg) {
    Args.ArgumentMarshaler am = stringArgs.get(arg);
    return am == null ? "" : (String) am.get();
}
... 
private abstract class ArgumentMarshaler {
    private int integerValue;
    public void setInteger(int i) {
        integerValue = i;
    }
    public int getInteger() {
        return integerValue;
    }
    public abstract void set(String s);
    public abstract Object get();
}
private class BooleanArgumentMarshaler extends ArgumentMarshaler {
    private boolean booleanValue = false;
    public void set(String s) {
        booleanValue = true;
    }
    public Object get() {
        return booleanValue;
    }
}
private class StringArgumentMarshaler extends ArgumentMarshaler {
    private String stringValue = "";
    public void set(String s) {
        stringValue = s;
    }
    public Object get() {
        return stringValue;
    }
}
private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    public void set(String s) {}
    public Object get() {
        return null;
    }
}
```

<div dir="rtl">

در نهایت، من همین فرایند را برای اعداد صحیح (`integers`) تکرار کردم. این کار کمی پیچیده‌تر بود زیرا اعداد صحیح نیاز به تجزیه (`parsing`) داشتند و عملیات تجزیه می‌تواند استثنا (`exception`) ایجاد کند. اما نتیجه بهتر شد زیرا کل مفهوم `NumberFormatException` در داخل کلاس `IntegerArgumentMarshaler` پنهان شد.
</div>

```java
private boolean isIntArg(char argChar) {
    return intArgs.containsKey(argChar);
}
private void setIntArg(char argChar) throws ArgsException {
    currentArgument++;
    String parameter = null;
    try {
        parameter = args[currentArgument];
        intArgs.get(argChar).set(parameter);
    } catch (ArrayIndexOutOfBoundsException e) {
        valid = false;
        errorArgumentId = argChar;
        errorCode = ErrorCode.MISSING_INTEGER;
        throw new ArgsException();
    } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar;
        errorParameter = parameter;
        errorCode = ErrorCode.INVALID_INTEGER;
        throw e;
    }
}
... private void setBooleanArg(char argChar) {
    try {
        booleanArgs.get(argChar).set("true");
    } catch (ArgsException e) {
    }
}
... public int getInt(char arg) {
    Args.ArgumentMarshaler am = intArgs.get(arg);
    return am == null ? 0 : (Integer) am.get();
}
... private abstract class ArgumentMarshaler {
    public abstract void set(String s) throws ArgsException;
    public abstract Object get();
}
... private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    private int intValue = 0;
    public void set(String s) throws ArgsException {
        try {
            intValue = Integer.parseInt(s);
        } catch (NumberFormatException e) {
            throw new ArgsException();
        }
    }
    public Object get() {
        return intValue;
    }
}
```

<div dir="rtl">

مطمئناً، تست‌ها همچنان موفق بودند. سپس، من سه نقشه مختلف (`Map`) که در ابتدای الگوریتم وجود داشتند را حذف کردم. این کار سیستم را به طور کلی جنریک‌تر کرد. اما نمی‌توانستم آن‌ها را فقط با حذف کردن از سیستم پاک کنم زیرا این کار باعث خراب شدن سیستم می‌شد. در عوض، من یک نقشه جدید برای `ArgumentMarshaler` اضافه کردم و سپس به طور تدریجی، هر یک از متدها را تغییر دادم تا از این نقشه جدید به جای سه نقشه اصلی استفاده کنند.
</div>

```java
public class Args {
    ... private Map<Character, ArgumentMarshaler> booleanArgs =
        new HashMap<Character, ArgumentMarshaler>();
    private Map<Character, ArgumentMarshaler> stringArgs =
        new HashMap<Character, ArgumentMarshaler>();
    private Map<Character, ArgumentMarshaler> intArgs =
        new HashMap<Character, ArgumentMarshaler>();
    private Map<Character, ArgumentMarshaler> marshalers =
        new HashMap<Character, ArgumentMarshaler>();
    ... private void parseBooleanSchemaElement(char elementId) {
        ArgumentMarshaler m = new BooleanArgumentMarshaler();
        booleanArgs.put(elementId, m);
        marshalers.put(elementId, m);
    }
    private void parseIntegerSchemaElement(char elementId) {
        ArgumentMarshaler m = new IntegerArgumentMarshaler();
        intArgs.put(elementId, m);
        marshalers.put(elementId, m);
    }
    private void parseStringSchemaElement(char elementId) {
        ArgumentMarshaler m = new StringArgumentMarshaler();
        stringArgs.put(elementId, m);
        marshalers.put(elementId, m);
    }
}
```

<div dir="rtl">

مطمئناً، تمام تست‌ها همچنان موفق بودند. سپس، من متد `isBooleanArg` را از این حالت تغییر دادم:
</div>

```java
private boolean isBooleanArg(char argChar) {
   return booleanArgs.containsKey(argChar);
}
```

<div dir="rtl">
به این : 
</div>

```java
private boolean isBooleanArg(char argChar) {
    ArgumentMarshaler m = marshalers.get(argChar);
    return m instanceof BooleanArgumentMarshaler;
}
```

<div dir="rtl">

تست‌ها همچنان موفق بودند. بنابراین، من همین تغییر را برای متدهای `isIntArg` و `isStringArg` نیز اعمال کردم.
</div>

```java
private boolean isIntArg(char argChar) {
    ArgumentMarshaler m = marshalers.get(argChar);
    return m instanceof IntegerArgumentMarshaler;
}
private boolean isStringArg(char argChar) {
    ArgumentMarshaler m = marshalers.get(argChar);
    return m instanceof StringArgumentMarshaler;
}
```

<div dir="rtl">

تست‌ها همچنان موفق بودند. بنابراین، من تمام تماس‌های تکراری به `marshalers.get` را به شکل زیر حذف کردم:
</div>

```java
private boolean setArgument(char argChar) throws ArgsException {
    ArgumentMarshaler m = marshalers.get(argChar);
    if (isBooleanArg(m))
        setBooleanArg(argChar);
    else if (isStringArg(m))
        setStringArg(argChar);
    else if (isIntArg(m))
        setIntArg(argChar);
    else
        return false;
    return true;
}
private boolean isIntArg(ArgumentMarshaler m) {
    return m instanceof IntegerArgumentMarshaler;
}
private boolean isStringArg(ArgumentMarshaler m) {
    return m instanceof StringArgumentMarshaler;
}
private boolean isBooleanArg(ArgumentMarshaler m) {
    return m instanceof BooleanArgumentMarshaler;
}
```

<div dir="rtl">

این کار هیچ دلیلی برای وجود متدهای سه‌گانه `isxxxArg` باقی نگذاشت. بنابراین، من آن‌ها را به صورت درون‌خطی (inline) تغییر دادم:
</div>

```java
private boolean setArgument(char argChar) throws ArgsException {
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m instanceof BooleanArgumentMarshaler)
        setBooleanArg(argChar);
    else if (m instanceof StringArgumentMarshaler)
        setStringArg(argChar);
    else if (m instanceof IntegerArgumentMarshaler)
        setIntArg(argChar);
    else
        return false;
    return true;
}
```

<div dir="rtl">

سپس، من شروع به استفاده از نقشه `marshalers` در توابع set کردم و استفاده از سه نقشه دیگر را متوقف کردم. من ابتدا با مقادیر بولی (`boolean`) شروع کردم.
</div>

```java
private boolean setArgument(char argChar) throws ArgsException {
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m instanceof BooleanArgumentMarshaler)
        setBooleanArg(m);
    else if (m instanceof StringArgumentMarshaler)
        setStringArg(argChar);
    else if (m instanceof IntegerArgumentMarshaler)
        setIntArg(argChar);
    else
        return false;
    return true;
}
...
private void setBooleanArg(ArgumentMarshaler m) {
    try {
        m.set("true"); // was: booleanArgs.get(argChar).set("true");
    } 
    catch (ArgsException e) {}
}
```

<div dir="rtl">

تست‌ها همچنان موفق بودند، بنابراین همین کار را برای رشته‌ها (`Strings`) و اعداد صحیح (`Integers`) انجام دادم. این کار به من اجازه داد تا برخی از کدهای پیچیده مدیریت استثناها را درون تابع `setArgument` ادغام کنم.
</div>

```java
private boolean setArgument(char argChar) throws ArgsException {
    ArgumentMarshaler m = marshalers.get(argChar);
    try {
        if (m instanceof BooleanArgumentMarshaler)
            setBooleanArg(m);
        else if (m instanceof StringArgumentMarshaler)
            setStringArg(m);
        else if (m instanceof IntegerArgumentMarshaler)
            setIntArg(m);
        else
            return false;
    } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar;
        throw e;
    }
    return true;
}
private void setIntArg(ArgumentMarshaler m) throws ArgsException {
    currentArgument++;
    String parameter = null;
    try {
        parameter = args[currentArgument];
        m.set(parameter);
    } catch (ArrayIndexOutOfBoundsException e) {
        errorCode = ErrorCode.MISSING_INTEGER;
        throw new ArgsException();
    } catch (ArgsException e) {
        errorParameter = parameter;
        errorCode = ErrorCode.INVALID_INTEGER;
        throw e;
    }
}
private void setStringArg(ArgumentMarshaler m) throws ArgsException {
    currentArgument++;
    try {
        m.set(args[currentArgument]);
    } catch (ArrayIndexOutOfBoundsException e) {
        errorCode = ErrorCode.MISSING_STRING;
        throw new ArgsException();
    }
}
```

<div dir="rtl">

من نزدیک بودم که بتوانم سه نقشه قدیمی را حذف کنم. اولین کاری که باید می‌کردم این بود که تابع `getBoolean` را از این حالت تغییر دهم:
</div>

```java
public boolean getBoolean(char arg) {
    Args.ArgumentMarshaler am = booleanArgs.get(arg);
    return am != null && (Boolean) am.get();
}
```

<div dir="rtl">
به این : 
</div>

```java
public boolean getBoolean(char arg) {
    Args.ArgumentMarshaler am = marshalers.get(arg);
    boolean b = false;
    try {
        b = am != null && (Boolean) am.get();
    } catch (ClassCastException e) {
        b = false;
    }
    return b;
}
```

<div dir="rtl">

این تغییر آخر ممکن است شگفت‌انگیز به نظر برسد. چرا ناگهان تصمیم گرفتم که با `ClassCastException` برخورد کنم؟ دلیل این است که من یک مجموعه از تست‌های واحد و یک مجموعه جداگانه از تست‌های پذیرش که در `FitNesse` نوشته شده‌اند، دارم. مشخص شد که تست‌های `FitNesse` تضمین می‌کنند که اگر از `getBoolean` برای یک آرگومان غیر بولی استفاده کنید، مقدار `false` دریافت خواهید کرد، اما تست‌های واحد این کار را انجام نمی‌دهند. تا به این نقطه، من تنها تست‌های واحد را اجرا کرده بودم.

این تغییر آخر به من این امکان را داد که استفاده دیگری از نقشه بولی را حذف کنم:
</div>

```java
private void parseBooleanSchemaElement(char elementId) {
    ArgumentMarshaler m = new BooleanArgumentMarshaler();
    //booleanArgs.put(elementId, m);
    marshalers.put(elementId, m);
}
```

<div dir="rtl">

و حالا می‌توانیم نقشه‌ی `Boolean` را حذف کنیم.
</div>

```java
public class Args {
    ...
    //private Map<Character, ArgumentMarshaler> booleanArgs =
        //new HashMap<Character, ArgumentMarshaler>();
    private Map<Character, ArgumentMarshaler> stringArgs =
        new HashMap<Character, ArgumentMarshaler>();
    private Map<Character, ArgumentMarshaler> intArgs =
        new HashMap<Character, ArgumentMarshaler>();
    private Map<Character, ArgumentMarshaler> marshalers =
        new HashMap<Character, ArgumentMarshaler>();
    ...
```

<div dir="rtl">

سپس آرگومان‌های رشته‌ای و عدد صحیح را به همان شیوه منتقل کردم و کمی پاک‌سازی در بخش `boolean` ها انجام دادم.
</div>

```java
private void parseBooleanSchemaElement(char elementId) {
    marshalers.put(elementId, new BooleanArgumentMarshaler());
}
private void parseIntegerSchemaElement(char elementId) {
    marshalers.put(elementId, new IntegerArgumentMarshaler());
}
private void parseStringSchemaElement(char elementId) {
    marshalers.put(elementId, new StringArgumentMarshaler());
}
... 
public String getString(char arg) {
    Args.ArgumentMarshaler am = marshalers.get(arg);
    try {
        return am == null ? "" : (String) am.get();
    } catch (ClassCastException e) {
        return "";
    }
}
public int getInt(char arg) {
    Args.ArgumentMarshaler am = marshalers.get(arg);
    try {
        return am == null ? 0 : (Integer) am.get();
    } catch (Exception e) {
        return 0;
    }
}
...
public class Args {
    ... 
    //private Map<Character, ArgumentMarshaler> stringArgs =
        //new HashMap<Character, ArgumentMarshaler>();
    //private Map<Character, ArgumentMarshaler> intArgs =
        //new HashMap<Character, ArgumentMarshaler>();
    private Map<Character, ArgumentMarshaler> marshalers =
        new HashMap<Character, ArgumentMarshaler>();
    ...
}
```

<div dir="rtl">

سپس سه متد `parse` را درون‌خطی (inline) کردم چون دیگر کار خاصی انجام نمی‌دادند.
</div>

```java
private void parseSchemaElement(String element) throws ParseException {
    char elementId = element.charAt(0);
    String elementTail = element.substring(1);
    validateSchemaElementId(elementId);
    if (isBooleanSchemaElement(elementTail))
        marshalers.put(elementId, new BooleanArgumentMarshaler());
    else if (isStringSchemaElement(elementTail))
        marshalers.put(elementId, new StringArgumentMarshaler());
    else if (isIntegerSchemaElement(elementTail)) {
        marshalers.put(elementId, new IntegerArgumentMarshaler());
    } else {
        throw new ParseException(
            String.format(
                "Argument: %c has invalid format: %s.", elementId, elementTail),
            0);
    }
}
```

<div dir="rtl">

خُب، حالا بیایید دوباره به تصویر کلی نگاه کنیم. Listing 14-12 نسخه‌ی فعلی کلاس `Args` را نشان می‌دهد.
</div>

Listing 14-12 - Args.java (After first refactoring)

```java
package com.objectmentor.utilities.getopts;
import java.text.ParseException;
import java.util.*;
public class Args {
    private String schema;
    private String[] args;
    private boolean valid = true;
    private Set<Character> unexpectedArguments = new TreeSet<Character>();
    private Map<Character, ArgumentMarshaler> marshalers =
        new HashMap<Character, ArgumentMarshaler>();
    private Set<Character> argsFound = new HashSet<Character>();
    private int currentArgument;
    private char errorArgumentId = '\0';
    private String errorParameter = "TILT";
    private ErrorCode errorCode = ErrorCode.OK;
    private enum ErrorCode {
        OK,
        MISSING_STRING,
        MISSING_INTEGER,
        INVALID_INTEGER,
        UNEXPECTED_ARGUMENT
    }
    public Args(String schema, String[] args) throws ParseException {
        this.schema = schema;
        this.args = args;
        valid = parse();
    }
    private boolean parse() throws ParseException {
        if (schema.length() == 0 && args.length == 0)
            return true;
        parseSchema();
        try {
            parseArguments();
        } catch (ArgsException e) {
        }
        return valid;
    }
    private boolean parseSchema() throws ParseException {
        for (String element : schema.split(",")) {
            if (element.length() > 0) {
                String trimmedElement = element.trim();
                parseSchemaElement(trimmedElement);
            }
        }
        return true;
    }
    private void parseSchemaElement(String element) throws ParseException {
        char elementId = element.charAt(0);
        String elementTail = element.substring(1);
        validateSchemaElementId(elementId);
        if (isBooleanSchemaElement(elementTail))
            marshalers.put(elementId, new BooleanArgumentMarshaler());
        else if (isStringSchemaElement(elementTail))
            marshalers.put(elementId, new StringArgumentMarshaler());
        else if (isIntegerSchemaElement(elementTail)) {
            marshalers.put(elementId, new IntegerArgumentMarshaler());
        } else {
            throw new ParseException(
                String.format("Argument: %c has invalid format: %s.", elementId,
                    elementTail),
                0);
        }
    }
    private void validateSchemaElementId(char elementId) throws ParseException {
        if (!Character.isLetter(elementId)) {
            throw new ParseException(
                "Bad character:" + elementId + "in Args format: " + schema, 0);
        }
    }
    private boolean isStringSchemaElement(String elementTail) {
        return elementTail.equals("*");
    }
    private boolean isBooleanSchemaElement(String elementTail) {
        return elementTail.length() == 0;
    }
    private boolean isIntegerSchemaElement(String elementTail) {
        return elementTail.equals("#");
    }
    private boolean parseArguments() throws ArgsException {
        for (currentArgument = 0; currentArgument < args.length;
             currentArgument++) {
            String arg = args[currentArgument];
            parseArgument(arg);
        }
        return true;
    }
    private void parseArgument(String arg) throws ArgsException {
        if (arg.startsWith("-"))
            parseElements(arg);
    }
    private void parseElements(String arg) throws ArgsException {
        for (int i = 1; i < arg.length(); i++) parseElement(arg.charAt(i));
    }
    private void parseElement(char argChar) throws ArgsException {
        if (setArgument(argChar))
            argsFound.add(argChar);
        else {
            unexpectedArguments.add(argChar);
            errorCode = ErrorCode.UNEXPECTED_ARGUMENT;
            valid = false;
        }
    }
    private boolean setArgument(char argChar) throws ArgsException {
        ArgumentMarshaler m = marshalers.get(argChar);
        try {
            if (m instanceof BooleanArgumentMarshaler)
                setBooleanArg(m);
            else if (m instanceof StringArgumentMarshaler)
                setStringArg(m);
            else if (m instanceof IntegerArgumentMarshaler)
                setIntArg(m);
            else
                return false;
        } catch (ArgsException e) {
            valid = false;
            errorArgumentId = argChar;
            throw e;
        }
        return true;
    }
    private void setIntArg(ArgumentMarshaler m) throws ArgsException {
        currentArgument++;
        String parameter = null;
        try {
            parameter = args[currentArgument];
            m.set(parameter);
        } catch (ArrayIndexOutOfBoundsException e) {
            errorCode = ErrorCode.MISSING_INTEGER;
            throw new ArgsException();
        } catch (ArgsException e) {
            errorParameter = parameter;
            errorCode = ErrorCode.INVALID_INTEGER;
            throw e;
        }
    }
    private void setStringArg(ArgumentMarshaler m) throws ArgsException {
        currentArgument++;
        try {
            m.set(args[currentArgument]);
        } catch (ArrayIndexOutOfBoundsException e) {
            errorCode = ErrorCode.MISSING_STRING;
            throw new ArgsException();
        }
    }
    private void setBooleanArg(ArgumentMarshaler m) {
        try {
            m.set("true");
        } catch (ArgsException e) {
        }
    }
    public int cardinality() {
        return argsFound.size();
    }
    public String usage() {
        if (schema.length() > 0)
            return "-[" + schema + "]";
        else
            return "";
    }
    public String errorMessage() throws Exception {
        switch (errorCode) {
            case OK:
                throw new Exception("TILT: Should not get here.");
            case UNEXPECTED_ARGUMENT:
                return unexpectedArgumentMessage();
            case MISSING_STRING:
                return String.format("Could not find string parameter for -%c.",
                    errorArgumentId);
            case INVALID_INTEGER:
                return String.format(
                    "Argument -%c expects an integer but was '%s'.",
                    errorArgumentId, errorParameter);
            case MISSING_INTEGER:
                return String.format(
                    "Could not find integer parameter for -%c.",
                    errorArgumentId);
        }
        return "";
    }
    private String unexpectedArgumentMessage() {
        StringBuffer message = new StringBuffer("Argument(s) -");
        for (char c : unexpectedArguments) {
            message.append(c);
        }
        message.append(" unexpected.");
        return message.toString();
    }
    public boolean getBoolean(char arg) {
        Args.ArgumentMarshaler am = marshalers.get(arg);
        boolean b = false;
        try {
            b = am != null && (Boolean) am.get();
        } catch (ClassCastException e) {
            b = false;
        }
        return b;
    }
    public String getString(char arg) {
        Args.ArgumentMarshaler am = marshalers.get(arg);
        try {
            return am == null ? "" : (String) am.get();
        } catch (ClassCastException e) {
            return "";
        }
    }
    public int getInt(char arg) {
        Args.ArgumentMarshaler am = marshalers.get(arg);
        try {
            return am == null ? 0 : (Integer) am.get();
        } catch (Exception e) {
            return 0;
        }
    }
    public boolean has(char arg) {
        return argsFound.contains(arg);
    }
    public boolean isValid() {
        return valid;
    }
    private class ArgsException extends Exception {}
    private abstract class ArgumentMarshaler {
        public abstract void set(String s) throws ArgsException;
        public abstract Object get();
    }
    private class BooleanArgumentMarshaler extends ArgumentMarshaler {
        private boolean booleanValue = false;
        public void set(String s) {
            booleanValue = true;
        }
        public Object get() {
            return booleanValue;
        }
    }
    private class StringArgumentMarshaler extends ArgumentMarshaler {
        private String stringValue = "";
        public void set(String s) {
            stringValue = s;
        }
        public Object get() {
            return stringValue;
        }
    }
    private class IntegerArgumentMarshaler extends ArgumentMarshaler {
        private int intValue = 0;
        public void set(String s) throws ArgsException {
            try {
                intValue = Integer.parseInt(s);
            } catch (NumberFormatException e) {
                throw new ArgsException();
            }
        }
        public Object get() {
            return intValue;
        }
    }
}
```

<div dir="rtl">

بعد از تمام این کارها، نتیجه کمی ناامیدکننده است. ساختار کمی بهتر شده، اما هنوز تمام آن متغیرها در بالای کلاس هستند؛ هنوز یک بررسی نوع وحشتناک در `setArgument` داریم؛ و تمام آن توابع `set` واقعاً زشت هستند. تازه به پردازش خطاها هم اشاره‌ای نکرده‌ایم. هنوز کار زیادی پیش رو داریم.

واقعاً دوست دارم از شر آن بررسی نوع در `setArgument` خلاص شوم `[G23]`. چیزی که می‌خواهم، یک فراخوانی واحد به `ArgumentMarshaler.set` است. این یعنی باید توابع `setIntArg،` `setStringArg` و `setBooleanArg` را به کلاس‌های مشتق‌شده‌ی `ArgumentMarshaler` منتقل کنم. اما یک مشکل وجود دارد.

اگر دقیق‌تر به `setIntArg` نگاه کنید، متوجه می‌شوید که از دو متغیر نمونه‌ای استفاده می‌کند: `args` و `currentArg`. برای اینکه بتوانم `setIntArg` را به داخل `IntegerArgumentMarshaler` منتقل کنم، باید هر دو را به‌عنوان آرگومان به تابع پاس بدهم. این کار ناپسند است `[F1]`. ترجیح می‌دهم فقط یک آرگومان پاس بدهم نه دو تا. خوشبختانه، یک راه‌حل ساده وجود دارد. می‌توانیم آرایه‌ی `args` را به یک لیست تبدیل کنیم و یک `Iterator` را به توابع `set` پاس بدهیم.

کاری که انجام دادم طی ده مرحله بود و پس از هر مرحله تمام تست‌ها را پاس می‌کردم. اما من فقط نتیجه را به شما نشان می‌دهم. شما باید بتوانید بیشتر مراحل کوچک را خودتان حدس بزنید.
</div>

```java
public class Args {
    private String schema;
    //private String[] args;
    private boolean valid = true;
    private Set<Character> unexpectedArguments = new TreeSet<Character>();
    private Map<Character, ArgumentMarshaler> marshalers =
        new HashMap<Character, ArgumentMarshaler>();
    private Set<Character> argsFound = new HashSet<Character>();
    private Iterator<String> currentArgument;
    private char errorArgumentId = '\0';
    private String errorParameter = "TILT";
    private ErrorCode errorCode = ErrorCode.OK;
    private List<String> argsList;
    private enum ErrorCode {
        OK,
        MISSING_STRING,
        MISSING_INTEGER,
        INVALID_INTEGER,
        UNEXPECTED_ARGUMENT
    }
    public Args(String schema, String[] args) throws ParseException {
        this.schema = schema;
        argsList = Arrays.asList(args);
        valid = parse();
    }
    private boolean parse() throws ParseException {
        if (schema.length() == 0 && argsList.size() == 0)
            return true;
        parseSchema();
        try {
            parseArguments();
        } catch (ArgsException e) {
        }
        return valid;
    }
    -- -private boolean parseArguments() throws ArgsException {
        for (currentArgument = argsList.iterator();
             currentArgument.hasNext();) {
            String arg = currentArgument.next();
            parseArgument(arg);
        }
        return true;
    }
    -- -private void setIntArg(ArgumentMarshaler m) throws ArgsException {
        String parameter = null;
        try {
            parameter = currentArgument.next();
            m.set(parameter);
        } catch (NoSuchElementException e) {
            errorCode = ErrorCode.MISSING_INTEGER;
            throw new ArgsException();
        } catch (ArgsException e) {
            errorParameter = parameter;
            errorCode = ErrorCode.INVALID_INTEGER;
            throw e;
        }
    }
    private void setStringArg(ArgumentMarshaler m) throws ArgsException {
        try {
            m.set(currentArgument.next());
        } catch (NoSuchElementException e) {
            errorCode = ErrorCode.MISSING_STRING;
            throw new ArgsException();
        }
    }
}
```

<div dir="rtl">

این‌ها تغییرات ساده‌ای بودند که باعث شدند تمام تست‌ها همچنان پاس شوند. حالا می‌توانیم شروع به انتقال توابع `set` به کلاس‌های مشتق‌شده‌ی مناسب کنیم. ابتدا باید تغییر زیر را در تابع `setArgument` اعمال کنم:
</div>

```java
private boolean setArgument(char argChar) throws ArgsException {
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m == null)
        return false;
    try {
        if (m instanceof BooleanArgumentMarshaler)
            setBooleanArg(m);
        else if (m instanceof StringArgumentMarshaler)
            setStringArg(m);
        else if (m instanceof IntegerArgumentMarshaler)
            setIntArg(m);
        //else
            //return false;
    } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar;
        throw e;
    }
    return true;
}
```

<div dir="rtl">

این تغییر مهم است چون می‌خواهیم زنجیره `if-else` را به‌طور کامل حذف کنیم. بنابراین، لازم بود شرط خطا را از آن خارج کنیم.
حالا می‌توانیم شروع به انتقال توابع set کنیم. تابع `setBooleanArg` ساده است، پس ابتدا آن را آماده می‌کنیم. هدف ما این است که تابع `setBooleanArg` را به‌گونه‌ای تغییر دهیم که فقط عملیات را به کلاس `BooleanArgumentMarshaler` واگذار کند.
</div>

```java
private boolean setArgument(char argChar) throws ArgsException {
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m == null)
        return false;
    try {
        if (m instanceof BooleanArgumentMarshaler)
            setBooleanArg(m, currentArgument);
        else if (m instanceof StringArgumentMarshaler)
            setStringArg(m);
        else if (m instanceof IntegerArgumentMarshaler)
            setIntArg(m);
    } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar;
        throw e;
    }
    return true;
}
---
private void setBooleanArg(ArgumentMarshaler m,
    Iterator<String> currentArgument) throws ArgsException {
    //try {
        m.set("true");
    //    catch (ArgsException e) {
    //    }
    }
```

<div dir="rtl">

مگر همین پردازش استثنا را تازه اضافه نکرده بودیم؟ اضافه کردن چیزهایی برای اینکه بعداً آن‌ها را حذف کنیم، در فرایند بازآرایی (Refactoring) بسیار رایج است. کوچکی گام‌ها و نیاز به حفظ عملکرد صحیح برنامه باعث می‌شود که چیزها را زیاد جابه‌جا کنید. بازآرایی خیلی شبیه حل کردن مکعب روبیک است. برای رسیدن به یک هدف بزرگ، گام‌های کوچکی زیادی لازم است. هر گام، امکان اجرای گام بعدی را فراهم می‌کند.

چرا آن `iterator` را ارسال کردیم وقتی که `setBooleanArg` قطعاً به آن نیازی ندارد؟ چون `setIntArg` و `setStringArg` نیاز خواهند داشت! و از آنجایی که می‌خواهم هر سه تابع را از طریق یک متد انتزاعی در `ArgumentMarshaller` پیاده‌سازی کنم، باید آن را به `setBooleanArg` نیز ارسال کنم.

پس حالا تابع `setBooleanArg` بی‌استفاده شده است. اگر تابعی به نام `set` در کلاس `ArgumentMarshaler` وجود داشت، می‌توانستیم مستقیماً آن را فراخوانی کنیم. بنابراین، زمان آن رسیده که این تابع را اضافه کنیم! اولین گام این است که یک متد انتزاعی (`abstract`) جدید به کلاس `ArgumentMarshaler` اضافه کنیم.
</div>

```java
private abstract class ArgumentMarshaler {
    public abstract void set(Iterator<String> currentArgument)
        throws ArgsException;
    public abstract void set(String s) throws ArgsException;
    public abstract Object get();
}
```

<div dir="rtl">

مطمئناً این تغییر باعث شکست در همه مشتقات (`derivatives`) می‌شود. پس بیایید متد جدید را در هر کدام پیاده‌سازی کنیم.
</div>

```java
private class BooleanArgumentMarshaler extends ArgumentMarshaler {
    private boolean booleanValue = false;
    public void set(Iterator<String> currentArgument) throws ArgsException {
        // booleanValue = true;
    }
    public void set(String s) {
        booleanValue = true;
    }
    public Object get() {
        return booleanValue;
    }
}
private class StringArgumentMarshaler extends ArgumentMarshaler {
    private String stringValue = "";
    public void set(Iterator<String> currentArgument) throws ArgsException {}
    public void set(String s) {
        stringValue = s;
    }
    public Object get() {
        return stringValue;
    }
}
private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    private int intValue = 0;
    public void set(Iterator<String> currentArgument) throws ArgsException {}
    public void set(String s) throws ArgsException {
        try {
            intValue = Integer.parseInt(s);
        } catch (NumberFormatException e) {
            throw new ArgsException();
        }
    }
    public Object get() {
        return intValue;
    }
}
```

<div dir="rtl">

و حالا می‌توانیم `setBooleanArg` را حذف کنیم!
</div>

```java
private boolean setArgument(char argChar) throws ArgsException {
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m == null)
        return false;
    try {
        if (m instanceof BooleanArgumentMarshaler)
            m.set(currentArgument);
        else if (m instanceof StringArgumentMarshaler)
            setStringArg(m);
        else if (m instanceof IntegerArgumentMarshaler)
            setIntArg(m);
    } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar;
        throw e;
    }
    return true;
}
```

<div dir="rtl">

تمام تست‌ها پاس شدند و تابع `set` به درستی به `BooleanArgumentMarshaler` منتقل شد! حالا می‌توانیم همین کار را برای رشته‌ها و اعداد صحیح انجام دهیم.
</div>

```java
private boolean setArgument(char argChar) throws ArgsException {
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m == null)
        return false;
    try {
        if (m instanceof BooleanArgumentMarshaler)
            m.set(currentArgument);
        else if (m instanceof StringArgumentMarshaler)
            m.set(currentArgument);
        else if (m instanceof IntegerArgumentMarshaler)
            m.set(currentArgument);
    } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar;
        throw e;
    }
    return true;
}
---
private class StringArgumentMarshaler extends ArgumentMarshaler {
    private String stringValue = "";
    public void set(Iterator<String> currentArgument) throws ArgsException {
        try {
            stringValue = currentArgument.next();
        } catch (NoSuchElementException e) {
            errorCode = ErrorCode.MISSING_STRING;
            throw new ArgsException();
        }
    }
    public void set(String s) {}
    public Object get() {
        return stringValue;
    }
}
private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    private int intValue = 0;
    public void set(Iterator<String> currentArgument) throws ArgsException {
        String parameter = null;
        try {
            parameter = currentArgument.next();
            set(parameter);
        } catch (NoSuchElementException e) {
            errorCode = ErrorCode.MISSING_INTEGER;
            throw new ArgsException();
        } catch (ArgsException e) {
            errorParameter = parameter;
            errorCode = ErrorCode.INVALID_INTEGER;
            throw e;
        }
    }
    public void set(String s) throws ArgsException {
        try {
            intValue = Integer.parseInt(s);
        } catch (NumberFormatException e) {
            throw new ArgsException();
        }
    }
    public Object get() {
        return intValue;
    }
}
```

<div dir="rtl">
و حالا ضربه نهایی: می‌توانیم نوع‌سازی را حذف کنیم! تبریک!
</div>

```java
private boolean setArgument(char argChar) throws ArgsException {
    ArgumentMarshaler m = marshalers.get(argChar);
    if (m == null)
        return false;
    try {
        m.set(currentArgument);
        return true;
    } catch (ArgsException e) {
        valid = false;
        errorArgumentId = argChar;
        throw e;
    }
}
```

<div dir="rtl">

حالا می‌توانیم برخی از توابع اضافی در `IntegerArgumentMarshaler` را حذف کرده و کمی آن را تمیز کنیم.
</div>

```java
private class IntegerArgumentMarshaler extends ArgumentMarshaler {
    private int intValue = 0 public void set(Iterator<String> currentArgument)
        throws ArgsException {
        String parameter = null;
        try {
            parameter = currentArgument.next();
            intValue = Integer.parseInt(parameter);
        } catch (NoSuchElementException e) {
            errorCode = ErrorCode.MISSING_INTEGER;
            throw new ArgsException();
        } catch (NumberFormatException e) {
            errorParameter = parameter;
            errorCode = ErrorCode.INVALID_INTEGER;
            throw new ArgsException();
        }
    }
    public Object get() {
        return intValue;
    }
}
```

<div dir="rtl">

ما همچنین می‌توانیم `ArgumentMarshaler` را به یک اینترفیس تبدیل کنیم.
</div>

```java
private interface ArgumentMarshaler {
    void set(Iterator<String> currentArgument) throws ArgsException;
    Object get();
}
```

<div dir="rtl">

حال ببینیم اضافه کردن یک نوع آرگومان جدید به ساختارمان چقدر آسان است. این کار باید تغییرات بسیار کمی را نیاز داشته باشد و این تغییرات باید مجزا باشند. ابتدا با افزودن یک تست جدید شروع می‌کنیم تا بررسی کنیم که آرگومان نوع `double` به درستی کار می‌کند.

</div>

```java
public void testSimpleDoublePresent() throws Exception {
    Args args = new Args("x##", new String[] {"-x","42.3"});
    assertTrue(args.isValid());
    assertEquals(1, args.cardinality());
    assertTrue(args.has('x'));
    assertEquals(42.3, args.getDouble('x'), .001);
}
```

<div dir="rtl">

حالا کد تجزیه اسکیمای خود را تمیز می‌کنیم و تشخیص `##` برای نوع آرگومان `double` را اضافه می‌کنیم.
</div>

```java
private void parseSchemaElement(String element) throws ParseException {
    char elementId = element.charAt(0);
    String elementTail = element.substring(1);
    validateSchemaElementId(elementId);
    if (elementTail.length() == 0)
        marshalers.put(elementId, new BooleanArgumentMarshaler());
    else if (elementTail.equals("*"))
        marshalers.put(elementId, new StringArgumentMarshaler());
    else if (elementTail.equals("#"))
        marshalers.put(elementId, new IntegerArgumentMarshaler());
    else if (elementTail.equals("##"))
        marshalers.put(elementId, new DoubleArgumentMarshaler());
    else
        throw new ParseException(
            String.format(
                "Argument: %c has invalid format: %s.", elementId, elementTail),
            0);
}
```

<div dir="rtl">

سپس کلاس `DoubleArgumentMarshaler` را مینویسیم.
</div>

```java
private class DoubleArgumentMarshaler implements ArgumentMarshaler {
    private double doubleValue = 0;
    public void set(Iterator<String> currentArgument) throws ArgsException {
        String parameter = null;
        try {
            parameter = currentArgument.next();
            doubleValue = Double.parseDouble(parameter);
        } catch (NoSuchElementException e) {
            errorCode = ErrorCode.MISSING_DOUBLE;
            throw new ArgsException();
        } catch (NumberFormatException e) {
            errorParameter = parameter;
            errorCode = ErrorCode.INVALID_DOUBLE;
            throw new ArgsException();
        }
    }
    public Object get() {
        return doubleValue;
    }
}
```

<div dir="rtl">

این ما را مجبور می‌کند تا یک `ErrorCode` جدید اضافه کنیم.
</div>

```java
private enum ErrorCode {
    OK,
    MISSING_STRING,
    MISSING_INTEGER,
    INVALID_INTEGER,
    UNEXPECTED_ARGUMENT,

    MISSING_DOUBLE,
    INVALID_DOUBLE
}
```

<div dir="rtl">

و ما نیاز به متد `getDouble` داریم.
</div>

```java
public double getDouble(char arg) {
    Args.ArgumentMarshaler am = marshalers.get(arg);
    try {
        return am == null ? 0 : (Double) am.get();
    } catch (Exception e) {
        return 0.0;
    }
}
```

<div dir="rtl">

و همه تست‌ها موفقیت‌آمیز بودند! این کار نسبتاً بی‌دردسر بود. حالا بیایید مطمئن شویم که پردازش خطا به درستی کار می‌کند. تست بعدی بررسی می‌کند که آیا در صورتی که یک رشته غیرقابل تجزیه به یک آرگومان `##` داده شود، خطا اعلام می‌شود یا خیر.

</div>

```java
public void testInvalidDouble() throws Exception {
    Args args = new Args("x##", new String[] {"-x", "Forty two"});
    assertFalse(args.isValid());
    assertEquals(0, args.cardinality());
    assertFalse(args.has('x'));
    assertEquals(0, args.getInt('x'));
    assertEquals("Argument -x expects a double but was 'Forty two'.",
        args.errorMessage());
}
-- -public String errorMessage() throws Exception {
    switch (errorCode) {
        case OK:
            throw new Exception("TILT: Should not get here.");
        case UNEXPECTED_ARGUMENT:
            return unexpectedArgumentMessage();
        case MISSING_STRING:
            return String.format(
                "Could not find string parameter for -%c.", errorArgumentId);
        case INVALID_INTEGER:
            return String.format(
                "Argument -%c expects an integer but was '%s'.",
                errorArgumentId, errorParameter);
        case MISSING_INTEGER:
            return String.format(
                "Could not find integer parameter for -%c.", errorArgumentId);
        case INVALID_DOUBLE:
            return String.format("Argument -%c expects a double but was '%s'.",
                errorArgumentId, errorParameter);
        case MISSING_DOUBLE:
            return String.format(
                "Could not find double parameter for -%c.", errorArgumentId);
    }
    return "";
}
```

<div dir="rtl">

و تست‌ها موفقیت‌آمیز بودند. تست بعدی بررسی می‌کند که آیا آرگومان `double` گم‌شده به‌درستی تشخیص داده می‌شود
</div>

```java
public void testMissingDouble() throws Exception {
    Args args = new Args("x##", new String[] {"-x"});
    assertFalse(args.isValid());
    assertEquals(0, args.cardinality());
    assertFalse(args.has('x'));
    assertEquals(0.0, args.getDouble('x'), 0.01);
    assertEquals(
        "Could not find double parameter for -x.", args.errorMessage());
}
```

<div dir="rtl">

این تست همان‌طور که انتظار داشتیم موفقیت‌آمیز بود. ما آن را صرفاً برای تکمیل کار نوشتیم.
کد مربوط به استثنا کمی نامرتب است و واقعاً در کلاس `Args` جایگاهی ندارد. همچنین ما `ParseException` را پرتاب می‌کنیم، در حالی که این استثنا به ما تعلق ندارد. بنابراین، بیایید تمام استثناها را در یک کلاس واحد به نام `ArgsException` ترکیب کنیم و آن را به یک ماژول اختصاصی منتقل کنیم.

</div>

```java
public class ArgsException extends Exception {
    private char errorArgumentId = '\0';
    private String errorParameter = "TILT";
    private ErrorCode errorCode = ErrorCode.OK;
    public ArgsException() {}
    public ArgsException(String message) {
        super(message);
    }
    public enum ErrorCode {
        OK,
        MISSING_STRING,
        MISSING_INTEGER,
        INVALID_INTEGER,
        UNEXPECTED_ARGUMENT,
        MISSING_DOUBLE,
        INVALID_DOUBLE
    }
}
-- -public class Args {
    ... private char errorArgumentId = '\0';
    private String errorParameter = "TILT";
    private ArgsException.ErrorCode errorCode = ArgsException.ErrorCode.OK;
    private List<String> argsList;
    public Args(String schema, String[] args) throws ArgsException {
        this.schema = schema;
        argsList = Arrays.asList(args);
        valid = parse();
    }
    private boolean parse() throws ArgsException {
        if (schema.length() == 0 && argsList.size() == 0)
            return true;
        parseSchema();
        try {
            parseArguments();
        } catch (ArgsException e) {
        }
        return valid;
    }
    private boolean parseSchema() throws ArgsException {
        ...
    }
    private void parseSchemaElement(String element) throws ArgsException {
        ... else throw new ArgsException(String.format(
            "Argument: %c has invalid format: %s.", elementId, elementTail));
    }
    private void validateSchemaElementId(char elementId) throws ArgsException {
        if (!Character.isLetter(elementId)) {
            throw new ArgsException(
                "Bad character:" + elementId + "in Args format: " + schema);
        }
    }
    ... private void parseElement(char argChar) throws ArgsException {
        if (setArgument(argChar))
            argsFound.add(argChar);
        else {
            unexpectedArguments.add(argChar);
            errorCode = ArgsException.ErrorCode.UNEXPECTED_ARGUMENT;
            valid = false;
        }
    }
    ... private class StringArgumentMarshaler implements ArgumentMarshaler {
        private String stringValue = "";
        public void set(Iterator<String> currentArgument) throws ArgsException {
            try {
                stringValue = currentArgument.next();
            } catch (NoSuchElementException e) {
                errorCode = ArgsException.ErrorCode.MISSING_STRING;
                throw new ArgsException();
            }
        }
        public Object get() {
            return stringValue;
        }
    }
    private class IntegerArgumentMarshaler implements ArgumentMarshaler {
        private int intValue = 0;
        public void set(Iterator<String> currentArgument) throws ArgsException {
            String parameter = null;
            try {
                parameter = currentArgument.next();
                intValue = Integer.parseInt(parameter);
            } catch (NoSuchElementException e) {
                errorCode = ArgsException.ErrorCode.MISSING_INTEGER;
                throw new ArgsException();
            } catch (NumberFormatException e) {
                errorParameter = parameter;
                errorCode = ArgsException.ErrorCode.INVALID_INTEGER;
                throw new ArgsException();
            }
        }
        public Object get() {
            return intValue;
        }
    }
    private class DoubleArgumentMarshaler implements ArgumentMarshaler {
        private double doubleValue = 0;
        public void set(Iterator<String> currentArgument) throws ArgsException {
            String parameter = null;
            try {
                parameter = currentArgument.next();
                doubleValue = Double.parseDouble(parameter);
            } catch (NoSuchElementException e) {
                errorCode = ArgsException.ErrorCode.MISSING_DOUBLE;
                throw new ArgsException();
            } catch (NumberFormatException e) {
                errorParameter = parameter;
                errorCode = ArgsException.ErrorCode.INVALID_DOUBLE;
                throw new ArgsException();
            }
        }
        public Object get() {
            return doubleValue;
        }
    }
}
```

<div dir="rtl">

این عالی است! اکنون تنها استثنایی که توسط `Args` پرتاب می‌شود، `ArgsException` است. انتقال `ArgsException` به یک ماژول اختصاصی به ما این امکان را می‌دهد که بسیاری از کدهای پراکنده مربوط به مدیریت خطا را در آن ماژول جای دهیم و `Args` را از این موارد جدا کنیم.
این کار نه‌تنها مکانی طبیعی و مناسب برای نگهداری این کدها فراهم می‌کند، بلکه باعث مرتب‌تر شدن ماژول `Args` در آینده می‌شود.
اکنون ما به‌طور کامل کدهای مربوط به مدیریت خطا و استثنا را از ماژول `Args` تفکیک کرده‌ایم. (به فهرست 14-13 تا 14-16 مراجعه کنید.) این تغییرات طی حدود 30 گام کوچک انجام شده‌اند، در حالی که تمامی تست‌ها در هر مرحله موفقیت‌آمیز باقی مانده‌اند.

</div>

Listing 14-13 - ArgsTest.java

```java
package com.objectmentor.utilities.args;
import junit.framework.TestCase;
public class ArgsTest extends TestCase {
    public void testCreateWithNoSchemaOrArguments() throws Exception {
        Args args = new Args("", new String[0]);
        assertEquals(0, args.cardinality());
    }
    public void testWithNoSchemaButWithOneArgument() throws Exception {
        try {
            new Args("", new String[] {"-x"});
            fail();
        } catch (ArgsException e) {
            assertEquals(
                ArgsException.ErrorCode.UNEXPECTED_ARGUMENT, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
        }
    }
    public void testWithNoSchemaButWithMultipleArguments() throws Exception {
        try {
            new Args("", new String[] {"-x", "-y"});
            fail();
        } catch (ArgsException e) {
            assertEquals(
                ArgsException.ErrorCode.UNEXPECTED_ARGUMENT, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
        }
    }
    public void testNonLetterSchema() throws Exception {
        try {
            new Args("*", new String[] {});
            fail("Args constructor should have thrown exception");
        } catch (ArgsException e) {
            assertEquals(ArgsException.ErrorCode.INVALID_ARGUMENT_NAME,
                e.getErrorCode());
            assertEquals('*', e.getErrorArgumentId());
        }
    }
    public void testInvalidArgumentFormat() throws Exception {
        try {
            new Args("f~", new String[] {});
            fail("Args constructor should have throws exception");
        } catch (ArgsException e) {
            assertEquals(
                ArgsException.ErrorCode.INVALID_FORMAT, e.getErrorCode());
            assertEquals('f', e.getErrorArgumentId());
        }
    }
    public void testSimpleBooleanPresent() throws Exception {
        Args args = new Args("x", new String[] {"-x"});
        assertEquals(1, args.cardinality());
        assertEquals(true, args.getBoolean('x'));
    }
    public void testSimpleStringPresent() throws Exception {
        Args args = new Args("x*", new String[] {"-x", "param"});
        assertEquals(1, args.cardinality());
        assertTrue(args.has('x'));
        assertEquals("param", args.getString('x'));
    }
    public void testMissingStringArgument() throws Exception {
        try {
            new Args("x*", new String[] {"-x"});
            fail();
        } catch (ArgsException e) {
            assertEquals(
                ArgsException.ErrorCode.MISSING_STRING, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
        }
    }
    public void testSpacesInFormat() throws Exception {
        Args args = new Args("x, y", new String[] {"-xy"});
        assertEquals(2, args.cardinality());
        assertTrue(args.has('x'));
        assertTrue(args.has('y'));
    }
    public void testSimpleIntPresent() throws Exception {
        Args args = new Args("x#", new String[] {"-x", "42"});
        assertEquals(1, args.cardinality());
        assertTrue(args.has('x'));
        assertEquals(42, args.getInt('x'));
    }
    public void testInvalidInteger() throws Exception {
        try {
            new Args("x#", new String[] {"-x", "Forty two"});
            fail();
        } catch (ArgsException e) {
            assertEquals(
                ArgsException.ErrorCode.INVALID_INTEGER, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
            assertEquals("Forty two", e.getErrorParameter());
        }
    }
    public void testMissingInteger() throws Exception {
        try {
            new Args("x#", new String[] {"-x"});
            fail();
        } catch (ArgsException e) {
            assertEquals(
                ArgsException.ErrorCode.MISSING_INTEGER, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
        }
    }
    public void testSimpleDoublePresent() throws Exception {
        Args args = new Args("x##", new String[] {"-x", "42.3"});
        assertEquals(1, args.cardinality());
        assertTrue(args.has('x'));
        assertEquals(42.3, args.getDouble('x'), .001);
    }
    public void testInvalidDouble() throws Exception {
        try {
            new Args("x##", new String[] {"-x", "Forty two"});
            fail();
        } catch (ArgsException e) {
            assertEquals(
                ArgsException.ErrorCode.INVALID_DOUBLE, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
            assertEquals("Forty two", e.getErrorParameter());
        }
    }
    public void testMissingDouble() throws Exception {
        try {
            new Args("x##", new String[] {"-x"});
            fail();
        } catch (ArgsException e) {
            assertEquals(
                ArgsException.ErrorCode.MISSING_DOUBLE, e.getErrorCode());
            assertEquals('x', e.getErrorArgumentId());
        }
    }
}
```

Listing 14-14 - ArgsExceptionTest.java

```java

public class ArgsExceptionTest extends TestCase {
    public void testUnexpectedMessage() throws Exception {
        ArgsException e = new ArgsException(
            ArgsException.ErrorCode.UNEXPECTED_ARGUMENT, 'x', null);
        assertEquals("Argument -x unexpected.", e.errorMessage());
    }
    public void testMissingStringMessage() throws Exception {
        ArgsException e = new ArgsException(
            ArgsException.ErrorCode.MISSING_STRING, 'x', null);
        assertEquals(
            "Could not find string parameter for -x.", e.errorMessage());
    }
    public void testInvalidIntegerMessage() throws Exception {
        ArgsException e = new ArgsException(
            ArgsException.ErrorCode.INVALID_INTEGER, 'x', "Forty two");
        assertEquals("Argument -x expects an integer but was 'Forty two'.",
            e.errorMessage());
    }
    public void testMissingIntegerMessage() throws Exception {
        ArgsException e = new ArgsException(
            ArgsException.ErrorCode.MISSING_INTEGER, 'x', null);
        assertEquals(
            "Could not find integer parameter for -x.", e.errorMessage());
    }
    public void testInvalidDoubleMessage() throws Exception {
        ArgsException e = new ArgsException(
            ArgsException.ErrorCode.INVALID_DOUBLE, 'x', "Forty two");
        assertEquals("Argument -x expects a double but was 'Forty two'.",
            e.errorMessage());
    }
    public void testMissingDoubleMessage() throws Exception {
        ArgsException e = new ArgsException(
            ArgsException.ErrorCode.MISSING_DOUBLE, 'x', null);
        assertEquals(
            "Could not find double parameter for -x.", e.errorMessage());
    }
}

```

Listing 14-15 - ArgsException.java

```java
public class ArgsException extends Exception {
    private char errorArgumentId = '\0';
    private String errorParameter = "TILT";
    private ErrorCode errorCode = ErrorCode.OK;
    public ArgsException() {}
    public ArgsException(String message) {
        super(message);
    }
    public ArgsException(ErrorCode errorCode) {
        this.errorCode = errorCode;
    }
    public ArgsException(ErrorCode errorCode, String errorParameter) {
        this.errorCode = errorCode;
        this.errorParameter = errorParameter;
    }
    public ArgsException(
        ErrorCode errorCode, char errorArgumentId, String errorParameter) {
        this.errorCode = errorCode;
        this.errorParameter = errorParameter;
        this.errorArgumentId = errorArgumentId;
    }
    public char getErrorArgumentId() {
        return errorArgumentId;
    }
    public void setErrorArgumentId(char errorArgumentId) {
        this.errorArgumentId = errorArgumentId;
    }
    public String getErrorParameter() {
        return errorParameter;
    }
    public void setErrorParameter(String errorParameter) {
        this.errorParameter = errorParameter;
    }
    public ErrorCode getErrorCode() {
        return errorCode;
    }
    public void setErrorCode(ErrorCode errorCode) {
        this.errorCode = errorCode;
    }
    public String errorMessage() throws Exception {
        switch (errorCode) {
            case OK:
                throw new Exception("TILT: Should not get here.");
            case UNEXPECTED_ARGUMENT:
                return String.format(
                    "Argument -%c unexpected.", errorArgumentId);
            case MISSING_STRING:
                return String.format("Could not find string parameter for -%c.",
                    errorArgumentId);
            case INVALID_INTEGER:
                return String.format(
                    "Argument -%c expects an integer but was '%s'.",
                    errorArgumentId, errorParameter);
            case MISSING_INTEGER:
                return String.format(
                    "Could not find integer parameter for -%c.",
                    errorArgumentId);
            case INVALID_DOUBLE:
                return String.format(
                    "Argument -%c expects a double but was '%s'.",
                    errorArgumentId, errorParameter);
            case MISSING_DOUBLE:
                return String.format("Could not find double parameter for -%c.",
                    errorArgumentId);
        }
        return "";
    }
    public enum ErrorCode {
        OK,
        INVALID_FORMAT,
        UNEXPECTED_ARGUMENT,
        INVALID_ARGUMENT_NAME,
        MISSING_STRING,
        MISSING_INTEGER,
        INVALID_INTEGER,
        MISSING_DOUBLE,
        INVALID_DOUBLE
    }
}
```

Listing 14-16 - Args.java

```java
public class Args {
    private String schema;
    private Map<Character, ArgumentMarshaler> marshalers =
        new HashMap<Character, ArgumentMarshaler>();
    private Set<Character> argsFound = new HashSet<Character>();
    private Iterator<String> currentArgument;
    private List<String> argsList;
    public Args(String schema, String[] args) throws ArgsException {
        this.schema = schema;
        argsList = Arrays.asList(args);
        parse();
    }
    private void parse() throws ArgsException {
        parseSchema();
        parseArguments();
    }
    private boolean parseSchema() throws ArgsException {
        for (String element : schema.split(",")) {
            if (element.length() > 0) {
                parseSchemaElement(element.trim());
            }
        }
        return true;
    }
    private void parseSchemaElement(String element) throws ArgsException {
        char elementId = element.charAt(0);
        String elementTail = element.substring(1);
        validateSchemaElementId(elementId);
        if (elementTail.length() == 0)
            marshalers.put(elementId, new BooleanArgumentMarshaler());
        else if (elementTail.equals("*"))
            marshalers.put(elementId, new StringArgumentMarshaler());
        else if (elementTail.equals("#"))
            marshalers.put(elementId, new IntegerArgumentMarshaler());
        else if (elementTail.equals("##"))
            marshalers.put(elementId, new DoubleArgumentMarshaler());
        else
            throw new ArgsException(
                ArgsException.ErrorCode.INVALID_FORMAT, elementId, elementTail);
    }
    private void validateSchemaElementId(char elementId) throws ArgsException {
        if (!Character.isLetter(elementId)) {
            throw new ArgsException(
                ArgsException.ErrorCode.INVALID_ARGUMENT_NAME, elementId, null);
        }
    }
    private void parseArguments() throws ArgsException {
        for (currentArgument = argsList.iterator();
             currentArgument.hasNext();) {
            String arg = currentArgument.next();
            parseArgument(arg);
        }
    }
    private void parseArgument(String arg) throws ArgsException {
        if (arg.startsWith("-"))
            parseElements(arg);
    }
    private void parseElements(String arg) throws ArgsException {
        for (int i = 1; i < arg.length(); i++) parseElement(arg.charAt(i));
    }
    private void parseElement(char argChar) throws ArgsException {
        if (setArgument(argChar))
            argsFound.add(argChar);
        else {
            throw new ArgsException(
                ArgsException.ErrorCode.UNEXPECTED_ARGUMENT, argChar, null);
        }
    }
    private boolean setArgument(char argChar) throws ArgsException {
        ArgumentMarshaler m = marshalers.get(argChar);
        if (m == null)
            return false;
        try {
            m.set(currentArgument);
            return true;
        } catch (ArgsException e) {
            e.setErrorArgumentId(argChar);
            throw e;
        }
    }
    public int cardinality() {
        return argsFound.size();
    }
    public String usage() {
        if (schema.length() > 0)
            return "-[" + schema + "]";
        else
            return "";
    }
    public boolean getBoolean(char arg) {
        ArgumentMarshaler am = marshalers.get(arg);
        boolean b = false;
        try {
            b = am != null && (Boolean) am.get();
        } catch (ClassCastException e) {
            b = false;
        }
        return b;
    }
    public String getString(char arg) {
        ArgumentMarshaler am = marshalers.get(arg);
        try {
            return am == null ? "" : (String) am.get();
        } catch (ClassCastException e) {
            return "";
        }
    }
    public int getInt(char arg) {
        ArgumentMarshaler am = marshalers.get(arg);
        try {
            return am == null ? 0 : (Integer) am.get();
        } catch (Exception e) {
            return 0;
        }
    }
    public double getDouble(char arg) {
        ArgumentMarshaler am = marshalers.get(arg);
        try {
            return am == null ? 0 : (Double) am.get();
        } catch (Exception e) {
            return 0.0;
        }
    }
    public boolean has(char arg) {
        return argsFound.contains(arg);
    }
}
```

<div dir="rtl">

کلاس `Args` بیشتر تغییراتش مربوط به حذف بخش‌هایی از کد بود. بخش زیادی از کد فقط از داخل `Args` منتقل شد و به `ArgsException` اضافه گردید. عالی است! همچنین تمام `ArgumentMarshallerها` به فایل‌های جداگانه منتقل شدند. خیلی بهتر!

بخش زیادی از طراحی خوب نرم‌افزار، در واقع در مورد بخش‌بندی است—ایجاد مکان‌های مناسب برای قرار دادن انواع مختلف کد. این تفکیک مسئولیت‌ها باعث می‌شود کد بسیار ساده‌تر قابل فهم و نگهداری باشد.

نکته جالب توجه، متد `errorMessage` در `ArgsException` است. به وضوح، قرار دادن کد فرمت کردن پیام خطا در داخل `Args` نقض اصل `SRP` بود. `Args` باید در مورد پردازش آرگومان‌ها باشد، نه فرمت پیام‌های خطا. اما آیا واقعاً منطقی است که کد فرمت پیام خطا را در داخل `ArgsException` قرار دهیم؟ به طور صریح، این یک مصالحه است. کاربرانی که از پیام‌های خطا که توسط `ArgsException` فراهم شده است راضی نیستند، باید پیام‌های خطای خودشان را بنویسند. اما راحتی در داشتن پیام‌های خطای آماده، بی‌اهمیت نیست.

تا به حال باید واضح شده باشد که ما تقریباً به راه‌حل نهایی که در ابتدای این فصل آورده شده، نزدیک شده‌ایم. من باقی‌مانده تغییرات را به شما می‌سپارم تا به عنوان تمرین انجام دهید.

## نتیجه‌گیری

کافی نیست که کد کار کند. کد کارا اغلب به شدت خراب است. برنامه‌نویس‌هایی که فقط به کارکردن کد اکتفا می‌کنند، به طور غیرحرفه‌ای عمل می‌کنند. آن‌ها ممکن است بترسند که وقت کافی برای بهبود ساختار و طراحی کد خود نداشته باشند، اما من با این نظر مخالفم. هیچ چیز به اندازه کد بد تأثیر مخرب و بلندمدتی بر روی یک پروژه توسعه نمی‌گذارد. برنامه‌نویسی که فقط به کد کارا اکتفا کند، در نهایت به یک کد خراب تبدیل می‌شود که بر پروژه تسلط پیدا می‌کند.

البته که کد بد قابل تمیز کردن است، اما این کار بسیار پرهزینه است. وقتی کد خراب می‌شود، ماژول‌ها به هم می‌پیوندند و وابستگی‌های پنهان و درهم‌تنیده‌ای ایجاد می‌کنند. پیدا کردن و شکستن این وابستگی‌های قدیمی کار زمان‌بر و سختی است. از طرف دیگر، نگهداری کد تمیز نسبتاً راحت است. اگر صبح یک ماژول را خراب کردید، می‌توانید بعدازظهر آن را تمیز کنید. بهتر از آن، اگر پنج دقیقه پیش یک ماژول را خراب کرده‌اید، بسیار آسان است که همین الان آن را تمیز کنید.

بنابراین راه‌حل این است که کد خود را به طور مداوم تمیز و ساده نگه دارید. هرگز اجازه ندهید که فساد کد آغاز شود.
</div>

* [فصل قبل](../13_Concurrency/Concurrency.md)
* [فصل بعد](../15_JUnit_Internals/JUnit_Internals.md)
