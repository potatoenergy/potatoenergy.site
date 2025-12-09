---
author: ["Potato Energy Team", "ponfertato"]
categories: ["gpt"]
date: "2023-07-06T11:15:00+03:00"
description: ""
draft: false
series: ["GPT4Free"]
slug: "gpt4free"
tags: ["git", "gpt", "python", "pip"]
title: "GPT4Free Python Version"
---

### GPT4Free is a project which is an attempt to provide the possibility of using the GPT-4 language model for free. GPT-4 (Generative Pre-trained Transformer 4) is an artificial intelligence model developed by OpenAI that is based on the Transformer architecture and is designed to generate text based on given input data

GPT-4 is capable of understanding and generating natural language, has a wide range of knowledge and can help in various tasks such as text generation, answering questions, customer support and others. It learns from huge amounts of text data and is able to understand the context and meaning of a given question or text.

GPT4Free is an initiative that gives you access to basic GPT-4 features without having to buy or sign up for a commercial license.

# Requirements

### Installing Python with PATH environment variable integration on a Windows system, follow these steps

1. Go to the official Python website - [Python.org](https://www.python.org), and download the Windows installer appropriate for your system (32-bit or 64-bit).

2. Run the downloaded Python installer. In the installation window that appears, select "Add Python to PATH" and click the "Customize installation" button to select additional options.

3. In the "Optional Features" window that opens, select the options you want to install (for example, "pip" to install packages) and click "Next".

4. In the "Advanced Options" window, you can choose the Python installation path and other options. If you don't know what to choose, leave the default and click "Install".

5. The installer will copy the Python files to your computer and add Python to the PATH environment variable. You will then see the "Setup was successful" window.

6. Close the Python installer and check if the installation was successful by opening a command prompt and typing `python --version`. If all was successful, you will see the version number of the Python you installed.

You should now have Python installed with integration in the PATH environment variable on your Windows system. You can use the `python` command in the terminal to run the Python interpreter.

### Update Pip on a Windows system with Python already installed, follow these steps

1. open a command prompt by pressing Win + R and typing "cmd", then press Enter.

2. In the terminal, type the following command to update Pip:

![](img/content/blog/gpt/gpt4free/115208.png)

```bash
python.exe -m pip install --upgrade pip
```

3. The Pip update will start and you will see the output of the command about the progress of the update.

4. If the update was successful, you will see a message saying that Pip has been successfully updated to the latest version.

### Update Wheel on a Windows system with Pip already installed, follow these steps

1. Open a command prompt by pressing Win + R and typing "cmd", then press Enter.

2. In the terminal, type the following command to install Wheel:

```bash
pip install --upgrade wheel
```

3. The installation of Wheel will start and you will see the command's output about the progress of the installation.

4. If the installation was successful, you will see a message saying that the Wheel has been successfully installed.

You should now have the latest version of Wheel installed on your Windows system. You can check the version of Wheel by entering the command `pip show wheel` in the terminal.

# Start

## Project setup

![](img/content/blog/gpt/gpt4free/114733.png)
Clone the GPT4Free repository from the GitHub site:

```bash
git clone https://github.com/xtekky/gpt4free.git
```

## Navigate to the project directory

```bash
cd gpt4free
```

## Create a virtual environment to manage Python packages and activate the virtual environment

![](img/content/blog/gpt/gpt4free/115326.png)

```bash
python.exe -m venv venv
```

```bash
.\venv\Scripts\activate
```

## Install the necessary Python packages from the requirements.txt file

![](img/content/blog/gpt/gpt4free/115449.png)

```bash
pip install -r requirements.txt
```

## Create a file test.py in the root folder with the following contents

![](img/content/blog/gpt/gpt4free/160824.png)

```python
#The "import g4f" line imports the "g4f" module, which is a Python module that provides access to GPT4Free functions. By importing this module, you can use the functions and classes defined in the `g4f` module to interact with the GPT-4 language model and perform tasks such as completing chat.
import g4f

#Line `print(g4f.Provider.Ails.params)` prints the Ails provider parameters in the GPT4Free Python project. It accesses the Ails provider params attribute in the Provider module of the g4f package and prints its value.
print(g4f.Provider.Ails.params)

#Line `response = g4f.ChatCompletion.create(model='gpt-3.5-turbo', messages=[{"role": "user", "content": "Hello world"}], stream=True)` creating a chat completion request using the GPT-3.5-turbo model.
response = g4f.ChatCompletion.create(model='gpt-3.5-turbo', messages=[
                                     {"role": "user", "content": "Hello world"}], stream=True)

#Code `for message in reply: print(message)` goes through the `response` object, which is a generator that outputs messages from the chat completion model. It outputs each message to the console.This allows you to process and display the generated messages in your application.
for message in response:
    print(message)

#Line `response = g4f.ChatCompletion.create(model=g4f.Model.gpt_4, messages=[{"role": "user", "content": "hi"}])` creates a chat completion request using the GPT-4 model. It sends a message from the user with the content "hi" and waits for a response from the model. The response variable will store the generated response from the model.
response = g4f.ChatCompletion.create(model=g4f.Model.gpt_4, messages=[
                                     {"role": "user", "content": "hi"}])

#Line «print(response)» prints the response generated by the GPT-4 model. It displays the output of the chat completion request made for the model.
print(response)

#The `response = g4f.ChatCompletion.create(model='gpt-3.5-turbo', provider=g4f.Provider.Forefront,messages=[{"role": "user", "content": "Hello world"} ], stream=True)` line creates a chat completion request using the GPT-3.5-turbo model. It sends a user message with the content "Hello world" and waits for a response from the model. The variable `response` will store the model's generated response.
response = g4f.ChatCompletion.create(model='gpt-3.5-turbo', provider=g4f.Provider.Forefront, messages=[
                                     {"role": "user", "content": "Hello world"}], stream=True)

#Code `for message in reply: print(message)` goes through the `response` object, which is a generator that outputs messages from the chat completion model. It outputs each message to the console. This allows you to process and display the generated messages in your application.
for message in response:
    print(message)
```

# Use

## Start the server

![](img/content/blog/gpt/gpt4free/120039.png)

```bash
python3 -m interference.app
```

The API will be available at: <http://127.0.0.1:1337>

Next, to start GPT4Free, turn on the VPN, open a terminal and execute the three commands below in sequence:

```bash
cd gpt4free
.\venv\Scripts\activate
python3 -m interference.app
```

# Models

<https://github.com/xtekky/gpt4free#gpt-35--gpt-4>

<https://github.com/xtekky/gpt4free#other-models>
