# Slow Change Automation
This document outlines the process for automating slow changes in our ETL (Extract, Transform, Load) workflows. Slow changes refer to all modifications in all feeature classes Except for the plot, As the plot feature class is updated frequently and requires a Fast change process.

## Overview 
The Slow Change Also checks for the changes only using the rowHash to identify changes in the data. Delete The old non matching rows and insert the new rows.

## Implementation Overview
The automation process is implemented using a main procedure that uses the mapping sheet (It's a configuration file that defines the source and target tables, along with the Fields mapping, values mapping if any) The mapping sheet is converted into a table in the database for better accessibility. After loading the mapping sheet as table in the db we have to make some preparations on it for facilitating the mapping process will be mentioned in mapping schema preparation section below. The main procedure iterates through each mapping entry and creates a dynamic SP to handle this specific feature class changes.

## Mapping Schema Preparation
Before starting the automation process, we need to prepare the mapping schema table. This involves adding necessary columns to facilitate the ETL process. The following columns should be added to the mapping schema table (Called fields table in sql server using sql script):
- ```source_SR_WKID```: The spatial reference WKID of the source data and we will be filled based on the Datasource field value, following the values below:
> [!NOTE] Datasource values and their corresponding WKID
> | Datasource Value | WKID |
> |------------------------|------|
> | Special Economic Zone at Duqm | 32640 |
> | Salalah Free Zone | 32639 |
> | Sohar Free Zone | 32640 |
> | Industrial Estates (MADAYN) | 32639 & 32640 | 
For Madayn it's a special case as it's a combination of both zones, so we need to have an additional info beside the datasource value to distinguish which zone the source data belongs to, this additional info will be the dataset name as it ends with _39 or _40 to indicate the zone.
- ```Is_Foreign_Key```: A boolean column to indicate if the field is a foreign key or not, this will help in the transformation process as foreign fields will be created in the destination using a specific form resulting from concatenating ```zoneid``` and ```zonecode(39, 40)``` as prefix to the field value.
- ```Is_Subtype```: A boolean column to indicate if the field is a subtype field or not to resolve its values during the transformation process, This column will be calculated in the fields table based on the result join between fields and objectclasses table using datasource and subtype field name in objectclasses and datasource and fieldname in fields table.

After adding these columns we need to change the destination tables to have a prefix depending on the zone they belong to (39, 40) as follows:
depending on the source_SR_WKID value we will add either 39 or 40 as suffix to the destination table name.

> [!TIP] The smallest unit until now is the dynamic SP for each feature class. And we will start building that in the next section.

> [!TIP] SP_Sync_Entity Creation Steps.
>
> 1. ***```SP input parameters```***: ```EntityName```: The name of the entity (feature class or regular table) in the destination, ```SourceDB```: The source database name to get the source tables that map to the feature class and they could be multiple tables joined to the destination or just one table, ```Source_SR_WKID```: The spatial reference WKID of the source data to distinguish which zone to go to (39, 40).
> 1. ***```Destination table preparation```***: Ensure the destination table exists and has the necessary Columns for comparison like ```RowHash``` and columns for keeping the source objectID and GlobalID ```(source_objectID, source_GlobalID)```, If not create them.
> 1. ***```Validation:```***: Validate some cases and throw meaningful errors if any of the below cases happen:
>     - Entity name and source DB Exist in the mapping table.
>     - Corresponding source tables to the entity exist in the mapping table.
>     - Not more than one field in the source tables map to the same field in the destination table.
>     - Entity exists in the destination table.
>     - Entity fields exist in the destination table.
>     - No NULL values in the mapping table as long as the field exists in the design (the non existing fields in the design are allowed to have NULL values in the mapping table).
> 1. ***```Select statement creation```***: Create the select statement to get the data from the source tables (the tables will be identified for the entity name from the mapping table making a distinct statement for the objectclassnames that maps to that entity and select the relation table for later use) based on the mapping table and convert their names to the destination field names using (As destination field name), the field in the select statement might be an equation in some cases (like concatenating multiple fields in the source to one field in the destination), in this cse we won't operate on it  and take it as it's in the select statement and put As destination field name after it, for the other fields that are not equations we need to write them in this form ```[SourceTableName].[SourceFieldName] As [DestinationFieldName]```, keep in mind if here multiple tables are used in the select statement we will create each select from each table alone and then concatenate them as strings, one more thing to consider in that the shape field is not in the mapping table so we need to add it manually in the select statement but first we have to check if it's in the destination table or not, if it's there we will add it to the select statement as ```Shape``` with no path to it, but there is a concern about this which is if we have more than one table joined together to the destination table we can't have more than one shape field in the select statement as it will throw an error, so in this case we have to make sure that the shape exists in only one of the source tables the primary one (Mr khater assumption), at the end add a place holder for the rowHash field in the select statement as we will calculate it later after selecting all the fields and concatenating them together, the placeholder will be like this: ```CAST(NULL AS VARCHAR(64)) AS RowHash```.
> 1. ***```From and Join statement creation```***: Create the from and join statements to get the data from the source tables based on the mapping table, we have 2 cases here:
>     - If there is only one source table for the entity, then the from statement will be like this: ```From [SourceDB].[SchemaName].[SourceTableName + suffix name from DBMappingMetaData table if exist]```.
>     - If there are multiple source tables for the entity, then we need to identify the primary table (the one that has the shape field and the primary key that links him with the other source tables) and make it the main table in the from statement and then join the other tables to it using their relation tables identified from the mapping table(will exist in a field named ```RelationshipOriginPrimaryKey``` if it null then this is the primary key if not then this is the name of the primary key and its field name in the same row is the foreign key for this table), so the from and join statements will be like this: ```From [SourceDB].[SchemaName].[PrimarySourceTableName + suffix] Left Join [SourceDB].[SchemaName].[SecondarySourceTableName] On [PrimarySourceTableName].[PrimaryKey] = [RelationTable].[ForeignKey] And [RelationTable].[ForeignKey] = [SecondarySourceTableName].[ForeignKey]```, and so on for more than one secondary table.
> 1. ***```Select all this inside a temp table```***: Select all this inside a temp table to facilitate the next steps, temp table naming convention is as follows: ```#[SourceDB]_[EntityName]_Temp```, i recommend to drop the temp table at the end of the process to avoid temp table accumulation in the tempdb. 
> 1. ***```RowHash calculation```***: Calculate the rowHash using all the rows concatenated together and update the temp table with the calculated rowHash, the calculation will be like this:
> 1. ```sql 
>        CONVERT(VARCHAR(64), HASHBYTES('SHA2_256', CONCAT(
>		    ISNULL(plotNo, ''),
>			ISNULL(CAST(calculatedArea AS NVARCHAR(50)), ''), 
>            ISNULL(CAST(maxBuildingHeightAllowed AS NVARCHAR(50)), ''),
>			ISNULL(CAST(maxNoOfFloorsAllowed AS NVARCHAR(50)), ''), 
>            ISNULL(CAST(OPAZZoneID AS NVARCHAR(50)), ''),
>			ISNULL(mainLanduseEn, ''), ISNULL(detailedLanduseEn, '')
>        )), 1)
>
> 1. keep in mind that the fields has to be casted to NVARCHAR for concatenation, fields used in the rowHash calculation should be all the fields that exist in the source table except the shape as the shape field will be used in the comparison directly to reduce hashing time and effort.
> 1. ***```Data Comparison and Synchronization (Deletion)```***: Compare The RowHash with RowHash in destination and the shape also to monitor any data change and delete the non matching rows in the destination, the comparison result could be achieved as follows:
> 1. ```sql
>  DELETE FROM [destinationSchema].[EntityName]
>  WHERE 
>		RowHash IN (
>		SELECT Dest.RowHash
>				FROM 
>					[destinationSchema].[EntityName] Dest LEFT JOIN [temp_table_name] Source
>				ON 
>					Dest.RowHash COLLATE Arabic_100_CI_AS = Source.RowHash AND Source.Shape.STEquals(Dest.Shape) = 1
>				WHERE 
>				    Source.RowHash IS NULL
>				    AND Dest.OPAZZoneID  = zoneid -- to make sure we are deleting from the correct zone only
>			)
>
> 1. ***```Data Comparison and Synchronization (Insertion)```***: Compare the temp table with the destination table and insert any new rows into the destination table, the insertion can be done as follows:
> 1. ```sql
>  INSERT INTO [destinationSchema].[EntityName] (dest_columns)
>  SELECT columns_from_temp_table (but some values might be transformed during the selection like subtype, type, domain and fields with data type mismatch string to number without domain specified we will use a certain function to resolve this issue)
>  FROM [temp_table_name] Source 
>  LEFT JOIN [destinationSchema].[EntityName] Dest
>  ON Source.RowHash = Dest.RowHash AND Source.Shape.STEquals(Dest.Shape) = 1
>  WHERE Dest.RowHash IS NULL
> 1. ***```dest_columns```***: The columns in the destination table: all the fields from the mapping table that map to the destination table + source_objectID + source_GlobalID + RowHash.
> 1. ***```columns_from_temp_table```***: The columns from the temp table but not straight forward as some values might be transformed during the selection like:
>     - ***Domain fields***: we identify the domain fields from the mapping table using the ```FieldDom_1``` column if it's not null then the value in it is the domain name and we use a function to resolve its value based on the datasource, domain name and the field value this is the function ```dbo.GetNewDomainCode(datasource, domainname, fieldvalue)```.
>     - ***Subtype fields***: we identify the subtype fields from the mapping table using the Is_Subtype column if true then it's a subtype field and we use a function to resolve its value based on the Datasource value this is the function ```dbo.fn_Get_Plot_Mapped_Landuse_Subtype_Code(subtypecode, datasource)```.
>     - ***Type fields***: we still don't identify them maybe we will put a rule on the mapping sheet to put the domain name in the destination field and treat the type as any regular field with domain.
>     - ***Fields with data type mismatch (string to number) without domain specified***: we will use a function to resolve this issue ```dbo.fn_ExtractDigitsAsFloat(fieldvalue)```.
>     - ***ObjectID***: we will generate a new objectID using the maximum objectID in the destination table + 1 as the objectID is an identity field in the destination table as follows: ```DECLARE @MaxObjectID INT; SELECT @MaxObjectID = ISNULL(max(objectid), 0) FROM (SELECT max(objectid) AS objectid FROM [destinationSchema].[EntityName]) AS t;```.
>     - ***GlobalID***: we will generate a new GlobalID using the NEWID() function in SQL Server as follows: ```NEWID()```.
>     - ***source_objectID***: we will use the objectID from the source table.
>     - ***source_GlobalID***: we will use the GlobalID from the source table.
> 1. ***```Cleanup```***: Drop the temp table to avoid accumulation of temp tables in the tempdb.


> > [!NOTE] Mapping value from the source to destination maybe either a string or a number, so one function won't fit all cases, so we need two functions or find solution to handle both cases in one function if possible.