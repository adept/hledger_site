# Mockups

Mockups, draft docs and notes exploring possible future features.

## Price syntax

### In Ledger and hledger

- In the journal, a `P DATE COMMODITY AMOUNT` directive some commodity's market price in some other commodity on DATE.
  (A timestamp may be added, but is ignored.)

- In a posting, `AMT @ UNITPRICE` declares the per-unit price that was used to convert AMT into the price's commodity.
  Eg: `2A @ 3B` records that 2A was posted, in exchange for 6B.

- `@@ TOTALPRICE` is another form of `@`, sometimes more convenient.
  Eg: `2A @@ 5.99B` records that 2A was posted in exchange for 5.99B.

### In Ledger

- `@ UNITPRICE`
  Any use of `@` also generates an implicit `P` directive.
  Eg:

      2019/1/1
        a  2A @ 3B
        b

  in the journal is equivalent to writing

      2019/1/1
        a  2A @ 3B
        b

      P 2019/1/1 A 1.5B

- `{UNITPRICE}`

- `{=FIXEDUNITPRICE}`

The following are variants of the above; they work the same way except
that you write the total instead of the unit price:

- `@@ TOTALPRICE`
- `{{TOTALPRICE}}`
- `{{=FIXEDTOTALPRICE}}`

### In hledger

- `@` does not generate a market price
- `{}` and `{=}` are ignored

## Capital gains

### A model for capital gains

Capital gain/loss (when the value of assets you hold increases/decreases
due to market price fluctuations) - is an important topic, since it can
generate tax liability.

