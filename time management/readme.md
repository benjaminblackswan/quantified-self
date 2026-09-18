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
