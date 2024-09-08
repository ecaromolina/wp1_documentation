# Fast Configuration

This section describes the necessary configuration to run the code and generate the database for WP1. For a comprehensive description of all configuration parameters, refer to the section [Configuration Module](configuration_module.md).

> **Important Notice**: Before configuring the project, ensure that the prerequisites described in the [Environment Setup](../enviroment_setup.md) section are met.

> **Note**: If you have downloaded the repository from GitHub, the structure of the entire repository should be correct. In this case, and if you want to run the code with the default configuration, only the following modifications would be needed:
>
> - Configure the `/data/cohort_name/databases` directory for each cohort. See the section [Structure of the `data/cohort_name/databases/` Directory](#structure-of-datacohort_namedatabases-directory).
> - Configure the MySQL DB connection parameters. See the section [MySQL Connection Configuration](#mysql-connection-configuration).

## Structure of the Project Directory

While the configuration allows some flexibility in the project directory structure, it is recommended to maintain the current structure to avoid errors. The project directory currently (as of 08/07/2024) has the following structure:

    LIVERAIM_WP1/
    ├── config/
    │   ├── cohort_config.py
    │   ├── connection_config.py
    │   ├── main_config.py
    │   ├── new_levels_config.py
    │   ├── new_var_config.py
    │   └── reference_names.py
    ├── data/
    │   ├── Alcofib/
    │   ├── Glucofib/
    │   ├── Decide/
    │   └── Liverscreen/
    ├── logs/
    │   ...
    ├── qc_report/
    │   ...
    ├── wp1_documentation/
    │   ...
    ├── requirements.txt
    ├── main.py
    ...
    other python modules

### Structure of `data/cohort_name/` Directory

Each subdirectory within `data/` contains the databases and files related to each cohort. These subdirectories should have the following structure:

    data/
    ├── cohort_name/
    │   ├── databases/
    │   │   ├── cohort_name_database_version_date.extension
    │   │   ...
    │   ├── cohort_name_level_data.xlsx
    │   └── cohort_name_var_data.xlsx
    ...

The `cohort_name_level_data.xlsx` and `cohort_name_var_data.xlsx` files are described in the section [Initial Data Configuration](../initial_data_configuration.md). If you downloaded the repository from GitHub, you should not need to modify these files.

### Structure of `data/cohort_name/databases/` Directory

The `/data/cohort_name/databases` directory should store the files containing the raw data for the databases of each cohort. Each file in the directory corresponds to one of the received database versions. For more information on how different versions are handled, see the section [Managing Raw Data Versions](). It is **very important** (for this particular case) that the file names follow this structure:

    cohort_name_database_version_date.extension

For example, the raw data file for the Liverscreen cohort received on 07/29/2024 in .dta format would be named:

    liverscreen_database_20240729.dta

Note that the name is in lowercase and the date is in the *yyyymmdd* format.

> **Note**: This folder may contain only one file. It is recommended that this corresponds to the latest version of the data.

Since these data are not uploaded to GitHub, it is necessary to manually place them in the correct directory and format. Once this is done, the directory should be correctly configured.

## MySQL Connection Configuration

If you want to export the data warehouse to the MySQL DB (presumably yes), the MySQL connection parameters must be configured. Therefore, it is necessary to have MySQL installed (if you want to generate the DB locally) and to have a username and password. [A section explaining how to download MySQL and create a user is missing].

Once these requirements are met, you need to configure the MySQL DB connection parameters. These are defined in the `config.connection_config` module, but for a quick configuration, you do not need to modify this file. Instead, you should create the following environment variables in your execution environment:

* `USER` (string): Username (must have permission to access the database).
* `PASSWORD` (string): Database access password.
* `DATABASE_NAME` (string): Name of the database you want to access.
* `PORT` (string): Connection port.
* `HOST` (string): Database host name. If running the code locally on a computer, the host is probably 'localhost'. If running on a server, the host may be the server name or its IP address.

For information on setting environment variables, see the section [Creating Environment Variables](#creating-environment-variables).

For more information on MySQL connection configuration, see the section [Connection Config Module](configuration_module.md#connection-configuration-module).

Once this is done, you should be able to run the code without any problems. In the [Outputs Configuration](#outputs-configuration) section, you can find some useful parameters to determine which outputs you want to generate.

**[A section explaining how to download MySQL and create a user is missing]**

## Outputs Configuration

There are some parameters in the `config.main_config` module that may be useful for users to quickly customize the outputs of the code execution:

* `EXPORT_QC_RECORD (bool)`: With a value of `True`, it exports the data generated during quality control. For more details, see the section [Quality Control](../quality_control.md). With a value of `False`, quality control is still performed, but the generated data is not saved after execution (except for those printed in the log).
* `EXPORT_FILES (bool)`: With a value of `True`, it exports the final data (the data warehouse, with the panel structure described in [???]()) in .csv and .feather format files. With a value of `False`, these files are not generated. For more details, see the section [File Exporting Utils Module](../modules_documentation/file_exporting_utils_doc.md).
* `CREATE_SQL_DB (bool)`: With a value of `True`, it exports the data to a MySQL database (i.e., the data warehouse is created). With a value of `False`, the data is not exported to MySQL.

Access the `config.main_config` file and modify these variables according to your needs.

> Note: This list is not exhaustive. For more information, see the [Configuration Module](configuration_module.md) section.

## Creating Environment Variables
