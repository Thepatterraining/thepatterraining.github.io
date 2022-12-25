---
title: JAVA-LocalDateTime时间格式化，转换时间戳和源码分析
date: 2022-11-30
tags: ['java','源码','LocalDateTime']
category: java
article: JAVA-LocalDateTime时间格式化，转换时间戳和源码分析
---

# JAVA-LocalDateTime时间格式化，转换时间戳和源码分析

## LocalDateTime

`LocalDateTime`作为java8新加的时间类型，也是后面开发中常用的时间类型。

作为时间类型，最关注的点无非是这几个
- 获取当前时间
- 获取指定时间
- 时间格式化
- 时间转时间戳
- 时间戳转时间
- 时间比较
- 时间加减

这些点通过`LocalDateTime`来操作，都会比较简单

### 获取当前时间

只需要now一下就可以获取到当前的时间，还是很方便的。

```java
LocalDateTime time = LocalDateTime.now();
```

再看一下之前的`Date`类

```java
Date date = new Date();
```

### 获取指定时间

这个有比较多的方式
- 通过原来的`date`和`dateTime`类型来生成
- 通过传年月日时分秒生成

```java
LocalDateTime time = LocalDateTime.of(2022,11,30,6,6,6);
```

原来`Date`类的方式。比较奇怪，他的年份会+1900，所以2022年就得是122，月份也会+1,所以11月就是10.但是这个方法呢后面会被删除，已经被标记为弃用了，使用`Calendar`代替。

```java
Date date = new Date(122,10,30,6,6,6);
```

看一下`Calendar`的使用。这个年份就正常了，是2022，但是月份还是会+1.

```java
Calendar calendar = Calendar.getInstance();
calendar.set(2022,10,30,6,6,6);
```

### 时间格式化

时间格式化都是通过`format`函数,需要传一个`DateTimeFormatter`对象进去，我们可以通过`DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")`来生成自己想要的格式。

DateTimeFormatter类里面也有一些定义好的格式可以直接用,除了下面列出的还有一些其他的，感兴趣可以看一下，不过我觉得都没啥用。
- ISO_DATE_TIME         2011-12-03T10:15:30
- ISO_OFFSET_DATE_TIME  2011-12-03T10:15:30+01:00
- ISO_LOCAL_DATE_TIME   2011-12-03T10:15:30

```java
time.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
```

看一下`Date`的格式化。这个需要借用`SimpleDateFormat`类来完成格式化。

```java
SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
format.format(date);
```

### 时间转时间戳

时间转时间戳分为两种，一种是当你已经有一个`LocalDateTime`类型的时间了，需要转换成秒或者毫秒的时间戳。

#### 时间转换秒级时间戳

只需要直接用`toEpochSecond`方法就可以了。
```java
LocalDateTime time = LocalDateTime.now();
time.toEpochSecond(ZoneOffset.ofHours(8));
```

`Date`类型没有办法直接获取秒级时间戳，只能获取毫秒级再转秒。

#### 时间转换毫秒级时间戳

转换毫秒需要先转换成`instant`对象，然后才能转换成毫秒级时间戳。

```java
LocalDateTime time = LocalDateTime.now();
time.toInstant(ZoneOffset.ofHours(8)).toEpochMilli();
```

`Date`获取毫秒就很简单了。

```java
Date date = new Date();
date.getTime();
```

### 字符串转换成时间戳


时间转时间戳分为两种，除了上面的，还有一种是有一个格式化好的字符串，比如`2022-12-18 10:00:00`这种，但是这个是字符串并不是时间类型。所以要先转换成`LocalDateTime`类型，然后就可以转换成时间戳了。

其实就是上面格式化的一种反向操作。使用`parse`方法就可以了。

```java
LocalDateTime.parse("2022-12-18 10:00:00", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
LocalDateTime.parse("2022-12-18", DateTimeFormatter.ofPattern("yyyy-MM-dd"));
```

`Date`类型的字符串转时间戳也是通过`SimpleDateFormat`类来完成。

```java
SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
Date date = format.parse("2022-12-18 10:00:00")
```

### 时间戳转时间

那如果我们现在转换成时间戳以后又想转换成时间呢？也可以通过`instant`对象来做到。

#### 毫秒时间戳转时间

`Instant.ofEpochSecond(1671365543834)`是将一个毫秒时间戳转换成一个instant对象。在通过`ofInstant`方法就可以将instant对象转换成LocalDateTime对象了。

```java
LocalDateTime.ofInstant(Instant.ofEpochSecond(1671365543834), ZoneOffset.ofHours(8));
```

`Date`类

```java
Date date = new Date(1669759566000L);
```

#### 秒时间戳转时间

`Instant.ofEpochMilli(1671365543)`是将一个秒时间戳转换成`instant`对象。和上面的区别就是使用的是`ofEpochMilli`方法。

```java
LocalDateTime.ofInstant(Instant.ofEpochMilli(1671365543), ZoneOffset.ofHours(8));
```

`Date`类不支持秒，只能把秒转成毫秒在转时间戳。

### 时间比较

通过`compareTo`方法可以进行时间的一个比较大小。如果大于会返回1，小于返回-1.

