# Create a micropython project

```bash
mkdir myproject
cd myproject
```
# Developing micropython

For several tools around micropython development, we also need a normal python installation. I use uv for this. 

```bash
# Install uv globally
brew install uv

# Creates a new project
uv init 

# Install the virtual python environment so that we don't mess up our system python
uv venv

# In case we have an existing project wich already hase some python dependencies in the pyproject.toml file
# we can install them with
uv sync
```

To have nice autocompletion and type checking in vscode we can use the `micropython-unix-stubs` package.

```bash

# Check the version of micropython
$ micropython --version
MicroPython v1.24.1 on 2024-11-29; darwin [GCC 4.2.1] version

# Install the micropython-unix-stubs package
# the version should match with your micropython version
$ uv add micropython-unix-stubs==1.24.1.post2
```


To be able to work with micropython microcontrollers like copying files to the device or executing files on it
we use the `mpremote` utility (a "normal" python package).

```bash
# Install the mpremote package
uv add mpremote
```


# Unittesting with micropython

This approach should help to build code/libraries for micropython that do not depend on 
specific hardware and can be tested in a desktop environment.

The idea is to use the micropython `unittest` module to test the library in a micropython desktop environment.

1) Install micropython so that it is available in the command line.
2) Create a test file that imports the code to test and defines some test methods.
3) Run the test file with micropython.



## Install micropython

On macOS we can use homebrew to easily install micropython. 

I don't know how to install it on other systems but you can definitely build it yourself for Linux and Windows if there is no more convenient way. See [text](https://docs.micropython.org/en/latest/develop/gettingstarted.html#building-the-unix-port-of-micropython)

```bash

# NOTE: this is micropython for your laptop to run micropython code without a microcontroler.
# To run the code on your microcontroller you need to install micropython on the microcontroller itself. 
# Docs for this are in the micropython documentation at https://docs.micropython.org/)
# Also, if you don't want to test/run your micropython code "locally" on your computer, 
# you don't need to install micropython on your computer just on the microcontroller.

# Micropython for macOS using homebrew 
brew install micropython
```


## Install the unittest module

Because the unittest module should test the micropython code with the micropython interpreter, we need to install the `unittest` module in the micropython environment and **NOT** in the "normal" python environment with ~~uv add~~.

```bash
# create a directory for the micropython libraries that we don't need on the microcontroller 
# but just locally on our computer for development/testing
mkdir lib-dev

# install the unittest module in the lib-dev directory
micropython -m mip install unittest --target lib-dev
```

## Create some code

Let's create a simple function that we want to test.

```bash
mkdir src
touch src/mycode.py

```python
# src/mycode.py
def add(a, b):
    return a + b
```


## Create the test file

```bash
mkdir test
touch test/test_mycode.py
```

```python
# test/test_mycode.py
import sys
# add the dev libs with the unittest module to the python path
sys.path.append('lib-dev')
# also add our src code to the python path
sys.path.append('src')

import unittest
import mycode

class TestMyCode(unittest.TestCase):
    def test_add(self):
        self.assertEqual(mycode.add(1, 2), 3)

if __name__ == '__main__':
    unittest.main()
```

## Run the tests

```bash
micropython test/test_mycode.py
```


## If we want to only test a specific test case

**NOTE**: This is not working yet when you follow the steps above. But in this repo I have implemented it.

```bash
micropython test/test_mycode.py test_mycode.TestMyCode.test_add
```


