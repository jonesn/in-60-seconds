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

  * Part 1: The dispatch of them, I.e. When does a trigger fire? DAO per Table.
  * Part 2: The providing of old rows default values in records.
  * Part 3: The logic.

## Example Workflow Status Update

### PLSQL Update Statement

```sql
UPDATE MktCase
               SET    CaseState          = WorkflowQ.gStateSolved,
                      ExecutionDuration  = ROUND((Timepac.GetGmtDate - ExecutionTime) * WorkflowQ.gDaySeconds)
               WHERE  CaseID             = pCaseID;

               vMessage := '@Case(' || vCaseName || ') ' || pCaseID || ' SOLVED';
               WorkflowQ.UserCaseMessage (pCaseID, 'Case', vMessage);
```

### PLSQL Trigger

```sql
create trigger MKTCASE_AR_OVERRIDESET
  after insert or update of OVERRIDESETID,CASENAME,CASESTARTTIME,CASEENDTIME or delete
  on MKTCASE
  for each row
BEGIN
--
-- Add the override set to the list to be processed in the after statement trigger
--
  If nvl(:new.OverrideSetID, :old.OverrideSetID) is not null Then
    Override.OvrdAdd(nvl(:new.OverrideSetID, :old.OverrideSetID));

    If :new.OverrideSetID is not null and
       nvl(:new.CaseName, :old.CaseName) <> nvl(:old.CaseName, :new.CaseName) Then -- fix override set name
       Update MktOverrideset set
              Name = :new.CaseName
        where OverrideSetID = :new.OverrideSetID;
    end if;

  end if;
END MktCase_AR_OverrideSet;
/
```

### Java Call

```java
rowcount = new Long(mktcasedao.update_12(workflowq.GSTATESOLVED(), workflowq.GDAYSECONDS(), PCASEID));
                    VMESSAGE = "@Case(" + VCASENAME + ") " + PCASEID + " SOLVED";
                    workflowq.USERCASEMESSAGE(PCASEID, "Case", VMESSAGE, "Info");
```

### Java SQL Dispatch

```java
public int update_12(BigDecimal GSTATESOLVED,
                         BigDecimal GDAYSECONDS,
                         BigDecimal pcaseid) {
        if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("update_12(" + "GSTATESOLVED: " + toStr(GSTATESOLVED) +
                                        ", GDAYSECONDS: " + toStr(GDAYSECONDS) +
                                        ", pcaseid: " + toStr(pcaseid) + ")");
        }

        Update11Dto input = new Update11Dto();
        input.setCasestate(GSTATESOLVED);
        input.setGdayseconds(GDAYSECONDS);
        input.setPcaseid(pcaseid);
        return update(SQL_UPDATE_11,
                      new TriggerRowSupplier<MKTCASEVO>() {
                          @Override
                          public List<MKTCASEVO> getOldRows() {
                              return query(SQL_UPDATE_13_old_rows, input);
                          }
                          @Override
                          public MKTCASEVO makeNewRow(MKTCASEVO oldRecord) {
                              MKTCASEVO newRow = cloneOldRow(oldRecord);
                              newRow.setCasestate(input.getCasestate());
                              newRow.setExecutionduration(
                                      round(bigDecimalMultiply(
                                                      (localDateTimeSubtract(com.tsri.lib.tputil.TIMEPAC.GETGMTDATE(),
                                                                                        oldRecord.getExecutiontime())),
                                                      input.getGdayseconds())));
                              return newRow;
                          }
                      });
    }
```

### Java Trigger Dispatch

```java
    protected int update(String sqlUpdate, TriggerRowSupplier<R> triggerRowSupplier) {
        
        int result = 0;

        List<R> oldVals = triggerRowSupplier.getOldRows();
        Set<String> alteredFields = makeAlteredFields(sqlUpdate);
        beforeUpdateStatement(alteredFields);
        if (!CollectionUtils.isEmpty(oldVals)) {
            for (int i = 0; i < oldVals.size(); i++) { // VO will need to know its PK and be able to return it.
                R newVals = triggerRowSupplier.makeNewRow(oldVals.get(i));
                beforeUpdateRow(oldVals.get(i), newVals); // makeNewRow(sqlUpdate, params, oldVals.get(i)));
                result += updateCall(sqlUpdate, newVals); // makeRowParams(sqlUpdate, params, oldVals.get(i).pk()));
                afterUpdateRow(oldVals.get(i), newVals); // makeNewRow(sqlUpdate, params, oldVals.get(i)));
            }
        }
        afterUpdateStatement(alteredFields);

        return result;
    }
```


---

## Intrinsic Behaviour

### Sessions 

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
