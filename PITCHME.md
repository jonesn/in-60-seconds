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

## Time

```plsql
SELECT TRUNC(TIMESTAMP '2018-01-01 13:14:15','hh') + 
       TRUNC(TO_CHAR(TIMESTAMP '2018-01-01 13:14:15','MI')/30)*30/60/24 
  FROM dual
```

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

## Numerics

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
