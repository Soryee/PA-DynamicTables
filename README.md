# Canvas PowerApp Component

This Canvas PowerApp component is designed to efficiently render and interact with data from an unspecified datasource. 

## Properties

### Core Data
- **Type:** Input Field
- **Data Type:** Table, see example below.
```php
Table(
    { 
        Id: 1, Name: "John Doe", Age: 30,
        JoinDate: DateValue("2023-01-15"),
        Score: 85.75, Active: true, Department: "IT",
        Email: "john.doe@example.com", 'Projects Completed': 5,
        Salary: 60000, Remote: false
    },
    { 
        Id: 2, Name: "Jane Smith", Age: 28,
        JoinDate: DateValue("2022-08-10"),
        Score: 92.5, Active: false, Department: "HR",
        Email: "jane.smith@example.com", 'Projects Completed': 7,
        Salary: 65000, Remote: true
    }
);
```
- **Description:** The datasource to be rendered within the component.

### UniqueColumnId
- **Type:** Input Field
- **Data Type:** String
```php
"Id"
```

- **Description:** Used to filter out the GUID column from the component to ensure it is not visible.

### VisualStyles
- **Type:** Input Field
- **Data Type:** Record
- **Description:** A complex record dictating the visual theme of the component.  

**Example Configuration:**
```php
With(
    {
        BaseColor: RGBA(51, 51, 51, 1),
        BaseBackgroundColor: RGBA(245, 245, 245, 1),
        AltColor: RGBA(115, 115, 115, 1),
        AltBackgroundColor: RGBA(255, 255, 255, 1),
        Typeface: "Segoe UI", 
        BaseFontSize: 12
    }, 
    {
        HeaderStyle: {
            Font: Typeface,
            FontColor: BaseColor, 
            FontSize: BaseFontSize + 2, 
            BackgroundColor: BaseBackgroundColor
        },
        LineItemStyle: {
            Font: Typeface,
            FontColor: BaseColor, 
            AltFontColor: AltColor,
            FontSize: BaseFontSize,
            BackgroundColor: BaseBackgroundColor,
            AltBackgroundColor: AltBackgroundColor
        }
    }
)

```
## Events

## CreateLocalCollection Event

The `CreateLocalCollection` function is designed to manage and manipulate data collections within the component. It initializes and organizes data for subsequent use and interaction.

### Function Details

- **Initialization:**
  - Resets the `SelectedItem` to an empty object: `Set(SelectedItem, {});`

- **Handling AppColumns:**
  - Checks if the `AppColumns` collection is empty, and if so, populates it with column names sorted by their position: 
    ```less
    If(CountRows(AppColumns) = 0,
        ClearCollect(AppColumns, 
            SortByColumns(DragColumns.GetColumnNames(), "Position", SortOrder.Ascending))
    );
    ```
      - ```DragColumns.GetColumnNames()``` is an internal function to provide the column names from the first row of the data source. See below.

    ```less
    With({
        LocalData: ForAll(
            Sequence(CountRows(Substitute(Substitute(MatchAll(JSON(First(DataCollection)), "(?<!\w):\s*([^{},]+).?").FullMatch, ":", Blank()), Char(34), Blank()))),
            {
                Value: Index(Substitute(Substitute(MatchAll(JSON(First(DataCollection)), ".(\s*\w+(?:\s+\w+)*)\s*.:").FullMatch, ":", Blank()), Char(34), Blank()), Value).Value,
                Position: Value,
                Id: Value, 
                GUID: If(Index(Substitute(Substitute(MatchAll(JSON(First(DataCollection)), ".(\s*\w+(?:\s+\w+)*)\s*.:").FullMatch, ":", Blank()), Char(34), Blank()), Value).Value = DragColumns.UniqueColumnId, true, false)
            }
        )}, 
        With({GUIDPosition: LookUp(LocalData, GUID)}, 
            ForAll(LocalData As Data,
                If(Data.Position > GUIDPosition.Position, 
                    {Value: Data.Value, Position: Data.Position-1, Id: Data.Id, GUID: Data.GUID}, 
                    If(Data.GUID, 
                        {Value: Data.Value, Position: -1, Id: Data.Id, GUID: Data.GUID}, 
                        {Value: Data.Value, Position: Data.Position, Id: Data.Id, GUID: Data.GUID}
                    )
                )
            )
        )
    )
    ```

- **JSON String Conversion:**
  - Converts `DataCollection` into a compact JSON string:
    ```less
    Set(JsonString, JSON(DataCollection, JSONFormat.Compact)); 
    ```

- **TableBuilder Collection:**
  - Clears the existing `TableBuilder` and processes each JSON object:
    ```less
    Clear(TableBuilder);

    ForAll(MatchAll(JsonString, "{([^{}]+)}"),
        Collect(TableBuilder, {
            FullObject: FullMatch,
            KeyValues: MatchAll(FullMatch, ".(\w+).\s*:\s*([^,{}]+).")
        })
    );
    ```

- **Record Parsing and Sorting:**
  - Processes each parsed JSON record to extract and sort key names and values based on their positions:
    ```less
    ClearCollect(TableData,
        ForAll(TableBuilder,
        SortByColumns(
            ForAll(Substitute(Substitute(Substitute(ThisRecord.KeyValues.FullMatch, "}", ""), ",", ""), Char(34), ""), 
                {
                    Name: First(Split(ThisRecord.Value, ":")).Value,
                    Value: Last(Split(ThisRecord.Value, ":")).Value, 
                    Position: LookUp(AppColumns As Columns, Columns.Value = First(Split(ThisRecord.Value, ":")).Value).Position
                }
        ), "Position", SortOrder.Ascending)));
    ```

This function ensures that data is structured and readily available for rendering and interactions within the PowerApp component.



### OnSelect
- **Type:** Output
- **Data Type:** Record
- **Description:** Captures the selected item from the gallery.
