# Canvas PowerApp Component

This Canvas PowerApp component is designed to efficiently render and interact with data from a specified datasource. With customizable visual styles and selectable interaction modes, it offers a modern and functional UI experience.

## Properties

### Core Data
- **Type:** Input Field
- **Data Type:** Table
- **Description:** The datasource to be rendered within the component.

### RowOrCell
- **Type:** Input Field
- **Data Type:** Boolean
- **Description:** Determines whether individual cells within the gallery can be selected.

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
