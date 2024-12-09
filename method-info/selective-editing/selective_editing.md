# Selective Editing User Notes

> [!CAUTION]  
> This is an old version of our User Notes.
> Our current User Guides and example data can be found in the [SML User Docs](https://github.com/ONSdigital/sml-user-docs) repository.

## Finding and Installing the Method
You can find instructions on downloading and installing the method in the [Help centre](https://statisticalmethodslibrary.ons.gov.uk/help-centre/index) of the [ONS Statistical Methods Library](https://statisticalmethodslibrary.ons.gov.uk)


## Using the Method
Once the selective editing method is available on your computer you will be
able to call the method and perform selective editing on a dataset. The
basic selective editing method is:

Score = 100 * (a * |z - y|) / T

* a is the design weight
* z is the return value
* y is the previous period value or the auxiliary value
* T is the standardising factor

As discussed in the methodological specification, the input dataset will
need to contain a specific suffix structure for each question used in
calculating the score. The adjusted return value will have '_ar', the
predicted value will have '_pv', the auxiliary predicted value will
have '_apv', the standardising factor will have '_sf' and the weight
(if weighted scores are chosen) will have '_wt', where the sum of '_wt'
has to add up to 1.

Below is a snapshot of an example dataset and how the input data should
look like:

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

Here we have 10 respondents with all the columns needed to perform the
selective editing method:

* reference: Unique to each respondent.
* design_weight: Also known as the a-weight.
* threshold: This is the selective editing threshold. This is unique for
each domain and are specified by Methodology.
* question_1_ar: This is the adjusted return for question 1. The adjusted
return has usually been through other editing strategies before Selective
Editing.
* question_1_pv: This is the predicted value for question 1, usually the
respondent's value from the previous period.
* question_1_apv: This is the auxiliary predicted variable used in case
there is not a previous period value available.
* question_1_sf: This is the standardising factor which is the weighted
domain estimate for the previous period.
* question_1_wt: This column is a weight column which is used in case the
weighted option is used in the combination method.

We can then run the method as shown below ensuring that we are calling the
right columns from the dataframe:

```
import pandas as pd
import sml_small.selective_editing as seled

# Location of csv file
datafile = "selective_editing_input_data_example_1.csv"

# Read in csv file above
df = pd.read_csv(datafile)

# Call the Selective Editing method
output = seled.selective_editing(input_dataframe = df, # DataFrame of the test data (above)
                                 reference_col = 'reference', # Reference column
                                 design_weight_col = 'design_weight', # Design weight column
                                 threshold_col = 'threshold', # Threshold column
                                 question_list = ['question_1'], # Question(s) we are performing Selective Editing on
                                 combination_method = 'maximum', # Type combination, will accept 'maximum', 'mean', 'weighted', 'minkowski', default = maximum
                                 minkowski_distance = 0, # Only used if Minkowski is selected in combination, set to 0 is minkowski is not selected
                                 show_sums = 0) # Provides additional data on score calculations
                                 
output.to_csv("selective_editing_output_data_example_1.csv")
```

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

New columns are produced from running the method and are described below:

* question_1_s: This is the score for question 1.
* question_1_pm: This is a predicted marker. This will indicate whether you
are using the predicted value (True) or using the auxiliary value (False).
* Final_score: the score after the combination method has been considered and
is the value that is compared to the threshold.
* Selective_edting_marker: This will show whether the respondent has passed
Selective Editing (True) or failed Selective Editing (False).

References 49900001 and 49900004 do not have predicted values available for the
score calculation and therefore uses the auxiliary value, we can see in the
sheet that it's blank but also question_1_pm is False which shows that the
auxiliary is used. The rest of the respondents have the predicted value
available, question_1_pm is True.

## Test Data
The test data mentioned in the example above can be found alongside this user documentation

## Additional Information
The ONS Statistical Methods Library at [https://statisticalmethodslibrary.ons.gov.uk/](https://statisticalmethodslibrary.ons.gov.uk/) contains further information about the methods including:
- a methodological specification, which contains further detail about the mathematical definition of the method algorithm
- a link to the github repository which contains detailed API information as part of the method code

## License
Unless stated otherwise, the SML codebase is released under the [MIT License](https://github.com/ONSdigital/sml-python-small/blob/main/LICENSE). This covers both the codebase and any sample code in the documentation.

The documentation is available under the terms of the [Open Government 3.0 license](https://github.com/ONSdigital/sml-supporting-info/blob/main/LICENSE).