Here is a description of how it works, intended for both users and
builders of accounting software (especially, plain text accounting
software). (I'm a software engineer, not an accountant. In places there
may be better accounting terms I'm not familiar with yet.)

- lots/units -
  A quantity of some commodity, acquired at a certain price on a certain date,
  is called a *lot*, or *unit*. (I'm not sure which is the most standard term. Using lot for now.)

- Since you might have purchased the lot on a stock exchange, received it as a gift,
  or something else, we'll call this event *lot acquisition*, on the *acquisition date*.

- Later you might sell the lot for cash, or exchange it for something else, or gift it.
  We'll call this *lot disposal*.

- You might have paid current market value for the lot, or you might have
  paid less or more than that. We'll call what you paid/exchanged the *acquisition amount*.
  
- I think the acquisition amount is also called the *basis* or *cost basis*.
  Or possibly the current market value is the basis, regardless of what you paid.
  Perhaps it depends. To be clarified. The basis at which you acquired a lot is important.

- After acquisition, while you are still holding the lot, if the market value of that commodity goes up (or down),
  your potential return from disposing of the lot increases (or decreases).
  This is known as *capital gain (or loss)* (we'll just call it "capital gain").
  At this stage, the gain is only "on paper", so it is called *unrealised capital gain* (URG).
  This is not considered revenue, or taxable.

- It's common to be holding multiple lots, perhaps many, even in a single account.
  Eg, say you buy a small amount of some stock or cryptocurrency each week.
  Each purchase adds a new lot to your assets. We'll call this a *multi-lot balance*, or *balance*.

- URG is calculated for a lot at a certain point in time.
  Likewise for a multi-lot balance.

- realised capital gain

- lot withdrawal strategies

- specific identification

### Capital gains in hledger

-  postings can have multiple commodities and multiple prices; each of
   these parts is a deposit or withdrawal to the account

- 
  ```haskell
  -- | Given a list of amounts all in the same commodity, interprets them
  -- as a sequence of lot deposits (the positive amounts) and withdrawals
  -- (the negative amounts), and applies them in order using the FIFO
  -- strategy for withdrawals, then returns the resulting lot balance (as
  -- another, shorter, list of amounts).
  sumLots :: [Amount] -> [Amount]
  ```
## Ease of getting started

What could make getting started substantially easier ?

- Official CI-generated binaries for all major platforms
- Builtin access to docs in web format

## Web docs

Provide the embedded user manuals as HTML also. Eg:

- hledger help --html   # temporary static html files
- hledger help --web    # serve from local hledger-web instance if installed
- hledger help --site   # on hledger.org
- hledger-ui ? h/w/s    # same as above
- hledger-web -> help   # served from hledger-web

## Config file

Name: hledger.conf (and possibly ~/.hledger.conf as well).

- easy to say and spell
- good highlighting support in editors

Format: toml/ini-ish format, but customised for our needs (if necessary).

Example:
```
# hledger.conf

[defaults]
# Set options/arguments to be always used with hledger commands.
# Each line is: HLEDGERCMD ARGS, or: hledger ARGS
hledger -f hledger.journal
bal -M --flat -b lastmonth
ui --watch
web -V
help --html

[commands]
# Define aliases for custom hledger commands.
# Each line is: CMDALIAS = HLEDGERCMD ARGS
assets = bal -M ^assets\b
liab   = bal -M ^liabilities\b

# Or use colon, like make ?
bs2:   bs --no-total date:thisyear

# Or just whitespace, like hledger csv rules ?
smui   ui ^sm\b

# Allow arbitrary shell commands ?
2019:    hledger -f 2019.journal
jstatus: git status -sb -- *.journal

# Allow multi-command shell scripts, with optional help string ?
bsis:
  "Show monthly balance sheet and income statement"
  hledger bs -M
  echo
  hledger is -M
  echo
```

Loaded: 

- at startup
and ideally:
- hledger-web: on each page load if changed, like journals
- hledger-ui --watch: on change, like journals

Location: 

Search a number of locations in order.
Values from multiple files are combined, with later files taking precedence.

User config file: should it be "modern"  ~/.config/hledger.conf or "old/simple" ~/.hledger.conf ? 
One or the other may be preferred/easier/more portable.
If we support both, should it be one or the other, or both ?

Parent directory config files: we'd probably like to recognise config files in parent directories.
How far up should we look - 
to the root dir ? 
to the user's home dir ? and if not under the user's home dir, don't look up at all ?
to the nearest VCS working directory root ?

This would be the simplest comprehensive scheme: use all of

1. ~/.config/hledger.conf
2. ~/.hledger.conf
3. hledger.conf in all directories from / down to the current directory

Eg: running hledger in /home/simon/project/finance would combine any of the following which exist:

- ~/.config/hledger.conf
- ~/.hledger.conf
- /hledger.conf
- /home/hledger.conf
- /home/simon/hledger.conf
- /home/simon/project/hledger.conf
- /home/simon/project/finance/hledger.conf

<style>
.wy-table-responsive { 
  overflow:visible;
}
</style>

## balance

*2021-01-30 draft of improved balance command docs, keeping [#1353](https://github.com/simonmichael/hledger/issues/1353) in mind.*

The `balance` command is a general-purpose report that lists accounts
(all of them, by default) along with the total amounts posted to them
(during the whole journal period, by default). You can use
[query](#queries) arguments or options to limit the report to specific
accounts, a different time period, only cleared transactions, etc.

hledger's balance command is based on Ledger's, and adds
hledger-specific features such as multi-period reports. It is
flexible; use it when you want maximum control. For everyday financial
reporting, however, consider using the following higher-level reports
instead; they are aware of [account types](hledger.html#account-types)
and have convenient defaults, making them easier to use correctly:
[`balancesheet`](hledger.html#balancesheet)/[`balancesheetequity`](hledger.html#balancesheetequity),
[`incomestatement`](hledger.html#incomestatement) and
[`cashflow`](hledger.html#cashflow). These commands also support many
of the balance command's optional features, described below.

As a quick overview, the balance command can show:

- accounts as a flat list or a tree, optionally depth-limited (`-l`, `-t`, `-[1-9]`)
- one time period, or multiple periods (`-D`, `-W`, `-M`, `-Q`, `-Y`, `-p INTERVAL`)
- balance changes in each period (`--change`)
- actual and planned balance changes, and their relative percentage, in each period (`--budget`)
- accumulated totals at the end of each period (counting from report start) (`--cumulative`)
- historical end balances at the end of each period (assuming a suitable opening balances transaction) (`--historical`)
- totals, averages, percentages, inverted sign (`-T`, `-A`, `-%`, `--invert`)
- custom-formatted line items (in single-period reports) (`--format`)
- transposed data - swapping the rows and columns (in multi-period reports) (`--transpose`)
- pivoted data - using a different field as the "account name" (`--pivot FIELD`)

The balance command supports the
[output destination](hledger.html#output-destination) and
[output format](hledger.html#output-format) options.
It supports output formats `txt`, `csv`, `json`, and (multi-period reports only:) `html`. 
In `txt` output in a colour-supporting terminal, negative amounts are shown in red.

<a name="classic-balance-report"></a>

### Single-period balance report

With no arguments, `balance` shows a list of all accounts and their
change of balance - ie, the sum of posting amounts, both inflows and
outflows - during the entire period of the journal.
Accounts are sorted by [declaration order](https://hledger.org/hledger.html#declaring-accounts)
if any, and then alphabetically by account name.
For instance, using [examples/sample.journal](https://github.com/simonmichael/hledger/blob/master/examples/sample.journal):

```shell
$ hledger bal
                  $1  assets:bank:saving
                 $-2  assets:cash
                  $1  expenses:food
                  $1  expenses:supplies
                 $-1  income:gifts
                 $-1  income:salary
                  $1  liabilities:debts
--------------------
                   0  
```

Accounts with a zero balance (and no non-zero subaccounts, in tree
mode - see below) are not shown by default. Use `-E/--empty` to see
them (assets:bank:checking here):
 
```shell
$ hledger -f examples/sample.journal  bal  -E
                   0  assets:bank:checking
                  $1  assets:bank:saving
                 $-2  assets:cash
                  $1  expenses:food
                  $1  expenses:supplies
                 $-1  income:gifts
                 $-1  income:salary
                  $1  liabilities:debts
--------------------
                   0  
```

You can see fewer accounts, a different time period, totals from
cleared transactions only, etc. by using [query](#queries) arguments
or options to limit the postings being matched:

```shell
$ hledger bal --cleared assets date:200806
                 $-2  assets:cash
--------------------
                 $-2  
```

The reported balances' total is shown as the last line, unless `-N`/`--no-total` is used.

### Balance changes or end balances?

Note that the balance command simply sums the amounts of the matched
postings (per account). This means that in general it shows balance
*changes* during the period. That is useful when reviewing income and
expense accounts.

By contrast, when reviewing asset, liability and equity accounts,
you'll usually want to see historically accurate point-in-time end
balances. For this, two things are needed:

1. The report must include all of the account's prior postings -
   eg by leaving the report start date unspecified, or by using
   the `-H/--historical` flag (see below) to include the prior
   postings in the calculation.

2. For accounts whose full history is not recorded in the journal,
   there must be an "opening balances" transaction that accurately
   initialises their balance on some past date (usually the journal's
   start date, often the first of the year).

We'll revisit this in the "All balance report types" section, below.

<!-- examples -->

<a name="tree-mode"></a>

### List or tree mode

By default, or with `-l/--flat`, accounts are shown as a flat list
with their full names visible, as in the examples above.

With `-t/--tree`, the account hierarchy is shown, with subaccounts'
"leaf" names indented below their parent:

```shell
$ hledger balance
                 $-1  assets
                  $1    bank:saving
                 $-2    cash
                  $2  expenses
                  $1    food
                  $1    supplies
                 $-2  income
                 $-1    gifts
                 $-1    salary
                  $1  liabilities:debts
--------------------
                   0
```

Also:

- "Boring" accounts are combined with their subaccount for more
compact output, unless `--no-elide` is used. Boring accounts have no
balance of their own and just one subaccount (eg `assets:bank` and
`liabilities` above).

- All balances shown are "inclusive", ie including the balances from
all subaccounts. Note this means some repetition in the output, which
requires explanation when sharing reports with
non-plaintextaccounting-users. A tree mode report's final total is the
sum of the top-level balances shown, not of all the balances shown.

- Each group of sibling accounts (ie, under a common parent) is sorted separately.

### Depth limiting

With a `depth:N` query, or `--depth N` option, or just `-N`, 
balance reports will show accounts only to the specified depth,
hiding the deeper subaccounts. 
Account balances at the depth limit always include the balances from
any hidden subaccounts (even in list mode). 
This can be useful for getting an overview. Eg, limiting to depth 1:

```shell
$ hledger balance -N -1
                 $-1  assets
                  $2  expenses
                 $-2  income
                  $1  liabilities
```

You can also hide top-level account name parts, using `--drop N`.
This can be useful for hiding repetitive top-level account names:

```shell
$ hledger bal expenses --drop 1
                  $1  food
                  $1  supplies
--------------------
                  $2  
```

<a name="multicolumn-balance-report"></a>

### Multi-period balance report

With a [report interval](#report-intervals) (set by the `-D/--daily`,
`-W/--weekly`, `-M/--monthly`, `-Q/--quarterly`, `-Y/--yearly`, or
`-p/--period` flag), `balance` shows a tabular report, with columns
representing successive time periods (and a title). (The higher-level
`balancesheet`/`incomestatement`/`cashflow` commands always use this
tabular report style, even for a single period.)

```shell
$ hledger balance --quarterly income expenses -E
Balance changes in 2008:

                   ||  2008q1  2008q2  2008q3  2008q4 
===================++=================================
 expenses:food     ||       0      $1       0       0 
 expenses:supplies ||       0      $1       0       0 
 income:gifts      ||       0     $-1       0       0 
 income:salary     ||     $-1       0       0       0 
-------------------++---------------------------------
                   ||     $-1      $1       0       0 

```

Also:

- The report's start/end dates will be expanded, if necessary, to fully
encompass the displayed subperiods (so that the first and last
subperiods have the same duration as the others).

- Leading and trailing periods (columns) containing all zeroes are not
shown, unless `-E/--empty` is used.

- Accounts (rows) containing all zeroes are not shown, unless
`-E/--empty` is used.

- Amounts with many commodities are shown in abbreviated form, unless
`--no-elide` is used. *experimental*

- Average and/or total columns can be added with the `-A/--average` and
`-T/--row-total` flags.

- The `--transpose` flag can be used to exchange rows and columns.
     
<!--
```shell
$ hledger balance income expenses -tQETA
Balance changes in 2008:

            ||  2008q1  2008q2  2008q3  2008q4    Total  Average 
============++===================================================
 expenses   ||       0      $2       0       0       $2       $1 
   food     ||       0      $1       0       0       $1        0 
   supplies ||       0      $1       0       0       $1        0 
 income     ||     $-1     $-1       0       0      $-2      $-1 
   gifts    ||       0     $-1       0       0      $-1        0 
   salary   ||     $-1       0       0       0      $-1        0 
------------++---------------------------------------------------
            ||     $-1      $1       0       0        0        0 

(Average is rounded to the dollar here since all journal amounts are)
```
-->

Multi-period reports with many periods can be wide.
Converting to a single currency with `-V`, or hiding the totals row
with `-N/--no-total`, are some ways to reduce the width.
When reports are still too wide for comfortable viewing, here are some tips:

- Maximize terminal, reduce font size
- View with a pager like less, eg: `hledger bal -D --color=yes | less -RS`
- Output as CSV and view with [visidata]: `hledger bal -D -O csv | vd -f csv`\
  or with a spreadsheet: `hledger bal -D -o a.csv && open a.csv`
- Output as HTML and view with a browser: `hledger bal -D -o a.html && open a.html`

[visidata]: https://www.visidata.org

### Balance report types

As mentioned in the "Balance changes or end balances?" section above,
it's important to be clear on the meaning of the numbers being shown.
Are they Balance change in period ? Or Historical balance at period
end ? With multi-period balance reports, there is a third possibility:
Balance change since report start. And a fourth variant, the budget
report, is described below.

The `balance` command can show any of these variants, selected by a
flag, with `--change` being the default:

| Balance report type       | Shows, for each account and period:                                                                                               |
|---------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| `balance [--change]`      | "Balance changes": the change of balance in each period.                                                                          |
| `balance --cumulative`    | "Cumulative balances": the change accumulated from report start to each period's end.                                             |
| `balance --historical/-H` | "Historical balances": the change accumulated from journal start to each period's end (usually including an opening balance txn). |
| `balance --budget`        | "Actual and target balance changes": like --change, but also shows a goal in each period.                                         |

Typically,
`--change` is used when reviewing revenues/expenses,
`--historical` is used when reviewing assets/liabilities/equity,
`--budget` is used for comparing revenues/expenses with budget goals, and
`--cumulative` is rarely used.

Note `--row-total/-T` is disabled with `--cumulative` or `--historical`,
since summing already-summed end balances usually does not make sense.

<!--
```shell
$ hledger balance --quarterly income expenses -E --cumulative
Ending balances (cumulative) in 2008:

                   ||  2008/03/31  2008/06/30  2008/09/30  2008/12/31 
===================++=================================================
 expenses:food     ||           0          $1          $1          $1 
 expenses:supplies ||           0          $1          $1          $1 
 income:gifts      ||           0         $-1         $-1         $-1 
 income:salary     ||         $-1         $-1         $-1         $-1 
-------------------++-------------------------------------------------
                   ||         $-1           0           0           0 

```
```shell
$ hledger balance ^assets ^liabilities --quarterly --historical --begin 2008/4/1
Ending balances (historical) in 2008/04/01-2008/12/31:

                      ||  2008/06/30  2008/09/30  2008/12/31 
======================++=====================================
 assets:bank:checking ||          $1          $1           0 
 assets:bank:saving   ||          $1          $1          $1 
 assets:cash          ||         $-2         $-2         $-2 
 liabilities:debts    ||           0           0          $1 
----------------------++-------------------------------------
                      ||           0           0           0 

```
-->

### Balance report valuation

When [valuation] is enabled (via `-V`, `-X`, or `--value`),
here is what the [balance report types](#balance-report-types) show 
for each [valuation type](hledger.html#value-flexible-valuation):

[valuation]: hledger.html#valuation

|                           | (no valuation)                          | `--value=then`                                                     | `--value=end`                                                                                                       | `--value=DATE/now`                                    |
|---------------------------|-----------------------------------------|--------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------|
| `balance [--change]`      | change in period                        | sum of posting-date market values in period                        | **1.20.4:** period-end value of change in period                        <br>**master:** change of period-end values | DATE-value of change in period                        |
| `balance --cumulative`    | change from report start to period end  | sum of posting-date market values from report start to period end  | **1.20.4:** period-end value of change from report start to period end  <br>**master:** change of period-end values | DATE-value of change from report start to period end  |
| `balance --historical/-H` | change from journal start to period end | sum of posting-date market values from journal start to period end | **1.20.4:** period-end value of change from journal start to period end <br>**master:** change of period-end values | DATE-value of change from journal start to period end |
| `balance --budget`        | like --change, plus budget goals        | "                                                                  | "                                                                                                                   | "                                                     |

### Sorting by amount

With `-S/--sort-amount`, accounts with the largest (most positive) balances are shown first.
Eg: `hledger bal expenses -MAS` shows your biggest averaged monthly expenses first.

Revenues and liability balances are typically negative, however, so `-S` shows these in reverse order.
To work around this, you can add `--invert` to flip the signs.
(Or, use one of the higher-level reports, which flip the sign automatically. Eg: `hledger incomestatement -MAS`).

### Percentages

With `-%/--percent`, balance reports show each account's value
expressed as a percentage of the (column) total:

```shell
$ hledger bal expenses -Q -%
Balance changes in 2008:

                   || 2008Q1   2008Q2  2008Q3  2008Q4 
===================++=================================
 expenses:food     ||      0   50.0 %       0       0 
 expenses:supplies ||      0   50.0 %       0       0 
-------------------++---------------------------------
                   ||      0  100.0 %       0       0 
```

Note it is not useful to calculate percentages if the amounts in a
column have mixed signs. In this case, make a separate report for each
sign, eg:

```shell
$ hledger bal -% amt:`>0`
$ hledger bal -% amt:`<0`
```

Similarly, if the amounts in a column have mixed commodities, convert
them to one commodity with `-B`, `-V`, `-X` or `--value`, or make a
separate report for each commodity:

```shell
$ hledger bal -% cur:\\$
$ hledger bal -% cur:€
```

### Customising single-period balance reports

Single-period balance reports can be customised by using `--format FMT`
to set the format of each line. Eg:

```shell
$ hledger balance --format "%20(account) %12(total)"
              assets          $-1
         bank:saving           $1
                cash          $-2
            expenses           $2
                food           $1
            supplies           $1
              income          $-2
               gifts          $-1
              salary          $-1
   liabilities:debts           $1
---------------------------------
                                0
```

The FMT format string (plus a newline) specifies the formatting
applied to each account/balance pair. It may contain any suitable
text, with data fields interpolated like so:

`%[MIN][.MAX](FIELDNAME)`

- MIN pads with spaces to at least this width (optional)
- MAX truncates at this width (optional)
- FIELDNAME must be enclosed in parentheses, and can be one of:

    - `depth_spacer` - a number of spaces equal to the account's depth, or if MIN is specified, MIN * depth spaces.
    - `account`      - the account's name
    - `total`        - the account's balance/posted total, right justified

Also, FMT can begin with an optional prefix to control how
multi-commodity amounts are rendered:

- `%_` - render on multiple lines, bottom-aligned (the default)
- `%^` - render on multiple lines, top-aligned
- `%,` - render on one line, comma-separated

There are some quirks. Eg in one-line mode, `%(depth_spacer)` has no
effect, instead `%(account)` has indentation built in.
<!-- XXX retest:
Consistent column widths are not well enforced, causing ragged edges unless you set suitable widths.
Beware of specifying a maximum width; it will clip account names and amounts that are too wide, with no visible indication.
-->
Experimentation may be needed to get pleasing results.

Some example formats:

- `%(total)`         - the account's total
- `%-20.20(account)` - the account's name, left justified, padded to 20 characters and clipped at 20 characters
- `%,%-50(account)  %25(total)` - account name padded to 50 characters, total padded to 20 characters, with multiple commodities rendered on one line
- `%20(total)  %2(depth_spacer)%-(account)` - the default format for the single-column balance report

### Budget report

There is also a special balance report mode for showing budget performance.
The `--budget` flag activates extra columns showing the budget goals for each account and period, if any.
For this report, budget goals are defined by [periodic transactions](journal.html#periodic-transactions).
This is very useful for comparing planned and actual income, expenses, time usage, etc.

For example, you can take average monthly expenses in the common expense categories to construct a minimal monthly budget:
```journal
;; Budget
~ monthly
  income  $2000
  expenses:food    $400
  expenses:bus     $50
  expenses:movies  $30
  assets:bank:checking

;; Two months worth of expenses
2017-11-01
  income  $1950
  expenses:food    $396
  expenses:bus     $49
  expenses:movies  $30
  expenses:supplies  $20
  assets:bank:checking

2017-12-01
  income  $2100
  expenses:food    $412
  expenses:bus     $53
  expenses:gifts   $100
  assets:bank:checking
```

You can now see a monthly budget report:
```shell
$ hledger balance -M --budget
Budget performance in 2017/11/01-2017/12/31:

                      ||                      Nov                       Dec 
======================++====================================================
 assets               || $-2445 [  99% of $-2480]  $-2665 [ 107% of $-2480] 
 assets:bank          || $-2445 [  99% of $-2480]  $-2665 [ 107% of $-2480] 
 assets:bank:checking || $-2445 [  99% of $-2480]  $-2665 [ 107% of $-2480] 
 expenses             ||   $495 [ 103% of   $480]    $565 [ 118% of   $480] 
 expenses:bus         ||    $49 [  98% of    $50]     $53 [ 106% of    $50] 
 expenses:food        ||   $396 [  99% of   $400]    $412 [ 103% of   $400] 
 expenses:movies      ||    $30 [ 100% of    $30]       0 [   0% of    $30] 
 income               ||  $1950 [  98% of  $2000]   $2100 [ 105% of  $2000] 
----------------------++----------------------------------------------------
                      ||      0 [              0]       0 [              0] 
```

This is different from a normal balance report in several ways:

- Only accounts with budget goals during the report period are shown, by default.

- In each column, in square brackets after the actual amount, 
  budget goal amounts are shown, and the actual/goal percentage.
  (Note: budget goals should be in the same commodity as the actual amount.)

- All parent accounts are always shown, even in list mode. 
  Eg assets, assets:bank, and expenses above.

- Amounts always include all subaccounts, budgeted or unbudgeted, even in list mode.

This means that the numbers displayed will not always add up!
Eg above, the `expenses`  actual amount includes the gifts and supplies transactions,
but the `expenses:gifts` and `expenses:supplies` accounts are not
shown, as they have no budget amounts declared.

This can be confusing. When you need to make things clearer, use the `-E/--empty` flag, 
which will reveal all accounts including unbudgeted ones, giving the full picture. Eg:

```shell
$ hledger balance -M --budget --empty
Budget performance in 2017/11/01-2017/12/31:

                      ||                      Nov                       Dec 
======================++====================================================
 assets               || $-2445 [  99% of $-2480]  $-2665 [ 107% of $-2480] 
 assets:bank          || $-2445 [  99% of $-2480]  $-2665 [ 107% of $-2480] 
 assets:bank:checking || $-2445 [  99% of $-2480]  $-2665 [ 107% of $-2480] 
 expenses             ||   $495 [ 103% of   $480]    $565 [ 118% of   $480] 
 expenses:bus         ||    $49 [  98% of    $50]     $53 [ 106% of    $50] 
 expenses:food        ||   $396 [  99% of   $400]    $412 [ 103% of   $400] 
 expenses:gifts       ||      0                      $100                   
 expenses:movies      ||    $30 [ 100% of    $30]       0 [   0% of    $30] 
 expenses:supplies    ||    $20                         0                   
 income               ||  $1950 [  98% of  $2000]   $2100 [ 105% of  $2000] 
----------------------++----------------------------------------------------
                      ||      0 [              0]       0 [              0] 
```


You can roll over unspent budgets to next period with `--cumulative`:
```shell
$ hledger balance -M --budget --cumulative
Budget performance in 2017/11/01-2017/12/31:

                      ||                      Nov                       Dec 
======================++====================================================
 assets               || $-2445 [  99% of $-2480]  $-5110 [ 103% of $-4960] 
 assets:bank          || $-2445 [  99% of $-2480]  $-5110 [ 103% of $-4960] 
 assets:bank:checking || $-2445 [  99% of $-2480]  $-5110 [ 103% of $-4960] 
 expenses             ||   $495 [ 103% of   $480]   $1060 [ 110% of   $960] 
 expenses:bus         ||    $49 [  98% of    $50]    $102 [ 102% of   $100] 
 expenses:food        ||   $396 [  99% of   $400]    $808 [ 101% of   $800] 
 expenses:movies      ||    $30 [ 100% of    $30]     $30 [  50% of    $60] 
 income               ||  $1950 [  98% of  $2000]   $4050 [ 101% of  $4000] 
----------------------++----------------------------------------------------
                      ||      0 [              0]       0 [              0] 
```

For more examples and notes, see [Budgeting](budgeting.html).

#### Budget report start date

This might be a bug, but for now:
when making budget reports, it's a good idea to explicitly set the
report's start date to the first day of a reporting period, because
a periodic rule like `~ monthly` generates its transactions on the 1st
of each month, and if your journal has no regular transactions on the 1st,
the default report start date could exclude that budget goal, which can
be a little surprising. Eg here the default report period is just the
day of 2020-01-15:

```journal
~ monthly in 2020
  (expenses:food)  $500

2020-01-15
  expenses:food    $400
  assets:checking
```
```shell
$ hledger bal expenses --budget
Budget performance in 2020-01-15:

              || 2020-01-15 
==============++============
 <unbudgeted> ||       $400 
--------------++------------
              ||       $400 
```

To avoid this, specify the budget report's period, or at least the start date,
with `-b`/`-e`/`-p`/`date:`, to ensure it includes the budget goal transactions
(periodic transactions) that you want. Eg, adding `-b 2020/1/1` to the above:

```shell
$ hledger bal expenses --budget -b 2020/1/1
Budget performance in 2020-01-01..2020-01-15:

               || 2020-01-01..2020-01-15 
===============++========================
 expenses:food ||     $400 [80% of $500] 
---------------++------------------------
               ||     $400 [80% of $500] 
```

#### Nested budgets

You can add budgets to any account in your account hierarchy. If you have budgets on both parent account and some of its children, then budget(s)
of the child account(s) would be added to the budget of their parent, much like account balances behave.

In the most simple case this means that once you add a budget to any account, all its parents would have budget as well. 

To illustrate this, consider the following budget:
```
~ monthly from 2019/01
    expenses:personal             $1,000.00
    expenses:personal:electronics    $100.00
    liabilities
```

With this, monthly budget for electronics is defined to be $100 and budget for personal expenses is an additional $1000, which implicitly means
that budget for both `expenses:personal` and `expenses` is $1100.

Transactions in `expenses:personal:electronics` will be counted both towards its $100 budget and $1100 of `expenses:personal` , and transactions in any other subaccount of `expenses:personal` would be
counted towards only towards the budget of `expenses:personal`.

For example, let's consider these transactions:
```journal
~ monthly from 2019/01
    expenses:personal             $1,000.00
    expenses:personal:electronics    $100.00
    liabilities

2019/01/01 Google home hub
    expenses:personal:electronics          $90.00
    liabilities                           $-90.00

2019/01/02 Phone screen protector
    expenses:personal:electronics:upgrades          $10.00
    liabilities

2019/01/02 Weekly train ticket
    expenses:personal:train tickets       $153.00
    liabilities

2019/01/03 Flowers
    expenses:personal          $30.00
    liabilities
```

As you can see, we have transactions in `expenses:personal:electronics:upgrades` and `expenses:personal:train tickets`, and since both of these accounts are without explicitly defined budget,
these transactions would be counted towards budgets of `expenses:personal:electronics` and `expenses:personal` accordingly:

```shell
$ hledger balance --budget -M
Budget performance in 2019/01:

                               ||                           Jan 
===============================++===============================
 expenses                      ||  $283.00 [  26% of  $1100.00] 
 expenses:personal             ||  $283.00 [  26% of  $1100.00] 
 expenses:personal:electronics ||  $100.00 [ 100% of   $100.00] 
 liabilities                   || $-283.00 [  26% of $-1100.00] 
-------------------------------++-------------------------------
                               ||        0 [                 0] 
```

And with `--empty`, we can get a better picture of budget allocation and consumption:
```shell
$ hledger balance --budget -M --empty
Budget performance in 2019/01:

                                        ||                           Jan 
========================================++===============================
 expenses                               ||  $283.00 [  26% of  $1100.00] 
 expenses:personal                      ||  $283.00 [  26% of  $1100.00] 
 expenses:personal:electronics          ||  $100.00 [ 100% of   $100.00] 
 expenses:personal:electronics:upgrades ||   $10.00                      
 expenses:personal:train tickets        ||  $153.00                      
 liabilities                            || $-283.00 [  26% of $-1100.00] 
----------------------------------------++-------------------------------
                                        ||        0 [                 0] 
```


## Upgrade notes

*Cf [#1353](https://github.com/simonmichael/hledger/issues/1353)*

User-visible changes when going from 1.20.4 to master:

|                            |                                                                                                                                                              |
|----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `-B/--cost`                | Now a primary flag.                                                                                                                                          |
| `--value=cost`             | Now an alias for `-B/--cost`, and deprecated.                                                                                                                |
| `--value=cost,COMM`        | No longer supported, suggests `-B --value=X,COMM`.                                                                                                           |
| `--value=end`              | With `--change`, shows change of end values instead of end value of change.<br>`--value=then` approximates and hopefully is preferable to the old behaviour. |

Meaning of the cost/valuation short flags in master:

| Short flag            | Equivalent to              |
|-----------------------|----------------------------|
| `-B`                  | `--cost`                   |
| `-V`                  | `--value=then` (soon)      |
| `-X/--exchange  COMM` | `--value=then,COMM` (soon) |

## Valuation examples

Minimal example for testing some valuation behaviours discussed in
[#1353](https://github.com/simonmichael/hledger/issues/1353).
See [Balance report valuation](#balance-report-valuation) above.

```journal
; every ~15 days: one A is purchased, and A's market price in B increases.

2020-01-01
  (a)  1 A

2020-01-15
  (a)  1 A

2020-02-01
  (a)  1 A

2020-02-15
  (a)  1 A

P 2020-01-01 A  1 B
P 2020-01-15 A  2 B
P 2020-02-01 A  3 B
P 2020-02-15 A  4 B
```

Old `balance --change --value=end` behaviour: shows period-end value of period's balance change:

```shell
$ hledger-1.20.4 bal -M --value=end  # --change is the default
Balance changes in 2020-01-01..2020-02-29, valued at period ends:

   || Jan  Feb 
===++==========
 a || 4 B  8 B 
---++----------
   || 4 B  8 B 
```

New `balance --change --value=end` behaviour in master: shows change between period-end-valued period-end balances:

```shell
$ hledger-master bal -M --value=end
Period-end value changes in 2020-01-01..2020-02-29:

   || Jan   Feb 
===++===========
 a || 4 B  12 B 
---++-----------
   || 4 B  12 B 
```

`balance --value=then` is also supported in master: shows sum of postings' then-values in each period:

```shell
$ hledger-master bal -M --value=then
Balance changes in 2020-01-01..2020-02-29, valued at posting date:

   || Jan  Feb 
===++==========
 a || 3 B  7 B 
---++----------
   || 3 B  7 B 
```