```java
LocalDateTime time = LocalDateTime.now();
time.compareTo(LocalDateTime.now());
```

`Date`也是通过`compareTo`方法进行比较
```java
Date date = new Date(1669759566000L);
date.compareTo(new Date());
```

### 时间加减

如果加上几天，就是`plusDays`。加几个小时就是`plusHours`。当然也可以使用`plus`。

```java
LocalDateTime time = LocalDateTime.now();
time.plusDays(1);
time.plusHours(1);
time.plus(Period.ofDays(1));
```

如果减去几天就是`minusDays`.减去几个小时就是`minusHours`。当然也可以使用`minus`。

```java
LocalDateTime time = LocalDateTime.now();
time.minusDays(1);
time.minusHours(1);
time.minus(Period.ofDays(1));
```

`Date`类不支持时间加减，只能通过`Calendar`类实现。

```java
Date date = new Date();
Calendar calendar = Calendar.getInstance();
calendar.setTime(date);
calendar.add(Calendar.DAY_OF_MONTH, 1);
//减去
calendar.add(Calendar.DAY_OF_MONTH, -1);
```

### 时间格式在入参出参中的使用

入参的时候需要通过`JsonFormat`注解来指定需要的是字符串类型和对应的时间格式。
```java
@JsonFormat(shape = JsonFormat.Shape.STRING, pattern="yyyy-MM-dd")
private LocalDate date;

@JsonFormat(shape = JsonFormat.Shape.STRING, pattern="yyyy-MM-dd HH:mm:ss")
private LocalDateTime time;
```

出参的时候就很简单了，因为只是返回了一个字符串。

```java
private String time;
```

### 格式化时间源码分析

格式化的时候这两个年是不一样的，具体的可以看一下源码。我们来找一下。

首先点进去是LocalDateTime这个类里面
```java
@Override  // override for Javadoc and performance
public String format(DateTimeFormatter formatter) {
    //判断参数是否空
    Objects.requireNonNull(formatter, "formatter");
    //真正的执行格式化
    return formatter.format(this);
}
```

接下来点进去，看一下怎么执行的，可以看到又调用了`formatTo`这个函数，说明主要的格式化代码都在这里面。

```java
public String format(TemporalAccessor temporal) {
       //创建了一个32长度的字符串构建器
        StringBuilder buf = new StringBuilder(32);
        //格式化主要代码
        formatTo(temporal, buf);
        //转成字符串
        return buf.toString();
    }
```

看一下`formatTo`函数，可以发现主要是调用`printerParser`这个对象的`format`方法，那我们这个对象是哪来的呢，是在一开始指定格式化类型的时候来的。不同的格式化类型对应不同的解析器，也就会执行不同的`format`方法。

```java
public void formatTo(TemporalAccessor temporal, Appendable appendable) {
        //判断参数，这里不用管
        Objects.requireNonNull(temporal, "temporal");
        Objects.requireNonNull(appendable, "appendable");
        try {
            //创建一个DateTimePrintContext对象
            DateTimePrintContext context = new DateTimePrintContext(temporal, this);
            //判断，显然我们之前传过来的就是一个StringBuilder
            if (appendable instanceof StringBuilder) {
                //主要看这个怎么处理  这里有个 printerParser 对象，这个对象是怎么来的呢，其实是上面DateTimeFormatter.ofPattern的时候给创建的。
                printerParser.format(context, (StringBuilder) appendable);
            } else {
                //这里其实就是如果传的不是个StringBuilder，就在创建一个然后执行
                // buffer output to avoid writing to appendable in case of error
                StringBuilder buf = new StringBuilder(32);
                printerParser.format(context, buf);
                appendable.append(buf);
            }
        } catch (IOException ex) {
            throw new DateTimeException(ex.getMessage(), ex);
        }
    }
```

接下来我们看一下`ofPattern`这个方法里面是怎样的吧。这里是创建了一个 时间格式化的建造者，然后把我们这个字符串添加进去了。

```java
//这里的字符串就是我们传的 yyyy-MM-dd HH:mm:ss
public static DateTimeFormatter ofPattern(String pattern) {
    return new DateTimeFormatterBuilder().appendPattern(pattern).toFormatter();
}
```

看一下`appendPattern`里面是怎么把字符串加进去的。

```java
public DateTimeFormatterBuilder appendPattern(String pattern) {
    //忽略
    Objects.requireNonNull(pattern, "pattern");
    //主要的解析逻辑
    parsePattern(pattern);
    return this;
}
```

继续追踪到`parsePattern`方法里面。这个方法代码比较多，这里只关注我们想知道的。其余的有兴趣的可以看一下。

