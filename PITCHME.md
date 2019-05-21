# Simplification Code

---

## At A Glance

 * @color[green](Deployment)
 * @color[green](Monitoring)
 * @color[orange](Code Clarity)
 * @color[orange](Complexity)
 * @color[red](Performance)
 * @color[red](Intrinsic Behaviour)

---

## Time And Numerics One Null Chaining

### PLSQL

```sql
SELECT TRUNC(TIMESTAMP '2018-01-01 13:14:15','hh') + 
       TRUNC(TO_CHAR(TIMESTAMP '2018-01-01 13:14:15','MI')/30)*30/60/24 
  FROM dual
```

### Java

```java
    private static LocalDateTime executeADeepArithmeticCallWithNulls(LocalDateTime mktPeriod) {
        return localDateTimeAdd(trunc(mktPeriod, "hh"),
                bigDecimalDivide(
                        bigDecimalDivide(
                                bigDecimalMultiply(
                                        trunc(bigDecimalDivide(
                                                        makeNewLong(BaseUtil.toChar(mktPeriod, "MI")),
                                                        30L)),
                                        30L),
                                makeNewBigDecimal(60L)),
                        makeNewBigDecimal(24L)));
    }

    /**
     * This test represents a check on handling a deeply nested NULL expression containing Date arithmetic.
     * Specifically: TRUNC(:new.MktPeriod,'hh') + TRUNC(TO_CHAR(:new.MktPeriod,'MI')/30)*30/60/24
     */
    @Test
    public void testDateArithmeticOperationsWillShortCircuitOnNull() {
        assertNull(executeADeepArithmeticCallWithNulls(null));
        // SELECT TRUNC(TIMESTAMP '2018-01-01 13:14:15','hh') + TRUNC(TO_CHAR(TIMESTAMP '2018-01-01 13:14:15','MI')/30)*30/60/24 FROM dual
        // 2018-01-01 13:00:00
        LocalDateTime expected = ldt("2018-01-01T13:00:00");
        LocalDateTime input = ldt("2018-01-01T13:14:15");
        assertEquals(expected, executeADeepArithmeticCallWithNulls(input));
    }
```

---

## Numerics Fraction Of A Day

### PLSQL

```sql
SELECT TRUNC(SYSDATE) + 1/86400 FROM dual
/

-- 2019-05-22 00:00:01

-- One Second Fraction Of a Day
SELECT 1/86400 FROM dual
/

-- 0.00001157407407407407407407407407407407407407
```

### Java

Introduction of Oracle Like Math Contexts for Numeric Precision

```java
    // ===================================================
    //                     Math Contexts
    // ===================================================

    // Line up with Oracle Decimal Results
    // Oracle returns decimal value which has 44 digits after '.' for date comparing, e.g.
    // select (to_date('2009-07-07 22:00:01', 'YYYY-MM-DD hh24:mi:ss') - to_date('2009-07-07 22:00:00', 'YYYY-MM-DD hh24:mi:ss')) as result from dual;
    // 0.0000115740740740740740740740740740740740
    private static final int BIG_DECIMAL_SCALE = 45; // The 45 is correct
    private static final RoundingMode BIG_DECIMAL_ROUNDING = RoundingMode.HALF_UP;
    private static final MathContext ORACLE_DECIMAL_STYLE_CONTEXT = new MathContext(BIG_DECIMAL_SCALE, BIG_DECIMAL_ROUNDING);

    // INTs in PLSQL are an alias for NUMBER(38).
    // If we retain more precision than that then become too accurate and get inconsistent results especially on Date
    // Arithmetic.
    private static final int LONG_SCALE = 39; // The 39 is correct
    private static final MathContext ORACLE_INT_STYLE_CONTEXT = new MathContext(LONG_SCALE, BIG_DECIMAL_ROUNDING);
```

### Java Translation To Durations

```java
    // ======================
    // Minute Based Durations
    // ======================

    // 1d / (60d * 24d)
    // 1d / 1440d
    public static final Duration ONE_MINUTE = Duration.ofMinutes(1L);
    public static final BigDecimal BD_ONE_MINUTE = new BigDecimal(1d / 1440d);

    // 5d / (60d * 24d)
    // 5d / 60d / 24d
    // 5d / 1440d
    // 1d / 24d / 2d/ 6d
    // 1d / 48d/ 6d
    public static final Duration FIVE_MINUTES = Duration.ofMinutes(5L);
    public static final BigDecimal BD_FIVE_MINUTES = new BigDecimal(5d / 1440d);
```

---

## Triggers

---

## Intrinsic Behaviour

---


@snap[east span-50]
![](assets/img/presentation.png)
@snapend

---?color=#E58537
@title[Add A Little Imagination]

@snap[north-west]
#### Add a splash of @color[cyan](**color**) and you are ready to start presenting...
@snapend

@snap[west span-55]
@ul[spaced text-white]
- You will be amazed
- What you can achieve
- *With a little imagination...*
- And **GitPitch Markdown**
@ulend
@snapend

@snap[east span-45]
@img[shadow](assets/img/conference.png)
@snapend

---?image=assets/img/presenter.jpg

@snap[north span-100 headline]
## Now It's Your Turn
@snapend

@snap[south span-100 text-06]
[Click here to jump straight into the interactive feature guides in the GitPitch Docs @fa[external-link]](https://gitpitch.com/docs/getting-started/tutorial/)
@snapend
