# sap-fiori-tools-offline-instructions

Guide to use SAP Fiori tools offline.

## Description

The SAP Fiori tools team recently received customer requests asking for instructions on how to work with SAP Fiori tools, when no internet connection is present. This repository addresses this request by providing information on how to setup such an environment.

To run the steps below, you'll need a 'connected system' which has internet connection and the 'isolated system' that will host the development environment without connection to the internet. Ideally, both systems, the connected system and the isolated system, are similar in terms of operating system and architecture (32bit/64bit).

## Assumptions

Desire to run SAP Fiori tools on an isolated system with following features:

- Windows 10 image
- No internet connection
- User with admin rights
- Security software that cannot be overwritten
- Only USB-like options can be used to put installation files and modules on the image

## Preparation on connected system

### Node.js

Download Node.js version 14.x.y from https://nodejs.org/en/download/releases/. At the time of writing this document this was https://nodejs.org/download/release/v14.20.0/.
Install node on the connected system, but keep a copy, you will later need to copy the installer also to the isolated system and install it there. In case you do not have permissions to install software on the connected system, you can also download and extract the zip archive instead of the msi installer and do a manual setup[^1].

[^1]:
    Instructions to install node without admin permissions can be found for instance at:  
    https://zwbetz.com/install-nodejs-on-windows-without-admin-access/  
    https://stackoverflow.com/questions/37029089/how-to-install-nodejs-lts-on-windows-as-a-local-user-without-admin-rights

### Install Verdaccio

Verdaccio is a lightweight node module registry that runs locally and we can use it to collect node modules on the connected system and later to host the module registry in the isolated system. Here, in the preparation, we install Verdaccio as local module in a new folder. This allows us to copy the installation later to the isolated system. To install it, run the commands:

```
>mkdir verdaccio
>cd verdaccio
>npm install --no-optional verdaccio
```

This step might take a while and will create a folder `node_modules` in the newly created folder `verdaccio`. Add all the folders in `node_module\*` to a zip archive named `verdaccio.zip`, we need to copy this later to the isolated system. Full installation instructions for Verdaccio can be found at https://verdaccio.org.

Now that we have Verdaccio installed, we can run it to collect and store node modules we need later for SAP Fiori tools.

Execute following command to run Verdaccio:

```shell
>node_modules\.bin\verdaccio
warn --- config file -C:\?\AppData\Roaming\verdaccio\config.yaml
warn --- Plugin successfully loaded: verdaccio-htpasswd
warn --- Plugin successfully loaded: verdaccio-audit
warn --- http address - http://localhost:4873/ - verdaccio/5.8.0
```

In the terminal you started Verdaccio you can see the host and port on which it runs, default is `localhost:4873`, and the path to the configuration file `config.yaml`. The config file defines the storage location, default is `storage` folder next to the config file. Both information, host:port and storage location are important in subsequent steps.

### Cache required modules

In order to get copies of all required node modules for development using SAP Fiori tools, you need to install them once through the Verdaccio registry we just installed.

But before, we need to make sure the the cache for node modules is empty, otherwise packages might be taken from the local npm cache and not copied to the Verdaccio storage. To clear the cache run command

```shell
npm cache clear --force
```

If you are using other node package managers, you might need to clean their cache as well. I use yarn and had to run `yarn cache clean` to avoid local cached modules being used.

Now we need to do the installation of node modules in order to fill up the Verdaccio storage. The list of all required node modules can be found at [dependencies.json](dependencies.json), but for convenience you'll find the command as one long string below. If you are still in `verdaccio` folder in terminal you should go one folder up. Now create a new temporary folder and switch to it, e.g.

```shell
mkdir sap-fiori-tools-modules
cd sap-fiori-tools-modules
```

In the terminal in folder `sap-fiori-tools-modules` run command to install all required modules:

```shell
npm install @sap/abap-deploy@^0.10.27 @sap/cds-compiler@3.1.0 @sap/eslint-plugin-ui5-jsdocs@2.0.5 @sap/generator-fiori@latest @sap/ux-specification@latest @sap-ux/ui5-middleware-fe-mockserver@2 @sap/ux-ui5-tooling@1 @sapui5/distribution-metadata@latest @sapui5/ts-types@latest @ui5/builder@^2.11.5 @ui5/cli@^2.14.10 @ui5/fs@^2.0.6 @ui5/logger@^2.0.1 @ui5/project@^2.6.0 @ui5/server@2.4.0 chokidar@3.5.3 rimraf@3.0.2 yo@^4.3.0 --registry=http://localhost:4873/
```

