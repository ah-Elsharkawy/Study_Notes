# Required Rules for the Mapping Sheet
Any violation of these rules will result in either errors while mapping or incorrect data in the destination database.

> [!TIP] Rules
>
> 1. **Unique Field Mapping**: 
No more than one source field may map to the same destination field.
> 1. **Expressions**: 
> >  - The expression should be included inside brackets ```(Expression)```.
> >  - Any field name should be preceded by the source table name with dot notation, e.g., ```(TableName.FieldName + '/' + TableName.AnotherFieldName)```.
> > - If multifields are being included in the expression no need to repeat the expression with each field, just include it once with all the fields.
> > - The expressions should only be applied to the source fields, not the destination fields. 
> 1. **Source Tables Mapping to Destination Exist**: 
At least one source table (or group of tables) must be defined to map to the destination entity.
> 1. **Destination Entity Exists**: 
The specified destination entity (table/feature class) must already exist in the destination database schema.
> 1. **Entity Fields Exist in Destination Table**: 
All destination fields defined in the mapping must exist as columns in the destination table.
> 1. **No Empty or Null Mapping Fields**: 
Source fields, destination fields, and related mapping information must not be null or empty as long as they are in the design.
> 1. **Primary Key Mapping**: 
For any destination entity that relies on a join between source tables, the related primary key field must be defined in the `RelationshipOriginPrimaryKey` column within Fields Mapping. In addition, the `RelationshipOriginObjectClass` column must specify the primary table name used in the join.
> 1. **Destination ID References**: 
Destination fields that reference the ID field as their primary key must be marked as Yes in the `ReferenceDestinationID` column in Fields Mapping.
> 1. **Subtype Field Names**: 
For any field that uses subtypes (e.g., detailedLanduse) and whose subtype values must be considered during mapping, the corresponding source subtype field name must be specified in the `SubtypeField` column in the Fields Mapping table.
> 1. **Restrict Source Databases**: 
The mapping sheet should only include source databases that are part of our ETL process. Any irrelevant databases should be removed to avoid confusion and incorrect mappings.
> 1. **Where Clause for Filtering**: 
If a source table requires filtering, the Where Clause column must contain a valid SQL WHERE clause with the following rules:
> >  - The clause should not include the `WHERE` keyword itself; it should be just the condition inside brackets, e.g., `(condition)`.
> >  - Only one condition is allowed per sourceEntityGroup.
> 1. **OpazZoneID shoudn't be represented in the mapping**: 
Mapping for opazZoneID field is populated during ETL process it shouldn't be represented in the mapping sheet.
> 1. **Custom Source Groups**: 
> > - When a source table must be mapped multiple times to the same destination table (for example, when pivoting a single source row into multiple destination rows), a distinct custom group name must be specified in the Group field of the mapping. This group name should clearly indicate which subset of fields is associated with each specific mapping instance.
> > - Also when multiple different source mapped to the same destination and those sources are joining the same source parent then "Custom Group" must be set



# Required Rules for Source databases
Any violation of these rules will result in either errors while mapping or incorrect data in the destination database

> [!TIP] Rules
> 1. **GlobalID should be enabled if there is attachment in the source**
> 1. **Source View**:
If archiving or versioning is enabled in the source database, a corresponding view for the feature class or table must be created using the appropriate ArcGIS tools.
> 1. **Fast Change Feature Class View**:
If the source feature class is registered as versioned or has archiving enabled, a fast change feature class view must be created for it, using the appropriate ArcGIS tools.


