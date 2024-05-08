# Winsorisation User Notes

## Finding and Installing the Method

You can find instructions on downloading and installing the method in the [Help centre](https://statisticalmethodslibrary.ons.gov.uk/help-centre/index) of the [ONS Statistical Methods Library](https://statisticalmethodslibrary.ons.gov.uk)

## Using the Method

### Overview

The Winsorisation method is available as part of the statistical-methods-library package. It takes a dataframe as input that contains the data to be considered for Winsorisation. This method implements one-sided Winsorisation only.

#### Input Data

An example dataset and a walkthrough of the processing is available in a subsequent section.

The input dataframe comprises of rows of data that must include the following fields:

* Reference - String - a unique identifier, e.g Business Reporting Unit
* Period - String - for example "YYYY" or "YYYYMM", the period to be Winsorised
* Cell or Group - String - a string representing a stratum (e.g Standard
Industrial Classification)
* Target Variable - Numeric - the value to be treated
* L-value - Numeric - a parameter necessary for the method to run [see [1]](#1)
* Design Weight - Numeric - a supplied weight that
reflects the sampling design
* Calibration Factor - Numeric - Nulls allowed, a weight that maintains the
estimated calibration totals. Mandatory for ratio estimation.
* Auxiliary - Numeric - Nulls allowed, a secondary value used for prediction
in ratio estimation. Mandatory for ratio estimation.

The presence of Auxiliary and Calibration Factor indicates that ratio
estimation must be performed rather than expansion estimation.

#### Output

The Winsorisation method returns a marker indicating whether a given row of data has been used for Winsorisation or not along with an outlier weight which indicates the reduction in the associated target variable due to Winsorisation.

* Reference - String - a unique identifier, e.g Business Reporting Unit
* Period - String - for example "YYYY" or "YYYYMM", the period to be Winsorised
* Outlier Weight - Numeric - A value between 0 and 1 which reflects the
reduction in a target variable due to Winsorisation
* Marker - String - Indicates the result of the Winsorisation method for
each target value
  * **W** - Winsorised. Marked against any value that could be Winsorised
  * **NW_FE** - Not Winsorised Fully Enumerated. Marked when design weight is
  1 for expansion estimation.
  * **NW_AG** - Not Winsorised a*g. Marked
  in ratio estimation when the product of
  design weight and calibration factor is <= 1.

The methodological specification available at [https://statisticalmethodslibrary.ons.gov.uk/](https://statisticalmethodslibrary.ons.gov.uk/) contains a detailed breakdown of the mathematics used in Winsorisation.

A set of data is assessed to determine if there are any outliers that may adversely affect further processing of the data. Ratio or Expansion estimation is used to calculate a value that is used as a threshold and used with pre-supplied information (e.g L-value) and calculated weights to determine whether a target value must have an outlier weight calculated. The outlier weight can be used to reduce the target value closer to the derived threshold.

#### Expansion Estimation Calculations

Used when values for auxiliary and calibration factor are **not** supplied.

A _weight_ **w** is calculated for each row of data, for expansion estimation this is the supplied design weight

$$ w = \text{design weight} $$

When _weight_ **w** is 1 the marker is set as "NW_FE" and the row is removed from further calculations for Winsorisation.

For a _weight_ **w** not equal to 1, where _target values_ are **y** the _mean target values_, **$\bar{y}$** are calculated as:

$$ \bar{y} = \frac{1}{n} \sum_{i=1}^{n} y_i $$

Using this, a _threshold_ **k** can be calculated to determine whether a _target value_ is an outlier or not:

$$ k = \bar{y} + (\frac{L}{w - 1}) $$

#### Ratio Estimation Calculations

Used when values for auxiliary and calibration factor **are** supplied.

A _weight_ **w** is calculated for each row of data

$$ w = \text{calibration factor} * \text{design weight} $$

When _weight_ **w** <= 1 the marker is set as "NW_AG" and the row is removed from further calculations for Winsorisation.

Note: The summations below are over model groups.

For a _weight_ **w** > 1 the method calculates:

_weighted target_ **ay**

$$ay = \text{design weight} * \text{target value}$$

_sum of weighted targets_ $sy$ is calculated:

$$sy = \sum ay $$

_weighted auxilary_ **ax**

$$ ax = \text{design weight} * auxiliary $$

_sum of weighted auxiliary_ $sx$ is calculated:

$$ sx = \sum ax $$

The _weighted ratio of target and auxiliary_ **wta** is calculated as:

$$ wta = \frac{sy}{sx} $$

A _unit_predict_value_ **mu**, is calculated as:

$$ mu = auxiliary * wta $$

Once these calculations are made a _threshold_ **k** can be calculated to determine whether a _target value_ is an outlier or not:

$$ k = mu + (\frac{L}{w - 1}) $$

### Outlier Weight Calculation

The threshold, **k** derived from either Expansion or Ratio Estimation is then used to determine whether a _target value_ exceeds the computed threshold. Values that exceed the threshold have an _outlier weight_, **o** calculated that is the ratio a value can be reduced by to bring it closer to the threshold. It is this _outlier weight_ value that the Winsorisation method returns.

For each row of data the _target value_ is compared with the calculated _threshold_, **k** and a _modified target value_ **$y^*$** is computed:

When _target value_, **$y$** is <= _threshold_, **k** the _modified target_ equals the _target value_ :

$$ y^* = y $$

When _target value_, **$y$** is > _threshold_, **k** the _modified target_ is calculated as :

$$ y^* = \frac{y}{w} +(1-(\frac{1}{w}))*k $$

The _outlier weight_, **$o$** can then be calculated as:

$$ o = \frac{y^*}{y} $$

Where the _target value_ was unchanged, because it didn't exceed the computed _threshold_, the _outlier weight_, **$o$** will be equal to **1**

## One-sided Winsorisation with Expansion Estimation Example

### Expansion Estimation - Input Data

|reference|period|group|target_value|design_weight|l_value|
|-|-|-|-|-|-|
|123456|202201|1001|50|100|3000|
|345678|202201|1001|55|100|3000|
|234567|202201|1001|60|100|3000|
|456789|202201|1001|40|100|3000|
|567890|202201|1001|45|100|3000|
|678901|202201|1002|120|50|3000|
|789012|202201|1002|110|50|3000|
|890123|202201|1002|115|50|3000|
|901234|202201|1002|125|50|3000|
|123456|202201|1002|700|50|3000|

**Note**: No Auxiliary or Calibration Factor values provided, hence Expansion Estimation

#### Walkthrough

##### Processing for Group 1001

_weight_ is taken from design weight:

$w = 100$

_mean target value_ is calulated:

$\bar{y} = \frac{50+55+60+40+45}{5} = 50$

_threshold_ is calculated using the _mean target value_, _design weight_ and the _L value_:

$k = 50 + (\frac{3000}{(100-1)}) = 80.3030$

All _target values_ are less than the computed _threshold_, k

_modified target value_ equals the _target value_, y

$y^* = y$

for each _target value_, _outlier weight_, o is calculated as **1**

$o = \frac{y^*}{y} =$ e.g $\frac{50}{50} = 1$

As each row was considered for Winsorisation the associated marker is set as "W"

##### Processing for Group 1002

_weight_ is taken from design weight:

$w = 50$

_mean target value_ is calulated:

$\bar{y} = \frac{120+110+115+125+700}{5} = 234$

_threshold_ is calculated using the _mean target value_, _design weight_ and the _L value_:

$k = 234 + (\frac{3000}{(50-1)}) = 295.2245$

Each _target values_ is compared to the _threshold_, k

Where the _target value_ is less than or equal to the _threshold_ the _modified target value_ equals the _target value_, y

$y^* = y$

Where the _target value_ exceeds the _threshold_ the _modified target value_ is calculated using the _weight_ and _threshold_

$y^* = \frac{700}{50} +(1-(\frac{1}{50}))*295.2245 = 303.32$

For each _target value_ that was less than or equal to the _threshold_, _outlier weight_, o is calculated as **1**

$o = \frac{y^*}{y} =$ e.g $\frac{120}{120} = 1$

Where the _target value_ exceeds the _threshold_ _outlier weight_, o is calculated as:

$o = \frac{y^*}{y} = \frac{303.32}{700} = 0.4333$

As each row was considered for Winsorisation the associated marker is set as "W"

### Expansion Estimation - Output Data

|reference|period|outlier_weight|winsorisation_marker|
|-|-|-|-|
|123456|202201|1|W|
|234567|202201|1|W|
|345678|202201|1|W|
|456789|202201|1|W|
|567890|202201|1|W|
|678901|202201|1|W|
|789012|202201|1|W|
|890123|202201|1|W|
|901234|202201|1|W|
|123456|202201|0.4333|W|

## One-sided Winsorisation with Ratio Estimation Example

### Ratio Estimation - Input Data

|reference|period|group|target_value|design_weight|calibration_factor|auxiliary|l_value|
|-|-|-|-|-|-|-|-|
|123456|1900|1|92|50|0.8|100|2368.579001|
|234567|1900|1|500|50|0.8|110|2368.579001|
|345678|1900|1|96|50|0.8|90|2368.579001|
|456789|1900|1|108|50|0.8|105|2368.579001|
|567890|1900|1|101|50|0.8|100|2368.579001|
|678901|1900|1|295|20|0.8|300|2368.579001|
|789012|1900|1|294|20|0.8|290|2368.579001|
|890123|1900|1|288|20|0.8|290|2368.579001|
|901234|1900|1|298|20|0.8|297|2368.579001|
|123450|1900|1|288|20|0.8|305|2368.579001|

#### Processing for Group 1

For each row:
_weight_ is taken from the product of the calibration factor and the design weight

e.g row 1:

$w = 0.8 * 50 = 40$

For rows that have a _weight_ exceeding 1 winsorisation is performed.

_weighted target_, ay is calculated using the _design weight_ and _target value_

$ay = 50 * 92 = 4600$

_weighted auxiliary_, ax is calculated using the _design weight_ and _auxiliary_

$ax = 50 * 100 = 5000$

The _weighted target_ values of the rows considered for Winsorisation are summed to calculate the _sum weighted target values_, sy

$sy = (50 * 92) + (50 * 500) + (50 * 96) + (50 * 108) + (50 * 101) + (20 * 295) + (20 * 294) + (20 * 288) + (20 * 298) + (20 * 288) = 74110$

The _weighted auxiliary_ values of the rows considered for Winsorisation are summed to calculate the _sum weighted auxiliary_, sx

$sx = (50 * 100) + (50 * 110) + (50 * 90) + (50 * 105) + (50 * 100) + (20 * 300) + (20 * 290) + (20 * 290) + (20 * 297) + (20 * 305) = 54890$

The _weighted ratio of target and auxiliary_ **wta** is calculated as:

$wta = \frac{74110}{54890} = 1.350155$

For each row considered for winsorisation, a _unit_predict_value_ **mu**, is calculated as:

e.g for row 1

$mu = 100 * 1.350155 = 135.01548$

e.g for row 2

$mu = 110 * 1.350155 = 148.51705$

Once these calculations are made a _threshold_ **k** can be calculated for each row to determine whether a _target value_ is an outlier or not:

e.g for row 1

$k = 135.01548 + (\frac{2368.579001}{40 - 1}) = 195.74828$

e.g for row 2

$k = 148.51705 + (\frac{2368.579001}{40 - 1}) = 209.24984$

Each _target value_ is compared to the _threshold_, k

Where the _target value_ is less than or equal to the _threshold_ the _modified target value_ equals the _target value_, y

$y^* = y$

Where the _target value_ exceeds the _threshold_ the _modified target value_ is calculated using the _weight_ and _threshold_

e.g row 2 exceeds the threshold:

$y^* = \frac{500}{40} +(1-(\frac{1}{40}))* 209.24984 = 216.51859$

For each _target value_ that was less than or equal to the _threshold_, _outlier weight_, o is calculated as **1**

e.g row 1

$o = \frac{y^*}{y} =$ e.g $\frac{92}{92} = 1$

Where the _target value_ exceeds the _threshold_ _outlier weight_, o is calculated as:

e.g row 2

$o = \frac{y^*}{y} = \frac{216.51859}{500} = 0.433037166$

As each row was considered for Winsorisation the associated marker is set as "W"

### Ratio Estimation - Output Data

|reference|period|outlier_weight|winsorisation_marker|
|-|-|-|-|
|123456|1900|1|"W"|
|234567|1900|0.433037166|"W"|
|345678|1900|1|"W"|
|456789|1900|1|"W"|
|567890|1900|1|"W"|
|678901|1900|1|"W"|
|789012|1900|1|"W"|
|890123|1900|1|"W"|
|901234|1900|1|"W"|
|123450|1900|1|"W"|

## Example Usage

The following is an example of importing and using Winsorisation on a dataframe
setup for ratio estimation (following the example outlined above).

```python
# Imports to allow a spark session to be created and schema defined
from pyspark.sql import SparkSession
from pyspark.sql.types import DecimalType, StringType, StructType, StructField

# Import the winsorisation function from the outliering module
from statistical_methods_library.outliering import winsorisation

# Import the validation error custom exception from the sml module
from statistical_methods_library.utilities.exceptions import ValidationError

# Create a SparkSession
spark = SparkSession.builder.getOrCreate()

# Define the schema of the dataframe
schema = StructType(
    [
        StructField("reference", StringType(), True),
        StructField("period", StringType(), True),
        StructField("group", StringType(), True),
        StructField("target_value", DecimalType(10, 2), True),
        StructField("design_weight", DecimalType(10, 2), True),
        StructField("calibration_factor", DecimalType(10, 2), True),
        StructField("auxiliary", DecimalType(10, 2), True),
        StructField("l_value", DecimalType(10, 2), True),
    ]
)

# Read data from csv file into pyspark dataframe
df = spark.read.csv(
    "winsorisation/example-data/winsorisation_ratio_input.csv",
    header=True,
    schema=schema,
)


# Print schema of the dataframe
df.printSchema()

# Call the winsorisation method (outlier) on the dataframe and
# catch any validation errors
try:
    output = winsorisation.outlier(
        input_df=df,
        reference_col="reference",
        period_col="period",
        grouping_col="group",
        target_col="target_value",
        design_col="design_weight",
        l_value_col="l_value",
        auxiliary_col="auxiliary",
        calibration_col="calibration_factor",
        outlier_col="outlier_weight",
    )

    # Display the output dataframe
    output.show()

except ValidationError as e:
    print("Dataframe validation failed: ", e)

except Exception as e:
    print("An error occurred: ", e)

```

## Example Data

The scenarios outlined above are captured in the csv files under the example-data directory where you found this user guide.

## Additional Information

The ONS Statistical Methods Library at [https://statisticalmethodslibrary.ons.gov.uk/](https://statisticalmethodslibrary.ons.gov.uk/) contains further information about the methods including:

* a methodological specification, which contains further detail about the mathematical definition of the method algorithm
* a link to the github repository which contains detailed API information as part of the method code

## References
<a id="1">[1]</a> [Winsorization for Identifying and Treating Outliers in Business Surveys](https://www.researchgate.net/publication/307632859_Winsorization_for_Identifying_and_Treating_Outliers_in_Business_Surveys) - Ray Chambers, University of Southampton, Philip Kokic, Insiders GmbH and Paul Smith and Marie Cruddas, Office for National Statistics

## License

Unless stated otherwise, the SML codebase is released under the [MIT License](https://github.com/ONSdigital/sml-python-small/blob/main/LICENSE). This covers both the codebase and any sample code in the documentation.

The documentation is available under the terms of the [Open Government 3.0 license](https://github.com/ONSdigital/sml-supporting-info/blob/main/LICENSE).