This assumes your Verdaccio is listening to http://localhost:4873/, if this is not the case, please adjust accordingly.

The storage folder should now be filled with all requested modules. Create an zip archive `storage.zip` of the storage folder, default location on Windows is:

`C:\Users\<USER_ID>\AppData\Roaming\verdaccio\storage`

You will need to transfer this archive later to the isolated system.

### Create a portable version of Visual Studio Code

Visual Studio Code can be setup in [portable mode](https://code.visualstudio.com/docs/editor/portable), which means settings like installed extensions or configurations are stored in the `data` folder next to the executable rather than in a user specific folders. This allows to copy and transfer the Visual Studio Code installation.

To do so, download the `.zip` file for your operating system and CPU from https://code.visualstudio.com/download and store it on your connected system. Extract the archive and create a `data` folder next to the code executable. If have troubles or question to these steps, navigate to https://code.visualstudio.com/docs/editor/portable and follow the instructions.

Once you've created the `data` folder you can start the `code` executable. Inside Visual Studio code, open the 'Extensions' and search and install 'SAP Fiori Tools - Extension Pack'. After the extension pack is installed zip the `VSCode` folder into `vscode.zip`, you'll need to copy it in following step to the isolated environment.

## Transfer to isolated environment

Copy the downloaded and prepared files to the isolated environment:

- Node.js 14 installation file
- `verdaccio.zip` that contains the modules to run Verdaccio
- `storage.zip` that contains the `storage` of Verdaccio
- `vscode.zip` containing Visual Studio Code portable mode with SAP Fiori tools extension pack

## Setup in isolated environment

### Node.js

Install Node.js 14 using the downloaded installer.
After installing node you can check the version by running following command in terminal

```shell
>node --version
v14.19.1
```

## Install and setup Verdaccio

We are now in the situation, that we need to install Verdaccio, which is a node module, but without connection to the internet. That is why we created the `verdaccio.zip` which contains the required modules. First we need to find the path where global node modules are installed. You can get this information by running following command in terminal:

```shell
>npm -g root
C:\Users\?\AppData\Roaming\npm\node_modules
```

Extract the folders from `verdaccio.zip` into the folder that is shown in the terminal after you execute the command. For instance, folder `verdaccio` from archive `verdaccio.zip` should be extracted to `C:\Users\<USER_ID>\AppData\Roaming\npm\node_modules\verdaccio`

After installing Verdaccio, run it using command (replace `?` with user id):

```shell
>C:\Users\?\AppData\Roaming\npm\node_modules\.bin\verdaccio
```

Same as when we prepared the node modules, you can find information about the URL and config file printed out in the terminal.

In the isolated system we are using Verdaccio as a local node registry providing the node modules. Copy and extract zip file that contains the `storage` into the location mentioned in the config file, by default this is:

`C:\Users\<USER_ID>\AppData\Roaming\verdaccio\storage`

Set the global setting for npm registry to point to the Verdaccio registry using command

```shell
>npm config set registry=http://localhost:4873/ -g
```

### Install global node modules

SAP Fiori tools includes templates to generate new SAP Fiori applications. These templates are provided as node module and should be in the Verdaccio storage that we copied.

Install the SAP Fiori tools application generator node module globally by executing command

```shell
>npm install -g @sap/generator-fiori
```

### Extract and run Visual Studio Code

Copy and extract the zip file that contains the portable version of Visual Studio Code we have prepared in one of the previous steps to a location of your choice in the isolated system and start `<EXTRACT_LOCATION>\Code.exe`.

To avoid error messages due to telemetry data, you can disable telemetry by opening the command palette and executing command `Fiori: Change Telemetry Settings` -> `Disable`.

Now you should be able to generate and develop projects using SAP Fiori tools. In Visual Studio Code you can test this by opening command palette (menu 'View' -> 'Command Palette...') and enter 'Open Template Wizard'.

## What is still to do

- Configure `ui5.yaml` in generated project to point to the SAP backend. By default UI5 is loaded from https://ui5.sap.com/.
- Download additional versions (with tags?) of specification.
