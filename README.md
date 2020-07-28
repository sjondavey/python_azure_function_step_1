# python_azure_function_step_1

The idea for this series of projects is to start with a Visual Studio Code, local Python project and ultimately to expose it as an Azure native, serverless calculator using Azure Functions and Azure Blob Storage. The valuation being performed is an overly simplified Monte Carlo simulation of a portfolio of equity forwards. 

This step is just the base Python project. It can be used to test the basic setup of a VSCode environment with a realistic project structure. There is no Azure Functionality at this stage. To get this project up and running will require the user to create the virtual environment and populate it with the appropriate packages
```python
python -m venv .venv
.venv\Scripts\activate.bat
pip install -r requirements.txt
```
The remainder of this document focuses on setting this up manually to run in VSCode. It is intended for someone who is completely new to the development environment and presents so of the concepts that were not obvious to me when I started.

Here I use Python 3 and develop in VSCode. Since I'm a noob, I'm documenting all the stuff I have to google to get up and running. This is here to ensure a common starting point, a python project, with a reasonable project structure, tested in VSCode. That's all. If you can run the tests here from within VSCode, you can move to the next step. One word of caution, I test that functions return exact 'float' values. This is not a good idea in general (perhaps different architectures or versions will have slightly different values) but I wanted to have something that was absolutely precise for me. If your tests fail at the nth decimal, just change the tests expected results!

Step 1 contains no Azure, it is here as an example of an existing project (code base) that we will gradually evolve into an Azure native, serverless calculator using Azure Functions and Azure Blob Storage to perform Monte Carlo simulation and persist the simulated values for later (xVA type) analysis.

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
If you want to use the `requirements.txt` from this project, the commands are 
```python
python -m venv .venv
.venv\Scripts\activate.bat
pip install -r requirements.txt
```

### Use the virtual environnement in VSCode 
[It is worth reading Microsoft's document](https://code.visualstudio.com/docs/python/environments) if you have not already done so. The trick is to ensure you have the last two lines of this `launch.json` file. The default version of this file created in the Microsoft document does not have this because it is not using the project structure I'm using.
```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Python: Current File",
            "type": "python",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal",
            "env": {"PYTHONPATH": "${workspaceRoot}"},
            "cwd": "${workspaceFolder}"
        }
    ]
}
```
### Setting up VSCode for testing
I use unittest. [Microsoft has a document](https://code.visualstudio.com/docs/python/testing) well worth reading if you have not done so already.

From VSCode `Ctrl+Shft+P` and select `Python: Configure Tests`. I use `unittest Standard test framework`. The default setup of `settings.json` does not need to be changed.

Hopefully that is all you need to run the tests in VSCode (assuming obviously you have Python extension installed. I also use Pylance for linting)

