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
 | Method Version   | 1.2.0                                           |

### Summary

The thousand pounds correction is commonly used across business surveys. A thousand pounds error occurs when the respondent should have reported values in thousands of pounds but 
has reported in actual pounds e.g., returned a value of £56,000 instead of correctly submitting 56. This method checks values against user-defined thresholds to automatically detect and correct thousand pounds errors.


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

If the predictive period’s data is missing, then the method is not applied unless
 an appropriate variable that is well correlated with the target variable is
 available. The auxiliary variable should not be read into the data
 if the user does not require it.

### 6.2 Error Detection

The error detection calculation is applied to each contributor and calculates the
 ratio of the principal variable and predictive variable at the contributor level.
 The principal variable is the current period data value and the predictive variable
 is the corresponding previous period data value (if the contributor was previously
 sampled). Previous period data can be a clean response, imputed or constructed data
 value. If there is no predictive value available (i.e., the contributor was
 not sampled in the previous period), then a well correlated alternative
 auxiliary variable can be used. Note that
 the auxiliary variable should be recorded in the same denomination as
 the target variable.

If the ratio is within the specified upper and lower thresholds, then a
 thousand pounds error is detected.

If the predictive or auxiliary variable's value is zero or missing, then the method
 does not continue. A thousand pounds error is neither detected nor corrected.

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
 lies within the defined lower or upper thresholds, $L_{Lower}$
 or $L_{Upper}$.

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

### Assumptions & Vailidity

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


### Issues for Consideration (optional) 

Detail any relevant considerations to be aware of that haven’t been covered in the rest of the specification. 



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


### Requirements and Dependencies 




### Assumptions and Validity 

- List the technical assumptions upon which the method is based. E.g. data formatted correctly 
- Explain whether the method is still valid if an assumption is violated (if the method is not valid what are the options to overcome this e.g. could a transformation be applied, can violating records be deleted) 


## How to Use the Method

There are several ways to use this method, according to your needs. Some example code is installed with this method and can be found in the directory where you installed sml_small
Lib -> site packages -> sml_small -> utils -> pandas_example.py. This example walks you through applying the thousand pound correction to data formatted as a pandas dataframe.

It is also possible to run the method to one row of data at a time. Example code for this approach can be found here 
https://github.com/ONSdigital/sml-python-small/blob/main/sml_small/editing/thousand_pounds/example.py 

### Method Input


* unique_identifier: Optional[str],  A question code/ruref/period/id/combination of all of these
* principal_variable: float,  Original response value provided for the 'current' period
* predictive: Optional[float], Value used for 'previous' response (Returned/Imputed/Constructed)
* auxiliary: Optional[float],  Calculated response for the 'previous' period
* upper_limit: float,  Upper bound of 'error value' threshold
* lower_limit: float,  Lower bound of 'error value' threshold
* target_variables: List[TargetVariable], identifier/value pairs
* precision: Optional[int],  Precision is used by the decimal package to ensure a specified accuracy used throughout method processing

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


An example output CSV file has been provided in the example_data folder.
 
  

### Example applying the correction one row at a time

Example input and output data can be found here https://github.com/ONSdigital/sml-supporting-info/tree/new-user-doc-template/method-info/thousand-pound-correction/example_data

## Worked Example

The code below shows you how to run the method with the example input in the table above. This is an instance of using the method on one row of data at a time. 

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


## Example using the Pandas Wrapper

To view the code of the pandas wrapper you can find the `pandas_wrapper.py` file within the `utils` directory.

### Prerequisites: Pandas Wrapper

In order to run some of the functions in the python `pandas_wrapper.py`, you will need to have `pandas` and `numpy` installed.

To install `pandas`:

```python
pip install pandas
```

To install `numpy`:

```python
pip install numpy
```

## Pandas Wrapper Usage

You will have to create a new python file importing in the `pandas_wrapper.py`. In this file you will have to write functions to read a CSV file and pass 
in the data as a DataFrame into the *`wrapper`* function from the `pandas_wrapper.py` file.


We have an example of how to do this in the `pandas_example.py` file within the `utils` directory. You can also find the file here 
https://github.com/ONSdigital/sml-python-small/blob/main/sml_small/utils/pandas_example.py






### Additional Information

The ONS Statistical Methods Library at https://statisticalmethodslibrary.ons.gov.uk/ 
contains:
-	Further information about the methods including a link to the GitHub repository which contains detailed API information as part of the method code.
-	Information about other methods available through the library.


### License

Unless stated otherwise, the SML codebase is released under the MIT License. This covers both the codebase and any sample code in the documentation.

The documentation is available under the terms of the Open Government 3.0 license.
