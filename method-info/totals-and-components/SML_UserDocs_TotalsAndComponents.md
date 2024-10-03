[comment]: # (File naming convention: SML_UserDocs_<MethodName>_<Lang>.md )
[comment]: # (Lang abbreviates to: Py / PySpk / R )


# Totals and Components in Python 

# Method Description


### Overview

 | Descriptive      | Details                                                  |
 |:---              | :----                                                    |
 | Support Area     | Branch or Expert Group who owns the method               | 
 | Method Theme     | Specify broad group of methods this method belongs to (e.g. imputation, editing, survival analysis, statistical disclosure control) |
 | Status           | Ready to Use / In development / In redevelopment         |
 | Inputs           | Name and description of inputs – high level description  |
 | Outputs          | Name and description of outputs - high level description |
 | Method Version   | Latest version                                           |

### Summary

A few sentences at most describing the method at a high level, to act as the method's "headline statement" on the SML Portal. 
 
This section should include, where relevant: 

- Describe briefly what the method is achieving, assuming no or very little prior knowledge (e.g. measuring the impact of outliers on business surveys). 
- Provide information about the method, including its variants. 
- Outline the strengths and limitations of the method along with explicit scenarios of where it should or shouldn’t be used. 
- Cautions (e.g. distributions of weights should lie within X range) 
- Present concisely all suitable alternative methods (e.g. trimming) or provide links to these methods, if relevant specifications exist in SML. 


### Terminology

This section should contain any terminology specific to this method which it would be useful to define at this stage – this should be statistical terminology, there is a section in the technical user notes to explain implementation-specific terminology.  


### Statistical Process Flow / Formal Definition

This section should: 

- Provide a formal definition of the method (e.g. mathematical notation, formulas used). 
- Describe the logic of the method step-by-step. Depending on circumstances and complexity this could be text only or a flow chart, etc. 

### Assumptions & Vailidity

- List the statistical assumptions upon which the method is based. (e.g. Normality of data) 
- Explain whether the method is still valid if an assumption is violated (if the method is not valid what are the options to overcome this e.g. could a transformation be applied, can violating records be deleted) 

### Worked Example (optional) 

Provide a simple worked through example of the method. This should focus on the mathematics of the method more-so than the code – a work-through of the technical implementation of the method should be included in the user notes. 


### Issues for Consideration (optional) 

Detail any relevant considerations to be aware of that haven’t been covered in the rest of the specification. 


### References (optional) 

This section should contain the list of references used in the specification. These should be publicly available resources accessible outside of ONS. This allows anyone reading the specification to read the reference material. 



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