```java
private void parsePattern(String pattern) {
    //这里给字符串做循环，注意 pattern = yyyy-MM-dd HH:mm:ss 这个字符串。
    for (int pos = 0; pos < pattern.length(); pos++) {
        //取出字符 比如第一个就是 y 对应的ASCII码就是121
        char cur = pattern.charAt(pos);
        //这里就是判断是否是大小写字母了，也就是A-Z或者a-z
        if ((cur >= 'A' && cur <= 'Z') || (cur >= 'a' && cur <= 'z')) {
            //初始化变量 start = 0 pos = 1
            int start = pos++;
            //这里做一个循环，目的其实就是找出相同的字符有几个，比如y有4个，pos就会变成4
            for ( ; pos < pattern.length() && pattern.charAt(pos) == cur; pos++);  // short loop
            //这里就是算出具体的数量 4 - 0 = 4
            int count = pos - start;
            // padding  这里忽略 我们这里面没有这个字符
            if (cur == 'p') {
                int pad = 0;
                if (pos < pattern.length()) {
                    cur = pattern.charAt(pos);
                    if ((cur >= 'A' && cur <= 'Z') || (cur >= 'a' && cur <= 'z')) {
                        pad = count;
                        start = pos++;
                        for ( ; pos < pattern.length() && pattern.charAt(pos) == cur; pos++);  // short loop
                        count = pos - start;
                    }
                }
                if (pad == 0) {
                    throw new IllegalArgumentException(
                            "Pad letter 'p' must be followed by valid pad pattern: " + pattern);
                }
                padNext(pad); // pad and continue parsing
            }
            //接下来是主要逻辑。
            // main rules
            //从hashMap里面取出对应的值，这个map放在下面了。y取出来就是 YEAR_OF_ERA
            TemporalField field = FIELD_MAP.get(cur);
            //判断map里面取出来的是否为空，如果不为空就直接解析，如果为空就接着往下走，看是不是 zvZOXxWwY 这几个，如果都不是就会报错了
            if (field != null) {
                //我们y是能取出来的，直接解析这里 cur = y, count = 4, field = YEAR_OF_ERA
                parseField(cur, count, field);
            } else if (cur == 'z') {
                if (count > 4) {
                    throw new IllegalArgumentException("Too many pattern letters: " + cur);
                } else if (count == 4) {
                    appendZoneText(TextStyle.FULL);
                } else {
                    appendZoneText(TextStyle.SHORT);
                }
            } else if (cur == 'V') {
                if (count != 2) {
                    throw new IllegalArgumentException("Pattern letter count must be 2: " + cur);
                }
                appendZoneId();
            } else if (cur == 'Z') {
                if (count < 4) {
                    appendOffset("+HHMM", "+0000");
                } else if (count == 4) {
                    appendLocalizedOffset(TextStyle.FULL);
                } else if (count == 5) {
                    appendOffset("+HH:MM:ss","Z");
                } else {
                    throw new IllegalArgumentException("Too many pattern letters: " + cur);
                }
            } else if (cur == 'O') {
                if (count == 1) {
                    appendLocalizedOffset(TextStyle.SHORT);
                } else if (count == 4) {
                    appendLocalizedOffset(TextStyle.FULL);
                } else {
                    throw new IllegalArgumentException("Pattern letter count must be 1 or 4: " + cur);
                }
            } else if (cur == 'X') {
                if (count > 5) {
                    throw new IllegalArgumentException("Too many pattern letters: " + cur);
                }
                appendOffset(OffsetIdPrinterParser.PATTERNS[count + (count == 1 ? 0 : 1)], "Z");
            } else if (cur == 'x') {
                if (count > 5) {
                    throw new IllegalArgumentException("Too many pattern letters: " + cur);
                }
                String zero = (count == 1 ? "+00" : (count % 2 == 0 ? "+0000" : "+00:00"));
                appendOffset(OffsetIdPrinterParser.PATTERNS[count + (count == 1 ? 0 : 1)], zero);
            } else if (cur == 'W') {
                // Fields defined by Locale
                if (count > 1) {
                    throw new IllegalArgumentException("Too many pattern letters: " + cur);
                }
                appendInternal(new WeekBasedFieldPrinterParser(cur, count));
            } else if (cur == 'w') {
                // Fields defined by Locale
                if (count > 2) {
                    throw new IllegalArgumentException("Too many pattern letters: " + cur);
                }
                appendInternal(new WeekBasedFieldPrinterParser(cur, count));
            } else if (cur == 'Y') {
                // Fields defined by Locale
                appendInternal(new WeekBasedFieldPrinterParser(cur, count));
            } else {
                throw new IllegalArgumentException("Unknown pattern letter: " + cur);
            }
            pos--;

        } else if (cur == '\'') {
            // parse literals
            int start = pos++;
            for ( ; pos < pattern.length(); pos++) {
                if (pattern.charAt(pos) == '\'') {
                    if (pos + 1 < pattern.length() && pattern.charAt(pos + 1) == '\'') {
                        pos++;
                    } else {
                        break;  // end of literal
                    }
                }
            }
            if (pos >= pattern.length()) {
                throw new IllegalArgumentException("Pattern ends with an incomplete string literal: " + pattern);
            }
            String str = pattern.substring(start + 1, pos);
            if (str.length() == 0) {
                appendLiteral('\'');
            } else {
                appendLiteral(str.replace("''", "'"));
            }

        } else if (cur == '[') {
            optionalStart();

        } else if (cur == ']') {
            if (active.parent == null) {
                throw new IllegalArgumentException("Pattern invalid as it contains ] without previous [");
            }
            optionalEnd();

        } else if (cur == '{' || cur == '}' || cur == '#') {
            throw new IllegalArgumentException("Pattern includes reserved character: '" + cur + "'");
        } else {
            // - : 这两个符号就会走到这里了
            appendLiteral(cur);
        }
    }
}
```

