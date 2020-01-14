---
lab:
    title: 'Lab 04: Connect IoT Device to Azure'
    module: 'Module 2: Devices and Device Communication'
---

# Connect IoT Device to Azure

## Lab Scenario

Contoso is known for producing high quality of cheeses. Due to the company rapidly growing in popularity and sales, they want to ensure that their cheeses stay at the same level of quality. At the moment, a worker temperature and humidity data is collected by floor workers every shift.

Contoso is exploring adding an IoT device to monitor the temperature and humidity of their batches of cheeses. For the asset monitoring solution, you will be connecting an IoT device with temperature and humidity sensors (temperature, humidity) to IoT Hub.

## In This Lab

In this lab, you will do the following:

* Verify Lab Prerequisites
* Register a Device ID in Azure IoT Hub using the Azure CLI.
* You will then configure and run a pre-built Simulated Device written in C# to connect to Azure IoT Hub and send Device-to-Cloud telemetry messages.
* Verify the device telemetry is being received by Azure IoT Hub using the Azure CLI.

## Exercise 1: Verify Lab Prerequisites

This lab assumes the following resources are available:

| Resource Type | Resource Name |
| :-- | :-- |
| Resource Group | AZ-220-RG |
| IoT Hub | AZ-220-HUB-_{YOUR-ID}_ |

If the resources are unavailable, please execute the **lab-setup.azcli** script before starting the lab.

> [!NOTE] The **lab-setup.azcli** script is written to run in a **bash** shell environment - the easiest way to execute this is in the Azure Cloud Shell.

