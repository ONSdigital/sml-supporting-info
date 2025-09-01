# Totals and Components User Notes

> [!CAUTION]  
> This is an old version of our User Notes.
> Our current User Guides and example data can be found in the [SML User Docs](https://github.com/ONSdigital/sml-user-docs) repository.

## Finding and Installing the Method

You can find instructions on downloading and installing the method in the [Help centre](https://statisticalmethodslibrary.ons.gov.uk/help-centre/access/run-a-method) of the [ONS Statistical Methods Library](https://statisticalmethodslibrary.ons.gov.uk)

## Using the Method

### Overview

Once you have downloaded the sml-small wheel package you will be able to import the T&C method and pass in data.

The TCC markers returned determine if and what the method has corrected.

* "S" is when the method stops early or the method is not completed
* "N" is for no correction
* "M" is for manual editing this is where the record needs to be reviewed by a user
* "C" is for component correction where the final components are corrected
* "T" is for totals correction where the final totals are corrected

The core mathematical corrections are total correction and component correction shown below respectively from the methodology specification.

```bash
final_total = sum_of_components
```

and

```bash
final_component = (original_component / sum_of_components) * total
```

It is worth noting these are mutually exclusive and a result of many mathematical and logical steps. Therefore, to get a complete understanding of the above equations it is advisable to read the methodology specification.

From the methodology specification we can see that the method accepts a structured input. The input parameters determine how the method operates and what kind of outputs we would expect from it.

### Example Run Through

Below is a snapshot of an example dataset and how the input data should
look like:

| identifier | total | components | amend_total | predictive | precision | auxiliary | absolute_difference_threshold | percentage_difference_threshold
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 1689 | [632,732,101,165] | False | 1689 | 10 | None | 28 | 0.1 |
| 2 | 0 | [7,0,2,2] | True | 0 | 28 | None | 11 | None |
| 3 | 11 | [0,0,0,0] | False | 11 | 28 | None | 11 | None |
| 4 | 10811 | [9201,866,632,112] | True | 10811 | 28 | None | None | 0.1 |
| 5 | 12492 | [9201,866,632,112] | True | 12492 | 28 | None | None | 0.1 |

This is constructed from the following:

* Unique Identifier – Any e.g. Business Reporting Unit
* Total Variable – Target period total, numeric
* Components Variable – Corresponding list of Total variable's components,
 numeric – nulls allowed
* Amend Total – Select whether Total Variable for the target period should be
 automatically corrected, Boolean
* Predictive Variable – Previous or current period total, numeric
* Precision - The precision value determines the level of accuracy for our floating point calculations
* Auxiliary Variable – optional, numeric – nulls allowed
* Absolute Difference Threshold - represented as a decimal
* Percentage Difference Threshold - represented as a decimal

To run the method using the example data above you can create a python file in your desired IDE and do the following below:

```python
# To import this method we are first navigating to the
# sml_small/editing/totals_and_components directory and
# the totals_and_components python file
# to import the totals_and_components function.
from sml_small.editing.totals_and_components.totals_and_components import totals_and_components

# We can pass our data into the method and save the output to the
# results variable.
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
# module, class, instance or any other object with a
# __dict__attribute
print(vars(result))
```

Running this command will then give you the results from the totals and components area.

### Example Output

The output data is determined by the TCC marker. Some values would be returned as None if they are not calculated.
The output is as follows:

| identifier | absolute_difference | lower_percentage_threshold | upper_percentage_threshold | final_total | final_components | tcc_marker |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | "59" | "1467" | "1793" | "1689" | ["654.8760735" ,"758.4957055", "104.6558282", "170.9723927"] | "C" | <!-- Components have been corrected  -->
| 2 | "11" | "None" | "None" | "11" | ["7", "0", "2", "2"] | "T" | <!-- Total value has been corrected -->
| 3 | "None" | "None" | "None" | "11" | ["0", "0", "0", "0"] | "S" |  <!-- Method has stopped and no outputs returned -->
| 4 | "None" | "9729.9" | "11892.1" | "10811" | ["9201", "866", "632", "112"] | "N" | <!-- No correction has been applied -->
| 5 | "None" | "9729.9" | "11892.1" | "12492" | ["9201", "866", "632", "112"] | "M" | <!-- Manual editing is required -->

The breakdown of the received outputs are as follows:

* Unique Identifier – Any e.g. Business Reporting Unit
* Absolute difference - the absolute difference between the predictive value and the sum of the original components returned as a string, None or NaN type
* Lower percentage threshold - the lower threshold calculated for the percentage range returned as a string, None or NaN type
* Higher percentage threshold - the higher threshold calculated for the percentage range returned as a string, None or NaN type
* Final total - the final total will be corrected if applicable or will remain as the original if not returned as a string, None or NaN type
* Final component - the final components will be corrected if applicable, if not it will remain as original components returned as a string, None or NaN type

### Additional Examples

For more examples of using this method with CSV files or by implementing the pandas package see the totals_and_components source code [documentation page](https://github.com/ONSdigital/sml-python-small/blob/main/sml_small/editing/totals_and_components).

## Test Data

The [test data mentioned in the example above](https://github.com/ONSdigital/sml-supporting-info/blob/main/method-info/totals-and-components/example-data) can be found alongside this user documentation.

## Additional Information

The ONS Statistical Methods Library website [methods page](https://statisticalmethodslibrary.ons.gov.uk/methods) contains further information about the methods including a methodological specification, which contains further detail about the mathematical definition of the method algorithm
and a link to the github repository which contains detailed API information as part of the method code.

## License

Unless stated otherwise, the SML codebase is released under the [MIT License](https://github.com/ONSdigital/sml-python-small/blob/main/LICENSE). This covers both the codebase and any sample code in the documentation.

The documentation is available under the terms of the [Open Government 3.0 license](https://github.com/ONSdigital/sml-supporting-info/blob/main/LICENSE).