看一下通过不同的key取值的map
```java
//时代
FIELD_MAP.put('G', ChronoField.ERA);                       // SDF, LDML (different to both for 1/2 chars)
//这个时代的年份，也就是我们常用的年份yyyy
FIELD_MAP.put('y', ChronoField.YEAR_OF_ERA);               // SDF, LDML
//单纯的年份
FIELD_MAP.put('u', ChronoField.YEAR);                      // LDML (different in SDF)
FIELD_MAP.put('Q', IsoFields.QUARTER_OF_YEAR);             // LDML (removed quarter from 310)
FIELD_MAP.put('q', IsoFields.QUARTER_OF_YEAR);             // LDML (stand-alone)
//一年里面的月份，也是我们常用的月份 MM
FIELD_MAP.put('M', ChronoField.MONTH_OF_YEAR);             // SDF, LDML
FIELD_MAP.put('L', ChronoField.MONTH_OF_YEAR);             // SDF, LDML (stand-alone)
//一年里面的天，我们基本不用这个作为日子
FIELD_MAP.put('D', ChronoField.DAY_OF_YEAR);               // SDF, LDML
//一个月里面的天，我们常用这个获取多少号
FIELD_MAP.put('d', ChronoField.DAY_OF_MONTH);              // SDF, LDML
FIELD_MAP.put('F', ChronoField.ALIGNED_DAY_OF_WEEK_IN_MONTH);  // SDF, LDML
FIELD_MAP.put('E', ChronoField.DAY_OF_WEEK);               // SDF, LDML (different to both for 1/2 chars)
FIELD_MAP.put('c', ChronoField.DAY_OF_WEEK);               // LDML (stand-alone)
FIELD_MAP.put('e', ChronoField.DAY_OF_WEEK);               // LDML (needs localized week number)
FIELD_MAP.put('a', ChronoField.AMPM_OF_DAY);               // SDF, LDML
//一天里面的小时，常用的小时 HH
FIELD_MAP.put('H', ChronoField.HOUR_OF_DAY);               // SDF, LDML
FIELD_MAP.put('k', ChronoField.CLOCK_HOUR_OF_DAY);         // SDF, LDML
FIELD_MAP.put('K', ChronoField.HOUR_OF_AMPM);              // SDF, LDML
FIELD_MAP.put('h', ChronoField.CLOCK_HOUR_OF_AMPM);        // SDF, LDML
//一个小时里面的分钟，常用的分钟 mm
FIELD_MAP.put('m', ChronoField.MINUTE_OF_HOUR);            // SDF, LDML
//一分钟里面的秒数，常用的秒数 ss
FIELD_MAP.put('s', ChronoField.SECOND_OF_MINUTE);          // SDF, LDML
//这个大S基本不用，这是秒里面的纳秒
FIELD_MAP.put('S', ChronoField.NANO_OF_SECOND);            // LDML (SDF uses milli-of-second number)
FIELD_MAP.put('A', ChronoField.MILLI_OF_DAY);              // LDML
FIELD_MAP.put('n', ChronoField.NANO_OF_SECOND);            // 310 (proposed for LDML)
FIELD_MAP.put('N', ChronoField.NANO_OF_DAY);               // 310 (proposed for LDML)
```

继续深入，直接解析y的方法`parseField`。可以看到这个是根据我们格式化的字母执行不同的代码，比如u,y都执行到一个代码块。4个y走到了`appendValue`方法里面。

