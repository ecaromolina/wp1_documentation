# `qc_checks_utils` Documentation

## Overview

The `qc_checks_utils` module is responsible for performing the various quality control and validation checks that are conducted before, during, and after data processing. The module also handles the export of the quality controls records and logs the relevant alerts in the execution log.

The module includes the following checks:

+ **Non-numeric value detection**: Detects the presence of non numeric values (`-inf`, `inf`). 
+ **NaN analysis**: Analizes the presence of `NaN` values (missings) in both raw_data and homogeneous_data.
+ **Conversion validation**: Checks if the transformation and formatting of variables has been done properly.
+ **Data consistency checks**: Checks if the final data structure fits the one specified in the metadata (`var_data`, `level_data`, `panel_metadata`). 
 
Each of these checks is managed by a class ('checker' from now on) designed specifically for that purpose. A final object, `QCChecker`, is responsible for coordinating all the aforementioned classes and functionalities.

Quality control is performed on a Cohort object, utilizing its attributes to conduct the various checks. For more information about this class, refer to the [cohort_utils](cohort_utils_doc.md) section.


## General Structure

The module has the following general structure, based on the following classes:

+ `GeneralChecker`: This class serves as a base template for all other "checker" classes. It requires a `Cohort` object as a parameter for initialization and sets the common attributes that all checker classes need, such as the cohort name, raw data, homogeneous data, among others.

Specific Checker Classes:

+ [`NonNumericChecker`](#nonnumericchecker): Responsible for checking the presence of non-numeric values in the numeric variables of the cohort's homogeneous data. Inherits from `GeneralChecker`.
+ [`NaNChecker`](#nanchecker): Analyzes the presence of NaN values in both raw and homogeneous data and records the differences between them. Inherits from `GeneralChecker`.
+ [`ConversionChecker`](#conversionchecker): Checks for errors that may have occurred during the conversion of numeric and categorical variables in the homogenization process. Inherits from `GeneralChecker`.
+ [`ConsistencyChecker`](#consistancychecker): Reviews the consistency of the final data by comparing it with the structure of the original metadata defined in the configuration files. Inherits from `GeneralChecker`.

+ [`QCChecker`](#qcchecker): This class centralizes all quality control (QC) checks by integrating the specific checker classes. It coordinates the execution of all checks and exports the results. Inherits from `GeneralChecker`.

## Checkers description

### **`GeneralChecker`**

#### Description:
The `GeneralChecker` class serves as a general template for all the "checker" classes in the quality control module. It acts as the base class (or parent class) from which all other specific checker classes inherit. It provides a common structure and essential attributes (extracted from the `Cohort` class) for performing quality control checks on data.

#### Attributes:
- **`name` (str)**: The name of the cohort being checked.
- **`raw_data` (pandas.DataFrame)**: A DataFrame containing the raw data from the cohort. This is the original, unprocessed dataset.
- **`homogeneous_data` (pandas.DataFrame)**: A DataFrame containing the homogenized data from the cohort. This dataset has undergone processing and formatting to unify formats and values. See section [`Dataflow`](../dataflow.md) for an overview of this process. Check section [`data_processing_util`](../modules_documentation/data_processing_utils_doc.md) and [`cohort_utils`](../modules_documentation/cohort_utils_doc.md)
- **`var_data_json` (dict)**: A dictionary containing metadata about the variables used to create the database. This metadata is also used in quality control checks.
- **`level_data_json` (dict)**: A dictionary containing metadata about variable levels, which is also used for quality control checks.
- **`var_translate_dict` (dict)**: A dictionary used to translate the original variable names to the common variable names used in the homogenized dataset.
- **`selected_variables` (list)**: A list of the selected (or core) variables of the cohort.
- **`selected_variables_homogenized` (list)**: A list of the selected variables, translated to the common format used in the homogenized dataset.
- **`alpha` (float)**: A parameter used as a threshold for detecting discrepancies in numerical conversions. Ideally, any difference greater than 0 should be reported. However, as some rounding errors are introduced while performin datatype adjustment and conversions, small differences between the expected and the real value can be due to this errors. If a discrepancy is bigger than alpha it is reported. Otherwise it is ignored. 
- **`exporting_exceptions` (tuple)**: A tuple containing common exceptions that might occur during the data export process. This includes `FileNotFoundError`, `PermissionError`, `OSError`, among others.

#### Methods:
The `GeneralChecker` class does not define specific methods, as its main purpose is to provide common structure and attributes for the derived classes. The specific methods for quality control checks are implemented in the subclasses.

#### Usage:
`GeneralChecker` is used as a base class for all other checker classes in the module. 

#### Logs
This class does not generate any logs.


### **`NonNumericChecker`**

#### Description:
The `NonNumericChecker` class is used to check, report, and export information about non-numeric values (such as `-inf` and `inf`) in the cohort `homogeneous_data`. This class inherits from `GeneralChecker` and handles the detection of non-numeric values in the numeric variables of the homogenized data. Non-numeric values (such as `-inf` and `inf`) behave differently than `NaN` values, so they are detected and managed separately.

#### Attributes:
- **Attributes inherited from `GeneralChecker`**: All attributes from the `GeneralChecker` class.
- **`non_numeric_record` (pandas.DataFrame)**: A DataFrame containing the count of non-numeric values for each numeric variable in the homogenized data of the cohort.

#### Methods:
- **`check_non_numeric_values()`**: This method checks for non-numeric values (e.g., `-inf`, `inf`, but not `NaN`) in every numeric variable (all the variables whose final datatype is in `main_config.NUMERIC_TYPES`) in the homogenized data. If non-numeric values are found, they are recorded in a list of errors and returned as a DataFrame.
    - **Return**: 
        - `error_df` (pandas.DataFrame) - A DataFrame containing the count of non-numeric values for each numeric variable.


- **`export_qc_data(export_path: str)`**: This method exports the non-numeric values check results to a specified path. It exports the `non_numeric_record` DataFrame as a CSV file.
    - **Returns**: 
        - `None`
    - **Arguments**:
        - `export_path` (str): The file path where the check record will be exported.

#### Logs
- In method `check_non_numeric_values`: 
    - If a variable being analyzed does not appear in the data, a warning is logged.
    - If a numeric variable (column) in the raw_data contains at lest one non_numeric value, a warning is logged. 
- In method `export_qc_data`: If there is any error while exporting generated data, an Error is logged. 


### **`NaNChecker`**

#### Description:
The `NaNChecker` class is responsible for analyzing the presence of `NaN` values in both the raw data (`raw_data`) and the homogenized data (`homogeneous_data`) of a cohort. This class inherits from `GeneralChecker` and provides functionality to check, compare, and export information about missing (`NaN`) values. It also compares the `NaN` counts between the raw and homogenized data to detect any discrepancies that might have occurred during data processing.

#### Attributes:
- **Attributes inherited from `GeneralChecker`**: All attributes from the `GeneralChecker` class.
- **`raw_nan_record_df` (pandas.DataFrame)**: A DataFrame containing the count of `NaN` values for each selected variable in the raw data.
- **`raw_nan_record_dict` (dict)**: A dictionary containing the count of `NaN` values for each selected variable in the raw data.
- **`homogeneous_nan_record_df` (pandas.DataFrame)**: A DataFrame containing the count of `NaN` values for each selected variable in the homogenized data.
- **`homogeneous_nan_record_dict` (dict)**: A dictionary containing the count of `NaN` values for each selected variable in the homogenized data.
- **`nan_diff_record_dict` (dict)**: A dictionary that stores the differences in `NaN` counts between the raw and homogenized data for each variable.

#### Methods:
- **`get_df_nan_record(df: pandas.DataFrame, column_list: list = None) -> tuple[pandas.DataFrame, dict]`**: 
  This method analyzes a DataFrame and returns both a DataFrame and a dictionary containing the `NaN` counts for each specified column in column_list(or all columns if no column list is provided).
    - **Arguments**:
        - `df` (pandas.DataFrame): The DataFrame to be analyzed for `NaN` values.
        - `column_list` (list, optional): A list of column names to check for `NaN` values. If not provided, all columns in the DataFrame are checked.
    - **Return**: 
        - `nan_record_df` (pandas.DataFrame): A DataFrame containing the count of `NaN` values for each column in the DataFrame.
        - `nan_record` (dict): A dictionary containing the same information as `nan_record_df`, with column names as keys and `NaN` counts as values.


- **`log_nan_record() -> dict`**: 
  This method compares the `NaN` counts between the raw data and the homogenized data to check for any discrepancies. It logs discrepancies in `NaN` counts and returns a dictionary with the differences for each variable.
    - **Return**: 
        - `nan_check_dict` (dict): A dictionary containing the difference in `NaN` counts between the raw and homogenized data for each variable.


- **`export_qc_data(export_path: str)`**: 
  This method exports the `NaN` check results to the specified path. It exports the `raw_nan_record_df`, `homogeneous_nan_record_df`, and `nan_diff_record_dict` as CSV files or JSON files as appropriate.
    - **Arguments**:
        - `export_path` (str): The file path where the check records will be exported.
    - **Return**:
        - `None`

#### Logs:
- In method `get_df_nan_record`:
    - If a column in the provided column list is not a subset of the columns in the DataFrame, a warning is logged indicating the missing variables.
    - If there is an error while calculating the percentage of `NaN` values, an error is logged.

- In method `log_nan_record`:
    - If there is a mismatch in the `NaN` counts for a variable between the raw and homogenized data, an error is logged with details of the discrepancy.
    - If no discrepancies are found, an info log confirms that the `NaN` counts were successfully checked.

- In method `export_qc_data`:
    - If there is any error while exporting the generated `NaN` check records, an error is logged with details about the exception.


### **`ConversionChecker`**

#### Description:
The `ConversionChecker` class is responsible for performing conversion checks in the cohort data. This class verifies if both numerical and categorical variables have been properly formatted during data processing. It inherits from `GeneralChecker`. Checks for discrepancies in numerical values (comparing means between the raw and homogenized data) and in categorical values (comparing category counts between the raw and homogenized data). It also allows exporting the results of these checks.

#### Attributes:
- **Attributes inherited from `GeneralChecker`**: All attributes from the `GeneralChecker` class.
- **`mean_diff_dict` (dict)**: A dictionary containing the difference between means for each numerical variable in the raw data and the homogenized data.

#### Methods:
- **`check_numeric_conversion(var: str, mean_diff_dict: dict) -> bool`**: 
  This method checks if the mean of a numerical variable in the raw data matches the mean in the homogenized data, taking into account any conversion factors. If the difference exceeds a specified threshold (`alpha`), it logs an error and updates the `mean_diff_dict`.
    - **Arguments**:
        - `var` (str): The name of the numerical variable to check.
        - `mean_diff_dict` (dict): A dictionary to store the mean differences for each variable.
    - **Return**: 
        - `means_check` (bool): `True` if no significant discrepancies are found, `False` otherwise.


- **`check_category_conversion(var: str) -> bool`**: 
  This method checks if the category counts for a categorical variable match between the raw data and the homogenized data, considering the specific variable mapping. If discrepancies are found, it logs a warning.
    - **Arguments**:
        - `var` (str): The name of the categorical variable to check.
    - **Return**: 
        - `category_check` (bool): `True` if no discrepancies are found, `False`  otherwise.


- **`check_format_conversion() -> dict`**: 
  This method centralizes the format conversion checks for both numerical and categorical variables. It calls the appropriate check methods for each variable and updates the `mean_diff_dict` with the results of the numerical checks.
    - **Return**: 
        - `mean_diff_dict` (dict): A dictionary containing the mean differences between the raw and homogenized data for all numerical variables.


- **`export_qc_data(export_path: str)`**: 
  This method exports the results of the conversion checks to the specified path. It exports the `mean_diff_dict` as a JSON file.
    - **Arguments**:
        - `export_path` (str): The file path where the check records will be exported.

#### Logs:
- In method `check_numeric_conversion`:
    - If the mean difference for a numerical variable exceeds the threshold, an error is logged with details about the discrepancy.

- In method `check_category_conversion`:
    - If there is a mismatch in category counts between the raw and homogenized data, a warning is logged with the details of the discrepancy.

- In method `check_format_conversion`:
    - Logs an informational message when the format conversion checks start.
    - Logs the results of both numerical and categorical variable checks.

- In method `export_qc_data`:
    - If there is any error while exporting the conversion check records, an error is logged with details about the exception.


### **`ConsistancyChecker`**

#### Description:
The `ConsistancyChecker` class is responsible for verifying the consistency of the cohort data. It checks if the final data (homogenized data) complies with the structure defined in the initial metadata (such as `var_data`, `level_data`, and `panel_data` files). This includes checking data types and ensuring that melted variables within panels have consistent data types. The class inherits from `GeneralChecker` and logs any discrepancies found during the consistency checks.

#### Attributes:
- **Attributes inherited from `GeneralChecker`**: All attributes from the `GeneralChecker` class.

#### Methods:
- **`check_data_types() -> None`**: 
    This method checks if there are any discrepancies between the data types in the final homogenized data and the data types specified in the `var_data` metadata. If discrepancies are found, they are logged as warnings.
    - **Return**: 
        - `None`

- **`panel_consistancy_datatypes_check(panel_data_json: dict[str, dict]) -> None`**: 
    This method checks for consistency in data types within panels. Specifically, it verifies that all melted variables in a panel have the same data type. If discrepancies are found, they are logged as warnings.
    - **Arguments**:
        - `panel_data_json` (dict): A dictionary containing metadata about the structure of the panels. This is used to check the consistency of data types within the panels.
    - **Return**: 
        - `None`

#### Logs:
- In method `check_data_types`:
    - If there are discrepancies between the data types in the homogenized data and those specified in the metadata, a warning is logged with details of the discrepancies.

- In method `panel_consistancy_datatypes_check`:
    - If there are discrepancies in data types within a panel (i.e., melted variables with different data types), a warning is logged with details of the panel and the conflicting data types.

### **`OutliersChecker`**

#### Description:
The `OutliersChecker` class is responsible for detecting outliers in the cohort's numerical data using the z-score method. It extends the `GeneralChecker` class, inheriting its attributes and methods, and adds functionalities for outlier detection across individual columns or the entire cohort. ç

#### Attributes:
- **Attributes inherited from `GeneralChecker`**: All attributes from the `GeneralChecker` class.
- **`cohort_outliers`**: A dictionary containing the outliers for all numerical columns in the dataset. Each key is a column/variable name, and the value is another dictionary with the count of outliers and the list of outlier IDs.

#### Methods:
- **`get_column_outliers(df: pd.DataFrame,column: str, z_score_bound: float = 3, id_var: str = 'cohort_id') -> dict`**:  
    This method calculates the z-score of a specified numerical column and identifies outliers. It returns a dictionary with the count of outliers and a list of the corresponding IDs for the observations where the absolute z-score exceeds the specified bound. If the column does not exist in the DataFrame, an empty dictionary is returned.
    
    - **Arguments**:
        - `df` (pandas.DataFrame): The DataFrame containing the data to be analyzed.
        - `column` (str): The name of the numerical column to be analyzed for outliers.
        - `z_score_bound` (float): The threshold for defining an outlier. If the absolute value of the z-score exceeds this threshold, the observation is flagged as an outlier. Default is 3.
        - `id_var` (str): The variable used as the identifier for each observation (e.g., `cohort_id`). Default is `'cohort_id'`.
    
    - **Return**: 
        - `outliers_dict` (dict): A dictionary containing the count of outliers and a list of IDs for the outliers. The structure is `{'outliers_count': int, 'outliers_list': [id_1, id_2, ...]}`.
    
- **`get_cohort_outliers() -> dict`**:  
    This method iterates over all numerical columns in the cohort's homogenized data and applies the `get_column_outliers` method to each one. It returns a dictionary where each key is the name of a numerical column, and the value is the output of the `get_column_outliers` method for that column.
    
    - **Return**: 
        - `outliers_dict` (dict): A dictionary containing outlier information for each numerical variable. Each key corresponds to a variable, and the value is the output from the `get_column_outliers` method.

- **`export_qc_data(export_path: str) -> None`**:  
    This method exports the outliers data stored in `cohort_outliers` to a JSON file at the specified export path. The exported file contains the outlier count and list of outlier IDs for each numerical column.

    - **Arguments**:
        - `export_path` (str): The file path where the outlier data should be exported.
    
    - **Return**: 
        - `None`

#### Logs:
- In method `export_qc_data`:
    - If there is any error while exporting the outlier data to the file, an error is logged with the relevant details.

### **`QCChecker`**

#### Description:
The `QCChecker` class centralizes the quality control (QC) checks for the cohort data. This class orchestrates the execution of all other checker classes, including `NonNumericChecker`, `NaNChecker`, `ConversionChecker`, and `ConsistancyChecker`, to ensure data quality. It also provides methods to generate cohort reports and export QC results. The class inherits from `GeneralChecker`.

#### Attributes:
- **Attributes inherited from `GeneralChecker`**: All attributes from the `GeneralChecker` class.
- **`non_numeric_checker`**: An instance of the `NonNumericChecker` class, responsible for checking non-numeric values in the cohort data.
- **`nan_checker`**: An instance of the `NaNChecker` class, responsible for checking `NaN` values in the cohort data.
- **`conversion_checker`**: An instance of the `ConversionChecker` class, responsible for verifying the conversion of numerical and categorical variables.
- **`consistency_checker`**: An instance of the `ConsistancyChecker` class, responsible for checking the consistency of the final data with the initial metadata.
- **`outlier_checker`**: An instance of the `OutliersChecker` class, responsible for recording the count of outliers for each numerical variable (and the corresponding ID's).
- **`export_qc_record` (bool)**: A boolean value indicating whether the QC records should be exported. This is configured via the `config` module.

#### Methods:
- **`cohort_report() -> None`**: 
    This method logs a brief description of the raw data and homogenized data from the cohort. It reports the number of variables and patients in each dataset.
    - **Return**: 
        - `None`

- **`export_qc_reports() -> None`**: 
    This method centralizes the process of exporting the QC results. It first creates the directory where the QC data will be stored (see [output description](#output-description)), then calls the export methods of the individual checker classes to save the QC results to files.
    - **Return**: 
        - `None`

- **`check_numeric_vars_coherence(data: pandas.DataFrame, var_data_json: dict) -> pandas.DataFrame`**: 
    This method checks the coherence of numeric variables across different cohorts. It calculates the mean and standard deviation for each numeric variable in the selected variables and returns a DataFrame summarizing this information. The aim of the method is to detect if there is any odd behaviour between data from the different cohorts (such as huge difference in mean or sd between cohorts). This method must be used with the merged-homogenized data. 
    - **Arguments**:
        - `data` (pandas.DataFrame): The homogenized and merged data from all cohorts.
        - `var_data_json` (dict): A dictionary containing metadata about the variables, including data types.
    - **Return**: 
        - `combined_data_df` (pandas.DataFrame): A DataFrame containing the mean and standard deviation for each numeric variable, grouped by cohort.

#### Logs:
- In method `cohort_report`:
    - Logs an informational message summarizing the structure of both the raw data and homogenized data, including the number of variables and patients in each.

- In method `export_qc_reports`:
    - If there is any error while exporting the QC results, an error is logged with details about the exception.

- In method `check_numeric_vars_coherence`:
    - No explicit logging is mentioned, but logs could be added to report the progress or outcome of the coherence checks if needed.


## Output Description

Several results generated during quality controls are exported if the `QC_EXPORT_RECORD` variable (declared in the [`main_config`](configuration_module.md#main_config-module) module) is set to `True`.

The `QC_REPORT_FOLDER` directory stores all results related to quality control. This directory contains the code (<span style="color: red">still to be implemented</span>) necessary to create the QC report if desired. `QC_REPORT_FOLDER` also contains a subfolder, defined by the `QC_DATA_FOLDER` variable, where all the raw outputs generated during QC are saved. Each execution (if `QC_EXPORT_RECORD` is set to `True`) will generate a directory within `QC_DATA_FOLDER` named:

    qc_execution_yyyy-mm-dd_hh-mm-ss

Where `yyyy-mm-dd_hh-mm-ss` corresponds to the `EXECUTION_DATETIME`. This directory will store, for each cohort, the files containing the QC-related data. Thus, the general structure of `QC_DATA_FOLDER` could be something like this:

    QC_REPORT_FOLDER/
    ├── qc_report.qmd   # Quarto/md file to create the report. Not yet impemented
    └── QC_DATA_FOLDER/
        ├── qc_execution_2024-08-12_13-42-42/
        │ ├── Liverscreen
        │ │   └── <QC_data_files>
        │ ├── Alcofib
        │ │   └── ...
        │ └── ...  
        ├── qc_execution_2024-08-12_13-42-42/
        └── ...

The resulting files for each cohort are as follows:

+ `raw_nan_record_df.csv`: Contains the count of NaN values for the core/selected variables of the raw data (`raw_data`).

    |     | Variable       | Liverscreen NaN Count | total_records | % NaN |
    | --- | -------------- | --------------------- | ------------- | ----- |
    | 0   | age_0          | 0                     | 23624         | 0.00% |
    | 1   | ethnicity      | 37                    | 23624         | 0.16% |
    | 2   | gender         | 0                     | 23624         | 0.00% |
    | 3   | hypertension   | 82                    | 23624         | 0.35% |
    | 4   | centralobesity | 19                    | 23624         | 0.08% |
    | 5   | hightrigly     | 488                   | 23624         | 2.07% |
    ...


+ `homogeneous_nan_record_df.csv`: Contains the count of NaN values for the selected/core variables of the homogeneous data (`homogeneous_data`).
  
    |     | Variable       | Liverscreen NaN Count | total_records | % NaN |
    | --- | -------------- | --------------------- | ------------- | ----- |
    | 0   | age            | 0                     | 23624         | 0.00% |
    | 1   | ethnicity      | 37                    | 23624         | 0.16% |
    | 2   | gender         | 0                     | 23624         | 0.00% |
    | 3   | hypertension   | 82                    | 23624         | 0.35% |
    | 4   | centralobesity | 19                    | 23624         | 0.08% |
    | 5   | hightrigly     | 488                   | 23624         | 0.00% |
    ...

+ `non_numeric_record_df.csv`: Records non-numeric values (such as -inf, inf) found in the numeric variables of the homogeneous data.

    |     | Variable | non_numeric_values_count |
    | --- | -------- | ------------------------ |
    | 0   | age      | 0                        |
    | 1   | weight   | 0                        |
    | 2   | height   | 0                        |
    | 3   | bmi      | 0                        |
    | 4   | hip      | 0                        |
    | 5   | te       | 0                        |
    ...

+ `nan_diff_record_dict.json`: Contains the differences in NaN counts between the raw and homogeneous data for each variable.

        {
            "age": 0,
            "ethnicity": 0,
            "gender": 0,
            "hypertension": 0,
            "centralobesity": 0,
            ...
        }


+ `mean_difference_dict.json`: Records the differences in means between the raw and homogeneous data for numeric variables, after applying the conversion factor.

        {"Liverscreen": [
            {"age": 0.0},
            {"weight": 2.5540074233276755e-06},
            {"height": 0.0},
            {"bmi": -1.8264589591865388e-06},
            {"hip": -2.641941080128163e-06},
            {"te": 8.128848616451023e-08}
            ...
        ]}

+ `cohort_outliers.json`: Records the outlier count and the id's of the outlier entries for each numerical variable. The structure is as follows:

```json
{
    "column_name_1": {
        "outliers_count": <int>, 
        "outliers_list": ["id_1", "id_2", ...]
    },
    "column_name_2": {
        "outliers_count": <int>, 
        "outliers_list": ["id_1", "id_2", ...]
    },
    ...
}
```

## Usage in main execution

The use of this module during the main execution of the code is through the instantiation of the `QCChecker`. Each time a `Cohort` class is instantiated (and the database is internally processed to obtain the `homogeneous_data`), a `QCChecker` object is created for that cohort (i.e., for each cohort listed in `COHORT_NAME_LIST` from the [`cohort_config`](configuration_module.md#cohort_config-module) module).

When `QCChecker` is instantiated, it automatically creates instances of all the classes responsible for various validations and checks as its attributes (see [`QCChecker` class](#qcchecker)). Each of these attributes will execute its respective validation and check methods automatically, storing the generated records in their own attributes. (<span style="color:red">Revisar docu de métodos init </span>)

If the `QC_EXPORT_RECORD` variable from the [`main_config`](configuration_module.md/#main_config-module) module is set to `True`, the `export_qc_reports` method of the `QCChecker` will be executed in the main code and quality control records for this specific cohort will be exported (see [output descrption](#output-description)). 

After all the cohorts have been created and merged into the final *Liveraim* cohort, another instance of the `QCChecker` class is created to perform a quality control check on this final cohort. In this case, quality control checks related to the structure of the final panels will also be applied through the `panel_consistancy_datatypes_check` method of the `ConsistancyChecker` class.


## Other complementary functions

*Not yet implemented*