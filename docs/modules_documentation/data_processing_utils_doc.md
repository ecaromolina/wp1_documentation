
# `data_processing_utils` Documentation 

# Overview 

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

In this sections are described the classes and fucntions defined in the module: 

### `check_df_formats(df_dict: dict[str, pd.DataFrame]) -> bool`
Checks if the format (variable names and data types) of all DataFrames in `df_dict` are compatible for concatenation. It ensures that the columns are in the same order (this can be modified by sorting the columns) and checks if the data types are consistent.

- **Arguments**:
  - `df_dict` (dict[str, pd.DataFrame]): A dictionary containing DataFrames to check for compatibility.

- **Returns**:
  - `bool`: Returns `True` if the DataFrames are compatible for concatenation, even if data types differ (a warning is logged). Returns `False` if the formats are incompatible.

### `DataPreprocessor`
This class centralizes data preprocessing tasks for cohort databases. It processes variables, adds metadata, and handles cohort-specific transformations.

#### `__init__(self, cohort_databases: dict[str, pd.DataFrame], var_data: pd.DataFrame, level_data: pd.DataFrame, cohort_name: str, **kwargs)`
Initializes the `DataPreprocessor` class with cohort databases, variable data, and level data. Also sets up cohort-specific information like age, inclusion date, and ID variables.

- **Arguments**:
  - `cohort_databases` (dict[str, pd.DataFrame]): A dictionary containing cohort databases.
  - `var_data` (pd.DataFrame): A DataFrame with variable metadata.
  - `level_data` (pd.DataFrame): A DataFrame with level metadata.
  - `cohort_name` (str): The name of the cohort being processed.

#### `set_new_variables(self)`
Sets new variables for the cohort, including status, birth date, exit date, and cohort name. Updates the variable and level metadata accordingly.

#### `preprocess(self)`
Centralizes the preprocessing logic for different cohorts by identifying the cohort and applying the corresponding cohort-specific function.

#### `preprocess_alcofib(self)`
Preprocesses the ALCOFIB cohort data by applying transformations specific to that cohort.

#### `preprocess_glucofib(self)`
Preprocesses the GLUCOFIB cohort data by applying transformations specific to that cohort.

#### `preprocess_liverscreen(self)`
Preprocesses the LIVERSCREEN cohort data. If a combination dictionary (`comb_var_dict`) is provided, it combines variables according to that configuration.

#### `preprocess_decide(self)`
Preprocesses the DECIDE cohort data by applying transformations specific to that cohort.

#### `preprocess_marina1(self)`
Preprocesses the MARINA1 cohort data by applying transformations specific to that cohort.

#### `set_status_metadata(self)`
Adds the "status" variable to the dataset and updates both the `var_data` and `level_data` DataFrames with the new metadata. This uses configurations from `new_var_config` and `new_levels_config`.

- **Returns**:
  - `var_data` (pd.DataFrame): The updated DataFrame with metadata for the status variable.
  - `level_data` (pd.DataFrame): The updated DataFrame with metadata for the status levels.

#### `set_birth_date_column(self) -> pd.DataFrame`
Calculates and adds a "birth_date" column to the dataset based on the inclusion date and age. Updates the `var_data` DataFrame with the new metadata for the birth date variable.

- **Returns**:
  - `var_data` (pd.DataFrame): The updated DataFrame with metadata for the birth date variable.

#### `set_exit_date_column(self)`
Adds the "exit_date" column to the dataset, which represents the date a patient leaves the cohort or the end of the study. If the study is ongoing, the current date is used.

- **Returns**:
  - `var_data` (pd.DataFrame): The updated DataFrame with metadata for the exit date variable.

#### `set_cohort_column(self)`
Adds the "cohort" column to the dataset, representing the cohort name for each patient. Updates the `var_data` DataFrame with metadata for this variable.

- **Returns**:
  - `var_data` (pd.DataFrame): The updated DataFrame with metadata for the cohort variable.

#### `get_last_version(self, cohort_databases: dict[str, pd.DataFrame]) -> pd.DataFrame`
Merges all versions of the cohort databases and assigns the patient status based on whether they appear in the latest version. Removes duplicates, keeping only the oldest data for each patient.

- **Arguments**:
  - `cohort_databases` (dict[str, pd.DataFrame]): A dictionary of cohort database versions.

- **Returns**:
  - `pd.DataFrame`: The final version of the cohort data, with the status variable updated.

#### `get_data(self)`
Returns all processed data, including the main dataset, `var_data`, and `level_data`.

- **Returns**:
  - `tuple`: A tuple containing the main dataset, `var_data`, and `level_data` DataFrames.

### `VarCombiner`
A class designed to combine variables in a DataFrame based on a reference dictionary (`comb_var_dict`). This class helps merge variables and apply conversion factors as needed.

#### `__init__(self, comb_var_dict: dict, df: pd.DataFrame)`
Initializes the `VarCombiner` class with a dictionary of reference variables and conversion factors, along with the DataFrame to be transformed.

- **Arguments**:
  - `comb_var_dict` (dict): A dictionary with reference variables and their corresponding conversion factors.
  - `df` (pd.DataFrame): The DataFrame containing the data to be combined.

#### `combine_all_data(self) -> 'VarCombiner'`
Combines all data according to the `comb_var_dict` and returns the updated instance of the `VarCombiner` class.

- **Returns**:
  - `VarCombiner`: The updated instance of the class with combined data.

## Logging

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