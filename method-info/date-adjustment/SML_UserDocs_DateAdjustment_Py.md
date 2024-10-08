[comment]: # (File naming convention: SML_UserDocs_<MethodName>_<Lang>.md )
[comment]: # (Lang abbreviates to: Py / PySpk / R )


# Date Adjustment in Python

# Method Description


### Overview

 | Descriptive      | Details                           |
 |:---              | :----                             |
 | Support Area     | Editing & Imputation              | 
 | Method Theme     | Editing                           |
 | Status           | Ready to Use                      |
 | Inputs           | input_dataframe, trading_weights, target_columns, contributor_returned_start_date_col, contributor_returned_end_date_col, expected_start_date_col, expected_end_date_col, domain_col, short_period_parameter_col, long_period_parameter_col, equal_weighted_col, set_to_mid_point_col, use_calendar_days_col, average_weekly_col, da_error_flag_col, trading_date_col, trading_period_start_col, trading_period_end_col, trading_weights_col, trading_domain_col |
 | Outputs          | dataframe with adjusted responses and sum of the weights |
 | Method Version   | v1.0.0                            |


### Summary

To generate summary and headline statistics for a reporting period it is important to ensure that data reported are on a consistent basis referencing the same period. However, sometimes it is not possible for data to be reported for the exact period of time required, e.g., monthly, quarterly, yearly. The responder may specify different start and end dates for which the observed (referenced as response throughout this specification) data cover. The Date Adjustment method weights the data values from the response period to approximate the values for the desired (expected) period, either by giving varying weights to trading days or giving each date the same weight.


### Terminology

- Expected period start date – The expected start date of the period set by the user.
- Expected period end date – The expected end date of the period set by the user.
- Variable(s) to be date adjusted – The user can select one or more variables to be date adjusted.
- Contributor’s returned start date – The start date of the period returned by a respondent/observed in the data.
- Contributor’s returned end date – The end date of the period returned by a respondent/observed in the data.
- Domain – Classification group.
- Set to mid-point – Allows the user to apply the mid-point method described below.
- Set to equal weighted – Allows the user to apply the equal weighted method described below.
- Use calendar days – Allows the user to apply the calendar days method described below.
- Average weekly – Allows the user to apply the average weekly method described below.
- Short period parameter – This is used to raise a flag to alert the user that the contributor’s returned period is short.
- Long period parameter – This is used to raise a flag to alert the user that the contributor’s returned period is long.
- Date mapping – This is a file which contains dates and relevant trading day weights per domain.
- Trading day weights: These are weights associated with each day to allow the user to give a higher value to certain days relative to others in a given period. For example, setting weights of 0.2 for weekdays and 0 for weekends when considering turnover data values would imply that turnover is not generated on the weekend and is stable throughout weekdays.
- Sum of trading day weights over contributor's period: The trading day weights will be summed over the dates returned; all dates are inclusive.
- Number of days in contributor's returned period: The number of days returned by the contributor; all dates are inclusive.
- Actual period start date: Will appear on the output dataset and is the start date that the output is based on.
- Actual period end date: Will appear on the output dataset and is the end date that the output is based on.
- Number of days in actual returned period: The number of days the user set.
- Sum of trading day weights over actual period: The sum of the trading weights over the days set by the user.
- Date adjusted variable: The adjusted data value based on the sum of the trading days weights ratio. 

### Statistical Process Flow / Formal Definition

The basic date adjustment method is:

```  Adjusted response = Original response * Sum of weights of desired period / Sum of weights of response  ```

A contributor will give a response for a period, this does not have to match the expected period, and will be fed into the method. Data which have the same returned and expected start and end dates may have this method applied but no adjustments will be made as the input data is fit for purpose. Where this is not the case trading day weights are assigned to each day covered by the contributor’s returned period and the expected period. These are then summed to provide a sum for the total trading day weights for the returned and expected periods respectively. The expected period total trading day weights are divided by the contributor’s returned period total trading day weights. This ratio is then applied to the contributor’s response so that it is representative of the expected period.

**Incomplete or erroneous data provided**
Contributors may not always provide full or correct data and the Date Adjustment method will handle it by producing error codes. There are 16 error codes and the explanation of each of the codes can be found below:

E00: Average Weekly parameter is invalid.
E01: The value to be date adjusted is missing from one of the target columns.
E02: The contributor returned end date is earlier than the contributor returned start date.
E03: A required record for calculating weight m is missing from the trading weights table.
E04: A required trading weight for calculating weight m is null or blank.
E05: A required trading weight for calculating weight m has a negative value.
E06: A required record for calculating weight n is missing from or duplicated in the trading weights table.
E07: A required trading weight for calculating weight n is null or blank.
E08: A required trading weight for calculating weight n has a negative value.
E09: Contributors return does not cover any of expected period.
E10: The sum of trading day weights over contributors returned period is zero.
E11: The sum of trading day weights over contributors returned period is zero.
E12: A required record for calculating midpoint date is missing from the trading weights table.
E13: A required record for setting APS and APE by midpoint is missing from or duplicated in the trading weights table.
E14: Expected period start date is missing or an invalid date.
E15: Expected period end date is missing or an invalid date.
These are NOT exceptions and do not cause the method to fail. Once an error flag has been placed on a row of data, no further processing is done to that row, preserving the data in the state it was when the flag was raised. The method will continue processing even when errors occur, this was done for historical reasons, unlike other methods in the Statistical Methods Library.

**Set to Mid-point – set as Y or YT or N**
A mid-point method can be used in Date Adjustment to check whether a contributor’s returned dates are within the expected period. If the mid-point of the contributor’s returned start and end dates are outside the expected period, then a “C” flag is raised in the error flag column to inform the user that the contributor’s response data aligns with a different reporting period (i.e., not the expected period). If the respondent’s mid-point does lie outside of the period, the user is interested in then date adjustment will not occur.

There are two ways to use the mid-point method: set the mid-point to “Y” and set the mid-point to “YT”. Setting the mid-point input parameter to “Y” will simply calculate the mid-point of the days returned. If the number of days in the contributor’s returned period are even, then divide the count by 2 and add that to the contributor’s returned start date to find the mid-point. If the number of days in the contributor’s returned period are odd, then add 1 to the count and divide by 2. This is then added onto the contributor’s returned start date. If the user sets the mid-point input parameter to “YT”, then the method will trim any weighted days at the start or end of the contributor’s returned dates if they have trading day weights set to 0 and do the same calculation as above with regards to the number of days in the period. Therefore, each method could provide a slightly different answer. If the user does not want the mid-point method to occur, then please set this to “N”.

**Use calendar days – set as Y or N**
If the mid-point lies outside of the expected period start and end dates, a “C” flag is raised in the Date change column. Calendar days function will only work when there is a “C” flag present. The calendar days function, when set to “Y”, allows the user to automatically set the expected start and end period dates to the first day and last day of the month where the mid-point lies. When the calendar days function is set to “N” it allows start and end period dates to be set to the start and end dates that “C” flag lies in.

**Set to equal weighted – set to Y or N**
The equal weighted method is where all the trading day weights are set to 1 instead of having a unique trading day weight associated with each day. This can be a useful feature when each day is of equal importance with respect to the data collected. To use this feature, the equal-weighted column should be marked as “Y”. If the user does not want the trading day weights to be equal-weighted, then please set this to “N”.

