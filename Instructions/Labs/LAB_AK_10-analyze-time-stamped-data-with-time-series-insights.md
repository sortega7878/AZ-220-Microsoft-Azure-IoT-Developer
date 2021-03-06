---
lab:
    title: 'Explore and analyze time stamped data with Time Series Insights'
    module: 'AZ-220T10-A: Insights and Business Integration'
---

# Explore and analyze time stamped data with Time Series Insights

## Lab Scenario

 Contoso's asset condition tracking revealed a spike in temperature for a specific asset, we want to find the root cause. We want to understand what happened: we correlate the IoT device's sensor data from transportation trucks and planes sensors. The temperature in one of the trucks rose and created the heat spike in one of the transport box equipped with an IoT device with temperature and humidity sensors. Your team will need to be able to explore data in near real time and perform root-cause analysis to help with improving delivery processes to ensure products are kept in the best condition.
 
 You will add Time Series Insights to the solution to quickly store, visualize, and query large amounts of time series data, that are generated by the IoT devices in the trucks, planes, and the transport boxes themselves to visualize changes over time.

## In This Lab

* Verify Lab Prerequisites
* Create an Azure Time Series Insights (TSI) environment
* Create IoT Hub and simulated device (using CLI)
* Connect to IoT Hub with Time Series Insights (TSI)
* Create and deploy TSI resources by using templates with Time Series Insights 

## Exercise 1: Verify Lab Prerequisites

This lab assumes the following resources are available:

| Resource Type | Resource Name |
| :-- | :-- |
| Resource Group | AZ-220-RG |
| IoT Hub | AZ-220-HUB-{YOUR-ID} |
| Device ID | SimulatedThermostat |

