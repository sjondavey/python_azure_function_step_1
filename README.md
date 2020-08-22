# python_azure_function_step_1

Step 1 of a series of projects which start with a non-trivial Python 3 library in VSCode (this step) and then to expose it as an HTTP Triggered Azure Function (step 2), to trigger from and store results to Azure Blob Storage (step 3) and to have reasonable logging.

The valuation being performed is a simplified Monte Carlo simulation of a portfolio of equity forwards. 

Step 1 is the Python project with testing. The goal is to test the basic setup of a VSCode environment with a realistic python project structure. There is no Azure Functionality at this stage, although some of the setup is done in a particular way because of the following steps. To get this project up and running will require the user to clone the project and open VSCode from the root `python_azure_function_step_1` folder. Once in VSCode, create the virtual environment in the root folder from the terminal (ctrl+`) and (ensure the terminal points to the root project folder first), run 
```python
python -m venv .venv
.venv\Scripts\activate.ps1 # if replace the .ps1 with .bat if running this from the traditional command prompt
pip install -r requirements.txt
```
Good instructions are available [here](https://code.visualstudio.com/docs/python/python-tutorial#_install-and-use-packages)

The remainder of this document focuses on setting this up manually to run in VSCode. It is intended for someone who is completely new to the development environment and presents some of the concepts that were not obvious to me when I started.

There were many steps that took me a lot of time to unpack. I am documenting those steps in this project. Step 1 is here to ensure a common starting point, a Python project, with a reasonable project structure, tested in VSCode. That's all. If you can run the tests here from within VSCode, you can move to the next step. One word of caution, I test that functions return exact 'float' values. This is not a good idea in general (perhaps different architectures or versions will have slightly different values) but I wanted to have something that was absolutely precise for me. If your tests fail at the nth decimal, just change the tests expected results!

**Note: __init__.py** Pay attention to the `__init__.py` files. You don't need them in the library and if you have all your tests as files in the `tests/` folder without further folder structure (like in all the Azure examples) but if you want to mimic your library folder structure for the tests, you need to include a blank `__init__.py` in each testing folder - including the root testing folder.

Target Project Structure 
```
python_azure_function_step_1  
├── .venv/                              # [Not in source control] Python Virtual Environment    
├── .vscode/                            # Local vscode environment variables  
│   ├── launch.json                     # To ensure we can run python from the project root  
│   └── settings.json                   # Settings to get unittest working correctly  
├── equityportfolioevolver/             # Folder for source files that will be deployed to Azure (so excludes testing for example)
│   ├── contracts/                      # Parent folder for groups of derivatives (called portfolios)  
│   │   └── portfolio.py                # Simple unmargined equity forwards (mostly hard coded)  
│   ├── rates/                          # Parent folder for rates environments  
│   │   └── rates_evolver.py            # Uncorrelated Brownian Motion with flat volatility and constant discount rates
├── tests/                              # Test folder outside of source folder (will not be packaged and sent to Azure Functions)
│   ├── __init__.py                     # While you don't need __init__.py in the library, you do need them in the tests to ensure the tests are found by VSCode  
│   ├── contracts/                      # Parent folder for contract tests   
│   │   ├── test_portfolio.py           # Tests for the portfolio code 
|   |   └── __init__.py                 # While you don't need __init__.py in the library, you do need them in the tests to ensure the tests are found by VSCode  
│   └── rates/                          # Parent folder for rates tests  
│       ├── rates_evolver.py            # Tests for the rates evolver
|       └── __init__.py                 # While you don't need __init__.py in the library, you do need them in the tests to ensure the tests are found by VSCode  
├── .gitignore                          # List of stuff not under source control  
└── requirements.txt                    # packages that are required and will need to be installed in Azure Functions to run your code  
```
**Note: Relative Imports** [You can read the official documentation here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-python#import-behavior). For more context, see how Microsoft sets up the project structure for multiple functions with common or shared code [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-python#folder-structure). While Step 1 does not get to the Azure parts, the way Azure Functions work will have a profound impact on this project structure. To minimise your trauma, it helps to use relative import statements within the library at this stage so for example, `portfolio.py` imports from `rates_evolver.py`. To avoid re-writing later I use
```python
from ..rates.rates_evolver import RatesEvolver
```
instead of 
```python
from equityportfolioevolver.rates.rates_evolver import RatesEvolver
```
In the test classes, you can use absolute import statements.

### The virtual environment 
Virtual environments are required to ensure Azure can recreate a suitable environnement for the function
```python
python -m venv .venv
.venv\Scripts\activate.bat
pip install numpy
pip install pandas
pip list                           # in case you are interested to see the packages in the virtual environment
pip freeze > requirements.txt      # create the file with installed packages
```

### Use the virtual environnement in VSCode 
[It is worth reading Microsoft's document](https://code.visualstudio.com/docs/python/environments) if you have not already done so. 

### Setting up VSCode for testing
I use unittest. [Microsoft has a document](https://code.visualstudio.com/docs/python/testing) well worth reading if you have not done so already.

From VSCode `Ctrl+Shft+P` and select `Python: Configure Tests`. I use `unittest Standard test framework`. The default setup of `settings.json` does not need to be changed.

Hopefully that is all you need to run the tests in VSCode (assuming obviously you have Python extension installed. I also use Pylance for linting)

### Other references
[Azure Functions Documentation](https://docs.microsoft.com/en-us/azure/azure-functions/)
[Functions Best Practices](https://docs.microsoft.com/en-us/azure/azure-functions/functions-best-practices)