**Average weekly – set to A or N**
The average weekly function allows the responses to be given on a weekly value rather than the period specified by the user. This can be a useful tool when the user’s period isn’t always equal, to allow the user to compare similar values. To use this feature, the average weekly value will need to be set to “A”. If the user does not want to have average weekly values in the output, then please set this to “N”.


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
import sml_small.date_adjustment as date_adjust
```


### Requirements and Dependencies 

- This method requires input data and trading day data. 
- These each need to be supplied as a pandas dataframe.

### Assumptions and Validity 

- All data inputs required by the method are available.
- Each respondent is clearly classified into one mutually exclusive domain group.
- All trading day weights are equal or greater than 0.
- Trading day weights are available for all periods and domains, or the equal weighted option is set to yes.



## How to Use the Method

### Method Input

The method requires input data and trading day data.

#### Input Data

The input data requires the following columns to perform the date adjustment method:

* Reference: Unique to each respondent
* Contributors start date: Start date returned by the contributor.
* Contributors end date: End date returned by the contributor.
* Q20: Variable response that needs to be date adjusted.
* Expected start date: Start date the user is expecting from the contributor.
* Expected end date: End date the user is expecting from the contributor
* Domain: Domain classification.
* Mid-point: Indicator as to whether the mid-point method needs to be used.
* Equal-weighted: Indicator as to whether the equal-weighted method needs to be used.
* Calendar days: Indicator as to whether the calendar days method needs to be used.
* Average weekly: Indicator as to whether the average weekly method needs to be used.
* Short period: A value that shows the user whether a response is of a short time frame.
* Long period: A value that shows the user whether a response is of a long-time frame.
* Error column: A column that will be populated if any errors occur.


**Example**

| Reference | Contributors start date | Contributors end date | Q20 | Expected start date | Expected end date | Domain | Mid-point | Equal-weighted | Calendar days | Average Weekly | Short period | Long period | 
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | 
| 1 | 20220601 | 20220630 | 1184 | 20220601 | 20220630 | A | N | N | N | N | 24 | 42 | 
| 2 | 20220604 | 20220630 | 2045 | 20220601 | 20220630 | A | N | N | N | N | 24 | 42 | 
| 3 | 20220601 | 20220624 | 2013 | 20220601 | 20220630 | A | N | N | N | N | 24 | 42 | 
| 4 | 20220601 | 20220704 | 1992 | 20220601 | 20220630 | A | N | N | N | N | 24 | 42 | 
| 5 | 20220530 | 20220628 | 1027 | 20220601 | 20220630 | A | N | N | N | N | 24 | 42 | 

#### Trading Day Data

The trading day data requires trading day weights for all dates present and for the domain we are interested in:

* Date: Date we are interested in
* Domain: Domain classification
* Weight: Weight for the specific trading day
* Period: An indicator of the period.
* Period_start: Start date of the period
* Period_end: End date of the period


**Example **

| date | domain | weight | period | period_start | period_end |
| --- | --- | --- | --- | --- | --- |
| 20220528 | A | 0 | 202205 | 20220501 | 20220531 |
| 20220529 | A | 0 | 202205 | 20220501 | 20220531 |
| 20220530 | A | 0.2 | 202205 | 20220501 | 20220531 |
| 20220531 | A | 0.2 | 202205 | 20220501 | 20220531 |
| 20220601 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220602 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220603 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220604 | A | 0 | 202206 | 20220601 | 20220630 |
| 20220605 | A | 0 | 202206 | 20220601 | 20220630 |
| 20220606 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220607 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220608 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220609 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220610 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220611 | A | 0 | 202206 | 20220601 | 20220630 |
| 20220612 | A | 0 | 202206 | 20220601 | 20220630 |
| 20220613 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220614 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220615 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220616 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220617 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220618 | A | 0 | 202206 | 20220601 | 20220630 |
| 20220619 | A | 0 | 202206 | 20220601 | 20220630 |
| 20220620 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220621 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220622 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220623 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220624 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220625 | A | 0 | 202206 | 20220601 | 20220630 |
| 20220626 | A | 0 | 202206 | 20220601 | 20220630 |
| 20220627 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220628 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220629 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220630 | A | 0.2 | 202206 | 20220601 | 20220630 |
| 20220701 | A | 0.2 | 202207 | 20220701 | 20220731 |
| 20220702 | A | 0 | 202207 | 20220701 | 20220731 |
| 20220703 | A | 0 | 202207 | 20220701 | 20220731 |
| 20220704 | A | 0.2 | 202207 | 20220701 | 20220731 |



### Method Output 

#### Output Data

New columns are produced from running the method and are described below:

* Sum of trading day weights over contributors period: The trading day weights will be summed over the dates returned, all dates are inclusive.
* Number of days in contributors returned period: The number of days returned by the contributor, all dates are inclusive.
* Actual period start date: The start date the user is looking for (usually similar to expected start date).
* Actual period end date: The end date the user is looking for (usually similar to expected end date).
* Number of days in actual returned period: The number of days the user set
* Sum of trading day weights over actual period: The sum of the trading weights over the days set by the user.
* Date adjusted Q20: The adjusted question value based on the sum of the trading days weights ratio.


**Example**

| Ref | Contributors start date | Contributors end date | Q20 | Expected start date | Expected end date | Domain | Mid-point | Equal-weighted | Calendar days | Average Weekly | Short period | Long period | Error Column | Sum trading day weights over contributors returned period | Num days in contributors returned period | Actual period start date | Actual period end date | Num days in actual returned period | Sum trading day weights over actual returned period | Date adjusted Q20 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 2022-06-01 | 2022-06-30 | 1184 | 2022-06-01 | 2022-06-30 | A | N | N | N | N | 24 | 42 | 	 | 4.400000000000001 | 30 | 2022-06-01 | 2022-06-30 | 30 | 4.400000000000001 | 1184.0 |
| 2 | 2022-06-04 | 2022-06-30 | 2045 | 2022-06-01 | 2022-06-30 | A | N | N | N | N | 24 | 42 | 	  | 3.800000000000001 | 27 | 2022-06-01 | 2022-06-30 | 30 | 4.400000000000001 | 2367.8947368421054 |
| 3 | 2022-06-01 | 2022-06-24 | 2013 | 2022-06-01 | 2022-06-30 | A | N | N | N | N | 24 | 42 | 	  | 3.600000000000001 | 24 | 2022-06-01 | 2022-06-30 | 30 | 4.400000000000001 | 2460.3333333333335 |
| 4 | 2022-06-01 | 2022-07-04 | 1992 | 2022-06-01 | 2022-06-30 | A | N | N | N | N | 24 | 42 | 	  | 4.800000000000002 | 34 | 2022-06-01 | 2022-06-30 | 30 | 4.400000000000001 | 1826.0 |
| 5 | 2022-05-30 | 2022-06-28 | 1027 | 2022-06-01 | 2022-06-30 | A | N | N | N | N | 24 | 42 |   | 4.400000000000001 | 30 | 2022-06-01 | 2022-06-30 | 30 | 4.400000000000001 | 1027.0 |


### Example (Synthetic) Data

Files containing the example input & output data given above can be found in the github repo https://github.com/ONSdigital/sml-supporting-info/tree/main/method-info/date-adjustment/example-data.

Input & Trading day data:

```  date_adjustment_input_data_example_1.csv  ```

```  date_adjustment_trading_day_weights_example_1.csv  ```

Expected output after running the worked example:

```  date_adjustment_output_data_example_1.csv  ```

## Worked Example

Using the example input data and trading day data, we can then run the method as shown below ensuring that we are calling the right columns from the dataframe:

```py
import pandas as pd
import sml_small.date_adjustment as date_adjust

