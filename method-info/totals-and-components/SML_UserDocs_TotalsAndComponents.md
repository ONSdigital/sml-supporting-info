[comment]: # (File naming convention: SML_UserDocs_<MethodName>_<Lang>.md )
[comment]: # (Lang abbreviates to: Py / PySpk / R )


# Totals and Components in Python 

# Method Description


### Overview

 | Descriptive      | Details                                                  |
 |:---              | :----                                                    |
 | Support Area     | Methodology - Editing & Imputation                       | 
 | Method Theme     | Editing                                                  |
 | Status           | Ready to Use                                             |
 | Inputs           | identifier, total, components, amend_total, predictive, precision, auxiliary, absolute_difference_threshold, percentage_difference_threshold |
 | Outputs          | object containing results |
 | Method Version   | v1.2.1                                                   |

### Summary

The automatic editing method for totals and components correction is currently used in ONS business surveys to ensure fixed relationships between variables are satisfied. For example, when a total (e.g., total employment is collected along with the component breakdown (e.g., full-time male, full-time female, part-time male, part-time female)

The primary use of the method is to automatically detect and correct errors in respondent data where fixed relationships have not been satisfied to improve the efficiency of the editing process, reduce the burden on respondents and survey validators and improve overall data quality.

This method can also be used to ensure fixed relationships between variables are satisfied in other data types such as imputed data to improve overall data quality.

This method can only be applied if all the components are of the same type e.g., all returned or imputed. Maintaining a total and components relationship where components are a combination of types (e.g., returned and imputed) is outside of the scope for this method.

### Terminology

- Contributor – A member of the sample; identified by a unique identifier
- Record – A set of values for each contributor and period
- Target Variable – The variable of interest that the method is working on, the total or components as determined by the Amend Total variable
- Target Record – A contributor's record in the target period
- Predictive Variable – A value used as a predictor for a contributor's target variable
- Predictive Record – The record containing a contributor's predictive value
- Auxiliary variable – The variable used as a predictor for a contributor’s target variable, where the predictive value is not available
- Responder – A contributor who has responded to the survey within a given period
- Precision - The precision value determines the level of accuracy for our floating point calculations 


### Statistical Process Flow / Formal Definition

This method firstly identifies if a given totals and components
 relationship has not been satisfied for a given contributor in
 the target period. If so, the method will determine
 whether the difference observed is within an acceptable tolerance.
 This tolerance is determined in one of three different ways:

1. If the absolute difference between the sum of the components and its
 corresponding predictive total is less than or equal to the predefined threshold
2. If the predictive total is within an acceptable absolute percentage of the
 components sum and an absolute difference is not considered
3. If the absolute difference described above is greater than the
 predefined threshold but the predictive total is within an acceptable
 absolute percentage of the components sum

If the above is satisfied, an automatic correction will be made depending
 on the Amend Total variable. If the above is not satisfied and the
 method is being applied to returned data, then the responder is contacted
 for confirmation. If the method is applied to data that is not returned
 data e.g., imputed data, then the thresholds can be set to a maximum
 value, allowing the method to automatically correct all cases where
 the total does not equal the sum of the components.

The rationale for automatically correcting totals/components can vary by
 data type. For example, with respondent data, it may be preferred to
 only automatically correct data if there is a small difference between
 the predictive and component variables sum in the target period as it is a
 cost-effective approach to data editing. However, if a difference greater
 than a set threshold is observed between the predictive variable and the sum
 of the component variables in the target period, then it may be preferred to not
 automatically correct data in favour of re-contacting the respondent to
 correct the data directly.

If the predictive total is missing, then the method is not
 applied, unless it is appropriate to use an auxiliary variable that
 is well correlated with the target variable. The auxiliary
 variable should not be read into the data if the user does not require it.
 If both variables are missing, then an appropriate marker will be output
 by the method.

**Error Detection**

If a total for the target period does not equal the sum of its corresponding
 components in the target period for a given contributor and the components
 sum is greater than
 zero, then depending on the approach chosen, the method checks whether
 the differences observed are within an acceptable tolerance. This tolerance
 can be determined by an absolute difference, a percentage difference or a
 combination of both.

If components sum is equal to zero and the user has selected to amend the
 total variable (Amend Total = TRUE), then the correction should not be applied.
 See the section on Zero Cases for more details.

The predictive period value can either be a cleaned return, an impute
 or a construction. If this data is not available, then an auxiliary
 variable should only be used when an appropriate
 auxiliary variable exists, and is populated by the user. For example,
 when checking employment data, ONS business surveys use a register-based
 annual auxiliary variable only for surveys with annual periodicity,
 when there is no predictive variable available as this variable is well
 correlated with the target variable. The predictive variable
 should take priority over the auxiliary variable. Otherwise,
 the method should not run if there is no predictive period data available.

**Error Correction**

If the selected condition(s) as described  are
 satisfied, then the target variable will be corrected, depending
 on which target variable has been selected:

1. If Amend Total = FALSE, then the components are automatically corrected
 for the target period to be equal to the total in the target period.
2. If Amend Total = TRUE, then the total is automatically corrected for
 the target period to equal the sum of the components in the target period.

The above should be applied if both the sum of the components and target period
 total are both greater than zero. Else, see the section on Zero Cases.

If the chosen conditions are not satisfied, then no correction
 is applied, and the responder is recontacted for confirmation if the
 target variable is a returned value.

**Zero Cases**

Some cases where either the target period total or the components sum equals
 zero need to be treated differently from as described previously
 depending on the case as detailed below:

1. If target total > 0 and components sum = 0 and amend total = TRUE:
 No correction should be applied in this case. A total only may be provided
 if the component breakdown is unknown so would not want to remove true
 values. A suitable marker will be output by the method.
2. If target total > 0 and components sum = 0 and amend total = FALSE: In this
 case, the proportions of the true components are unknown so the method cannot
 apply a correction. A suitable marker will be output by the method.
3. If target total = 0 and components sum > 0 and amend total = TRUE: The total
 should be corrected if the difference observed is within the tolerances
 determined by the detection method. Else, the difference should be flagged for
 manual checking.
4. If target total = 0 and components > 0 and amend total = FALSE. Apply
 correction to override the components with zeros if the difference observed is
 within the tolerances determined by the detection method. Else, the difference
 should be flagged for manual checking.
 
 
### Calculations
 
**1. Error Detection Calculations**

For all error detection approaches, the method initially identifies contributors where the following is
 not satisfied for *n* components at time *t*:

$$ \large y_{1, t} + \dots + y_{n, t} = y_{total, t} $$

If the above is satisfied, then no error is detected, and the method stops.
Otherwise, the error detection process will proceed depending on the chosen approach.

**1.1 Absolute Difference**

If the condition described in section 1 is not satisfied and the sum of the
 components is not equal to zero, the method then checks whether the
 following is satisfied for a given contributor:

$$ \large |(y_{1, t} + \dots + y_{n, t}) - y_{total,\ predictive}|
 \leq x_{absolute} $$

Where $y_{1, t}$ to $y_{n, t}$ are the *n* corresponding components
 of the total $y_{total, t}$, $y_{total, predictive}$ is the total for the
 predictive period and $x_{absolute}$ is the predefined threshold for
 the absolute difference.

If the predictive value is missing, then $y_{total, predictive}$ should
 be replaced with the auxiliary variable, if
 required by the user.

**1.2 Percentage Difference**

Where the condition described in section 1 is not satisfied and the sum
 of the components is not equal to zero, the method then checks
 whether the following is satisfied for a given contributor:

Let $y_{1,t} +...+ y_{n,t} = y_{derived}$, then:

$$ \large low_{percent} \leq y_{total,\ predictive} \leq high_{percent} $$

Where,

$$ \large low_{percent} = y_{derived} - (y_{derived}*x_{percent}) $$

$$ \large high_{percent} = y_{derived} + (y_{derived}*x_{percent}) $$

Where $x_{percent}$ is the predefined percentage threshold
 represented as a decimal.

If the predictive value is missing, then $y_{total, predictive}$
 should be replaced with the auxiliary variable, if
 required by the user.

**1.3 Combined Error Detection**

The user may choose to use either the absolute
 difference or the percentage difference in isolation. If the user chooses
 to combine both methods, then the absolute difference check should first
 be applied. If the absolute difference is within the acceptable tolerance,
 then the method stops, and the correction is applied. If the absolute
 difference is greater than the tolerance, then the percentage check is
 applied.

If the percentage check is within an acceptable tolerance, then the
 correction is applied.

If the percentage check is outside of the acceptable tolerance, then
 the contributor is flagged for manual checking.

**2. Error Correction Calculation**

If the conditions described in the subsections of section 1 are satisfied
 and the Amend Total variable = FALSE (amend the components), then
 for a given contributor:

$$ \large y_{1, t} + \dots + y_{n, t} = y_{total, t} $$

Where $y_{1, t}$ to $y_{n, t}$ are prorated to
 equal $y_{total, t}$. For a given component $y_{k,t}$, the adjusted
 $\hat{y}_{k,t}$ is calculated by:

$$ \large \hat{y}\_{k,t} = \Bigl(\frac{y_{k,t}}{y_{1, t} + \dots + y_{n, t}}
 \Bigl) * y_{total, t} $$

If the conditions in subsections 1.1 to 1.3 are satisfied
 and the Amend Total variable = TRUE (amend the total), then
 for a given contributor, the adjusted
 $\hat{y}_{total,t}$ is calculated by:

$$ \large \hat{y}\_{total, t} = y_{1, t} + \dots + y_{n, t} $$

If the difference between the predictive total and the sum of the components
 is greater than the thresholds, then the data is not automatically corrected, 
 and the data is flagged for manual checking.
 
 
 **Core mathematical Corrections**
 
 In summary, total correction is:

```  final_total = sum_of_components  ```

and component correction is:

```  final_component = (original_component / sum_of_components) * total  ```


# User Notes

### Finding and Installing the method 

**This method requires Python >=3.7.1, <4.0.0 and uses the Pandas package v1.5.3.**

If you are using Pandas >=2.0 this will be uninstalled and v1.5.3 installed.

><sub>To prevent downgrading software on your system, we recommend creating a virtual environment to install and run SML methods. This will enable you to install the method with the required version of Python, etc, without disrupting the newer versions you may be running on your system. If you’re new to virtual environments, please see our guidence on installing a method in the Help centre of our SML website to get started. Otherwise, use your preferred method to create a virtual environment with the correct software versions.</sub>


The method package can be installed from Artifactory/PyPI using the following code in the terminal or command prompt:

```py
pip install sml_small 
```

In your code you can import the method using:

```py
from sml_small.editing import totals_and_components
```

### Requirements and Dependencies 

- This method requires input data. 

### Assumptions and Validity 

- Target value and predictive value are well correlated
- Target value and auxiliary value, if required, are well correlated
- Thresholds set are a good indication of whether a value should be corrected
- The method can only observe and satisfy one fixed relationship at a time (i.e., only one set of components and total)
- Both the total value and at least one corresponding component are populated for the target period
- The components are all the same data type e.g., all returned or imputed. The total variable type does not impact this.


## How to Use the Method


### Method Input

Input records must include the following fields of the correct types:

- Unique Identifier – Any e.g. Business Reporting Unit
- Total Variable – Target period total, numeric
- Components Variable – Corresponding list of Total variable's components, numeric – nulls allowed
- Amend Total – Select whether Total Variable for the target period should be automatically corrected, Boolean
- Predictive Variable – Previous or current period total, numeric
- Precision - The precision value determines the level of accuracy for our floating point calculations – Numeric – nulls allowed
- Auxiliary Variable – optional, numeric – nulls allowed
- Absolute Difference Threshold - represented as a decimal
- Percentage Difference Threshold - represented as a decimal

Note, populating the parameters with zeros is not equivalent to leaving
them blank.

**Example**

| identifier | total | comp_1 | comp_2 | comp_3 | comp_4 | amend_total | predictive | precision | auxiliary | absolute_difference_threshold | percentage_difference_threshold | expected_result | test_id |
| --- | --- | ---  | --- | --- | --- | ---   | ---   | --- |--- | --- | --- | --- | --- |
| 1 | 1689  |  632 | 732 | 101 | 165 | False | 1689  |  10 |    |  28 | 0.1 |     |     |
| 2 |    0  |    7 |   0 |   2 |   2 | True  |    0  |  28 |    |  11 |     |     |     |
| 3 |   11  |    0 |   0 |   0 |   0 | False |   11  |  28 |    |  11 |     |     |     |
| 4 | 10811 | 9201 | 866 | 632 | 112 | True  | 10811 |  28 |    |     | 0.1 |     |     |
| 5 | 12492 | 9201 | 866 | 632 | 112 | True  | 12492 |  28 |    |     | 0.1 |     |     |

### Method Output 

Output records shall always contain the following fields with the following types:

* Unique Identifier – Any e.g., Business Reporting Unit
* Absolute Difference – Numeric, nulls allowed
* Low Percent – Numeric, nulls allowed
* High Percent – Numeric, nulls allowed
* Final Total Variable – Numeric
* Final Components Variable – Numeric
* TCC Marker – To indicate the result of the Totals Components Correction method, string

The TCC marker returned show if and what the method has corrected:

* T = Total corrected
* C = Components corrected
* N = No correction required, i.e., the total is equal to the components sum
* M = Manual editing required. This marker will identify contributors where the discrepancy between the total and component is deemed too large for automatic correction.
* S = Method stops. This may be due to insufficient data to run the method, or one of the relevant zero cases as described in 6.3 has occurred.


**Example**

| identifier | absolute_difference | lower_percentage_threshold | upper_percentage_threshold | final_total | final_components | tcc_marker |
| --- | --- | --- | ---     | ---   | ---                                                          | --- |
| 1 | 59 | 1467   |  1793   | 1689  | ['654.8760735' ,'758.4957055', '104.6558282', '170.9723927'] |   C | 
| 2 | 11 |        |         | 11    | ['7', '0', '2', '2']                                         |   T | 
| 3 |    |        |         | 11    | ['0', '0', '0', '0']                                         |   S | 
| 4 |    | 9729.9 | 11892.1 | 10811 | ['9201', '866', '632', '112']                                |   N | 
| 5 |    | 9729.9 | 11892.1 | 12492 | ['9201', '866', '632', '112']                                |   M | 

### Example (Synthetic) Data

Files containing the example input & output data given above can be found in the github repo https://github.com/ONSdigital/sml-supporting-info/blob/new-user-doc-template/method-info/totals-and-components/example-data.

Input data:

```  totals_and_components_input_data_example.csv  ```

Expected output after running the worked example:

```  totals_and_components_output_data_example.csv  ```

## Worked Example

- The method can be used in two ways:
- 1. a single set of data can be specified as the input parameters to the method
- 2. mutiple records can be supplied as a pandas dataframe to a wrapper function


The following code can be used to run totals & components on a single record. 
This example uses the first record in the example dataset.

```py
from sml_small.editing import totals_and_components

result = totals_and_components(
    identifier="1",
    total=1689,
    components=
    [
        (632),
        (732),
        (101),
        (165)
    ],
    amend_total=False,
    predictive=1689,
    precision=10,
    auxiliary=None,
    absolute_difference_threshold=28,
    percentage_difference_threshold=0.1
)

# The output will be returned as an object.
# You will need to destructure the object to extract the values and
# one way of doing this is using the built-in function vars().
# vars() is used to return the __dict__attribute for the specified
# module, class, instance or any other object with a
# __dict__attribute

print(vars(result))
```

Alternatively, a wrapper function is supplied with the method to run 
totals & components for all records in a pandas dataframe.

```py
import pandas as pd
from sml_small.utils.pandas_wrapper import wrapper

datafile = "totals_components_input_data_example.csv"
df = pd.read_csv(datafile)

# Specify all columns to be appended to output dataframe
totals_and_components_output_columns = [
    "abs_diff",
    "perc_low",
    "perc_high",
    "final_total",
    "final_components",
    "tcc_marker"    
    ]

totals_and_components_output = wrapper(
    input_frame = df,
    method = "totals_and_components",
    output_columns = totals_and_components_output_columns,
    unique_identifier_column = "identifier",
    total_column = "total",
    components_list_columns = ["comp_1","comp_2","comp_3","comp_4"], 
    amend_total_column = "amend_total",
    predictive_column = "predictive",
    precision = "precision",
    auxiliary_column = "auxiliary",
    absolute_threshold_column = "absolute_difference_threshold",
    percentage_threshold_column = "percentage_difference_threshold"
    )

# Save the output dataframe to a csv file
totals_and_components_output.to_csv(
    "totals_components_output_data_example.csv",index=False)
```

### Additional Information

The ONS Statistical Methods Library at https://statisticalmethodslibrary.ons.gov.uk/ 
contains:
-	Further information about the methods including a link to the GitHub repository which contains detailed API information as part of the method code.
-	Information about other methods available through the library.


### License

Unless stated otherwise, the SML codebase is released under the MIT License. This covers both the codebase and any sample code in the documentation.

The documentation is available under the terms of the Open Government 3.0 license.