1. Using a browser, open the [Azure Cloud Shell](https://shell.azure.com/) and login with the Azure subscription you are using for this course.

1. If you are prompted about setting up storage for Cloud Shell, accept the defaults. 

1. To ensure the Azure Shell is using **Bash**, ensure the dropdown selected value in the top-left is **Bash**.

1. To upload the setup script, in the Azure Shell toolbar, click **Upload/Download files** (fourth button from the right).

1. In the dropdown, select **Upload** and in the file selection dialog, navigate to the **lab-setup.azcli** file for this lab. Select the file and click **Open** to upload it.

    A notification will appear when the file upload has completed.

1. You can verify that the file has uploaded by listing the content of the current directory by entering the `ls` command.

1. To create a directory for this lab, move **lab-setup.azcli** into that directory, and make that the current working directory, enter the following commands:

    ```bash
    mkdir lab4
    mv lab-setup.azcli lab4
    cd lab4
    ```

1. To ensure the **lab-setup.azcli** has the execute permission, enter the following commands:

    ```bash
    chmod +x lab-setup.azcli
    ```

1. To edit the **lab-setup.azcli** file, click **{ }** (Open Editor) in the toolbar (second button from the right). In the **Files** list, select **lab4** to expand it and then select **lab-setup.azcli**.

    The editor will now show the contents of the **lab-setup.azcli** file.

1.  In the editor, update the values of the `{YOUR-ID}` and `{YOUR-LOCATION}` variables. Set `{YOUR-ID}` to the Unique ID you created at the start of this course - i.e. **CAH191211**, and set `{YOUR-LOCATION}` to the location that matches your resource group.

    ```bash
    #!/bin/bash

    RGName="AZ-220-RG"
    IoTHubName="AZ-220-HUB-{YOUR-ID}"

    Location="{YOUR-LOCATION}"
    ```

    > [!NOTE] The `{YOUR-LOCATION}` variable should be set to the short name for the region where you are deploying all of your resources. You can see a list of the available locations and their short-names (the **Name** column) by entering this command:

    > ```bash
    > az account list-locations -o Table
    >
    > DisplayName           Latitude    Longitude    Name
    > --------------------  ----------  -----------  ------------------
    > East Asia             22.267      114.188      eastasia
    > Southeast Asia        1.283       103.833      southeastasia
    > Central US            41.5908     -93.6208     centralus
    > East US               37.3719     -79.8164     eastus
    > East US 2             36.6681     -78.3889     eastus2
    > ```

1. To save the changes made to the file and close the editor, click **...** in the top-right of the editor window and select **Close Editor**.

    If prompted to save, click **Save** and the editor will close.

    > [!NOTE] You can use **CTRL+S** to save at any time and **CTRL+Q** to close the editor.

1. To create a resources required for this lab, enter the following command:

    ```bash
    ./lab-setup.azcli
    ```

    This will take a few minutes to run. You will see JSON output as each step completes.

Once the script has completed, you will be ready to continue with the lab.

## Exercise 2: Create Azure IoT Hub Device ID using Azure CLI

The `iot` Azure CLI modules includes several commands for managing IoT Devices within Azure IoT Hub under the `az iot hub device-identity` command group. These commands can be used to manage IoT Devices within scripts or directly from the command-line / terminal.

### Task 1: Create the IoT Hub Device ID

1. If necessary, log in to your Azure portal using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. At the top of the Azure Portal click on the **Cloud Shell** icon to open up the **Azure Cloud Shell** within the Azure Portal. 

1. If you are prompted about setting up storage for Cloud Shell, accept the defaults. 

1. When the pane opens, choose the option for the **Bash** terminal within the Cloud Shell.

1. Within the Cloud Shell, run the following command to ensure the Cloud Shell has the IoT extension installed.

    ``` sh
    az extension add --name azure-cli-iot-ext
    ```

1. Still within the Cloud Shell, run the following Azure CLI command to create **Device Identity** in Azure IoT Hub that will be used for a Simulated Device.

    ```sh
    az iot hub device-identity create --hub-name {IoTHubName} --device-id SimulatedDevice1
    ```

    > [!NOTE] Be sure to replace the _{IoTHubName}_ placeholder with the name of your Azure IoT Hub. If you have forgotten your IoT Hub name, you can enter the following command:
    >
    >```sh
    >az iot hub list -o table
    >```

### Task 2: Get the Device Connection String

1. Within the Cloud Shell, run the following Azure CLI command to get _device connection string_ for the Device ID that was just added to the IoT Hub. This connection string will be used to connect the Simulated Device to the Azure IoT Hub.

    ```cmd/sh
    az iot hub device-identity show-connection-string --hub-name {IoTHUbName} --device-id SimulatedDevice1 --output table
    ```

1. Make note of the **Device Connection String** that was output from the previous command. You will need to save this for use later.

    The connection string will be in the following format:

    ```text
    HostName={IoTHubName}.azure-devices.net;DeviceId=SimulatedDevice1;SharedAccessKey={SharedAccessKey}
    ```

## Exercise 3: Configure and Test a Simulated Device (C#) 

In this exercise you will configure a simulated device written in C# to connect to Azure IoT Hub using the Device ID and Shared Access Key created in the previous exercise. You will then test the device and ensurethat IoT Hub is receiving telemetry from the device as expected.

### Task 1: Open the C# Code Project

1. Using **Visual Studio Code**, open the `/home/luckposht/AZ-220-Microsoft-Azure-IoT-Developer/Allfiles/Labs/04-Connect IoT Device to Azure/Starter/` folder.

    Check with your instructor to locate the code project, either in the GitHub folder or on the Host PC.

### Task 2: Update the Device Connection String

1. Open the `SimulatedDevice.cs` file.

1. Locate the `s_connectionString` variable, and replace the value placeholder `{Your device connection string here}` with the **Device Connection String** that was copied previously. This will enable the Simulated Device to authenticate, connect, and communicate with the Azure IoT Hub.

    Once configured, the variable will look similar to the following:

    ```csharp
    private readonly static string s_connectionString = "HostName={IoTHubName}.azure-devices.net;DeviceId=SimulatedDevice1;SharedAccessKey={SharedAccessKey}";
    ```

1. In Visual Studio Code, click on the **View** menu, then click **Terminal** to open the _Terminal_ pane.

1. Run the following command within the **Terminal** to build and run the Simulated Device application. Be sure the terminal location is set to the directory with the `SimulatedDevice.cs` file.

    ```cmd/sh
    dotnet run
    ```

    > [!NOTE] If the command outputs a `Malformed Token` or other error message, then make sure the **Device Connection String** is configured correctly as the value of the `s_connectionString` variable.

1. Once the Simulated Device application is running, it will be sending event messages to the Azure IoT Hub that include `temperature` and `humidity` values.

    The terminal output will look similar to the following:

    ```text
    IoT Hub C# Simulated Device. Ctrl-C to exit.

    10/25/2019 6:10:12 PM > Sending message: {"temperature":27.714212817472504,"humidity":63.88147743599558}
    10/25/2019 6:10:13 PM > Sending message: {"temperature":20.017463779085066,"humidity":64.53511070671263}
    10/25/2019 6:10:14 PM > Sending message: {"temperature":20.723927165718717,"humidity":74.07808918230147}
    10/25/2019 6:10:15 PM > Sending message: {"temperature":20.48506045736608,"humidity":71.47250854944461}
    10/25/2019 6:10:16 PM > Sending message: {"temperature":25.027703996760632,"humidity":69.21247714628115}
    10/25/2019 6:10:17 PM > Sending message: {"temperature":29.867399432634656,"humidity":78.19206098010395}
    10/25/2019 6:10:18 PM > Sending message: {"temperature":33.29597232085465,"humidity":62.8990878830194}
    10/25/2019 6:10:19 PM > Sending message: {"temperature":25.77350195766124,"humidity":67.27347029711747}
    ```

### Task 3: Verify Telemetry Stream sent to Azure IoT HUb

In this task, you will use the Azure CLI to verify telemetry sent by the simulated device is being received by Azure IoT Hub.

1. Run the following command in the **Azure Cloud Shell** (or a different command-line window), to view a stream of the event messages being sent to the Azure IoT Hub endpoint by the Simulated Device.

    ```cmd/sh
    az iot hub monitor-events --hub-name {IoTHubName} --device-id SimulatedDevice1
    ```

    _Be sure to replace the **{IoTHubName}** placeholder with the name of your Azure IoT Hub._

    > [!NOTE] If you receive a message stating _"Dependency update required for IoT extension version"_ when running the Azure CLI command, then press `y` to accept the update and press `Enter`. This will allow the command to continue as expected.

    The `--device-id` parameter is optional and allows you to monitor the events from a single device. If the parameters is omitted, the command will monitor all events sent to the specified Azure IoT Hub.

    The `monitor-events` command within the `az iot hub` Azure CLI module offers the capability to monitor device telemetry & messages sent to an Azure IoT Hub from within the command-line / terminal.

2. The `az iot hub monitor-events` Azure CLI command will output a JSON representation of the events that are being sent to the Azure IoT Hub. This command allows you to monitor the events being sent, and verify the device is able to connect to and communicate with the Azure IoT Hub.

    ```cmd/sh
    Starting event monitor, filtering on device: SimulatedDevice1, use ctrl-c to stop...
    {
        "event": {
            "origin": "SimulatedDevice1",
            "payload": "{\"temperature\":25.058683971901743,\"humidity\":67.54816981383979}"
        }
    }
    {
        "event": {
            "origin": "SimulatedDevice1",
            "payload": "{\"temperature\":29.202181296051563,\"humidity\":69.13840303623043}"
        }
    }
    ```

3. When finished, you can press `Ctrl-C` in both windows to stop monitoring telemetry & messages being sent to Azure IoT Hub.
