# Time Tracking and Time Management

## Background
Time tracking is the foundation of all other Quantified Self analytics.

Time tracking data is exported from excel to SQL Server at the end of each year.

```
Server : Legion
Database: Ben
Schema: productivity
```
Table names: timetrackingYYYY

<img width="372" height="82" alt="image" src="https://github.com/user-attachments/assets/788ee3d3-23a1-4470-8a63-94164e5c2a6a" />


## DateTable

Date Table is stored as dbo.datetable.

<img width="416" height="343" alt="image" src="https://github.com/user-attachments/assets/fc3c9360-f653-4019-af5d-499c76f45434" />


**DayID** :Primary Key that identifies the day.  
**DateValue** : is the date in the format of YYYY-MM-DD.  
**ExcelDateNum** : The value of the date in Excel's numeric format.  
**DaysSinceBirth** : Number of days since birthday. With birthday being 0  
**EpYearID** : Number of Epic Year since birth year with birth year being 1.  
**EpYear** : Epic Year is a way of demarcating years based on blocks of weeks (either 52 weeks or 53 weeks) instead of days. See how to get CalenderYear below.  
**EpYearDayNum** : Number of days since the beginning of an Epic Year, with the first day of an Epic Year being 1.  
**QuarterID** : Number of Quarters since birthday.  
**EpYearQuarterNum** : Number of quarters since the beginning of an Epic Year.  
**QuarterDayNum** : Number of days since the beginning of a quarter, with the first day of a quarter being 1.  
**FortnightID** : Number of Quarters since birthday.  
**FortnightDayNum** : Number of days since the beginning of a quarter, with the first day of a quarter being 1.  
**WeekID** : Number of Quarters since birthday.  
**EpYearWeekNum** : Number of quarters since the beginning of an Epic Year.  
**WeekDayNum** : Number of days since the beginning of a quarter, with the first day of a quarter being 1.  


#### Calendar year, months and days
```
SELECT
    YEAR(datevalue) AS CalendarYear,
    MONTH(datevalue) AS MonthNumber,
    DATENAME(MONTH, datevalue) AS MonthName,
    DAY(datevalue) AS Day
FROM datetable;
```

#### Fortnight
```
SELECT
    format(ceiling(EpYearWeekNum/2.0), '00') as EpYearFortnightNum
FROM datetable;
```

note: must divide by 2.0 because EpYearWeekNum is NOT float data type.

### SQL for loading into Power Query
```
select DateValue
, EpYear
, epyearweeknum
, epYearDayNum
, MONTH(datevalue) AS MonthNum
, DATENAME(MONTH, datevalue) AS MonthName
, epyearweeknum as Week_Num
, concat('Week ', epyearweeknum) as Week_Num_Text
, epyearweeknum as Weekday_Number
, format(datevalue, 'ddd') AS Weekday_Text
, format(ceiling(EpYearWeekNum/2.0), '0') as EpYearFortnightNum
, 'Fortnight ' + format(ceiling(EpYearWeekNum/2.0), '0') as Fortnight_text
from [Ben].[dbo].[DateTable]
where EpYear between 2021 and 2026
order by DayID;
```




## Analytics

In Power Query, in order to combine all the time tracking tables from SQL Server and then reorder the activity code columns in ascending order.

```
let
    Source = Sql.Database("legion", "ben"),
    Tables = Table.SelectRows(
        Source,
        each [Kind] = "Table"
            and Text.StartsWith([Name], "productivity.timetracking")
    ),
    Combined = Table.Combine(Tables[Data]),
    ReplacedNulls = Table.ReplaceValue(
        Combined,
        null,
        0,
        Replacer.ReplaceValue,
        Table.ColumnNames(Combined)
    ),
    ReorderedColumns = Table.ReorderColumns(
        ReplacedNulls,
        List.Sort(Table.ColumnNames(ReplacedNulls))
    ),
    #"Reordered Columns" = Table.ReorderColumns(ReorderedColumns,{"datestr", "A010", "A011", "A012", "A020", "A021", "A030", "A040", "A050", "A060", "A061", "A062", "A063", "A064",
                                                                  "A070", "A080", "A081", "A090", "A100", "A101", "A110", "A111", "A112", "A120", "A121", "A122", "A123", "A130", "A140",
                                                                  "A150", "A160", "A170", "A190", "A200", "A210", "A211", "A212", "A213", "A220", "A230", "A240", "A241", "A250", "A251", "A260"})
in
    #"Reordered Columns"
```
