# Canvas PowerApp Component

This Canvas PowerApp component is designed to efficiently render and interact with data from a specified datasource. With customizable visual styles and selectable interaction modes, it offers a modern and functional UI experience.

## Properties

### Core Data
- **Type:** Input Field
- **Data Type:** Table
- **Description:** The datasource to be rendered within the component.

### UniqueColumnId
- **Type:** Input Field
- **Data Type:** String
- **Description:** Used to filter out the GUID column from the component to ensure it is not visible.

### VisualStyles
- **Type:** Input Field
- **Data Type:** Record
- **Description:** A complex record dictating the visual theme of the component.  

**Example Configuration:**
```javascript
With(
    {
        PrimaryColor: RGBA(59, 89, 152, 1),
        SecondaryColor: RGBA(139, 157, 195, 1),
        AccentColor: RGBA(223, 249, 251, 1),
        BackgroundColor: RGBA(236, 239, 241, 1),
        Font: "Segoe UI",
        FontSize: 12
    },
    {
        Colors: {
            Primary: PrimaryColor,
            Secondary: SecondaryColor,
            Accent: AccentColor,
            Background: BackgroundColor
        },
        HeaderStyle: {
            Font: Font,
            FontColor: PrimaryColor,
            FontSize: FontSize + 2,
            BackgroundColor: BackgroundColor
        },
        LineItemStyle: {
            Font: Font,
            FontColor: RGBA(67, 74, 84, 1),
            FontSize: FontSize,
            BackgroundColor: SecondaryColor,
            BorderColor: AccentColor
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
    ```javascript
    If(CountRows(AppColumns) = 0,
        ClearCollect(AppColumns, SortByColumns(DragColumns.GetColumnNames(), "Position", SortOrder.Ascending))
    );
    ```

- **JSON String Conversion:**
  - Converts `DataCollection` into a compact JSON string:
    ```javascript
        Table(
            { 
                Id: 1,
                Name: "John Doe",
                Age: 30,
                JoinDate: DateValue("2023-01-15"),
                Score: 85.75,
                Active: true,
                Department: "IT",
                Email: "john.doe@example.com",
                'Projects Completed': 5,
                Salary: 60000,
                Remote: false
            },
            { 
                Id: 2,
                Name: "Jane Smith",
                Age: 28,
                JoinDate: DateValue("2022-08-10"),
                Score: 92.5,
                Active: false,
                Department: "HR",
                Email: "jane.smith@example.com",
                'Projects Completed': 7,
                Salary: 65000,
                Remote: true
            }
    );
    ```
    ```javascript
    Set(JsonString, JSON(DataCollection, JSONFormat.Compact)); 
    ```

- **TableBuilder Collection:**
  - Clears the existing `TableBuilder` and processes each JSON object:
    ```javascript
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
    ```javascript
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
