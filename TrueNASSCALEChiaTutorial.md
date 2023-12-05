---
title: "Installing TrueNAS SCALE Chia Application"
description: "Provides basic installation instructions for the TrueNAS SCALE Chia application using both the SCALE web UI."
weight: 
---

The TrueNAS SCALE Chia app installs the Chia Blockchain architecture in a Kubernetes container.
Chia Blockchain is a cryptocurrency ecosystem that uses Proof of Space and Time, and allows users to work with digital money and interact with their assets and resources.
Instead of using expensive hardware that consumes exorbitant amounts of electricity to mine crypto, Chia leverages existing empty hard disk space on your computer(s) to farm crypto with minimal resources.

## First Steps
Before you install the SCALE Chia application, you have the option to create the two datasets, **config** and **plots**, for the Chia app storage volumes, or you can allow SCALE to automatically create these storage volumes during the installation process.

You also have the option to mount additional datasets for other Chia storage needs inside the container pod.
Either allow SCALE to create these storage volumes or create and name datasets according to your intended use or as sequentially-named datasets (i.e., *volume1*, *volume2*, etc.) for each extra volume to mount inside the container.

To use existing datasets and the host path option, create all datasets before beginning the SCALE app installation process.
See [Creating a Dataset](https://www.truenas.com/docs/scale/scaletutorials/storage/datasets/datasetsscale/) for information on correctly configuring datasets.

## Installing the SCALE Chia App
To install the SCALE Chia app:
1. [Configure and deploy the SCALE Chia app](#deploying-the-scale-chia-app) in the Kubernetes container.
2. [Obtain the authentication keys](#obtaining-and-preserving-keys) from Chia and make the keys persist across container reboots.
3. [Add the keys to the SCALE Chia app](#adding-keys-to-the-scale-chia-app).
3. Setup the [Chia GUI](#setting-up-the-chia-gui) or Chia command line (CLI) to configure Chia and start farming.

### Deploying the SCALE Chia App
Log into TrueNAS SCALE, go to **Apps**, click on **Discover Apps**, then either begin typing Chia into the search field or scroll down to locate the **Chai** application widget.

![DiscoverScreenChiaApp](/static/img/truenas/DiscoverScreenChiaApp.png)

Click on the widget to open the **Chia** app information screen.

![ChiaAppDetailsScreen](/static/img/truenas/ChiaAppDetailsScreen.png)

Click **Install** to open the **Install Chia** configuration screen.

![InstallChiaScreen](/static/img/truenas/InstallChiaScreen.png)

Application configuration settings are grouped in several sections, each of which are explained in [Understanding SCALE Chia App Settings](#understanding-scale-chia-app-settings) below.
To find specific fields click in the **Search Input Fields** search field, scroll down to a particular section or click on the section heading on the navigation area in the upper-right corner.

Accept the default value or enter a name in **[Application Name](#application-name)**.
Accept the default value in **Version**.

Select the timezone for your TrueNAS system location from the **Timezone** dropdown list.

![InstallChiaConfigFullNodeService](/static/img/truenas/InstallChiaConfigFullNodeService.png)

Select the service from the **Chia Service Mode** dropdown list.
The default option is **Full Node**, but you can select **Farmer** or **Harvester**.
**Harvester** displays additional settings, each described in [Chia Configuration](#chia-configuration) below. 
Refer to Chia-provided documentation for more information on these services.

You can enter the network address (IP address or host name) for a trusted peer in **Full Node Peer** now or after completing the app installation.
This is the trusted/known or untrusted/unknown server address you want to use in sync operations to speed up the sync time of your Chia light wallet.
If not already configured in Chia, you can add this address as a trusted peer after completing the app installation by editing the app settings.

Accept the default values in **Chia Port** and **Farmer Port**.
You can enter port numbers below 9000, but check the [TrueNAS Default Ports list](https://www.truenas.com/docs/references/defaultports/) to verify the ports are available. 
Setting ports below 9000 automatically enables host networking.

By default, SCALE can create the storage volumes (datasets) for the app by accepting the default ixVolume option.

![InstallChiaStorageConfigixVolume](/static/img/truenas/InstallChiaStorageConfigixVolume.png)

To use existing datasets created for the Chia app, select **Host Path (Path that already exists on the system)**.
Enter or browse to select the mount paths for the **config** dataset, then for the **plot** dataset. 
These are the datasets created in [First Steps](#first-steps). 
Selecting the dataset populates the **Host Path** field for storage volume.

Accept the defaults in [**Resource Configuration**](#resource-configuration) or change the CPU and memory limits to suit your use case.

Click **Install**.
The system opens the **Installed Applications** screen with the SCALE Chia app in the **Deploying** state.
When the installation completes, it the status changes to **Running**.

![ChiaAppInstalled](/static/img/truenas/ChiaAppInstalled.png")

The first time the SCALE Chia app launches, it automatically creates and sets a new private key for your Chia plotting and wallet, but you must preserve this key to persist across container restarts.

### Obtaining and Preserving Keys
To make sure your plots and wallet private key persists across container or system restarts, save the mnemonic seed created during the app installation and deployment.

On the **Installed** apps screen, click on the **Chia** app row, scroll down to the **Workloads** widget, and locate the **Shell** and **Logs** icons.

![ChiaAppWorkloadsWidget](/static/img/truenass/ChiaAppWorkloadsWidget.png")

Click on the shell <span class="iconify" data-icon="ci:window-terminal"></span> icon to open the **Choose pod** window.

![ChiaChoosePodforShell](/static/img/truenas/ChiaChoosePodforShell.png)

Click **Choose** to open the **Pod shell** screen.

To show Chia key file details and the 24 word recovery key, enter `/chia-blockchain/venv/bin/chia keys show --show-mnemonic-seed` at the **Shell** prompt.
The command should return the keyfile and the 24-word recovery key.

![ChiaShellShowMnemonicSeed](/static/img/truenas/ChiaShellShowMnemonicSeed.png)

If you loose the keyfile at any time, use this information to recover your account.
To copy from the SCALE **Pod Shell**, highlight the part of the screen to copy, then press <kbd>Ctrl+Insert</kbd>.
Open a text editor like Notepad and paste the information into the file. 
Save this file where only authorized people can access it and where it gets backed up.

Now save this mnemonic-seed phrase to one of the host volumes on TrueNAS. Enter this command at the **Pod Shell** prompt:

<code>echo <i>type all 24 unique secret words in this command string</i> > /plots/keyfile</code>

Where <i>type all 24 unique secret words in this command string</i> is all 24 words in the mnemonic-seed.

Next, edit the SCALE Chia app to add the key file.

### Adding Keys to the SCALE Chia App
Click **Installed** on the breadcrumb at the top of the **Pod Shell** screen to return to the **Apps > Installed** screen.

Click on the Chia app row, then click **Edit** in the **Application Info** widget to open the **Edit Chia** screen.

Click on **Chia Configuration** on the menu on the right of the screen or scroll down to this section.
Click **Add** to the right of **Additional Environments** to add the **Name** and **Value** fields.

![EditChiaConfigAddEnvironmentVariable](/static/img/truenas/EditChiaConfigAddEnvironmentVariable.png)

Enter **keys** in **Name** and **/plots/keyfile** in **Value**.

Scroll down to the bottom of the screen and click **Save**.
The container should restart automatically.

After the system restarts, click on **Apps** to open the **Installed** screen.
The Chia app returns to the **Running** state. If the screen does not appear to change, refresh the browser cache.
You can confirm the keys by returning to the **Pod shell** screen and entering the `/chia-blockchain/venv/bin/chia keys show --show-mnemonic-seed` command again.
If the keys are not identical, edit the Chia app again and check for any errors in the name of values entered.
If identical, the keys file persists for this container.

The TrueNAS SCALE Chia application installation and deployment is complete. 
You can now complete your Chia software and client configuration using either the Chia command line (CLI) or web interface (GUI).

### Setting Up the Chia GUI
To complete the Chia software and client setup, refer to the [Chia Crash Course](https://docs.chia.net/guides/crash-course/introduction) and [Chai Getting Started](https://docs.chia.net/installation) guides and follow the instructions provided.
The following shows the simplest option to install the Chia GUI.

Click on the [Chia downloads](https://www.chia.net/downloads/) link and select the option that fits your operating system environment. 
This example shows the Windows setup option.

After downloading the setup file and opening the **Chia Setup** wizard, agree to the license to show the Chia setup options.

![ChiaGUISetupInstallOptions](/static/img/truenas/ChiaGUISetupInstallOptions.png)

Click **Next**. Choose the installation location.

![ChiaSetupGUIChooseLocation](/static/img/truenas/ChiaSetupGUIChooseLocation.png)

Click **Install** to begin the installation. 
When complete, click **Next** to show the **Chia Setup Installation Complete** wizard window.
**Launch Chia** is selected by default. 
Select the **Add the Chia Command Line executable to PATH** advanced option if you want to include this. Click **Finish**.

After the setup completes, the Chia web portal opens in a new window where you configure your Chia wallet, farming modes, and other settings to customize Chia for your use case.

![ChiaWebPortalGUI](/static/img/truenas/ChiaWebPortalGUI.png)

Use the [Chia Documentation](https://docs.chia.net/) guide you through completing the configuration of your Chia software and client.

At this point, you are ready to begin farming Chia.
{{< hint type=info >}}
The CLI process is beyond the scope of this TrueNAS quick how-to.
We recommend you start by reading the Chia [CLI reference materials](https://github.com/Chia-Network/chia-blockchain/wiki/CLI-Commands-Reference), [Quick Start guide](https://github.com/Chia-Network/chia-blockchain/wiki/Quick-Start-Guide), and other [documentation](https://github.com/Chia-Network/chia-blockchain/wiki).
{{< /hint >}}

## Understanding SCALE Chia App Settings
The following sections provide more details on the settings found in each section of the SCALE **Install Chia** and **Edit Chia** screens. 
To access the **Edit Chia** screen, click on the **Chia** app row on the **Installed** applications screen, then click **Edit** on the **Application Info** widget.

### Application Name Settings

Accept the default value or enter a name in **Application Name** field.
In most cases use the default name, but if adding a second deployment of the application you must change the name.

Accept the default version number in **Version**.
When a new version becomes available, the application displays an update badge, and the **Installed Applications** screen shows the option to update applications.

### Chia Configuration
The **Chia Configuration** section includes four settings: **Timezone**, **Chia Service Node**, **Full Node Peer**, and **Additional Environments**.

Select the timezone for the location of the TrueNAS server from the dropdown list.

The **Chia Service Node** has three options: **Full Node**, **Farmer**, and **Harvester**. 
The default **Full Node** and **Farmer** options do not have extra settings.

![InstallChiaConfigFullNodeService](/static/img/truenas/InstallChiaConfigFullNodeService.png)

Selecting **Harvester** shows the required **Farmer Address** and **Farmer Port** settings, and **CA** for the certificate authority for the farmer system.
Refer to Chia documentation on each of these services and what to enter as the farmer address and CA.

![InstallChiaConfigHarvesterService](/static/img/truenas/InstallChiaConfigHarvesterService.png)

After configuring the Chia software and client in the Chia GUI or CLI, you can edit these configuration settings. 
If you want to create a second app deployment with the **Harvester** service node setting, repeat the instructions above, use a different name for the second deployment of the SCALE Chia app (for example, *chia-harvester*).

You can enter a network address (IP address or host name) for a trusted peer in **Full Node Peer** now or later, after completing the SCALE Chia app installation and setting up the Chia software and client in the Chia GUI or CLI. 
Enter the trusted/known or untrusted/unknown server address to use for sync operations and to speed up the sync time of your Chia light wallet.

Click **Add** to the right of **Additional Environments** to add a **Name** and **Value** field. 
You can add environment variables to further customize the Chia app. 
Click **Add** for each environment variable you want to add. Refer to Chia documentation for information on environment variables you might want to implement. 

The **Additional Environments** fields are used to [make the initial key file persist](#adding-keys-to-the-scale-chia-app) after a container restart.

### Network Configuration
Accept the default port numbers in **Chia Port** and **Farmer Port**.
The SCALE Chai app listens on port **38444** and **38447**.
To change the port numbers, enter an available number within the range 9000-65535. 
Refer to the TrueNAS [default port list](https://www.truenas.com/docs/references/defaultports/) for a list of assigned port numbers.

![InstallChiaNetworkConfig](/static/img/truenas/InstallChiaNetworkConfig.png)

### Storage Configuration
You can allow SCALE to create the datasets for Chia plots and configuration storage or you can create the datasets to use for the app storage volumes or to mount in the container.

To allow SCALE to create the datasets, accept the default ixVolume settings.

If manually creating and using datasets, follow the instructions in [Creating a Dataset](https://www.truenas.com/docs/scale/scaletutorials/storage/datasets/datasetsscale/) to correctly configure the datasets before beginning the Chia app installation.
Add one dataset named **config** and another named **plots**.
If also mounting datasets in the container pod for the Chia app, add and name additional datasets according to your intended use. You can  or use *volume1*, *volume2*, etc. for each additional volume.

In the SCALE Chia app **Storage Configuration** section, select **Host Path (Path that already exists on the system)** as the **Type** for the **Data** storage volume.
Enter or browse to and select the location of the existing dataset to populate the **Host Path** field. Repeat this for the **Plots** storage volume.

![InstallChiaStorageConfigHostPaths](/static/img/truenas/InstallChiaStorageConfigHostPaths.png)

If adding storage volumes inside the container pod, click **Add** to the right of **Additional Volumes** for each dataset or ixVolume you want to mount inside the pod.

![InstallChiaStorageConfigAdditionalVolsHostPath](/static/img/truenas/InstallChiaStorageConfigAdditionalVolsHostPath.png)

You can edit the SCALE Chia app after installing it to add additional storage volumes.

### Resource Configuration
The **Resources Configuration** section allows you to limit the amount of CPU and memory the SCALE Chia application can use.
By default, the application is limited to use no more than **4** CPU cores and **8** Gibibytes available memory, but the application might use considerably less system resources.

![InstallChiaResourcesConfig](/static/img/truenas/InstallChiaResourcesConfig.png)

Tune these limits as needed to prevent the application from over-consuming system resources and introducing performance issues.
