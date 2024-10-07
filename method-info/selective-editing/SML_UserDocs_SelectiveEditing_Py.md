[comment]: # (File naming convention: SML_UserDocs_<MethodName>_<Lang>.md )
[comment]: # (Lang abbreviates to: Py / PySpk / R )


# Selective Editing in Python

# Method Description

### Overview

 | Descriptive      | Details                                                  |
 |:---              | :----                                                    |
 | Support Area     | Methodology - Editing & Imputation                       | 
 | Method Theme     | Editing                                                  |
 | Status           | Ready to Use                                             |
 | Inputs           | Reference, question list, adjusted return, predicted value, auxillary predicted value, standardising factor, design weight, threshold |
 | Outputs          | Reference, Score1, ScoreM, Final_Score, Selective Editing Marker, Predicted Marker |
 | Method Version   | 1.1.0                                           |

### Summary

Selective Editing is an internationally recognised editing method where potential errors are prioritised according to their expected impact on key outputs, for one time period. Only respondents that are having a significant impact on published estimates will be recontacted for validation.

Selective Editing works by assigning a score to each important variable for a contributor where, the score reflects the impact that editing the respondent will have on the estimates. Only contributors with a score above a predetermined threshold are flagged for manual review to be validated, low scoring contributors pass through unchecked.

### Terminology

- Contributor reference - A respondent identified by a unique
  identifier.
- Adjusted return - The most recent unedited returned data
  value (for a given variable) in the current period, t.
- Predicted value - The first predictor value (for a given
  variable) for the current period adjusted return.
- Auxiliary predicted value - This is the (secondary)
  auxiliary predictor (for a given variable) for the current
  period adjusted return.
- Standardising factor - The domain group estimate used to
  standardise scores within a given domain group.
- Design weight - An a-weight, usually generated in another
  method.
- Selective Editing domain group - States which Selective
  Editing domain group, a given respondent belongs to.
- Selective Editing threshold - Unique threshold, to compare
  to the score, for each domain group.
- Combining function - If the method is applied to more than
  one variable, then the scores can be combined in the
  following ways: Average, Sum, Max, Weighted mean, Minkowski.

### Statistical Process Flow / Formal Definition

A selective editing score is calculated for each reporting unit i in
time t. The selective editing score is calculated by multiplying the
design weight by the modulus of the adjusted return for reporting unit
i at time t by the predicted value for reporting unit i at time t,
divided by the standardising factor, and all multiplied by 100 as shown
by the equation below.

```asciimath
Score = 100*{{a-weight*|current value-predicted value|}/Standardising factor}
```

The predicted value is equal to a clean response for (i.e. free from
error adjusted return) for reporting unit i at time t-1. However, if
a clean response for reporting unit i at time t-1 is not available
then imputed or constructed previous period data is used. If this
value isn't available, then the auxiliary predicted value for reporting
unit i at time t is used.

If the predicted value is the adjusted return for reporting unit i at
time t-1 then the method output will contain a column with the ending
"_pm" for predictive marker which will be set to 'True'.

If the predicted value is the auxiliary predicted value for reporting
unit i at time t then the method output will contain a column with the
ending "_pm" for predictive marker which will be set to 'False'.

The standardising factor is the weighted domain estimate for a given
variable at time t-1, which is used as a means to determine a
respondent's potential impact on key output estimates.



### Assumptions & Vailidity

- All data inputs required by the method are available and are on a
  standardised basis.
- The predicted variable and auxiliary predicted value should both
  be good predictors of the returned data for time t.
- The predicted value in the score calculation must have been cleaned
  and free from errors. This is not limited to genuine returns, and
  it may be an imputed or constructed value if a clean response is
  not available.
- Each respondent is clearly classified into one mutually exclusive
  domain group.
- The thresholds specified are valid and appropriate (>0)


# User Notes

### Finding and Installing the method (written by SML Team)

This method requires (Python / R) version (n.n.n) and uses the (...) package(s).

</sub> (If method requires older Python/R and/or packages): To prevent downgrading software on your system, we recommend creating a virtual environment to install and run SML methods. This will enable you to install the method with the required version of Python, etc, without disrupting the newer versions you may be running on your system.  
If you’re new to virtual environments, feel free to use this guide (link to Python or R portal guidance as appropriate) to get started. Otherwise, use your preferred method to create a virtual environment with the correct software versions.</sub>


The method package can be installed from (Artifactory / PyPI / CRAN) using the following code in the terminal or command prompt:

```py
pip install (package_name)
```

```r
install.packages("package_name")
```

In your code you can import/load the method using:

```py
from (package_name.module_name) import (function_name)
```

```r
library(package_name)
```


### Terminology 

This section should contain any terminology specific to this method which it would be useful to define at this stage. 

### Requirements and Dependencies 

This section should: 

- Summarize any required pre-processing requirements. 
- List any SML methods/utilities that call or depend on this method (if there are dependent SML methods please provide links). 


### Assumptions and Validity 

- If the "weighted mean score" is selected to combine score, then the
  sum of the weights provided should total to 1.