1. Using a browser, open the [Azure Cloud Shell](https://shell.azure.com/) and login with the Azure subscription you are using for this course.

1. To ensure the Azure Cloud Shell is using **Bash**, ensure the dropdown selected value in the top-left is **Bash**.

1. To upload the setup script, in the Azure Cloud Shell toolbar, click **Upload/Download files** (fourth button from the right).

1. Before the Azure CLI can be used with commands for working with Azure IoT Hub, the **Azure IoT Extensions** need to be installed. To install the extension, run the following command:

    ```sh
    az extension add --name azure-cli-iot-ext
    ```

1. In the dropdown, select **Upload** and in the file selection dialog, navigate to the **lab-setup.azcli** file for this lab. Select the file and click **Open** to upload it.

    A notification will appear when the file upload has completed.

1. You can verify that the file has uploaded by listing the content of the current directory by entering the `ls` command.

1. To create a directory for this lab, move **lab-setup.azcli** into that directory, and make that the current working directory, enter the following commands:

    ```bash
    mkdir lab9
    mv lab-setup.azcli lab9
    cd lab9
    ```

1. To ensure the **lab-setup.azcli** has the execute permission, enter the following commands:

    ```bash
    chmod +x lab-setup.azcli
    ```

1. To edit the **lab-setup.azcli** file, click **{ }** (Open Editor) in the toolbar (second button from the right). In the **Files** list, select **lab9** to expand it and then select **lab-setup.azcli**.

    The editor will now show the contents of the **lab-setup.azcli** file.

1. In the editor, update the values of the `YourID` and `Location` variables. Set `YourID` to your initials and todays date - i.e. **CAH121119**, and set `Location` to the location that makes sense for your resources.

    > [!NOTE] The `Location` variable should be set to the short name for the location. You can see a list of the available locations and their short-names (the **Name** column) by entering this command:
    >
    > ```bash
    > az account list-locations -o Table
    > ```
    >
    > ```text
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

1. To create a resource group named **AZ-220-RG**, create an IoT Hub named **AZ-220-HUB-{YourID}**, add a device with a Device ID of **SimulatedThermostat**, and display the device connection string, enter the following command:

    ```bash
    ./lab-setup.azcli
    ```

    This will take a few minutes to run. You will see JSON output as each step completes.

1. Once complete, the **Connection Strings** for all 3 IoT Devices, starting with "HostName=", are displayed. Copy all three of these connection strings into a text document, and save for later reference.


## Exercise 2: Setup Time Series Insights

Azure Time Series Insights (TSI) is an end-to-end platform-as-a-service (PaaS) offering used to collect, process, store, analyze, and query data from Internet of Things (IoT) solutions at scale. TSI is designed for ad hoc data exploration and operational analysis of data that's highly contextualized and optimized for time series.

In this exercise, you will setup Time Series Insights (TSI) integration with Azure IoT Hub.

1. Login to [portal.azure.com](https://portal.azure.com) using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. In the Azure Portal, click **+Create a resource** to open the Azure Marketplace.

1. On the **New** blade, in the **Search the Marketplace** box, type in and search for **Time Series Insights**.

1. In the search results, select the **Time Series Insights** item.

1. On the **Time Series Insights** item, click **Create**.

1. On the **Create Time Series Insights** blade, enter `AZ-220-TSI` in the **Environment name** field

1. On the **Resource group** dropdown, select the **AZ-220-RG** resource group.

1. On the **Location** dropdown, select the Azure region used by the Resource group.

1. On the **Tier** field, select the **S1** pricing tier, and leave the **Capacity** at the default of `1`.

1. Click **Next: Event Source**.

1. In the **EVENT SOURCE DETAILS** section, select **Yes** on the **Create an event source?** option.

1. Enter `AZ-220-HUB-{YOUR-ID}` in the **Name** field to specify a unique name for this Event Source.


1. In the **Source type** dropdown, select **IoT Hub**.

1. In the **Select a hub** dropdown, select the **Select existing** option. This will allow you to select an existing IoT Hub that's already been provisioned.

1. In the **IoT Hub name** dropdown, select the `AZ-220-HUB-{YOUR-ID}` Azure IoT Hub service that's already been provisioned.

1. In the **IoT Hub access policy name** dropdown, select the **iothubowner** option.

    In a production environment, it's best practice to create a new _Access Policy_ within Azure IoT Hub to use for configuring Time Series Insights (TSI) access. This will enable the security of TSI to be managed independently of any other services connected to the same Azure IoT Hub.

1. Under the **CONSUMER GROUP** section, click the **New** button next to the **IoT Hub consumer group** dropdown.

1. Enter `tsievents` into the **IoT Hub consumer group** box, and click **Add**.

    This will add a new _Consumer Group_ to use for this Event Source. The Consumer Group needs to be used exclusively for this Event Source, as there can only be a single active reader from a given Consumer Group at a time.

1. Click the **Review + create** button.

1. Click the **Create** button.

    > [!NOTE] Deployment of Time Series Insights (TSI) will take a couple minutes to complete.

1. Once **Time Series Insights** is deployed, navigate to the **AZ-220-TSI** resource.

1. On the **Time Series Insights environment** blade, click on the **Event Sources** link under the **Settings** section.

1. On the **Event Sources**, notice the **AZ-220-HUB-{YOUR-ID}** Event Source in the list. This is the Event Source that was configured when the TSI resource was created.

1. Click the **AZ-220-HUB-{YOUR-ID}** Event Source in the list to view the Event Source details.

1. Notice the configuration of the **Event Source** matches what was set when the Time Series Insights resource was created.

## Exercise 3: Run Simulated IoT Devices

In this exercise, you will run the Simulated Thermostat device so it starts sending telemetry events to Azure IoT Hub.

1. Within **Visual Studio Code**, open the **LabFiles** directory for this unit.

1. Open the **DeviceSimulation.cs** file.

1. Locate the variable for the Container, Truck, and Airplane **Connection Strings**, and change their values to the **Azure IoT Hub Connection Strings** for the IoT Devices registered in IoT Hub.

    ```csharp
    private readonly static string connectionString_Truck = "{Your Truck device connection string here}";
    private readonly static string connectionString_Airplane = "{Your Airplane device connection string here}";
    private readonly static string connectionString_Container = "{Your Container device connection string here}";
    ```

    Be sure to replace the `{Your XXXX device connection string here}` placeholder with the IoT Device Connection String that was copied previously for that IoT Device.

1. Within **Visual Studio Code**, open the **Terminal** pane by clicking the **View** menu, then selecting **Terminal**.

1. Within the **Terminal** pane, navigate to the directory for the `/LabFiles` directory for this unit.

1. Within the **Terminal** execute the following command to build and run the **DeviceSimulation** app.

    ```cmd/sh
    dotnet run
    ```

1. Once the **SimulatedThermostat** device is running, it will begin outputting telemetry data to the Terminal. This will be the telemetry data that it is sending to Azure IoT Hub.

    The **Terminal** output when the **DeviceSimulation** app is running will look similar to the following:

    ```text
    12/27/2019 8:51:30 PM > Sending TRUCK message: {"temperature":35.15660452608195,"humidity":48.422323938240865}
    12/27/2019 8:51:31 PM > Sending AIRPLANE message: {"temperature":17.126545186374237,"humidity":36.46941012936869}
    12/27/2019 8:51:31 PM > Sending CONTAINER message: {"temperature":21.986403302500637,"humidity":47.847680384455096}
    12/27/2019 8:51:32 PM > Sending TRUCK message: {"temperature":36.10474464823629,"humidity":48.82029906486022}
    12/27/2019 8:51:32 PM > Sending AIRPLANE message: {"temperature":16.55005930170971,"humidity":36.49988437459935}
    12/27/2019 8:51:32 PM > Sending CONTAINER message: {"temperature":21.811727088543286,"humidity":50.0}
    ```

1. Leave the **DeviceSimulation** app running for the remaining duration of this lab. This will ensure device telemetry from the three devices (Container, Truck, and Airplane) remain being sent to Azure IoT Hub.

1. Once the **DeviceSimulation** app is running for greater than 30 seconds, you will see output that the **Container** device is changing transport methods between **Truck** and **Airplane** every 30 seconds. The **Terminal** output will look like the following when this happens:

    ```text
    12/27/2019 8:51:40 PM > CONTAINER transport changed to: TRUCK
    ```

    > [!NOTE] In production the shipping container would on change transport methods during the normal course of shipping. For the simulated scenario in this lab, it's performed every 30 seconds to give a short enough data duration that will fit during the course of performing the steps in this lab.

## Exercise 4: View Time Series Insights Explorer

In this exercise, you will be introduced to working with time series data using the Time Series Insights (TSI) Explorer.

1. Login to [portal.azure.com](https://portal.azure.com) using your Azure account credentials.

    If you have more than one Azure account, be sure that you are logged in with the account that is tied to the subscription that you will be using for this course.

1. On your Resource group tile, click the **AZ-220-TSI** Time Series Insights (TSI) resource.

1. On the **Time Series Insights environment** blade, click the **Go to Environment** button at the top.

1. This will open the **Time Series Insights Explorer** in a new browser tab.

1. On the **Analyze** view within TSI Explorer, locate the **MEASURE** dropdown within the box for creating new queries, and select the `temperature` value.

1. Within the **SPLIT BY** dropdown, select the `iothub-connection-device-id` value. This will split the graph to show the telemetry from each of the IoT Devices separately on the graph.

1. Click **Add**.

1. At the top of the **Time Series Insights Explorer**, click the toggle to enable **Auto Refresh**.

    When **Auto refresh** is enabled, the display will be updated every _30 seconds_ to display the latest data. This only applies to the last 1 hour of available data.

1. Notice the graph now displays the **temperature** sensor event data from the IoT Devices within Azure IoT Hub in a _Line Chart_.

1. You can see the list of **Device IDs** to the left of the graph. Hovering the mouse over a specific Device ID will highlight it's data on the graph display.

1. As you watch the graph data auto-refresh as telemetry is streaming into the system from the simulated devices, notice that the spikes in **temperature** of the **ContainerDevice** correlate with the temperature spikes of the **TruckDevice**. This gives you an indication that the ContainerDevice is being transported by Truck at those times.

1. Add another new query, by setting the **MEASURE** dropdown to `humidity`, the **SPLIT BY** dropdown to `iothub-connection-device-id`, and click **Add**.

1. Notice that there are now two graphs displayed; a graph for each of the **temperature** and **humidity** telemetry.

1. Notice that when you hover the mouse cursor over the graph, a popup will display when you hover over the data on the graph. This popup will display the minimum (**min**), average (**avg**), and maximum (**max**) values for the data points in the graph.

    When hovering over the **temperature** graph, the popup will display the **min**, **avg**, and **max** values for the data point under the mouse cursor.

1. Notice you can adjust the time span selection interface bar above the graphs to adjust the time series selection of data to view in the graphs.

Once you have completed exploring the data, don't forget to stop the device simulator app by pressing **CTRL+C** in the terminal.
