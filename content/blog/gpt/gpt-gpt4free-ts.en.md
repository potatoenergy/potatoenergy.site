---
author: ["Potato Energy Team", "ponfertato"]
categories: ["gpt"]
date: "2023-07-06T17:20:00+03:00"
description: ""
draft: false
series: ["GPT4Free"]
slug: "gpt4free-ts"
tags: ["git", "gpt", "typescript", "yarn"]
title: "GPT4Free TypeScript Version"
---

### GPT4Free-TS is a project which is an attempt to provide the ability to use the GPT-4 language model for free. GPT-4 (Generative Pre-trained Transformer 4) is an artificial intelligence model developed by OpenAI that is based on the Transformer architecture and is designed to generate text based on given input data

GPT-4 is capable of understanding and generating natural language, has a wide range of knowledge and can help in various tasks such as text generation, answering questions, customer support and others. It learns from huge amounts of text data and is able to understand the context and meaning of a given question or text.

GPT4Free is an initiative that gives you access to basic GPT-4 features without having to buy or sign up for a commercial license.

The TS version is a fork of the original GPT4Free project, using Yarn for building, instead of Python.

# Requirements

### Install Node.js on a Windows system, follow these steps

1. Go to the official Node.js website at [Nodejs.org](https://nodejs.org/), select "Download" and download the Windows installer appropriate for your system (32-bit or 64-bit).

2. Run the downloaded Python installer. In the installer window, select Install for the standard Node.js installation, or select Customize to customize various installation options.

If you choose the standard installation, simply follow the installer instructions until the installation is complete. If you chose a customized installation, you can select additional components and settings before installation.

4. Once the installation is complete, open a command prompt by pressing Win + R and typing "cmd". Press Enter.

5. Enter the command `node -v` to check the version of Node.js. If the installation was successful, you will see the current version of Node.js.

Node.js is now installed on your Windows system. You can use it to run JavaScript applications.

### Update Yarn on your Windows system with Node.js already installed

1. Open a command prompt by pressing Win + R and typing "cmd", then press Enter.

2. In the terminal, type the following command to update Yarn:

![](img/content/blog/gpt/gpt4free-ts/132707.png)

```bash
npm install --global yarn
```

1. The Yarn update will start and you will see the command output about the progress of the update.

2. If the update was successful, you will see a message saying that Yarn has been updated successfully.

You should now have the latest version of Yarn installed on your Windows system. You can check the version of Yarn by entering the command `yarn --version` in the terminal.

### Setting up a workload for Node.js development in Visual Studio involves the following steps

1. Install Visual Studio on your computer, following the official Microsoft documentation.

2. open Visual Studio and, in the Workloads tab, select the Node.js Development option.

![](img/content/blog/gpt/gpt4free-ts/134508.png)

3. In the "Installation Details" section of the window that appears, select "MSVC versions...".

4. To install the modules and packages, click "Change" and wait for the process of downloading and installing all the necessary packages.

You now have a workload for developing Node.js in Visual Studio in the future action we will need it.

### Update node-gyp on a Windows system with Node.js and MSVC already installed, do the following steps

1. open a command prompt by running "cmd.exe" or "Powershell" on your computer.

2. Check that you have MSVC installed . You can do this by running the command:

```bash
npm config list
```

Make sure that the "msvs_version" parameter is present.

5. Update node-gyp to the latest version by running the following command:

```bash
npm install -g node-gyp
```

You should now have an updated version of node-gyp on your Windows system.

# Start

## Project setup

Clone the GPT4Free-TS repository from GitHub:

![](img/content/blog/gpt/gpt4free-ts/132335.png)

```bash
git clone https://github.com/xiangsx/gpt4free-ts.git
```

## Navigate to the project directory

```bash
cd gpt4free-ts
```

## Create a .env file in the root folder with the following contents

![](img/content/blog/gpt/gpt4free-ts/133030.png)

```env
#EMAIL_TYPE=tempmail-lol in the .env file sets the EMAIL_TYPE environment variable to tempmail-lol. This variable is used in the GPT4Free-TS project to specify the type of email service used to create temporary email addresses. In this case it is set to `tempmail-lol`, which, is the specific email service provider supported by the project.
EMAIL_TYPE=tempmail-lol
#`DEBUG=0`in the .env file sets the value of the`DEBUG`environment variable to`0`. This variable is used in the GPT4Free-TS project to control debug output. Setting it to `0` means that debug output is disabled.
DEBUG=0
#`POOL_SIZE=0` in the .env file is used to set the number of worker processes to be created by the GPT4Free-TS server.
POOL_SIZE=0
#`PHIND_POOL_SIZE=0` in the .env file is used to set the number of worker processes to be created by the GPT4Free-TS server. Setting it to "0" means no worker processes will be created.
PHIND_POOL_SIZE=0
```

## Install the necessary Yarn packages

![](img/content/blog/gpt/gpt4free-ts/140258.png)

```bash
yarn
```

# Use

## Start the server

![](img/content/blog/gpt/gpt4free-ts/140332.png)

```bash
yarn start
```

The API will be available at: <http://127.0.0.1:3000>

Next, to start GPT4Free-TS, turn on your VPN, open a terminal and run the two commands below in sequence:

```bash
cd gpt4free-ts
yarn start
```

# Models

<https://github.com/xiangsx/gpt4free-ts#site-support-model->