# Import datafile containing the respondent's data
datafile = "date_adjustment_input_data_example_1.csv"

df = pd.read_csv(datafile)

# Import trading day weights file
trading_day_datafile = "date_adjustment_trading_day_weights_example_1.csv"

trading_df = pd.read_csv(trading_day_datafile)

# Match up column names with with variables below
output = date_adjust.date_adjustment(
    input_dataframe = df, # Respondent's input dataframe
    trading_weights = trading_df, # trading day weights dataframe
    target_columns = ['Q20'], # The variable that needs to be date adjusted
    contributor_returned_start_date_col = "Contributors start date", # Contributor returned start date
    contributor_returned_end_date_col = "Contributors end date", # Contributor returned end date
    expected_start_date_col = "Expected start date", # Expected start date
    expected_end_date_col = "Expected end date", # Expected end date
    domain_col = "Domain", # Domain classification
    short_period_parameter_col = "Short period", # Short period
    long_period_parameter_col = "Long period", # Long period
    equal_weighted_col = "Equal-weighted", # Equal-weighted column
    set_to_mid_point_col = "Mid-point", # Mid-point indicator column
    use_calendar_days_col = "Calendar days", # Calendar days column
    average_weekly_col = "Average Weekly", # Average weekly indicator
    da_error_flag_col = "Error column", # Column for errors to be populated
    trading_date_col = "date", # Date in the trading day dataframe
    trading_period_start_col = "period_start", # Start date column in trading day dataframe
    trading_period_end_col = "period_end", # End date column in trading day dataframe
    trading_weights_col = "weight", # Weight values for each date
    trading_domain_col = "domain" # Domain for each date and weight
    )
                    
# Exporting the output file to csv
output.to_csv("date_adjustment_output_data_example_1.csv")
```

The output can be exported as a csv file and will give you the adjusted responses along with the sum of the weights.



### Additional Information

The ONS Statistical Methods Library at https://statisticalmethodslibrary.ons.gov.uk/ 
contains:
-	Further information about the methods including a link to the GitHub repository which contains detailed API information as part of the method code.
-	Information about other methods available through the library.


### License

Unless stated otherwise, the SML codebase is released under the MIT License. This covers both the codebase and any sample code in the documentation.

The documentation is available under the terms of the Open Government 3.0 license.
