# Test note — Assistant write test

Created: 2025-11-03

This note was appended by the assistant to test writing into your vault.

# ETL Tool Overview (Slow Change Flow)

## Purpose

This ETL tool transfers data from multiple source databases (each with different structures) into a single, unified destination database. The mapping rules for how data is moved and transformed are defined in an Excel file, making the process flexible and easy to update.

## Mapping Rules

The Excel file contains 4 sheets:

- **Object Classes:** Defines which source tables map to which destination tables.
- **Fields:** Specifies how individual fields (columns) are mapped from source to destination.
- **Domains:** Lists domain mappings (like enums or categories).
- **Mappings (Domains | Subtypes | Values):** Details how specific source values are converted to destination values.

## Slow Change Flow

1. **Mapping Preparation**
    
    - The tool reads the Excel file and builds two key tables:
        - **FieldsMapping:** Contains all table and field mapping rules.
        - **ValuesMapping:** Contains all value and domain transformation rules.
    - This is done using stored procedure `SP_ETL_Mapping_Tables_Preparation`.
    - Preparation is done once 
2. **Job Execution**
    
    - ETL jobs are managed in a table called `ETL_Jobs`.
    - When a new job is added, a trigger automatically starts the ETL process.
    - The main procedure (`SP_Sync_Entity`) uses the mapping tables to move and transform data from the source to the destination database.
3. **Job Management and Monitoring**
    
    - Each job is tracked, and its progress and results are logged.
    - Errors and successes are recorded for review.

## User Interface

The tool provides four main pages:

1. **Jobs**
    
    - Add new ETL jobs.
    - When a job is added, the ETL process starts automatically.
    - You can view the status of each job.
2. **Logs**
    
    - Shows the results of each ETL job.
    - Displays whether jobs succeeded or failed, along with any error messages.
3. **FieldsMapping**
    
    - Allows you to view and update how fields are mapped between source and destination databases.
4. **ValuesMapping**
    
    - Allows you to view and update how specific values are transformed during the ETL process.

## How to Use

- **To run an ETL job:** Go to the Jobs page and add a new job. The system will handle the rest.
- **To check results:** Visit the Logs page to see if jobs completed successfully or if there were errors.
- **To review or change mappings:** Use the FieldsMapping and ValuesMapping pages to see or update how data is mapped and transformed.

## Error Handling

- All errors are logged and visible in the Logs page.
- Error messages help identify issues with data, mappings, or source systems.