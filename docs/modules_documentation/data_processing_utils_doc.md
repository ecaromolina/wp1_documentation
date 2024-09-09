
# `data_processing_utils` Documentation 

## Overview 

The goal of this module is to prepare the raw data received from various partners so that it can later be homogenized (using the [`cohort_utils`](cohort_utils_doc.md) module) and exported (using the [`file_exporting_utils`](file_exporting_utils_doc.md) and [`sql_exporting_utils`](sql_exporting_utils_doc.md) modules). This module performs the following tasks:

- **Add calculated variables** based on the original data. This includes updating the objects with configuration data, such as `var_data` and `level_data` with new information, which will be used in the `cohort_utils` module to create the `Cohort` object. Examples of added variables include: `status`, `birth_date`, `exit_date`, `cohort`, etc.
- **Combine variables** that measure the same quantity but in different units to reduce the number of missing values in a variable.
- **Merge versions** of the same database to obtain each patient's data in its latest version. This includes identifying patients with `status='withdrawn'`, i.e., patients who do not appear in the latest version but do appear in previous versions.
- **Check compatibility** of the databases from different cohorts.

## General Structure

The module consists of the following key components:

- **`DataPreprocessor`**: A class responsible for centralizing the preprocessing described above. It is designed to be flexible and customizable based on the specifics of each cohort's data: for each cohort, `DataPreprocessor` allows for personalized transformations through cohort-specific methods that apply the appropriate preprocessing steps.
- **`VarCombiner`**: A helper class responsible for combining the variables specified in the `var_comb_data` dictionary. It is used within `DataPreprocessor` during data preprocessing.
- **Auxiliary functions**: helper functions to check and transform data structure.

## Functions and Classes

In this sections are described the classes and functions defined in the module: 

### **Functions**: 

### `check_df_formats(df_dict: dict[str, pd.DataFrame]) -> bool`
Checks if the format (variable names and data types) of all DataFrames in `df_dict` are compatible for concatenation. It ensures that the columns are in the same order (this can be modified by sorting the columns) and checks if the data types are consistent.

- **Arguments**:
  - `df_dict` (dict[str, pd.DataFrame]): A dictionary containing DataFrames to check for compatibility.

- **Returns**:
  - `bool`: Returns `True` if the DataFrames are compatible for concatenation, even if data types differ (a warning is logged). Returns `False` if the formats are incompatible.

#### logs 

- In `check_df_formats` function:
    - If there are discrepancies between the names of the columns of the different versions of the same databse, an error is logged. 
    - If there are discrepancies between the data types of the same column between the different versions of the same database, a warning is logged.

### **Classes**:

> **Note**: Due to how pandas handles DataFrame modifications, changes made to the `data` attribute (`data`) (generally modifying and/or adding columns) occur 'inplace'. Therefore, when passing the DataFrame by reference into a method or function, any modifications to it will be reflected in the original, and returning the object is not necessary. However, changes made to `var_data` and `level_data` objects do require returning the modified object in the function.


### Class `DataPreprocessor`

#### Description
This class centralizes data preprocessing tasks for cohort databases. It processes the dataframe columns/variables, updates configuration data (`var_data` and `level_data`), and handles cohort-specific transformations. In particular, it first merges all the database versions into a single DataFrame, keeping the data of each patient from the las version. This single dataframe will be stored in the attribute `data`. Then new calculated/secundary variables are added to it, such as `status`, `exit_date`, etc. Moreover, it combines variables using the `var_combine_data` atrribute (if it exists). During the preprocessing, the configuration data (`var_data` and `level_data` dataframes) are updated to fit the structure of `data`. 

Once the preprocessing is finished, `data`, `var_data` and `level_data` is returned so they can be used, for example, to instantiate a `Cohort`. 

#### Attributes

#### Methods

