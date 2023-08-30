# Totals and Components User Notes

## Finding and Installing the Method
You can find instructions on downloading and installing the method in the [Help centre](https://statisticalmethodslibrary.ons.gov.uk/help-centre/index) of the [ONS Statistical Methods Library](https://statisticalmethodslibrary.ons.gov.uk)

Open the project in the IDE of choice and run 

```bash
poetry install --sync
```

## Using the Method
Once the totals and components method is available on your computer you will be
able to call the method on a set of data. The tcc markers returned determine
if and what the method has corrected.

* "S" is when the method stops
* "N" is for no correction
* "M" is for manual editing
* "C" is for component correction
* "T" is for totals correction

From the methodology and technical specification we can see that the method accepts a structured input.
The input parameters determine how the method operates and what kind of outputs we would expect from it.

The core mathematical corrections are as total correction and component correction shown below respectively.

```bash
    final_total = sum_of_components
```

and 

```bash
    final_component = (original_component / sum_of_components) * total
```

It is worth noting these are mutually exclusive and a result of many mathematical and logical steps. Therefore, to get a complete understanding of the above equations it is advisable to read the methodical or technical specifications.

Below is a snapshot of an example dataset and how the input data should
look like:

| identifier | total | components | amend_total | predictive | precision | auxiliary | absolute_difference_threshold | percentage_difference_threshold
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 1689 | [632,732,101,165] | False | 1689 | 10 | None | 28 | 0.1 |
| 2 | 0 | [7,0,2,2] | True | 0 | 28 | None | 11 | None |
| 3 | 11 | [0,0,0,0] | False | 11 | 28 | None | 11 | None |
| 4 | 10811 | [9201,866,632,112] | True | 10811 | 28 | None | None | 0.1 |
| 5 | 12492 | [9201,866,632,112] | True | 12492 | 28 | None | None | 0.1 |
To run the method using the example above you can follow these steps:

```python

# Importing the totals_and_components method from the totals_and_components.py file
from totals_and_components import totals_and_components

# The data we are going to pass into the T&C method
data = ["1", 1689, [(632), (732), (101), (165)], False, 1689, 10, None, 28, 0.1]

# We can the totals_and_components function and pass in our data and save the return outputted by the T&C method in the variable result
# We use * to unpack the above list into separate arguments to pass into the T&C method
result = totals_and_components(*data)
```

Note there are other ways to run this method these can be seen [here](https://github.com/ONSdigital/sml-python-small/blob/main/sml_small/editing/totals_and_components/example.py)

The output data is determined by the tcc marker. Some values would be returned as null if they are not calculated.
The output is as follows:

| identifier | absolute_difference | lower_percentage_threshold | upper_percentage_threshold | final_total | final_components | tcc_marker |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | 59 | 1467 | 1793 | 1689 | [654.8760735,758.4957055,104.6558282,170.9723927] | C | <!-- Components have been corrected  -->
| 2 | 11 | None | None | 11 | [7,0,2,2] | "T" | <!-- Total value has been corrected -->
| 3 | None | None | None | 11 | [0,0,0,0] | "S" |  <!-- Method has stopped and no outputs returned -->
| 4 | None | 9729.9 | 11892.1 | 10811 | [9201,866,632,112] | "N" | <!-- No correction has been applied -->
| 5 | None | 9729.9 | 11892.1 | 12492 | [9201,866,632,112] | "M" | <!-- Manual editing is required -->

## Test Data
The test data mentioned in the example above can be found alongside this user documentation

## Additional Information
The ONS Statistical Methods Library at [https://statisticalmethodslibrary.ons.gov.uk/](https://statisticalmethodslibrary.ons.gov.uk/) contains further information about the methods including:
- a methodological specification, which contains further detail about the mathematical definition of the method algorithm
- a link to the github repository which contains detailed API information as part of the method code

## License
Unless stated otherwise, the SML codebase is released under the [MIT License](https://github.com/ONSdigital/sml-python-small/blob/main/LICENSE). This covers both the codebase and any sample code in the documentation.

The documentation is available under the terms of the [Open Government 3.0 license](https://github.com/ONSdigital/sml-supporting-info/blob/main/LICENSE).