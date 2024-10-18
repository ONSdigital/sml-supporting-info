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
 | Method Version   | 1.1.0                                           |

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
 by the predictive variable is within a fixed set of upper and lower thresholds.
 If the ratio lies within these thresholds, then a thousand pounds correction is
 automatically applied to the principal variable and the rest of the target
 variables, if any, are automatically corrected.

If the predictive period’s data is missing, then the method is not applied, unless
 an appropriate variable that is well correlated with the target variable is
 available to the user. The auxiliary variable should not be read into the data
 if the user does not require it.

### 6.2 Error Detection

The error detection calculation is applied to each contributor and calculates the
 ratio of the principal variable and predictive variable at the contributor level.
 The principal variable is the current period data value, and the predictive variable
 is the corresponding previous period data value, if the contributor was previously
 sampled. Previous period data can be a clean response, imputed or constructed data
 value. If there is no predictive value available (i.e., the contributor was
 not sampled in the previous period), then a well correlated alternative
 auxiliary variable can be used if available required by the user. Note that
 the auxiliary variable should be recorded in the same denomination as
 the target variable.

If the ratio is within the predefined upper and lower thresholds, then a
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

This method requires input data supplied as a Pandas dataframe.


### Assumptions and Validity 

- List the technical assumptions upon which the method is based. E.g. data formatted correctly 
- Explain whether the method is still valid if an assumption is violated (if the method is not valid what are the options to overcome this e.g. could a transformation be applied, can violating records be deleted) 


## How to Use the Method

This section must contain: 

- an example and description of the input data 
- an example snippet of code importing and using the method 
- an example and description of the output data produced 

Note: example data sets referenced in the notes must be available in an 
appropriate format (e.g csv) alongside the user notes.

### Method Input

This section should list and describe: 

- The input required (to apply the method). 
- The necessary variables from the input (i.e. only the variables that will be used to create the outputs). 

See below for an example table used to describe input variables. 

 | Variable definition | Type of variable | Format of specific variable (if applicable) | Expected range of the values | Meaning of the values | Expected level of aggregation  | Frequency | Comments | 
 | :---  | :---- | :---- | :---- | :---  | :---- | :---- | :---- | 
 | e.g. 10-digit enterprise reference number | e.g. character; integer; doubl | e.g. date YYYY-MM-DD | e.g. weights should be greater than 0 | e.g. Stagger = 0 indicates that the reporting period is a month | e.g. RU level; VAT unit level; Enterprise level | e.g. quarterly, monthly, annual |      | 
 |           |      |      |      |          |      |      |      | 
   

### Method Output 

This section should: 

- List the expected output data including any auxiliary output/information that 
would be useful for analysis (e.g. summary statistics of weights). 
- List the variables in the output and describe the new variables generated. 

See below for an example table used to describe new output variables, columns 
are context dependent so will vary between specifications. 

 | Variable definition | Type of variable | Format of specific variable (if applicable) | Expected range of the values | Meaning of the values | Expected level of aggregation  | Frequency | Comments | 
 | :---  | :---- | :---- | :---- | :---  | :---- | :---- | :---- | 
 | e.g. 10-digit enterprise reference number | e.g. character; integer; doubl | e.g. date YYYY-MM-DD | e.g. weights should be greater than 0 | e.g. Stagger = 0 indicates that the reporting period is a month | e.g. RU level; VAT unit level; Enterprise level | e.g. quarterly, monthly, annual |      | 
 |           |      |      |      |          |      |      |      | 
  

### Example (Synthetic) Data

This section should contain links to synthetic input and output data.

## Worked Example

Provide a simple worked through example of the method. 

### Treatment of Special Cases 

Are there any special cases to be aware of? For example: 

- How should the method should treat missing/null/zero/extreme values  
- For a time-series method how are the ends of the time-series treated? 

Details of special cases will depend on the method and context. 

### Other Outputs and Metadata (optional) 

This section should describe the metadata that is produced by the method. This could include, for example, intermediate datasets created during the process or summary statistics that can later be used to investigate the performance/suitability of a method. A few examples could include: 

- Number of times a given donor was used in an imputation run 
- The distribution of g weights in estimation (perhaps highlighting the number outside the range 0.7<g<1.4, summary stats on design weight distribution) 
- Selective editing - Business level data showing every iteration of selective editing score that was calculated for each contributor in a given period or a dataset containing all versions of returned and adjusted data values for a given reference at that current point in time 
- History of the type of imputation for a given business (e.g. forward, backward, construction) 

### Issues for Consideration (optional) 

Detail any relevant considerations to be aware of that haven’t been covered in the rest of the specification. 

### Appendix (optional) 

This section should include: 

- Details not covered in the “Background” section. 
- The variable description tables discussed in the sections “Input Data” and “Output Data” 

### Additional Information

The ONS Statistical Methods Library at https://statisticalmethodslibrary.ons.gov.uk/ 
contains:
-	Further information about the methods including a link to the GitHub repository which contains detailed API information as part of the method code.
-	Information about other methods available through the library.


### License

Unless stated otherwise, the SML codebase is released under the MIT License. This covers both the codebase and any sample code in the documentation.

The documentation is available under the terms of the Open Government 3.0 license.
