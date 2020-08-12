# python_azure_function_step_1

The aim is to have a series of projects which start with a non-trivial python library in VSCode (this step) and then to expose it as an HTTP Triggered Azure Function (step 2), to trigger from and store results to Azure Blob Storage and to have reasonable logging.

The valuation being performed is a simplified Monte Carlo simulation of a portfolio of equity forwards. 

This step is the Python project. The goal is to test the basic setup of a VSCode environment with a realistic python project structure. There is no Azure Functionality at this stage, although some of the setup is done in a particular way because of the following steps. To get this project up and running will require the user to clone the project and open VSCode from the root `python_azure_function_step_1` folder. Once in VSCode, create the virtual environment from the terminal (ctrl+`) and (ensure the terminal points to the root project folder first), run 
```python
python -m venv .venv
.venv\Scripts\activate.ps1 # if replace the .ps1 with .bat if running this from the traditional command prompt
pip install -r requirements.txt
```
Good instructions are available [here](https://code.visualstudio.com/docs/python/python-tutorial#_install-and-use-packages)

The remainder of this document focuses on setting this up manually to run in VSCode. It is intended for someone who is completely new to the development environment and presents some of the concepts that were not obvious to me when I started.

Here I use Python 3 and develop in VSCode. Since I'm a noob, I'm documenting all the stuff I have to google to get up and running. This is here to ensure a common starting point, a python project, with a reasonable project structure, tested in VSCode. That's all. If you can run the tests here from within VSCode, you can move to the next step. One word of caution, I test that functions return exact 'float' values. This is not a good idea in general (perhaps different architectures or versions will have slightly different values) but I wanted to have something that was absolutely precise for me. If your tests fail at the nth decimal, just change the tests expected results!

I start with an example of an existing project / library / code base that we will gradually evolve into an Azure native, serverless calculator using Azure Functions and Azure Blob Storage to perform Monte Carlo simulation and persist the simulated values for later (xVA type) analysis.

Target Project Structure (NB Azure Functions stuff at this point)
```
equityportfolioevolver  
├── .venv/                              # [Not in source control] Python Virtual Environment    
├── .vscode/                            # Local vscode environment variables  
│   ├── launch.json                     # To ensure we can run python from the project root  
│   └── settings.json                   # Settings to get unittest working correctly  
├── equityportfolioevolver/             # Folder for source files that will be deployed to Azure (so excludes testing for example)
│   ├── contracts/                      # Parent folder for groups of derivatives (called portfolios)  
│   │   └── portfolio.py                # Simple unmargined equity forwards (mostly hard coded)  
│   ├── rates/                          # Parent folder for rates environments  
│   │   └── rates_evolver.py            # Uncorrelated Brownian Motion with flat volatility and constant discount rates
├── test/                               # Test folder outside of source folder (will not be packaged and sent to Azure Functions)
│   ├── test_portfolio.py               # Tests for the portfolio code  
│   └── test_rates_evolver.py           # Tests for the rates evolver  
├── .gitignore                          # List of stuff not under source control  
└── requirements.txt                    # packages that are required and will need to be installed in Azure Functions to run your code  
```
**Gotcha No 1: Azure Imports** [You can read the official documentation here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-python#import-behavior). While this step does not get to the Azure parts, the way Azure Functions work will have a profound impact on this project structure (I will move the library or lower level  `equityportfolioevolver` folder into another, intermediate folder). To minimise your trauma, it helps to use relative import statements within the library at this stage so for example, `portfolio.py` imports from `rates_evolver.py`. To avoid re-writing later I use
```python
from ..rates.rates_evolver import RatesEvolver
```
instead of 
```python
from equityportfolioevolver.rates.rates_evolver import RatesEvolver
```
Unfortunately, there is no way to minimise the pain in the test classes! The import statements there will need to change in Step 2.

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

