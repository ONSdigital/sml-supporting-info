[comment]: # (File naming convention: SML_UserDocs_<MethodName>_<Lang>.md )
[comment]: # (Lang abbreviates to: Py / PySpk / R )


# Thousand Pound Correction in Python

# Method Description


### Overview

 | Descriptive      | Details                                                  |
 |:---              | :----                                                    |
 | Support Area     | Methodology – Processing, Editing & Imputation           | 
 | Method Theme     | Automatic Editing |
 | Status           | Ready to Use         |
 | Inputs           | Unique identifier, Principle variables, Target variable(s), Predictive variable, Auxillary variable, Upper limit, Lower limit, Precision   |
 | Outputs          | TCP ratio, Final principle variable, Final target variables, TCP marker |
 | Method Version   | 1.2.1                                           |

### Summary

The thousand pounds correction is commonly used across business surveys. A thousand pounds error occurs when the respondent should have reported values in thousands of pounds but has reported in actual pounds e.g., returned a value of £56,000 instead of correctly submitting 56. This method checks values against user-defined thresholds to automatically detect and correct thousand pounds errors.


### Terminology

* Contributor - A member of the sample; identified by a unique identifier.
* Record - A set of values for each contributor and period.
* Target Period - The period currently undergoing data validation.
* Principal Variable – Variable that the method is working on and will
 determine if the remaining monetary variables, if any, will be
 automatically corrected.
* Target Variable(s) – List of all monetary variables that may be automatically
 corrected, excluding the principal variable.
* Target Record - A contributor's record in the target period.
* Predictive Variable - The corresponding value used as predictor for the
 principal variable for each contributor.
* Predictive Record - The record containing a contributor's predictive value.
* Predictive Period - The period containing predictive records; defined
 relative to the target period.
* Auxiliary variable - An alternative variable used as a predictor for a
 contributor's principal variable, where the predictive value is not available
 for a given contributor (i.e. where the contributor was not sampled in the
 predictive period).
* Responder - A contributor who has responded to the survey within a given period.
* Precision – The precision value determines the level of accuracy for the
 floating point calculations in significant figures.

### Statistical Process Flow / Formal Definition

A principal variable must be specified as a priority indictor for whether a
 thousand pounds error has occurred. The method checks the principal variable
 for a given contributor to determine whether the ratio of the principal variable
 to the predictive variable is within specified upper and lower thresholds.
 If the ratio lies within these thresholds, then a thousand pounds correction is
 automatically applied to the principal variable and the rest of the target
 variables, if any, are automatically corrected.

If the predictive period’s data is missing, then the method is not applied unless an appropriate variable that is well correlated with the target variable is available. The auxiliary variable should not be read into the dataif the user does not require it.

### 6.2 Error Detection

The error detection calculation is applied to each contributor and calculates the
 ratio of the principal variable and predictive variable at the contributor level.
 The principal variable is the current period data value and the predictive variable
 is the corresponding previous period data value (if the contributor was previously
 sampled). Previous period data can be a clean response, imputed or constructed data
 value. If there is no predictive value available (i.e., the contributor was not sampled in the previous period), then a well correlated alternative auxiliary variable can be used. Note that the auxiliary variable should be recorded in the same denomination as the target variable.

If the ratio is within the specified upper and lower thresholds, then a thousand pounds error is detected.

If the predictive or auxiliary variable's value is zero or missing, then the method does not continue. A thousand pounds error is neither detected nor corrected.

### 6.3 Error Correction

A detected thousand pounds error will be automatically corrected by dividing the
 principal variable (i.e., suspicious returned value) by 1000 then rounding to
 the nearest whole number.

All other monetary questions, the target variables excluding the principal variable,
 on the form will be automatically corrected as described without checking the
 returned or corresponding previous values.

### 6.4 Exception Handling

In the case of the method experiencing processing issues, the method shall not result
 in any output records. Instead, a suitable error description shall be emitted.

## 7.0 Calculations

### 7.1 Error Detection Calculation

If a predictive value for the principal question *q* is available for contributor
 *i* at time *t-1*, then a thousand pounds error is detected if the following ratio
 lies within the defined lower or upper thresholds, $L_{Lower}$ or $L_{Upper}$.