```java
private void parseField(char cur, int count, TemporalField field) {
    boolean standalone = false;
    switch (cur) {
        case 'u':
        case 'y':
            //判断数量
            if (count == 2) {
                //yy走这里
                appendValueReduced(field, 2, 2, ReducedPrinterParser.BASE_DATE);
            } else if (count < 4) {
                //y or yyy走这里
                appendValue(field, count, 19, SignStyle.NORMAL);
            } else {
                // yyyy走这里 field = YEAR_OF_ERA count = 4
                appendValue(field, count, 19, SignStyle.EXCEEDS_PAD);
            }
            break;
        case 'c':
            if (count == 2) {
                throw new IllegalArgumentException("Invalid pattern \"cc\"");
            }
            /*fallthrough*/
        case 'L':
        case 'q':
            standalone = true;
            /*fallthrough*/
        case 'M':
        case 'Q':
        case 'E':
        case 'e':
            switch (count) {
                case 1:
                case 2:
                    //两个MM输出月份走到这里
                    if (cur == 'c' || cur == 'e') {
                        appendInternal(new WeekBasedFieldPrinterParser(cur, count));
                    } else if (cur == 'E') {
                        appendText(field, TextStyle.SHORT);
                    } else {
                        if (count == 1) {
                            appendValue(field);
                        } else {
                            //经过判断走到这里
                            appendValue(field, 2);
                        }
                    }
                    break;
                case 3:
                    appendText(field, standalone ? TextStyle.SHORT_STANDALONE : TextStyle.SHORT);
                    break;
                case 4:
                    appendText(field, standalone ? TextStyle.FULL_STANDALONE : TextStyle.FULL);
                    break;
                case 5:
                    appendText(field, standalone ? TextStyle.NARROW_STANDALONE : TextStyle.NARROW);
                    break;
                default:
                    throw new IllegalArgumentException("Too many pattern letters: " + cur);
            }
            break;
        case 'a':
            if (count == 1) {
                appendText(field, TextStyle.SHORT);
            } else {
                throw new IllegalArgumentException("Too many pattern letters: " + cur);
            }
            break;
        case 'G':
            switch (count) {
                case 1:
                case 2:
                case 3:
                    appendText(field, TextStyle.SHORT);
                    break;
                case 4:
                    appendText(field, TextStyle.FULL);
                    break;
                case 5:
                    appendText(field, TextStyle.NARROW);
                    break;
                default:
                    throw new IllegalArgumentException("Too many pattern letters: " + cur);
            }
            break;
        case 'S':
            appendFraction(NANO_OF_SECOND, count, count, false);
            break;
        case 'F':
            if (count == 1) {
                appendValue(field);
            } else {
                throw new IllegalArgumentException("Too many pattern letters: " + cur);
            }
            break;
        case 'd':
        case 'h':
        case 'H':
        case 'k':
        case 'K':
        case 'm':
        case 's':
            if (count == 1) {
                appendValue(field);
            } else if (count == 2) {
                //可以看到dd HH mm ss也是走到这里，最终也是通过NumberPrinterParser这个对象来格式化
                appendValue(field, count);
            } else {
                throw new IllegalArgumentException("Too many pattern letters: " + cur);
            }
            break;
        case 'D':
            if (count == 1) {
                appendValue(field);
            } else if (count <= 3) {
                appendValue(field, count);
            } else {
                throw new IllegalArgumentException("Too many pattern letters: " + cur);
            }
            break;
        default:
            if (count == 1) {
                appendValue(field);
            } else {
                appendValue(field, count);
            }
            break;
    }
}
```

看一下`appendValue`方法。field = YEAR_OF_ERA,minWidth = 4, maxWidth = 19, signStyle = SignStyle.EXCEEDS_PAD。前面是一些判断，重点是创建了一个`NumberPrinterParser`的对象。最后转换的时候其实就是通过这个对象来转换的。

```java
public DateTimeFormatterBuilder appendValue(
    TemporalField field, int minWidth, int maxWidth, SignStyle signStyle) {
    //这里不执行 忽略
    if (minWidth == maxWidth && signStyle == SignStyle.NOT_NEGATIVE) {
        return appendValue(field, maxWidth);
    }
    //参数校验
    Objects.requireNonNull(field, "field");
    Objects.requireNonNull(signStyle, "signStyle");
    //一些校验规则
    if (minWidth < 1 || minWidth > 19) {
        throw new IllegalArgumentException("The minimum width must be from 1 to 19 inclusive but was " + minWidth);
    }
    if (maxWidth < 1 || maxWidth > 19) {
        throw new IllegalArgumentException("The maximum width must be from 1 to 19 inclusive but was " + maxWidth);
    }
    if (maxWidth < minWidth) {
        throw new IllegalArgumentException("The maximum width must exceed or equal the minimum width but " +
                maxWidth + " < " + minWidth);
    }
    //重点是这里，创建了一个 NumberPrinterParser的对象，把参数传进去了。
    NumberPrinterParser pp = new NumberPrinterParser(field, minWidth, maxWidth, signStyle);
    appendValue(pp);
    return this;
}
```

看一下`NumberPrinterParser`类。还记得最开始格式化的时候那一段代码`printerParser.format(context, (StringBuilder) appendable);`吗，实际调用的就是这里。？

```java

//构造方法赋值
NumberPrinterParser(TemporalField field, int minWidth, int maxWidth, SignStyle signStyle) {
    // validated by caller
    this.field = field;
    this.minWidth = minWidth;
    this.maxWidth = maxWidth;
    this.signStyle = signStyle;
    this.subsequentWidth = 0;
}

//格式化方法
@Override
public boolean format(DateTimePrintContext context, StringBuilder buf) {
    //从context上下文中获取字段 field = YEAR_OF_ERA  context实际包含了真正的时间 2022-12-01T00:00:00
    Long valueLong = context.getValue(field);
    if (valueLong == null) {
        return false;
    }
    //获取到以后 value = 2022
    long value = getValue(context, valueLong);
    DecimalStyle decimalStyle = context.getDecimalStyle();
    String str = (value == Long.MIN_VALUE ? "9223372036854775808" : Long.toString(Math.abs(value)));
    if (str.length() > maxWidth) {
        throw new DateTimeException("Field " + field +
            " cannot be printed as the value " + value +
            " exceeds the maximum print width of " + maxWidth);
    }
    //转换一个格式类型
    str = decimalStyle.convertNumberToI18N(str);

    //这些条件都不满足，忽略
    if (value >= 0) {
        switch (signStyle) {
            case EXCEEDS_PAD:
                if (minWidth < 19 && value >= EXCEED_POINTS[minWidth]) {
                    buf.append(decimalStyle.getPositiveSign());
                }
                break;
            case ALWAYS:
                buf.append(decimalStyle.getPositiveSign());
                break;
        }
    } else {
        switch (signStyle) {
            case NORMAL:
            case EXCEEDS_PAD:
            case ALWAYS:
                buf.append(decimalStyle.getNegativeSign());
                break;
            case NOT_NEGATIVE:
                throw new DateTimeException("Field " + field +
                    " cannot be printed as the value " + value +
                    " cannot be negative according to the SignStyle");
        }
    }
    //填充0 也就是yyyy minWidth = 4就会填充0 MM minWidth = 2如果 1月就会填充01，一个M就不会走到循环填充0
    for (int i = 0; i < minWidth - str.length(); i++) {
        buf.append(decimalStyle.getZeroDigit());
    }
    //输出到buf中
    buf.append(str);
    return true;
}
```