- The method will automatically assign
adjusted return (ar), predicted value (pv), auxiliary predicted
value (apv) and standardising factor (sf) based on the column names
of the inputs. 
- Unless otherwise noted, fields must not contain Null values. All other
fields shall be ignored.

## How to Use the Method

Once the selective editing method is available on your computer you will be
able to call the method and perform selective editing on a dataset. 

The input dataset will need to contain a specific suffix structure for each question used in
calculating the score. The adjusted return value will have '_ar', the
predicted value will have '_pv', the auxiliary predicted value will
have '_apv', the standardising factor will have '_sf' and the weight
(if weighted scores are chosen) will have '_wt', where the sum of '_wt'
has to add up to 1.

### Method Input

Input records must include the following fields of the
correct types:

- reference (any type): Unique to each respondent. 
- design_weight (numeric): Also known as the a-weight. 
- threshold (numeric): This is the selective editing threshold. This is unique for
each domain and are specified by Methodology. 
- question_1_ar (numeric): This is the adjusted return for question 1. The adjusted
return has usually been through other editing strategies before Selective
Editing. 
- question_1_pv (numeric): This is the predicted value for question 1, usually the
respondent's value from the previous period. 
- question_1_apv (numeric): This is the auxiliary predicted variable used in case
there is not a previous period value available. 
- question_1_sf (numeric): This is the standardising factor which is the weighted
domain estimate for the previous period. 
- question_1_wt (numeric): This column is a weight column which is used in case the
weighted option is used in the combination method. 




**Example**

 | Reference | design_weight | threshold | question_1_ar | question_1_pv | question_1_apv | question_1_sf | question_1_wt |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 49900001 | 20 | 0.6 | 800 | | 424 | 800000 | 1 |
| 49900002 | 20 | 0.6 | 656 | 390 | 259 | 800000 | 1 |
| 49900003 | 20 | 0.6 | 997 | 773 | 912 | 800000 | 1 |
| 49900004 | 20 | 0.6 | 676 |  | 334 | 800000 | 1 |
| 49900005 | 20 | 0.6 | 632 | 871 | 684 | 800000 | 1 |
| 49900006 | 20 | 0.6 | 985 | 345 | 312 | 800000 | 1 |
| 49900007 | 20 | 0.6 | 468 | 963 | 773 | 800000 | 1 |
| 49900008 | 20 | 0.6 | 772 | 733 | 833 | 800000 | 1 |
| 49900009 | 20 | 0.6 | 621 | 673 | 898 | 800000 | 1 |
| 49900010 | 20 | 0.6 | 736 | 377 | 646 | 800000 | 1 |
   

### Method Output 

Output records will contain the following fields with the
following types:
New columns are produced from running the method and are described below:

- question_1_s: This is the score for question 1.
- question_1_pm: This is a predicted marker. This will indicate whether you
are using the predicted value (True) or using the auxiliary value (False).
- Final_score: the score after the combination method has been considered and
is the value that is compared to the threshold.
- Selective_edting_marker: This will show whether the respondent has passed
Selective Editing (True) or failed Selective Editing (False).

The output gets exported as a csv file and will give you the scores and what
values were used, whether it is the previous period value or the auxiliary
value.

| Reference | design_weight | threshold | question_1_ar | question_1_pv | question_1_apv | question_1_sf | question_1_s | question_1_pm | final_score | selective_editing_marker
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 49900001 | 20 | 0.6 | 800 | | 424 | 800000 | 0.94 | FALSE | 0.94 | FALSE |
| 49900002 | 20 | 0.6 | 656 | 390 | 259 | 800000 | 0.665 | TRUE | 0.665 | FALSE |
| 49900003 | 20 | 0.6 | 997 | 773 | 912 | 800000 | 0.56 | TRUE | 0.56 | TRUE |
| 49900004 | 20 | 0.6 | 676 |  | 334 | 800000 | 0.855 | FALSE | 0.855 | FALSE |
| 49900005 | 20 | 0.6 | 632 | 871 | 684 | 800000 | 0.5975 | TRUE | 0.5975 | TRUE |
| 49900006 | 20 | 0.6 | 985 | 345 | 312 | 800000 | 1.6 | TRUE | 1.6 | FALSE |
| 49900007 | 20 | 0.6 | 468 | 963 | 773 | 800000 | 1.2375 | TRUE | 1.2375 | FALSE |
| 49900008 | 20 | 0.6 | 772 | 733 | 833 | 800000 | 0.0975 | TRUE | 0.0975 | TRUE |
| 49900009 | 20 | 0.6 | 621 | 673 | 898 | 800000 | 0.13 | TRUE | 0.13 | TRUE |
| 49900010 | 20 | 0.6 | 736 | 377 | 646 | 800000 | 0.8975 | TRUE | 0.8975 | FALSE |



References 49900001 and 49900004 do not have predicted values available for the
score calculation and therefore uses the auxiliary value, we can see in the
sheet that it's blank but also question_1_pm is False which shows that the
auxiliary is used. The rest of the respondents have the predicted value
available, question_1_pm is True.

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