$$ \large L_{Lower} < \frac{y_{i, q, t}}{y_{i, q, t-1}} < L_{Upper} $$

```asciimath
L_{Lower} < \frac{y_{i, q, t}}{y_{i, q, t-1}} < L_{Upper}
```

Where $y_{i, q, t}$ is the returned value for the principal question *q* and
 $y_{i, q, t-1}$ is its corresponding previous period value. Previous period
 data may be a clean response, impute or constructed data.

If the ratio lies within, and not equal to, the limits, then a thousand pounds error
 is detected; else a thousand pounds error has not been identified.

If a cleared, predictive value does not exist and the user has chosen to input an
 auxiliary variable, then the error detection calculation is as follows:

$$ \large L_{Lower} < \frac{y_{i, q, t}}{x_{i, q, t}} < L_{Upper} $$

```asciimath
L_{Lower} < \frac{y_{i, q, t}}{x_{i, q, t}} < L_{Upper}
```

Where $x_{i, q, t}$ is the well correlated auxiliary variable for contributor
 *i* and appropriately converted to the same denomination as the principal
 variable, if necessary.

If the ratio lies within the limits, then a thousand pounds error is detected;
 else a thousand pounds error has not been identified.

### 7.2 Error Correction Calculation

A detected error for question *q* is automatically corrected for
 contributor *i* at time *t* by:

$$ \large \hat{y}\_{i, q, t} = \frac{y_{i, q, t}}{1000} $$

```asciimath
\hat{y}_{i, q, t} = \frac{y_{i, q, t}}{1000}
```

Where $\hat{y}\_{i, q, t}$ is the corrected value.

Once the principal target variable has been corrected, all other monetary values,
 the remaining target variables, for a given contributor will be automatically
 corrected by the same method as described above.


# User Notes

### Finding and Installing the method 

**This method requires Python >=3.7.1, <4.0.0 and uses the Pandas package >=1.3.5, <=v1.5.3.**

If you are using Pandas >=2.0 this will be uninstalled and v1.5.3 installed.

><sub>To prevent downgrading software on your system, we recommend creating a virtual environment to install and run SML methods. This will enable you to install the method with the required version of Python, etc, without disrupting the newer versions you may be running on your system. If you’re new to virtual environments, please see our guidence on installing a method in the Help centre of our SML website to get started. Otherwise, use your preferred method to create a virtual environment with the correct software versions.</sub>


The method package can be installed from Artifactory/PyPI using the following code in the terminal or command prompt:

```py
pip install sml_small 
```

In your code you can import the method using:

```py
from sml_small.editing import thousand_pounds
```


## How to Use the Method


### Method Input


* unique_identifier: Optional,  A question code/ruref/period/id/combination of all of these, string
* principal_variable: Original response value provided for the 'current' period, float
* predictive: Optional, Value used for 'previous' response (Returned/Imputed/Constructed), float
* auxiliary: Optional,  Calculated response for the 'previous' period, float
* upper_limit: Upper bound of 'error value' threshold, float
* lower_limit: Lower bound of 'error value' threshold, float
* target_variables: List[TargetVariable], identifier/value pairs
* precision: Optional,  Precision is used by the decimal package to ensure a specified accuracy used throughout method processing, int

This table shows an example of input values for a single row of data. 

| unique_identifier | principal_variable | predictive | auxiliary | upper_limit | lower_limit | precision |
| --- | --- | --- | --- | --- | --- | --- |
| 12340000001-201409-q100 | 50000000 | 60000 | 30000 | 1350 | 350 | 2 |

Target variables are stored in this format:

| identifier | value |
| --- | --- |
| q101 | 500 | 
| q102 | 1000 | 
| q103 | 1500 | 
| q104 | None | 



### Method Output 

Example output table:

| unique_identifier | principal_final_value | target_variables | tpc_ratio | tpc_marker |
| --- | --- | --- | --- | --- |
| '12340000001-201409-q100' | '5.0E+4' | [TargetVariable(identifier='q101', original_value='500', final_value='0.5'), TargetVariable(identifier='q102', original_value='1000', final_value='1'), TargetVariable(identifier='q103', original_value='1500', final_value='1.5'), TargetVariable(identifier='q104', original_value=None, final_value=None)] | '8.3E+2' | 'C' |

Output attributes:
* unique_identifier - A question code/ruref/period/id/combination of all of these
* principal_final_value – Output value that may or may not be adjusted
* target_variables - Identifier/value pairs which may or may not be adjusted (returned as TargetVariable objects)
* tpc_ratio – Ratio of the principal variable against good/predictive/auxillary response
* tpc_marker - 'C' for correction applied, 'N' for no correction applied, 'S' for method stop / error


### Requirements and Dependencies 

- This method requires input data. 

### Assumptions and Validity 


* A ratio inside of the upper or lower thresholds is due to a thousand pounds error
* If the principal variable is found to be a thousand pounds error, then it is
 assumed that all other monetary values are also thousand pounds errors
* The principal variable and predictive value are well correlated and are intended
 to be reported in the same denomination (i.e., thousand pounds)
* The principal variable and auxiliary value are well correlated and intended
 to be reported in the same denomination (i.e., thousand pounds)
* The auxiliary variable is known and available for all contributors processed
 by the method
* Thresholds set are a good indication of whether a value should be corrected
  

### Example (Synthetic) Data

Files containing the example input & output data given above can be found in the github repo
https://github.com/ONSdigital/sml-supporting-info/tree/new-user-doc-template/method-info/thousand-pound-correction/example_data

Input data (single record example):

```  thousand_pounds_example.csv  ```
``` target_variables.csv ```

Expected output (single record example):

```  thousand_pounds_example_output.csv  ```

File containing the example input data to apply the pandas wrapper (second worked example, below) can be found in the github repo
https://github.com/ONSdigital/sml-python-small/blob/main/tests/editing/thousand_pounds/example_data/example_test_data.csv

Input data (pandas wrapper example):

``` example_test_data.csv ```



### Worked Examples

- The method can be used in two ways:
1. a single record can be specified as the input parameters to the method
2. mutiple records can be supplied as a pandas dataframe to a wrapper function


The following code can be used to run the thoudand pounds correction on a single record. This uses the input data for the single record example above.


```python

from sml_small.editing import thousand_pounds

output = thousand_pounds(
    unique_identifier = "12340000001-201409-q100",
    principal_variable = "50000000",
    predictive = "60000",
    auxiliary = "30000",
    upper_limit = "1350",
    lower_limit = "350",
    target_variables = {"q101": 500,
                        "q102": 1000,
                        "q103": 1500,
                        "q104": None},
    precision = 2
)
```
The output will be returned as an object. You will need to destructure the object to extract the values and one way of doing this is using the built-in function vars(). vars() is used to return the \_\_dict\_\_ attribute for the specified module, class, instance or any other object with a \_\_dict\_\_ attribute.
```python
print(vars(output))
```

```bash
{'unique_identifier': '12340000001-201409-q100', 'principal_final_value': '5.0E+4', 'target_variables': [TargetVariable(identifier='q101', original_value='500', final_value='0.5'), TargetVariable(identifier='q102', original_value='1000', final_value='1'), TargetVariable(identifier='q103', original_value='1500', final_value='1.5'), TargetVariable(identifier='q104', original_value=None, final_value=None)], 'tpc_ratio': '8.3E+2', 'tpc_marker': 'C'}
```


Alternatively, a wrapper function is supplied with the method to run the thousand pound correction for all records in a pandas dataframe.

This example uses the input data for the pandas wrapper example above. 

### Prerequisites: Pandas Wrapper

In order to run some of the functions in the python `pandas_wrapper.py`, you will need to install `regex`. You should already have installed `pandas` >= v 1.3.5 <= v 1.5.3.


To install `regex`:

```python
pip install regex
```

## Pandas Wrapper Usage

To use the wrapper you will have to create a new python file that sets up the necessary functions to execute the wrapper. 

The code to do this is shown below. You can copy it directly into your project. 



First, import the required packages.

