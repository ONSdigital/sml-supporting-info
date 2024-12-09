# Thousand Pound Correction User Notes

> [!CAUTION]  
> This is an old version of our User Notes.
> Our current User Guides and example data can be found in the [SML User Docs](https://github.com/ONSdigital/sml-user-docs) repository.

## Finding and Installing the Method

You can find instructions on downloading and installing the method in the [Help centre](https://statisticalmethodslibrary.ons.gov.uk/help-centre/access/run-a-method) of the [ONS Statistical Methods Library](https://statisticalmethodslibrary.ons.gov.uk)

## Using the Method

### Overview

Once you have downloaded the sml-small wheel package you will be able to import the Thousand Pound Correction method and pass in data.

The TPC markers returned determine if and what the method has corrected.

* "S" is when the method stops early or the method is not completed
* "N" is for no correction
* "C" is for correction, where a correction has been applied to the principal variable and selected target variables

The core mathematical correction applied to the principal variable and selected target variables is shown below.

```bash
corrected_value = value / 1000
```
It is worth noting that this correction is only applied if a target variable has been selected for correction. The process by which a target variable is selected for correction is a result of many mathematical and logical steps. Therefore, to get a complete understanding of when the above correction is applied, it is advisable to read the methodology specification.

From the methodology specification we can see that the method accepts a structured input. The input parameters determine how the method operates and what kind of outputs we would expect from it.

### Example Usage

Below is a snapshot of an example dataset for the Thousand Pound Correction method:

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

The method has the following interface:

```python
def thousand_pounds(
    unique_identifier: Optional[str],  # Unique identifier e.g. a question
    # code/ruref/period/id/combination of all of these
    principal_variable: float,  # Original response value provided for
    # the 'current' period
    predictive: Optional[float],  # Value used for 'previous'
    # response (Returned/Imputed/Constructed)
    auxiliary: Optional[float],  # Calculated response for the
    # 'previous' period
    upper_limit: float,  # Upper bound of 'error value' threshold
    lower_limit: float,  # Lower bound of 'error value' threshold
    target_variables: List[TargetVariable], # identifier/value pairs
    precision: Optional[int],  # Precision is used by the decimal
    # package to ensure a specified accuracy
    # used throughout method processing
)
```

Calling the method with the example data:
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

Example output table:

| unique_identifier | principal_final_value | target_variables | tpc_ratio | tpc_marker |
| --- | --- | --- | --- | --- |
| '12340000001-201409-q100' | '5.0E+4' | [TargetVariable(identifier='q101', original_value='500', final_value='0.5'), TargetVariable(identifier='q102', original_value='1000', final_value='1'), TargetVariable(identifier='q103', original_value='1500', final_value='1.5'), TargetVariable(identifier='q104', original_value=None, final_value=None)] | '8.3E+2' | 'C' |

Output attributes:
* unique_identifier - Unique identifier e.g. a question code/ruref/period/id/combination of all of these
* principal_final_value – Output value that may or may not be adjusted
* target_variables - adjusted identifier/value pairs which may or may not be adjusted (returned as TargetVariable objects)
* tpc_ratio – Ratio of the principal variable against good/predictive/aux response
* tpc_marker - 'C' for correction applied, 'N' for no correction applied, 'S' for method stop / error
An example output CSV file has been provided in the example_data folder.

## Pandas Wrapper

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

- You will have to create a new python file importing in the `pandas_wrapper.py`.
- Where you will have to write functions to read a CSV file and pass in the data as a DataFrame into the *`wrapper`* function from the `pandas_wrapper.py` file.


We have an example of how to do this in the `pandas_example.py` file within the `utils` directory.


### Additional Examples

For more examples of using this method with CSV files or by implementing the pandas package see the thousand_pounds source code [documentation page](https://github.com/ONSdigital/sml-python-small/blob/main/sml_small/editing/thousand_pounds).

## Test Data

The [test data mentioned in the example above](https://github.com/ONSdigital/sml-supporting-info/tree/b42cf2d112bee6937efd54171aada5346e4df532/method-info/thousand-pound-correction/example_data) can be found alongside this user documentation.

## Additional Information
The ONS Statistical Methods Library at [https://statisticalmethodslibrary.ons.gov.uk/](https://statisticalmethodslibrary.ons.gov.uk/) contains further information about the methods including:
- a methodological specification, which contains further detail about the mathematical definition of the method algorithm
- a link to the github repository which contains detailed API information as part of the method code

## License
Unless stated otherwise, the SML codebase is released under the [MIT License](https://github.com/ONSdigital/sml-python-small/blob/main/LICENSE). This covers both the codebase and any sample code in the documentation.

The documentation is available under the terms of the [Open Government 3.0 license](https://github.com/ONSdigital/sml-supporting-info/blob/main/LICENSE).
