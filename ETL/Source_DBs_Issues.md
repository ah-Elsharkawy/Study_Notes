# Source Databases Issues

> [!TIP] Invalid views in the source databases
> 
> `Duqm` database contains views that are not valid and cannot be queried. Examples:
> ```Hobbies_evw```: This view references a non-existent column `Y` in the `Hobbies` table.
> ```GROUND_WATER_WELLS_SAMPLE_evw```: This view exists but has no columns.
> ```SOIL_SAMPLING_LOCATIONS_evw```: This view exists but has no columns.
> ```SOIL_SUITABILITY_AGRICULTURE_evw```: This view exists but has no columns.
> ```Building_evw```: This view doesn't have ```last_edited_date``` column which is required.


> [!Bug] Please note that the following issues could stem from either the source database or the mapping sheet (e.g., misspellings in the mapping). Kindly verify internally first before reaching out to the client.

> [!TIP] Not Existing tables in the source databases
>
> `Salalah`: The table `ProjectPhase` does not exist.

> [!TIP] Invalid columns in the source databases
>
> `Sohar`: Field `Desc_` doesn't exist in table `SWRC_Pit`.