可以看到上面的代码，但是`NumberPrinterParser`其实只是解析了`yMdHms`这些格式的。也可以再看一下`M`的确认一下。

首先是`appendValue`这个方法。大差不差，除了传到解析器的参数不一样，没啥区别，其实dd这些也都一样。

```java
public DateTimeFormatterBuilder appendValue(TemporalField field, int width) {
    //参数校验
    Objects.requireNonNull(field, "field");
    if (width < 1 || width > 19) {
        throw new IllegalArgumentException("The width must be from 1 to 19 inclusive but was " + width);
    }
    //可以发现MM也是用的yyyy这个解析器格式化的，但是后面三个参数不一样
    NumberPrinterParser pp = new NumberPrinterParser(field, width, width, SignStyle.NOT_NEGATIVE);
    appendValue(pp);
    return this;
}
```

那我们`-`,`:`这些格式化符号的输出呢？是通过另外一个解析器，它先是取到`char`类型的一个字符来判断的时候会走到else里面然后走`appendLiteral(cur);`这个方法。看一下这个方法里面。这里可以看到主要使用的是 CharLiteralPrinterParser 这个解析器。

```java
public DateTimeFormatterBuilder appendLiteral(char literal) {
    //这里可以看到主要使用的是 CharLiteralPrinterParser 这个解析器
    appendInternal(new CharLiteralPrinterParser(literal));
    return this;
}
```

看一下 CharLiteralPrinterParser 这个解析器

```java
//构造方法
CharLiteralPrinterParser(char literal) {
    this.literal = literal;
}

@Override
public boolean format(DateTimePrintContext context, StringBuilder buf) {
    //简单粗暴 直接把 - : 这种符号添加到字符串里面
    buf.append(literal);
    return true;
}
```

接下来看一下为啥我们刚才上面说的，y代表 YEAR_OF_ERA，为啥就能从`2022-12-01`里面取到`2022`呢？这个可以看到我们`NumberPrinterParser`这个解析器里面主要调用了一个`context.getValue(field)`方法。


主要是`temporal.getLong(field)`方法，其实temporal就是我们的日期时间，在我们一开始创建上下文的时候过来的。回忆一下上面的创建。这里的temporal可以再往上一层传过来的，传的其实就是`LocalDateTime的对象`。

> new DateTimePrintContext(temporal, this)

```java
Long getValue(TemporalField field) {
    try {
        //主要是这里，其实temporal就是我们的日期时间，在我们一开始创建上下文的时候过来的。
        return temporal.getLong(field);
    } catch (DateTimeException ex) {
        if (optional > 0) {
            return null;
        }
        throw ex;
    }
}
```

所以我们再看一下`getLong`方法。可以看到有一个类型判断，`yMdHms`这几个类型就会走到if里面，如果是时间的 Hms这几个调用time.getLong方法，yMd日期的调用日期的getLong方法。Y的话就会走到 `getFrom` 这个方法。而且是通过`field`调用。

```java
@Override
public long getLong(TemporalField field) {
    //类型判断 yMdHms这几个走的这里面
    if (field instanceof ChronoField) {
        ChronoField f = (ChronoField) field;
        //如果是时间的 Hms这几个调用time.getLong方法，yMd日期的调用日期的getLong方法
        return (f.isTimeBased() ? time.getLong(field) : date.getLong(field));
    }
    //Y走这个方法
    return field.getFrom(this);
}
```

看一下`getFrom`方法。

```java
@Override
public long getFrom(TemporalAccessor temporal) {
    if (rangeUnit == WEEKS) {  // day-of-week
        return localizedDayOfWeek(temporal);
    } else if (rangeUnit == MONTHS) {  // week-of-month
        return localizedWeekOfMonth(temporal);
    } else if (rangeUnit == YEARS) {  // week-of-year
        return localizedWeekOfYear(temporal);
    } else if (rangeUnit == WEEK_BASED_YEARS) {
        return localizedWeekOfWeekBasedYear(temporal);
    } else if (rangeUnit == FOREVER) {
        // YYYY 大写的Y走的是这里
        return localizedWeekBasedYear(temporal);
    } else {
        throw new IllegalStateException("unreachable, rangeUnit: " + rangeUnit + ", this: " + this);
    }
}
```

