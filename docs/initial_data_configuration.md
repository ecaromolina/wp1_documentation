# Initial data/metadata
This section describes the initial data and metadata files that the program needs to build the data warehouse. For each cohort, there are three types of files, which are described below:

* [`database`](#database-files): contains the raw data of the cohort
* [`var_data`](#var_data-files): contains metadata about the core variables needed to create the data warehouse
* [`level_data`](#level_data-files): contains metadata about the levels of categorical core variables needed to create the data warehouse
* [`comb_var_data`](#comb_var_data-file): For the variables that need to be combined, it contains the necessary data to perform this process.

Finally, an other file, [`panel_data`](#panel_data-file) is needed for the proper creation of the data warehouse. In this file the structure of the final panels is described. 

## `Database` files

The files we refer to as *database* files contain the raw database of the cohort. These are the files that the partners responsible for each cohort have sent us with the patient data.

The content of these files is never modified, neither manually nor through the code: all data processing is done after reading these files, which remain intact after each execution. The only modification they receive is a renaming necessary for their correct reading. For more details, see the section [Structure of `data/cohort_name/databases/` directory](configuration/fast_configuration.md#structure-of-datacohort_namedatabases-directory).

The format of these files varies depending on the cohort. For more information on reading the data, see the section [File reading utils](modules_documentation/file_reading_utils_doc.md). When this files are read, they are loaded into the code as `pandas.DataFrame` objects. 

> **Note**: For obvious data privacy reasons, these files are not uploaded to GitHub.

## `var_data` files

This file, along with level_data, is essential for the creation of the data warehouse. It is an Excel file (.xlsx) where each row corresponds to one of the selected/core variables. It contains the following columns:

* `original_name (str)`: Name of the core variable in the original database. It varies between cohorts.
* `liveraim_name (str)`: Name of the core variable in the common format, i.e., in the liveraim format.
* `panel (str)`: Panel where the variable will be stored.
* `data_type (str)`: Final datatype of the variable.
* `initial_units (str)`: Units of the variable in the original cohort.
* `description (str)`: Short text describing the variable.
* `final_units (str)`: Final units of the variable, i.e., units in the liveraim data warehouse.
* `lower_bound (float)`: Lower bound of the variable. For checking purposes.
* `upper_bound (float)`: Upper bound of the variable. For checking purposes.
* `conversion_factor (float)`: Conversion factor from initial_units to final_units. Set to 1 if not needed.

Before running the code, you must verify that these files are in the correct directories, can be read correctly, have the structure defined above, and that the information they contain is correct. Any variation in structure/content may result in an error in execution or the creation of an erroneous data warehouse. The correct functioning of the program is highly dependent on these files.

These files are tracked in GitHub like the rest of the scripts, so there is no need to create them from scratch, and they should work fine if the repository is cloned. 

When this file is read, it is loaded into the code as a `pandas.DataFrame` object. Throughout all the documentation, we will refer to this DataFrame also as `var_data`.

> **Note 1**: There must be certain coherence between the var_data files from each cohort: the number of variables in each file (i.e., the number of rows) should be the same. In addition, the set of variables in `liveraim_name` should be the same between cohorts, and the `final_data` column should be consistent. Although this is checked in the code, making sure that there are no errors is recomended. 

> **Note 2**: If needed, addition of other columns is possible and should not create incompatibilities (but the management of those new columns should be implemented).

## `level_data` files

This file, along with `var_data`, is essential for the creation of the data warehouse. These files describe the different levels of the categorical core variables. In this case, each row does not correspond to a variable, but to a level of the original variable. The file contains the following columns:

* `original_name (str)`: Name of the core variable in the original database. It varies between cohorts.
* `original_level_value (Variable)`: Original value of the level in the cohort.
* `original_data_type (str)`: Original datatype of the variable.
* `liveraim_level_value (Variable)`: Value of the level in the final database (i.e., in the LIVERAIM data warehouse).
* `level_desc (str)`: Description/label of the level.
* `liveraim_name (str)`: Name of the core variable in the common format, i.e., in the liveraim format.

Before running the code, you must verify that these files are in the correct directories, can be read correctly, have the structure defined above, and that the information they contain is correct. Any variation in structure/content may result in an error in execution or the creation of an erroneous data warehouse. The correct functioning of the program is highly dependent on these files.

These files are tracked in GitHub like the rest of the scripts, so there is no need to create them from scratch, and they should work fine if the repository is cloned.

When this file is read, it is loaded into the code as a `pandas.DataFrame` object. Throughout all the documentation, we will refer to this DataFrame also as `level_data`.

> **Note**: There must be certain coherence between the `level_data` files from each cohort: In this case, the number of rows in each file does not need to match, as in some cohorts two or more different levels might coincide in the same final level. However, the `original_name`, `liveraim_name`, and `liveraim_level_value` columns should be consistent between files. Although this is checked in the code, it is recommended to ensure there are no errors.

## `comb_var_data` file

This is a `json` file that will be loaded into the code as a dictionary. It contains the necessary information to obtain a final variable by combining original variables present in the database. For more information about this process, see the section [`data_processing_utils`](modules_documentation/data_processing_utils_doc.md), subsection [`class VarCombiner`](modules_documentation/data_processing_utils_doc.md#class-varcombiner). This section describes the class responsible for combining the variables listed in `comb_var_data` according to the specified configuration.

The primary goal of this dictionary is to combine variables that refer to the same magnitude but use different units. Combining these variables helps reduce the number of missing values. For example, in the LIVERSCREEN cohort, for the magnitude blood glucose (in visit 1), there are the variables `glc` (expressed in mmol/L) and `glc_mg_dl` (expressed in mg/dL). Both will be combined to fill missing at least one of the variables have a proper value.

> **Note**: Currently, only the functionality described above has been implemented. However, it can be extended (by modifying this dictionary and the `VarCombiner` class) to combine categorical variables, generate secondary or calculated variables, and more.

The structure of the `comb_var_data` dictionary is as follows:

- **Primary Key**: The name of the final variable resulting from the combination. This variable must appear in the [`var_data`](#var_data-files) file described in this documentation. It usually matches the name of one of the variables to be used in the combination.
- **Primary Value**: A dictionary with the following structure:
    - **Secondary Key**: Names of the variables (in the database) that will be used for the combination.
    - **Secondary Value**: Conversion factor to transform the variable from its original units to the final units (i.e., the units of the variable named by the primary key).

The structure of the dictionary would be:

```yaml
comb_var_data:
  <final_variable_name_1>:            # Primary Key: Name of the final variable
    <original_variable_name_1>: <conversion_factor_1>    # Secondary Key-Value: Original variable and its conversion factor
    <original_variable_name_2>: <conversion_factor_2>    # Secondary Key-Value: Original variable and its conversion factor
  <final_variable_name_2>:            # Primary Key: Name of another final variable
    <original_variable_name_3>: <conversion_factor_3>    # Secondary Key-Value: Original variable and its conversion factor
    <original_variable_name_4>: <conversion_factor_4>    # Secondary Key-Value: Original variable and its conversion factor
  ...
```

And an example: 

    {
        "glc": {
            "glc": 1,
            "glc_mg_dl": 0.055
        },
        "crea_mg_dl": {
            "creat": 0.017,
            "crea_mg_dl": 1
        }
    }

In this case, it indicates that the `glc` variable will be created by combining the `glc` itself and `glc_mg_dl` variables. Specifically, the value of `glc` will be used if it exists, and if not, the value of `glc_mg_dl` will be used, multiplied by the corresponding conversion factor (in this case, 0.055). A similar procedure will be aplied to `crea_mg_dl`.

These files are tracked in GitHub like the rest of the scripts, so there is no need to create them from scratch, and they should work fine if the repository is cloned.

## panel_data file

This file is not specific to any cohort and is used as a guide to create and structure the different final panels. The file `panel_metadata.xlsx` is an Excel file that contains a sheet for each final panel. Each of these panels includes the following columns:

* `var_name (str)`: Name of the variable (in the common liveraim format).
* `melt (int)`: Integer (acting as a boolean, so it is 0 or 1) indicating:

    * `0`: The variable is not melted: it will appear in the panel as a column.
    * `1`: The variable is melted: the variable name and value will be restructured in a long format. A column `variable` will contain the name of the variable, and the column `value` will contain its value.

These files are tracked in GitHub like the rest of the scripts, so there is no need to create them from scratch, and they should work fine if the repository is cloned.

When this file is read, since it is an .xlsx file with multiple sheets, it is loaded into the code as a dictionary of `pandas.DataFrames`. Each sheet name becomes a key, and the corresponding table becomes its value (a `DataFrame`). Throughout the documentation, we will refer to this dictionary as `panel_metadata`.

It is important to maintain the structure of this file (and the previously described). Changes can be easily made to modify the final structure of the data warehouse by adding new sheets to the file (i.e., adding new panels), adding variables to a panel, or changing their structure (long or wide format).

For more information about the structure of the final data, check the section [Data Warehouse structure](data_warehouse_structure.md).
