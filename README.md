# lattice

Declarative validation for tabular data.

Rules read like sentences about a column. Pipelines transform values before
assertions. Row-level and aggregate rules live side by side. A rich
built-in function library covers the common cases, and custom operators
plug in at runtime.

```yaml
version: 1
dataset: orders

columns:
  customer_email:
    type: string
    rules:
      - NOT NULL
      - LEN < 254
      - LOWER -> CONTAINS '@'

  order_total:
    type: float
    rules:
      - '>= 0'
      - $AVG BETWEEN 20 AND 500
      - $FILTER(> 10000) -> COUNT < 100
        code: TOO_MANY_LARGE_ORDERS
        message: "Found {actual} orders over $10K (expected fewer than 100)"

  status:
    type: string
    rules:
      - LOWER -> STRIP -> IN ['pending', 'paid', 'shipped', 'delivered']

  tracking_number:
    type: string
    rules:
      - NOT NULL WHEN status IN ['shipped', 'delivered']
```

## A tour of the language

### Pipelines

The `->` operator chains transforms before a final assertion:

```yaml
- LOWER -> STRIP -> IN ['pending', 'shipped']
- UPPER -> LEN == 2
- LOWER -> MATCHES /^[^@]+@[^@]+\.[a-z]{2,}$/
```

### Aggregates

A `$` prefix marks a rule as column-scoped. The aggregator names how the
column collapses to a single value:

```yaml
- $MIN >= 0
- $AVG BETWEEN 20 AND 500
- $UNIQUE_COUNT == $ROW_COUNT
- $FILTER(> 10000) -> COUNT < 100
```

### Conditional rules

`WHEN` clauses make a rule fire only when a predicate holds:

```yaml
- NOT NULL WHEN status IN ['shipped', 'delivered']
- MATCHES /^[A-Z0-9]{10,30}$/ WHEN NOT NULL
```

### Cross-column rules

Rules that reference more than one column live in their own block:

```yaml
cross_column_rules:
  - shipped_at >= created_at WHEN shipped_at IS NOT NULL
  - refund_amount <= order_total WHEN refund_amount IS NOT NULL
```

### Error codes and messages

```yaml
- LEN < 254
  code: EMAIL_TOO_LONG
  message: "Email '{value}' is {actual} characters (max 254)"
  severity: error

- $FILTER(LOWER -> ENDS_WITH '@test.com') -> COUNT == 0
  code: TEST_EMAILS_IN_PROD
  message: "{actual} test emails found in production data"
  severity: warning
  tags: [data-hygiene]
```

Messages support `{value}`, `{row}`, `{column}`, `{actual}`, and
`{expected}`. When omitted, sensible defaults are generated from the rule.

## Built-in functions

**Math** (35 functions) — `ABS` `ROUND` `ROUNDUP` `ROUNDDOWN` `FLOOR`
`CEILING` `POWER` `SQRT` `MOD` `LOG` `LOG10` `LN` `EXP` `SIN` `COS` `TAN`
`ASIN` `ACOS` `ATAN` `ATAN2` `SINH` `COSH` `TANH` `PI` `RAND`
`RANDBETWEEN` `SIGN` `TRUNC` `INT` `EVEN` `ODD` `GCD` `LCM` `FACT`
`DEGREES` `RADIANS`

**String** (27 functions) — `LEN` `LEFT` `RIGHT` `MID` `TRIM` `CLEAN`
`UPPER` `LOWER` `PROPER` `CONCAT` `REPLACE` `SUBSTITUTE` `REPT` `EXACT`
`FIND` `SEARCH` `TEXT` `VALUE` `NUMBERVALUE` `TEXTBEFORE` `TEXTAFTER`
`CHAR` `CODE` `UNICHAR` `UNICODE` `FIXED` `DOLLAR`

**Logical** (16 functions) — `IF` `AND` `OR` `NOT` `XOR` `IFERROR` `IFNA`
`IFS` `SWITCH` `ISBLANK` `ISNUMBER` `ISTEXT` `ISLOGICAL` `ISERROR`
`ISODD` `ISEVEN`

**Date** (20 functions) — `YEAR` `MONTH` `DAY` `HOUR` `MINUTE` `SECOND`
`TODAY` `NOW` `DATEVALUE` `TIMEVALUE` `DATE` `TIME` `DATEDIF` `DAYS`
`DAYS360` `EDATE` `EOMONTH` `WEEKDAY` `WEEKNUM` `ISOWEEKNUM`

**Statistical aggregates** (14 functions, used with `$`) — `SUM` `AVG`
`MIN` `MAX` `COUNT` `MEDIAN` `MODE` `STDEV` `VAR` `VARIANCE` `PERCENTILE`
`QUARTILE` `ANY` plus aliases

**Array aggregates** (5 functions, used with `$`) — `CUMSUM` `CUMPROD`
`DIFF` `MEAN` `CLIP`


## Extending the language

To be defined.

## Status

The language design is settled; tooling is in progress.