如果大写的`Y`格式化就会走下面的函数，主要就是取出年份以后计算周数，如果周数=0就认为是上一年的，年份-1，如果周数大于等于下一年的周数就年份+1

```java
private int localizedWeekBasedYear(TemporalAccessor temporal) {
    //获取到这周的第几天 第5天
    int dow = localizedDayOfWeek(temporal);
    //获取日期中的年份 2021
    int year = temporal.get(YEAR);
    //获取今年的第几天 2021-12-30 是 364天
    int doy = temporal.get(DAY_OF_YEAR);
    //这周开始的偏移量 5
    int offset = startOfWeekOffset(doy, dow);
    //今年的第几周 53周
    int week = computeWeek(offset, doy);
    //如果这周是0周，就是上一年的，年份就-1
    if (week == 0) {
        // Day is in end of week of previous year; return the previous year
        return year - 1;
    } else {
        //如果接近年底，使用更高精度的逻辑
        //检查 如果年份的日期包含在下一年的部分的星期里面了
        // If getting close to end of year, use higher precision logic
        // Check if date of year is in partial week associated with next year
        //获取一年里面的天数 对象里面包含 最小1天 - 最大365天
        ValueRange dayRange = temporal.range(DAY_OF_YEAR);
        //获取到年份的长度，也就是365
        int yearLen = (int)dayRange.getMaximum();
        //下一年的周数 根据下面的计算公式得出 (7 + 5 + 366 - 1) / 7 = 53
        //这里为啥是366呢，因为yearLen是今年的天数也就是365 + 1，其实也就是到下一年去了。为的是计算下一年的第一周
        int newYearWeek = computeWeek(offset, yearLen + weekDef.getMinimalDaysInFirstWeek());
        //比较如果今年的这周大于等于下一年的周 就年份 +1 所以这里格式化就会出错了。
        if (week >= newYearWeek) {
            return year + 1;
        }
    }
    return year;
}

```

那么是怎么计算今年的第几周的呢，看一下`computeWeek`方法。其实就是一个计算公式。

```java
//offset = 5 , day = 今年的第几天 364 天
private int computeWeek(int offset, int day) {
    //计算公式 ( 7 + 5 + （364 - 1）) / 7
    return ((7 + offset + (day - 1)) / 7);
}
```

还有一个问题，就是我们用到了一个周的偏移量，这个偏移量怎么计算的呢，看一下这个方法`startOfWeekOffset`。以`2021-12-30`为例，day = 364,dow = 5

```java
private int startOfWeekOffset(int day, int dow) {
    // offset of first day corresponding to the day of week in first 7 days (zero origin)
    //算出上一周 (364 - 5) % 7 = 2
    int weekStart = Math.floorMod(day - dow, 7);
    // offset = -2
    int offset = -weekStart;
    //这里 2 + 1  > 1会走进去
    if (weekStart + 1 > weekDef.getMinimalDaysInFirstWeek()) {
        // The previous week has the minimum days in the current month to be a 'week'
        //这里 7 - 2 = 5 返回的就是5
        offset = 7 - weekStart;
    }
    return offset;
}
```

上面看完了大写的`Y`，再来看一下小写的`y`。走的`getLong`方法。

日期的`getLong`方法。经过判断后主要看`get0`这个方法。可以看到这个命名就很随意了。。

```java
@Override
public long getLong(TemporalField field) {
    //这个判断也会走进来
    if (field instanceof ChronoField) {
        //这两个判断忽略
        if (field == EPOCH_DAY) {
            return toEpochDay();
        }
        if (field == PROLEPTIC_MONTH) {
            return getProlepticMonth();
        }
        //走到这里
        return get0(field);
    }
    return field.getFrom(this);
}
```

看一下日期的`get0`方法。可以发现了，这里主要处理了这几种类型。我们常用的
- y也就是YEAR_OF_ERA 处理很简单，判断了一下year >= 1就返回 year。
- M也就是MONTH_OF_YEAR 处理很简单，返回日期的month.
- d也就是DAY_OF_MONTH 返回日期的day.

从这里也可以看出我们格式化成`YEAR`和`ERA`作为年其实也是可以的。

```java
private int get0(TemporalField field) {
    switch ((ChronoField) field) {
        case DAY_OF_WEEK: return getDayOfWeek().getValue();
        case ALIGNED_DAY_OF_WEEK_IN_MONTH: return ((day - 1) % 7) + 1;
        case ALIGNED_DAY_OF_WEEK_IN_YEAR: return ((getDayOfYear() - 1) % 7) + 1;
        case DAY_OF_MONTH: return day;
        case DAY_OF_YEAR: return getDayOfYear();
        case EPOCH_DAY: throw new UnsupportedTemporalTypeException("Invalid field 'EpochDay' for get() method, use getLong() instead");
        case ALIGNED_WEEK_OF_MONTH: return ((day - 1) / 7) + 1;
        case ALIGNED_WEEK_OF_YEAR: return ((getDayOfYear() - 1) / 7) + 1;
        case MONTH_OF_YEAR: return month;
        case PROLEPTIC_MONTH: throw new UnsupportedTemporalTypeException("Invalid field 'ProlepticMonth' for get() method, use getLong() instead");
        case YEAR_OF_ERA: return (year >= 1 ? year : 1 - year);
        case YEAR: return year;
        case ERA: return (year >= 1 ? 1 : 0);
    }
    throw new UnsupportedTemporalTypeException("Unsupported field: " + field);
}
```

