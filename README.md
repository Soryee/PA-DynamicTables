# Canvas PowerApp Component

This Canvas PowerApp component is designed to efficiently render and interact with data from an unspecified datasource. 

## How it works
```
With({Items:
    Sort(
        Filter(ForAll(ForAll(Split(Substitute(Substitute(JSON(_c.Data),"[{",""),"}]",""),"},{"),
            With({ThisRecord:ThisRecord,ID:GUID()},
            ForAll(Split(Substitute(Value,Char(34),""),","), {
                ID: ID, 
                ColumnName:First(Split(ThisRecord.Value,":")).Value,
                ColumnValue:Last(Split(ThisRecord.Value,":")).Value
                }
            ))),
            ForAll(Value,{
                ID:ThisRecord.ID,
                ColumnName:ThisRecord.ColumnName,
                ColumnValue:ThisRecord.ColumnValue
                })),
                If(!IsBlank(SearchVal), !IsBlank(LookUp(Value As Records,
                If(SearchVal.Type="Text",
                    SearchVal.Value in Records.ColumnValue && Records.ColumnName = SearchVal.Field,
                    Records.ColumnValue=SearchVal.Value&&Records.ColumnName=SearchVal.Field)
                )), true)),
        LookUp(Value,ColumnName=Gallery_Titles.Selected.Value).ColumnValue,
    If(SortDirection,SortOrder.Ascending,SortOrder.Descending))},

    ForAll(Sequence(CountRows(Items)),Patch(Last(FirstN(Items,Value)),{rowNumber:Value})))
```

Step by step: 

1. Assume that we're working with a JSON table [{}].
2. Remove the [] and split on new row },{ in order to create a PowerFX table
   -
   - Substitute(String, $Placeholder, "")
3. Loop through and clean up any remaining unwanted strings via replace
4. Create a nested table that contains each row's column name and column value with a randomized GUID 

```
ForAll(Split(Substitute(Substitute(JSON(_c.Data),"[{",""),"}]",""),"},{"),
    With({ThisRecord:ThisRecord, ID:GUID()},
        ForAll(Split(Substitute(Value,Char(34),""),","), {
                ID: ID, 
                ColumnName: First(Split(ThisRecord.Value,":")).Value,
                ColumnValue: Last(Split(ThisRecord.Value,":")).Value
            }
        )
    )
)
```

At this point we have a table of untyped 'Values', Add a gallery and a gallery within that gallery. 
Top level gallery should use the above ForAll(Split) and the sub level gallery will use ThisItem.Value

You should now have a basic framework to display the datasource 
![image](https://github.com/user-attachments/assets/a5e7502c-d8e1-43ff-ac5d-dee81f850ece)
![image](https://github.com/user-attachments/assets/972601c4-c3c8-4dd8-9e82-f248b1c7d511)


Now adding Filters and Sorting is tricky because we're using untyped data within nested table. 

Add another ForAll around the code, and then add a forall to parse through the values 

![image](https://github.com/user-attachments/assets/88c1a2ba-2d26-4b17-be92-c3f70414e21a)

It should look something like this: 
![image](https://github.com/user-attachments/assets/5fbed78b-434f-4a1a-a951-952a5bc47494)

Now we'll be able to Filter the datasource at the end. Wrap the code in a Filter() then LookUp the Value by the Column you wanted to search and the value searched for.

For Example, I'm searching the Active column by "False"
![image](https://github.com/user-attachments/assets/65bdb37b-f17d-43b5-b887-f44d3abe1e00)

We'll use the same trick for Sorting. Wrap the code in a Sort() and then sort by the LookedUp column's value
![image](https://github.com/user-attachments/assets/826bca70-a210-4728-a1e3-1f8bfaebc488)

