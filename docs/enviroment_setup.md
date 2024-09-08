# Enviroment setup

In this section, we explain how to set up the virtual environment to run the source code (and generate the data warehouse).

> ***Important Notice:*** It is recommended to work in an active virtual environment when installing the requirements and running the code. This isolates the project and its configuration, ensuring that it does not interfere with other projects. Make sure a Python virtual environment is set up and active before continuing with the ***setup***. If you haven't done so yet, refer to the section [Set Up and Activate a Virtual Environment](#set-up-and-activate-a-virtual-environment).

## Basic Requirements

To run the code, you need to have the following installed:

* `python` version 3.12.2 or higher.
* `pip` version 24.0 or higher.

To check if you have `pip` installed, open your terminal and run the command:
```sh
pip --version
> pip 24.0 from C:**\enviroment\Lib\site-packages\pip (python 3.12)
```

To check your version of `python`, open your terminal and run the command:

```sh
python --version
> Python 3.12.2
```

If you do not have a compatible version or are missing any of the listed requirements, you can see how to install them in the section [Installation of Requirements](#installing-requirements).

> **Note**: It is possible that the code may work with earlier versions of Python, although this has not been tested. It is strongly recommended to update Python to a compatible version. 

## Package Installation

All the packages required to run the code (and their respective versions) are listed in the `requirements.txt` file. When running the main code (`main.py`), it checks if these requirements are met. If any are not installed or have an earlier version than required, they will be installed and/or updated to the latest version. The `setup.py` module handles this process. For more details, see [setup_utils](#module-setup).

However, it may be useful for the user to install the necessary packages beforehand. Run the following command to see the packages installed in the active environment:

```sh
pip list
```

The output should look something like this:

    Package Version
    ------- -------
    pip     24.0

> **Note**: This result only contains `pip` as the installed module because the command was run in a newly created virtual environment.

To install the packages listed in `requirements.txt`, run the command:

```sh
pip install -r requirements.txt
```

Alternatively, you can run the setup_utils module individually. This will check if the necessary requirements are met in the same way as during the execution of main.py. To do this, run the command:

```sh
python setup_utils.py
```

If you now check the installed packages again with the pip list command, you should get something like this:

    Package                Version
    ---------------------- -----------
    et-xmlfile             1.1.0
    feather-format         0.4.1
    greenlet               3.0.3
    mysql-connector-python 9.0.0
    numpy                  2.0.1
    openpyxl               3.1.5
    pandas                 2.2.2
    pip                    24.0
    pyarrow                17.0.0
    pyreadstat             1.2.7
    ...
    pytz                   2024.1
    six                    1.16.0
    SQLAlchemy             2.0.32
    typing_extensions      4.12.2
    tzdata                 2024.1

This is a way to verify that the packages have been installed correctly.

## Installing requirements

## Set Up and Activate a Virtual Environment

To create and set up a Python virtual environment, we will use the `venv` module, which is included by default in Python starting from version 3.3. Other tools (e.g., the `virtualenv` module or the `conda` package manager) also allow you to create virtual environments, but they will not be covered in this documentation. The steps are as follows:

1. **Install Python**: Ensure that you have correctly installed Python beforehand. You can verify the installation by checking the current Python version with the command `python --version`.

2. **Navigate to the Project**: Open a terminal and navigate to the project directory where you want to create the virtual environment. Once there, run the command:

        python -m venv environment_name

    This command should create a virtual environment named `environment_name`. A folder with the name of the new environment should appear in the current directory. This folder should have a structure similar to the following:

        environment_name/
        ├── Include/
        ├── Lib/
        ├── Scripts/
        └── pyvenv.cfg

3. **Activate the Virtual Environment**: Once the virtual environment is created, and each time you want to work from it, you will need to activate it. Depending on the operating system you are using, you will need to use a different command. Make sure you are in the directory where the environment you want to activate is located and run one of the following commands:

    * On ***Windows***: From the terminal, run the command:

            enviroment_name\Scripts\activate

    * On ***Linux***: From the terminal, run the command:

            source enviroment_name/bin/activate

    * On ***macOS***: From the terminal, run the command:

            source enviroment_name/bin/activate

If the virtual environment is set up correctly, you should see the environment name in parentheses before the command line in your terminal. This indicates that the environment is active. Your terminal should look something like this:

    (environment_name) C:/Users/user_name/path/to/project

Now all the packages you install will be isolated in this virtual environment and will not interfere with other projects. Additionally, if necessary, you can configure which version of Python you want to use in each virtual environment.

To deactivate the virtual environment and return to the *global environment* or *system environment*, use the `deactivate` command in the terminal. You should see the environment name in parentheses disappear from the terminal.


## Cloning GitHub repository