- **`__init__(self,`
    `cohort_databases: dict[str, pd.DataFrame],`
    `var_data: pd.DataFrame, level_data: pd.DataFrame,`
    `cohort_name: str, **kwargs)`:**
        
    Initializes the `DataPreprocessor` class with cohort databases (all the avaliable versions) and the configuration data objects:  `var_data` and `level_data`. Also sets up cohort-specific information like the name of some relevant variables in the specific cohort: age, inclusion date, and ID variables.

    - **Arguments**:
        - `cohort_databases` (dict[str, pd.DataFrame]): A dictionary containing cohort databases. Keys should be the version date (ckeck the format specified in [Structure of data/cohort_name/databases/ Directory](../quick_start_guide.md#structure-of-datacohort_namedatabases-directory))
        - `var_data` (pd.DataFrame): A DataFrame with variable configuration data.
        - `level_data` (pd.DataFrame): A DataFrame with level configuration data.
        - `cohort_name` (str): The name of the cohort being processed.
     
     
- **`set_new_variables(self) -> tuple`**:
  Sets new variables for the cohort, including status, birth date, exit date, and cohort name. Updates the variable and level configuration objects accordingly. To do so, it calls an specific method for each variable, such as: `set_status_metadata`, `set_birth_date_columns`, etc. 

    - **Returns**
        - **tuple**:
            - `var_data`: the `var_data` object updated with the information about the new variables added. 
            - `level_data`: the `level_data` object updated with the information aobut the new variable levels added. 


- **`preprocess(self) -> self`**:
Centralizes the preprocessing logic for different cohorts by identifying the cohort and applying the corresponding cohort-specific method.

    - **Returns**: 
        - **self (PreProcessor)**: Returns the updated class itself. 

- **`preprocess_alcofib(self)`**:
Preprocesses the ALCOFIB cohort data by applying transformations specific to that cohort.

- **`preprocess_glucofib(self)`**:
Preprocesses the GLUCOFIB cohort data by applying transformations specific to that cohort.

- **`preprocess_liverscreen(self)`**:
Preprocesses the LIVERSCREEN cohort data. If a combination dictionary (`comb_var_dict`) is provided, it combines variables according to that configuration.

- **`preprocess_decide(self)`**:
Preprocesses the DECIDE cohort data by applying transformations specific to that cohort.

- **`preprocess_marina1(self)`**:
Preprocesses the MARINA1 cohort data by applying transformations specific to that cohort.

- **`set_status_metadata(self)`**:
Method to update the configuration objects `var_data` and `level_data` with the new variable `status`. This variable is added as a column to the `_data` attribute when the method `get_last_version` is called. In addition, it updates both the `var_data` and `level_data` DataFrames with the new configuration data. This method uses the configuration defined in `config.new_var_config` and `config.new_levels_config` to update the `var_data` and `level_data` DataFrames.

    - **Returns**:
        - `var_data` (pd.DataFrame): The updated DataFrame with metadata for the status variable.
        - `level_data` (pd.DataFrame): The updated DataFrame with metadata for the status levels.

- **`set_birth_date_column(self) -> pd.DataFrame`**:
Calculates and adds a "`birth_date`" column to the dataset based on the inclusion date and age varaibles (specified during the instantiation of the class). Updates the `var_data` DataFrame with the new configuration data for the birth date variable. If first checks if both columns, age and inclusion date, are in the raw data dataframe. Then, for those patients with values in both columns, computes the `birth_date`. 

    - **Returns**:
        - `var_data` (pd.DataFrame): The updated DataFrame with configuration data for the birth date variable.

- **`set_exit_date_column(self)`**:
Adds the "`exit_date`" column to the dataset, which represents the date a patient leaves the cohort or the end of the study. If the study is ongoing, the current date is used.

     - **Returns**:
         - `var_data` (pd.DataFrame): The updated DataFrame with metadata for the exit date variable.

- **`set_cohort_column(self)`**:
Adds the "cohort" column to the dataset, representing the cohort name for each patient. Uses the `cohort_name` attribute to do so. Updates the `var_data` DataFrame with metadata for this variable.

    - **Returns**:
        - `var_data` (pd.DataFrame): The updated DataFrame with metadata for the cohort variable.

- **`get_last_version(self, cohort_databases: dict[str, pd.DataFrame]) -> pd.DataFrame`**:
Merges all versions of the cohort databases and assigns the patient status based on whether they appear in the latest version. Removes duplicates, keeping only the oldest data for each patient. If incompatibilites are found that might prevent the databases to merge properly (this is checked using the function [`check_df_formats`](#check_df_formatsdf_dict-dictstr-pddataframe---bool)) only the last version is returned and a warning is logged. 

    - **Arguments**:
        - `cohort_databases` (dict[str, pd.DataFrame]): A dictionary of cohort database versions. 

    - **Returns**:
        - `pd.DataFrame`: The final version of the cohort data, with the status variable updated.

- **`get_data(self)`**
Returns all processed data, including the main dataset, `var_data`, and `level_data`. This is a sort of a getter for all the processed data. 

    - **Returns**:
        - `tuple`: A tuple containing the main dataset, `var_data`, and `level_data` DataFrames.

#### Logs

- In `__init__`:
    - An error is logged if problems ocurre while calling configuration variables in the `config` package. 

- In `set_status_metadata`:
    - An error is logged if any issues occur while updating `var_data` or `level_data`, particularly when concatenating the new rows in the former DataFrame.

- In `set_birth_date_column`:
    - If any of the variables used to compute `birth_date` (i.e. age or inclusion date) is not a subset of the columns in `data`, an error is logged. 
    - In any errors occurs while computing and assigning the new colimn, an error message is logged. 

- In `set_exit_date_column`:
    - If there is an error while assigning the correct `exit_date` based on the status, an error is logged. 
  
- In `get_last_version`:
    - If an error occurs while assigning the column date_version, an error is logged. 
    - If an error occurs while merging al the version databases, an error is logged. 
    - If an error occurs while dropping the duplicates in the merged data and assigning the `status` column, an error message is logged. 

### Class `VarCombiner`

#### Description

A class designed to combine variables in a DataFrame based on a reference dictionary (`comb_var_dict`). This class helps merge variables and apply conversion factors as needed. In this version, the main functionality of the class is to combine variables of the same magnitude that are expressed in different units so the count of missings is reduced. To do so, it uses the `var_combine_dict`, where the variables (and their conversion factor) to be combined are specified. For more information about the structure of this dictionary check the section [comb_var_data file](../liveraim_database_structure.md#comb_var_data-file). 

The **process of combining the data** works as follows (it's important to understand the structure of `comb_var_dict`). For each key-value pair in `comb_var_dict` with the following structure:

    <final_variable>: {<variable_to_combine_1>: <conversion_factor_1>, <variable_to_combine_2>: <conversion_factor_2>, ...}

The process performs the following steps:

1. If `final_variable` is also one of the `variables_to_combine_n` and its value is not `NaN`, that value is directly used as the `final_variable` value.
   
2. If `final_variable` is not present among the `variables_to_combine_n` or its value is `NaN`, the method iterates through all the `variables_to_combine_n`. It searches for the first variable with a numeric value, applies the corresponding conversion factor, and assigns the result as the value for `final_variable`.
   
3. If no numeric values are found among the `variables_to_combine_n`, the value for `final_variable` is set to `NaN`.


#### Attributes

- **`comb_var_dict (dict)`**: A dictionary where keys are reference variables (the name of the final variable to be obtained) and values are dictionaries with variables to combine and their respective conversion factors.
  
- **`ref_vars (list[str])`**: A list of reference variables extracted from the keys of `comb_var_dict`, i.e. the final variables to be obtained.
  
- **`original_data (pd.DataFrame)`**: A DataFrame containing the original data before any transformations. This is the raw data from each cohort. 
  
- **`transformed_data (pd.DataFrame)`**: A copy of the original DataFrame that will be transformed by combining variables. 

#### Methods

- **`__init__(self, comb_var_dict: dict, df: pd.DataFrame)`**
Initializes the `VarCombiner` class with a dictionary of reference variables and conversion factors, along with the DataFrame to be transformed. It creates the attributes described above, setting `transformed_data` as a copy of `original_data` (i.e. a copy of `df`). 

  - **Arguments**:
    - `comb_var_dict` (dict): A dictionary with reference variables and their corresponding conversion factors.
    - `df` (pd.DataFrame): The DataFrame containing the data to be combined.

- **`_combine_vars(self, row: pd.Series, ref_var: str, var_conv_factors: dict) -> float`**
Combines variables within a row by applying conversion factors if the reference variable is missing (using the algorithm described in the [class *Description*](#description-1)). This function is intended to be used inside the apply method from pandas, specifically in the method `_combine_data`. 

    - **Arguments**:
        - `row` (pd.Series): A row of the DataFrame.
        - `ref_var` (str): The reference variable to check.
        - `var_conv_factors` (dict): A dictionary where keys are alternative variables and 
              values are conversion factors.

    - **Returns**:
          - `float`: The combined value for the reference variable or NaN if no valid variables are found.
        """

- **`_combine_data(self, df:pd.DataFrame, comb_var_dict: dict) -> pd.DataFrame:`**
Applies the variable combination logic (i.e. the _combine_vars method) across all the variables in the comb_var_dict keys.

    - **Arguments**:
        - `df` (pd.DataFrame): The DataFrame to apply the variable combination to.
        - `comb_var_dict` (dict): The dictionary containing the reference variables and their 
          corresponding combining variables and factors.

    - **Returns**:
        - `pd.DataFrame`: The DataFrame with the combined variables.

- **`combine_all_data(self) -> 'VarCombiner'`**
Combines all data according to the `comb_var_dict` and returns the updated instance of the `VarCombiner` class.

  - **Returns**:
    - `VarCombiner`: The updated instance of the class with combined data.

#### Logs

## General Operation

## Usage Example

```python
from data_processing_utils import DataPreprocessor

# Initialize DataPreprocessor with required arguments
preprocessor = DataPreprocessor(cohort_databases, var_data, level_data, cohort_name)

# Preprocess the data
preprocessor.preprocess()

# Get the final processed data
processed_data, var_data, level_data = preprocessor.get_data()
```

## Usage in main execution