```python

import pandas as pd
import regex as re

from sml_small.utils.pandas_wrapper import wrapper
```
Next, define additional functions needed to execute the wrapper.

```python
def load_csv(filepath):
    df = pd.read_csv(filepath)
    return df


# Function used to extract columns from a dataframe that are treated as a list, e.g comp_1, comp_2, comp_3
def filter_columns_by_pattern(df, pattern):
    return [col for col in df.columns if re.match(pattern, col)]


# This function takes a given dataframe and will expand a provided column that contains a list
# into separate columns based on the provided prefix (for plain lists) or attribute (for custom objects)
def expand_list_column(
    df,
    list_column_name,
    prefix_attribute=None,
    custom_prefix=None,
    custom_suffix="final_value",
):
    final_data = []

    for index, row in df.iterrows():
        data = row.to_dict()
        list_data = row[list_column_name]

        for i, item in enumerate(list_data, start=1):
            if prefix_attribute:
                # If the attribute doesn't exist N/A is added
                column_prefix = getattr(item, prefix_attribute, "N/A")
                column_name = f"{column_prefix}_{custom_suffix}"
                data[column_name] = item.final_value

            else:
                column_name = f"{custom_prefix}_{i}"
                data[column_name] = item

        final_data.append(data)

    return pd.DataFrame(final_data)
```
Now, define a function to apply the wrapper

```python
def run_thousand_pounds_with_pandas(path, input_csv, output_csv):
    input_dataframe_thousand_pounds = load_csv(path + input_csv)

    # match target variable columns q<number>
    list_column_pattern = r"q\d+"
    target_variables = filter_columns_by_pattern(
        input_dataframe_thousand_pounds, list_column_pattern
    )

    thousand_pounds_output_columns = [
        "principal_final_value",
        "target_variables",
        "tpc_ratio",
        "tpc_marker",
    ]
    test_thousand_pounds = wrapper(
        input_dataframe_thousand_pounds,
        "thousand_pounds",
        output_columns=thousand_pounds_output_columns,
        unique_identifier_column="RU",
        principal_variable_column="principal_val",
        target_variables_columns=target_variables,
        upper_limit_column="threshold_upper",
        lower_limit_column="threshold_lower",
        predictive_column="predictive_val",
        auxiliary_column="aux_val",
    )

    # Expand columns that contain a list of objects (e.g target_variable [TargetVariable(identifier='q42', original_value='32', final_value='0.032'),...])
    # and create separate columns e.g: q42_final_value
    final_data = expand_list_column(
        df=test_thousand_pounds,
        list_column_name="target_variables",
        prefix_attribute="identifier",
    )

    # Create a new DataFrame from the final_data list
    final_df = pd.DataFrame(final_data)

    # Drop the "target_variables" column
    final_df.drop(columns=["target_variables"], inplace=True)

    # Move the tpc_marker to be the last column
    column_to_move = "tpc_marker"

    # Define the desired column order with the specified column at the end
    column_order = [col for col in final_df.columns if col != column_to_move] + [
        column_to_move
    ]

    # Update the DataFrame with the columns in the desired order
    final_df = final_df[column_order]

    # Write the DataFrame to a CSV, excluding the index column
    csv_filename = path + output_csv
    final_df.to_csv(csv_filename, index=False)
```
Finally, apply the wrapper to your data. You will need to edit this code to match 1) the file path to the directory where you have your input data, 2) the file name of the input data, and 3) the desired file name of the output file that will be created. 

```python

run_thousand_pounds_with_pandas(
    "<your_file_path>",
    "<input_data_name>.csv",
    "<output_file_name>.csv",
)

```
After running this function, your output data will be stored in the specified directory.


### Additional Information

The ONS Statistical Methods Library at https://statisticalmethodslibrary.ons.gov.uk/ 
contains:
-	Further information about the methods including a link to the GitHub repository which contains detailed API information as part of the method code.
-	Information about other methods available through the library.


### License

Unless stated otherwise, the SML codebase is released under the MIT License. This covers both the codebase and any sample code in the documentation.

The documentation is available under the terms of the Open Government 3.0 license.
