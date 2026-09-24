# DateTable

Date Table is stored in SQL Server ben.dbo.datetable (default schema)

```
CREATE TABLE [dbo].[DateTable](
	[DayIndex] [int] IDENTITY(1,1) NOT NULL,
	[DateValue] [date] NOT NULL,
	[ExcelDateNum] [int] NOT NULL,
	[DaysSinceBirth] [smallint] NOT NULL,
	[Year] [smallint] NOT NULL,
	[YearIndex] [smallint] NOT NULL,
	[DayOfYearNum] [smallint] NOT NULL,
	[QuarterIndex] [smallint] NOT NULL,
	[QuarterNum] [smallint] NOT NULL,
	[DayOfQuarterNum] [smallint] NOT NULL,
	[FortnightIndex] [smallint] NOT NULL,
	[FortnightNum] [smallint] NOT NULL,
	[DayOfFortnightNum] [smallint] NOT NULL,
	[WeekIndex] [smallint] NOT NULL,
	[WeekNum] [smallint] NOT NULL,
	[DayOfWeekNum] [smallint] NOT NULL
) 
GO
```


**DayIndex** :Primary Key that identifies the day.  
**DateValue** : is the date in the format of YYYY-MM-DD.  
**ExcelDateNum** : The value of the date in Excel's numeric format.  
**DaysSinceBirth** : Number of days since birthday. With birthday being 0  

**Year** : An Epic Year is a way of demarcating years based on blocks of weeks (either 52 weeks or 53 weeks) instead of days. See how to get CalenderYear below.  
**YearIndex** : Number of Epic Year since birth year with birth year being 1.  
**DayOfYearNum** : Number of days since the beginning of an Epic Year, with the first day of an Epic Year being 1.  

**QuarterIndex** : Number of Quarters since birthday.  
**QuarterNum** : Number of quarters since the beginning of an Epic Year.  
**DayOfQuarterNum** : Number of days since the beginning of the quarter.

**FortnightIndex** : Number of Fortnight since birthday.  
**FortnightNum** : Number of Fortnight since the beginning of a year.  
**DayOfFortnightNum** : Number of days since the beginning of the fortnight.

**WeekIndex** : Number of week since birthday.  
**WeekNum** : Number of week since the beginning of a year.  
**DayOfWeekNum** : Number of days since the beginning of the week, with the Sunday being 1.




### Calendar year, months and days

Calendar year, months and days can be extracted from the *datevalue* column.
```
SELECT
    YEAR(datevalue) AS CalendarYear,
    MONTH(datevalue) AS MonthNumber,
    DATENAME(MONTH, datevalue) AS MonthName,
    DAY(datevalue) AS Day
FROM datetable;
```