看完了日期的处理再看一下时间的吧，其实大同小异了。

时间的`getLong`方法。同样的经过判断走到`get0`里面，注意这是时间的`getLong`和`get0`。

```java
@Override
public long getLong(TemporalField field) {
    if (field instanceof ChronoField) {
        if (field == NANO_OF_DAY) {
            return toNanoOfDay();
        }
        if (field == MICRO_OF_DAY) {
            return toNanoOfDay() / 1000;
        }
        return get0(field);
    }
    return field.getFrom(this);
}
```

时间的`get0`方法。处理的就是这些类型了。主要看我们关注的几个
- H 也就是 HOUR_OF_DAY, 直接返回时间的 hour
- m 也就是MINUTE_OF_HOUR，直接返回时间的 minute
- s 也就是 SECOND_OF_MINUTE, 直接返回时间的 second

```java
private int get0(TemporalField field) {
    switch ((ChronoField) field) {
        case NANO_OF_SECOND: return nano;
        case NANO_OF_DAY: throw new UnsupportedTemporalTypeException("Invalid field 'NanoOfDay' for get() method, use getLong() instead");
        case MICRO_OF_SECOND: return nano / 1000;
        case MICRO_OF_DAY: throw new UnsupportedTemporalTypeException("Invalid field 'MicroOfDay' for get() method, use getLong() instead");
        case MILLI_OF_SECOND: return nano / 1000_000;
        case MILLI_OF_DAY: return (int) (toNanoOfDay() / 1000_000);
        case SECOND_OF_MINUTE: return second;
        case SECOND_OF_DAY: return toSecondOfDay();
        case MINUTE_OF_HOUR: return minute;
        case MINUTE_OF_DAY: return hour * 60 + minute;
        case HOUR_OF_AMPM: return hour % 12;
        case CLOCK_HOUR_OF_AMPM: int ham = hour % 12; return (ham % 12 == 0 ? 12 : ham);
        case HOUR_OF_DAY: return hour;
        case CLOCK_HOUR_OF_DAY: return (hour == 0 ? 24 : hour);
        case AMPM_OF_DAY: return hour / 12;
    }
    throw new UnsupportedTemporalTypeException("Unsupported field: " + field);
}
```

## 总结

好了，到这里我们知道了时间格式的各种使用方法和格式化的源码。

对于不同格式化的区别。总结一下。
- `y` 处理简单，只是判断了year > 1 就返回了year。
- `Y` 处理较复杂，还判断了周，根据情况对年份+1和-1。某些年份的某些日期会有坑。**一定要注意！！！**
- `Md Hms`处理非常简单，直接返回了日期时间上面对应的数。
- `-: `一些特殊字符，格式化的时候是直接增加到字符串里面的。 


下面总结一下源码对应文件和方法的追踪链。感兴趣的可以自己在多翻翻源码。

ofPattern指定格式的调用链
- DateTimeFormatter.java -> public static DateTimeFormatter ofPattern(String pattern)
    - DateTimeFormatterBuilder.java -> public DateTimeFormatterBuilder appendPattern(String pattern)
    - DateTimeFormatterBuilder.java -> private void parsePattern(String pattern)
    - DateTimeFormatterBuilder.java -> private void parseField(char cur, int count, TemporalField field)
    - DateTimeFormatterBuilder.java -> public DateTimeFormatterBuilder appendValue(TemporalField field, int width)
    - 在这里创建的解析器
    - DateTimeFormatterBuilder.java -> `static class NumberPrinterParser implements DateTimePrinterParser`
    - DateTimeFormatterBuilder.java -> `static final class CharLiteralPrinterParser implements DateTimePrinterParser`

format方法调用链
- LocalDateTime.java -> public String format(DateTimeFormatter formatter)
    - DateTimeFormatter.java -> public String format(TemporalAccessor temporal)
    - DateTimeFormatter.java -> public void formatTo(TemporalAccessor temporal, Appendable appendable)
        - 接下来根据不同的处理解析器进行处理，主要有两个解析器
        - DateTimeFormatterBuilder.java -> `static class NumberPrinterParser implements DateTimePrinterParser`
        - DateTimeFormatterBuilder.java -> `static final class CharLiteralPrinterParser implements DateTimePrinterParser`
            - DateTimePrintContext.java -> Long getValue(TemporalField field)
                - LocalDateTime.java -> public long getLong(TemporalField field)
                    - 这里日期调日期的 LocalDate.java -> public long getLong(TemporalField field)
                    - LocalDate.java -> private int get0(TemporalField field) 
                    - 时间调时间的 LocalTime.java -> public long getLong(TemporalField field)
                    - LocalTime.java -> private int get0(TemporalField field)



