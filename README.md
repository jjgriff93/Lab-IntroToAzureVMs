# Lab-IntroToAzureVMs

Okay, so let's assume you've just got your Azure subscription (if not, [click here](https://azure.microsoft.com/en-gb/free/) to get started for free) and you're staring at a myriad of scary-looking options. Something like the picture below. Well don't worry, at the end of this lab, you'll have two VMs running and a good understanding of the various methods of deployment and post-deployment configuration. ![Portal](http://jamesgriff.in/wp-content/uploads/2018/01/Portal-1024x535.png)

> This lab will take you through the off-the-shelf VM options and the process of provisioning a VM with a single unmanaged disk, then one with a managed disk, and finally how to use configuration management to build a simple web server environment.

Let's begin.  

## **Exercise 1 - Looking through the VM images & sizes available**

Azure VMs are available in many shapes and sizes, and they're organised into families to make things easier. We'll look through what's available in different Azure regions and then what OS images are available in the galleries, before using one to build a bootable OS disk for the VM we're going to create.

##### **Step 1 - Go to the Azure portal and open up the Cloud Shell**

Head to [portal.azure.com](http://portal.azure.com/) and then click on the Cloud Shell icon. This will open up a terminal in the Azure portal itself and will make us look like we're extremely advanced to people standing behind us. You can use either Bash or PowerShell depending on your preference, but we'll start with Bash and the Azure Command Line Interface (more info on the [Azure CLI here](https://docs.microsoft.com/en-us/cli/azure/get-started-with-azure-cli?view=azure-cli-latest)). ![CLI](http://jamesgriff.in/wp-content/uploads/2018/01/CLI-2-1024x396.png)

##### **Step 2 - Select the subscription you want to use (if you have more than one)**

To show what subscriptions you have at your disposal (based on the account you're logged in with and it's permissions), type in the following command: `az account list` This will list out all of the subscriptions and their associated details that your account has access to, like so: ![AZ account list](http://jamesgriff.in/wp-content/uploads/2018/01/AZ-account-list-1024x677.png) Then select the subscription you want to use (make sure you have quote marks around the name/id): `az account set --subscription 'INSERT NAME OR ID HERE'`

##### **Step 3 - Check out the VM sizes available in the North Europe and UK South regions using the CLI**

Enter the following in the Cloud Shell to output the available sizes for North Europe into a table: `az vm list-sizes -l northeurope -o table` That will give you something like this: ![VM Sizes in North Europe](http://jamesgriff.in/wp-content/uploads/2018/01/5_ListVMSizesNorthEurope-1024x376.png) As you can see there are varying numbers of data disks, cores and memory depending on VM type.

> When working with Azure, it's important to assess which is the right VM for the workload you will use it for. For example, you might need an F-series for compute-optimised work, or an N-series for something more graphically intensive that could benefit from a dedicated GPU. Check out [this guide](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes) to help you decide when the time comes.

For now though, let's compare what's available in the UK South region, modifying the previous command: `az vm list-sizes -l uksouth -o table` Now compare what you got from each region. You'll probably notice that there were a few differences (at the time of writing), namely that the G series (very high balanced performance), L series (storage optimised) and M series (memory optimised) have appeared. This is because not all regions have the same VMs available (when new ones roll out certain regions get them first for example), so it's worth bearing this in mind when you're selecting a region for your workload.

##### **Step 4 - Inspect the available VM images in North Europe using PowerShell**

Okay, just to shake things up, let's switch over to PowerShell in the Cloud Shell. Click the dropdown on the top left to switch, like so: ![Switch to powershell](http://jamesgriff.in/wp-content/uploads/2018/01/Switch-to-powershell-1024x241.png) Once you've done that, everything will turn a beautiful shade of blue to indicate you've swapped to PowerShell, and you'll notice the commands we enter from now on are different to the Azure CLI commands used with Bash.

> PowerShell has its own Azure module, which means different commands to the ones we've been using in Bash; however you can use the CLI commands in both Bash and PowerShell as it's built to be cross-platform. After you've used both methods you'll find one you're more comfortable with, so stick with that (I think the CLI is much simpler and easier to get to grips with, but less powerful). A useful guide on Azure PowerShell is [available here.](https://docs.microsoft.com/en-gb/azure/cloud-shell/quickstart-powershell)

Anyway, let's use Azure PowerShell to list out all the image publishers in the North Europe region: `Get-AzureRmVMImagePublisher -l northeurope` ![Publisher by region](http://jamesgriff.in/wp-content/uploads/2018/01/Capture-1024x296.png) You'll notice there are quite a few. Let's pick out Canonical and get their image offers: `Get-AzureRmVMImageOffer -l northeurope -Publisher Canonical | Select offer` ![Publisher offers](http://jamesgriff.in/wp-content/uploads/2018/01/Capture-1-1024x222.png) That gives us all of the images that Canonical offer. Now let's get the available skus for the UbuntuServer offer: `Get-AzureRmVMImagesku -l northeurope -Publisher Canonical -Offer UbuntuServer | Select Skus` ![Skus](http://jamesgriff.in/wp-content/uploads/2018/01/Capture-2-1024x282.png) Wow, a lot of skus. Let's see what versions are available for the 17.10 sku: `Get-AzureRmVMImage -l northeurope -Publisher Canonical -Offer UbuntuServer -Sku 17.10 | Select Version` ![Versions](http://jamesgriff.in/wp-content/uploads/2018/01/Capture2-1024x282.png) A lot of versions as well. Ok, let's bring this all together to select a specific image: `Get-AzureRmVMImage -l northeurope -Publisher Canonical -Offer UbuntuServer -Sku 17.10 -Version 17.10.201801260` ![Select vm image](http://jamesgriff.in/wp-content/uploads/2018/01/Capture-3-1024x265.png) Phew. And breathe. So there you have it, we've navigated a vast library of images and picked out a specific one. We'll come back to this later when we create a VM.  

## **Exercise 2 - Navigating the Azure marketplace in the portal**

Okay, let's have a quick break from the command line. Now we're going to look at all of the full solutions available to deploy to Azure through the marketplace.

##### **<input type="checkbox"> Step 1 - Head to the marketplace**

To start, open a new tab in your browser and head to the following url: [https://azuremarketplace.microsoft.com/](https://azuremarketplace.microsoft.com/) ![Azure marketplace](http://jamesgriff.in/wp-content/uploads/2018/01/Capture1-1024x616.png) You'll see a plethora of options available, and specific recommendations for you which will get more intelligent as you continue using Azure.

##### **<input type="checkbox"> Step 2 - Search for the Ubuntu image we selected in the command line**

Let's search for the same Ubuntu image like we did in the command line to show different ways of finding what you're looking for. In the "Search Everything" box, type **Ubuntu**. ![market search](http://jamesgriff.in/wp-content/uploads/2018/01/Capture3-1024x437.png) As you can see, the same Ubuntu Server 17.10 by Canonical has come up, as well as some by other publishers - including one co-published with Microsoft with Docker pre-installed for running containers. Pretty cool (if containers are the kind of thing that excites you).

##### **<input type="checkbox"> Step 3 - Select an image**

Finally, let's pick the 17.10 image. Click on it and you'll see the information blade come up, with the options to deploy it at the bottom: ![Marketplace deploy](http://jamesgriff.in/wp-content/uploads/2018/01/Capture4-1024x213.png) It's that simple. In summary, we've searched for our desired image in the Azure Marketplace, and after identifying the machine we want to deploy, we're just a click away.  

## **Exercise 3 - Provisioning a VM with unmanaged storage in the portal**

Next we're going to deploy a virtual machine using the portal, and we're going to provision it with unmanaged storage for now (which means we'll have to then sort the storage ourselves), as we'll look at managed storage in the next exercise.

##### **Step 1 - Configure the basics**

So, since we're already there, let's click **Create** on the Ubuntu server image we've selected from the marketplace.

> Make sure you've kept the deployment model selected as "Resource Manager", as this is the modern model used by Azure that centres around resource groups (the classic model is mainly there for legacy support). More on resource groups later.

Once you've clicked that, you'll be presented with the basics configuration pane. Fill it out like so:

1.  **Name:** what you want to call the VM - I've gone with **MyUbuntuTestServer**. Make sure there are no spaces or non-ASCII characters.
2.  **VM disk type:** stick with **SSD** (solid state drive) as this is faster, but you have the option of HDD also.
3.  **Username:** username that you'll use to log into the server. Make sure your username is all lowercase.
4.  **Authentication type:** you can use either an SSH public key or a password for your account. For simplicity let's switch to **Password** and enter a password of your choice.
5.  **Subscription****:** make sure you've selected the subscription you want the VM to be under. This is where the VM will be linked to and where the cost will be charged.
6.  **Resource group:** think of this as a container for all the resources that will be associated with the VM/the workload. Let's **Create new** and call it **UbuntuRG** (unless you want to call it something else more edgy).
7.  **Location****:** as we've discovered before, different datacentre locations sometimes have different options, and obviously this affects latency as well, so it's best to pick an established datacentre that's near where the consumers of your workload will be. For now, let's pick **West Europe**.

Got all that? Once you've filled it out it should look a bit like this: ![VM basics config](http://jamesgriff.in/wp-content/uploads/2018/01/Capture5-1024x646.png) Once that's all done, click **Next** down at the bottom of the pane.

##### **<input type="checkbox"> Step 2 - Choosing the size**

Next we need to select the machine size we want for our workload. You'll be presented with various options and VM families, and each one has a different balance of max IOPS, memory, processing power and, of course, estimated per month cost. I linked[this guide](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes) earlier on which will help you understand the different VM options in more detail, so if you haven't already, check it out. For this lab, let's select a balanced inexpensive VM, the **D2S_V3 Standard**: ![VM sizes](http://jamesgriff.in/wp-content/uploads/2018/01/Capture-4-1024x598.png) Once you've selected that, click **Select** to go to the next pane.

##### **Step 3 - Set up the storage and virtual network**

Right, now it's time to set up the important stuff. Our virtual machine will need a network and associated storage to operate. For this stage of the lab, we're going to set up unmanaged storage, which means Azure won't manage the disk for us as part of the VM, it will sit in a storage account instead for us to look after. Fill out the options like below:

1.  **Availability set**: leave this set to **None** for now. Configuring an availability set is always a good idea for important workloads, as instances of your VM will be spread out across different zones in a datacentre to ensure fault tolerance.
2.  **Use managed disks**: change this to **No**. You'll then be prompted to associate a storage account for the disk, and we'll **Create new** for this lab. Leave the pre-populated name as is and **Premium** and **LRS** as the selected performance and replication.
3.  **Network**: leave these as is. This will create a new virtual network for our machine to operate in, along with a public IP address for us to access our VM later.
4.  **Extensions:** leave this set to **No Extensions.** This is where you can set up things like configuration management and antivirus, but we'll do this later.
5.  **Boot diagnostics**: change this to **Disabled.** Diagnostics are saved to a storage account, so we'll leave this for now.

![Optional features](http://jamesgriff.in/wp-content/uploads/2018/01/Capture2-1-1024x493.png) Once you've done all that, click **Ok**. Almost there.

##### **Step 4 - Save a template before deploying**

We're nearly ready to deploy our machine, but first, let's save our VM as a template that we can use later on to quickly deploy a similar machine. Click on **Download template and parameters** next to the 'Create' button. ![Save template](http://jamesgriff.in/wp-content/uploads/2018/01/Capture5-1-1024x522.png) This will open up the deployment template blade. Clicking 'Add to library' which will save it to your Azure account for future use, but we're going to click **Download** to save the template to our PC as a .zip, which we'll use later. Save it to your **Desktop**. ![download template](http://jamesgriff.in/wp-content/uploads/2018/01/download-template-1024x493.png) Template packages consist of a JSON file for describing the resources to be deployed, a JSON file for the parameters, and a script to automate the deployment (available as PowerShell, Ruby, C# and CLI scripts). More on this in the next exercise.

##### **Step 5 - Deploy the VM**

Close off the Template blade by clicking the cross or by scrolling left back to the previous blade. Let's click the big blue **Create** button, which will take us back to our dashboard and create a blue tile for our deploying VM: ![Deploying vm](http://jamesgriff.in/wp-content/uploads/2018/01/Deploying-vm-1024x591.png) Now let's stare at the mesmerising animation while our server deploys. (This usually doesn't take too long but feel free to grab a cup of tea while you wait).

##### **Step 6 - Connect to the VM**

Okay, once it's deployed all that's left to do is connect to our machine. As it's Linux, we'll connect using SSH.

> Pre-requisite: if you're using Windows for this lab, you'll need to make sure you have either an SSH client like PuTTY installed ([download here](http://www.putty.org/)) or the Ubuntu app with the Linux subsystem for Windows enabled ([guide here](https://docs.microsoft.com/en-us/windows/wsl/install-win10)). I'll be using the latter for this lab.

When the deployment finishes, the VM's dashboard will open up automatically. You can also access this by clicking on its tile. All you need to do now is click on **Connect** at the top of the Overview pane: ![ssh](http://jamesgriff.in/wp-content/uploads/2018/01/ssh-1024x404.png) This will give you the Bash command you need to use to connect to the machine. Copy it to your clipboard, then open up the Ubuntu app and paste it in by right-clicking: ![ubuntu cli](http://jamesgriff.in/wp-content/uploads/2018/01/ubuntu-cli-1024x326.png) Once you've pressed ENTER, it will ask you if you want to continue connecting. Type **Yes** and press ENTER again: ![ubuntu cli2](http://jamesgriff.in/wp-content/uploads/2018/01/ubuntu-cli2-1024x257.png) Then type in the password you defined earlier on when creating the VM. If you've not used Bash before, when you type into a password field it doesn't display the characters as you type, so keep on typing and press ENTER when done: ![Ubuntu cli3](http://jamesgriff.in/wp-content/uploads/2018/01/Ubuntu-cli3-1024x567.png) As you can see, the title bar and user ID has now changed to reflect that we've successfully logged into the server. You can now Bash away to your heart's content. So to summarise, we have successfully created an Ubuntu VM in Azure, saved a deployment template for later use, and connected to our VM from our local machine. Nice one.  

## **Exercise 4 - Provisioning another VM using our template**

Remember the template that we saved? Well we're going to use that now to deploy another VM quickly, without having to touch the portal, but we're also going to modify the template first so we can deploy a different kind of machine. Exciting I know.

##### **Step 1 - Open up the parameters JSON file**

Go and find the **template.zip** file on your Desktop (unless you saved it somewhere else) and extract the files: ![extract template](http://jamesgriff.in/wp-content/uploads/2018/01/extract-template-1024x359.png) Then open up the **parameters.json** file with your favourite code editor.

> I can definitely recommend Visual Studio Code as a great lightweight open-source code editor, with many packages available to help write quick templates and scripts for Azure, and it's the editor I'll be using for the lab. [Download it here](https://code.visualstudio.com/).

##### **Step 2 - Modify the parameters for the deployment**

The parameters file is used by the deployment script to populate all of the, well, parameters for the resources it will deploy, so we can change the VM size, region and loads of other stuff very easily. Make the following changes to the parameters JSON:

1.  Change `"location": {"value": "westeurope"}` to `"location": {"value": "northeurope"}`
2.  Replace `null` with a password for the `"adminPassword"` parameter (make sure you include quotation marks around the password and that it's sufficiently complex for Azure's requirements)
3.  Change `"virtualMachineName": {"value": "MyUbuntuTestServer"}` to `"virtualMachineName": {"value": "MyWinTestServer"}`
4.  Change `"virtualNetworkName": {"value": "UbuntuRG-vnet"}` to `"virtualNetworkName": {"value": "WinRG-vnet"}`
5.  Change `"networkInterfaceName": {"value": "myubuntutestserver447"}` to `"networkInterfaceName": {"value": "mywintestserver447"}`
6.  Change `"networkSecurityGroupName": {"value": "MyUbuntuTestServer-nsg"}` to `"networkSecurityGroupName": {"value": "MyWinTestServer-nsg"}`
7.  Delete these two parameters (as we'll be using managed storage so don't need a separate storage account): `"storageAccountName": {"value": "ubunturgdisks333"}, "storageAccountType": {"value": "Premium_LRS"},`
8.  Change `"publicIpAddressName": {"value": "MyUbuntuTestServer-ip"}` to `"publicIpAddressName": {"value": "MyWinTestServer-ip"}`

Once you're done, your parameter file should look something like this:

![parameters.json](https://gist.github.com/jjgriff93/b96ace53b4c241e7c3c4e8ef7d70881a)

##### **Step 3 - Modify the deployment template file**

Okay, now we just need to make a couple of modifications to the template file as well, to change it from an unmanaged disk deployment to use managed disks. Open up the **template.json** file and replace the content with the below: 

![template.json](https://gist.github.com/jjgriff93/94c0a480dfcaabdfcbe6612dfbfeeed6)

Make sure you save the file once it's been modified.

##### **Step 4 - Deploy the modified template**

Okay, we're ready. You have a few options to deploy the template with (CLI, Ruby etc.), but we're going to use PowerShell this time.

> Prerequisite: you'll need to have the Azure module installed on your machine to be able to use the deploy script. If you haven't yet, it only takes a sec to set up - [see here for instructions.](https://docs.microsoft.com/en-us/powershell/azure/install-azurerm-ps?view=azurermps-5.1.1)

Open up a PowerShell window on your machine and change the working directory to the template folder. Easiest way is to grab the absolute folder path from your file explorer and then paste it in like below (make sure you add the quotation marks): `cd "C:\Users\jamesgr\Desktop\template"` ![cd pshell](http://jamesgriff.in/wp-content/uploads/2018/01/cd-pshell-1024x185.png) Next we need to change the execution policy for our current PowerShell session, as the deploy script will be blocked from running due to it not being digitally signed. Run the following command, then type **A** to accept: `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` ![execution policy](http://jamesgriff.in/wp-content/uploads/2018/01/execution-policy-1024x252.png) Now, before running the script, we want to get our subscription ID, as we'll be prompted for it. Type in the following command then copy the Subscription ID from the one you wish to deploy to to your clipboard: `Get-AzureRmSubscription` And now we just need to run the **deploy.ps1** script by typing the following: `.\deploy.ps1` You'll then be prompted to fill in your subscription ID, so paste this in from your clipboard, followed by the Resource Group name you want to create or an existing one to use, which we'll put as **WinRG**, and finally the deployment name, which we'll put as **MyWinTestServer**. ![listsubs](http://jamesgriff.in/wp-content/uploads/2018/01/listsubs-1024x460.png) Then you'll be asked to authenticate with Azure in a popup, so make sure you use the account associated with the subscription you're using for the deployment. Once you've done that, the deployment will start. Time for another cup of tea while you wait. If all goes well, after a few minutes, you should get a Success dialogue along with all the details of your deployment: ![provisioning succeeded](http://jamesgriff.in/wp-content/uploads/2018/01/provisioning-succeeded-1024x228.png)

##### **Step 4 - Connect to our shiny new Windows VM**

Great, so now we've got our server up and running, all that's left to do is connect to it. As it's Windows, we'll do this via Remote Desktop. Type in the following command into PowerShell, which will generate an RDP file for us to use to set up the connection. `Get-AzureRmRemoteDesktopFile -ResourceGroupName "WinRG" -Name "MyWinTestServer" -LocalPath "C:\MyWinTestServer.rdp"` This will create the MyWinTestServer.rdp file on our C drive. Now all we need to do is open it, which we'll do from PowerShell: `Invoke-Item "C:\MyWinTestServer.rdp"` This should open up the RDP connection in your default RDP client (I'm using Remote Desktop from the Windows 10 store). Then all that's left is to log in with the credentials you set up earlier and voila: ![Windows Server RD](http://jamesgriff.in/wp-content/uploads/2018/01/Windows-Server-RD-1024x585.png) We've connected to our Windows Server VM running in Azure. Nice. To summarise this exercise, we've taken a deployment template we made earlier, modified it to swap from an Ubuntu image to Windows Server, changed the storage from unmanaged to managed and the region from West Europe to North Europe, deployed it using local PowerShell and connected via RDP, all without touching the Azure web portal. Not too shabby.  

## **Exercise 5 - adding an extension for configuration management**

So, last exercise. We have our VMs up and running, but now we want to extend them with additional functionality. Well, luckily for us, that's where extensions come in.

> VM Extensions are software components that extend the VM functionality and simplify various VM management operations. For example, the VMAccess extension can be used to reset an administrator's password, or the Custom Script extension can be used to execute a script on the VM. Marketplace images all come pre-installed with the Azure VM Agent extension, which manages interaction between an Azure virtual machine and the Azure fabric controller. The VM agent is responsible for many functional aspects of deploying and managing Azure virtual machines, including running VM extensions. [More info here.](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/extensions-features)

We're going to install the PowerShell Desired State Configuration (DSC) Extension on our Windows VM, which will enable us to use PowerShell cmdlets to enforce a configuration on our machine.

##### **Step 1 - Create a basic configuration**

Okay, let's create a quick configuration document that we'll use to enforce our 'desired state' on the VM. Copy the code below and save it to your desktop as a .ps1 (PowerShell) file using Notepad, Visual Studio Code or your favourite text editor: 

![IISInstall.ps1](https://gist.github.com/jjgriff93/ff5f1c4a68d724557f2dc88616bc1cc7)

Call it **IISInstall.ps1**. This will enforce our VM to have Internet Information Services (IIS) present, which will in effect make our machine a web server.

##### **Step 2 - Package the configuration file**

Next we need to package up the configuration file we've created as a .zip. Open up a PowerShell window, change the working directory (`cd`) to your desktop, then type the following: `Publish-AzureVMDscConfiguration .\IISInstall.ps1 -ConfigurationArchivePath .\IISInstall.zip` ![IISInstallzip](http://jamesgriff.in/wp-content/uploads/2018/01/IISInstallzip-1024x244.png) This will create a .zip package on your desktop for us to upload to Azure, and it will also parse our configuration and move any local module dependencies into the package for us if present, so it's a good habit to get into using this command.

##### **Step 3 - Install the PowerShell DSC extension on our VM**

Now we're going to head to the good ol' portal again, at [portal.azure.com](http://portal.azure.com/). Navigate to your Windows VM by typing in **MyWinTestServer** in the search bar at the top. ![Server search](http://jamesgriff.in/wp-content/uploads/2018/01/Server-search-1024x312.png) Click on the **Virtual Machine** resource, which will take you to the VM's dashboard. Then click on **Extensions** on the left-hand side: ![VM Extension](http://jamesgriff.in/wp-content/uploads/2018/01/VM-Extension-1024x356.png) As you can see there's already a monitoring extension pre-installed on our VM as it's a marketplace image. This allows us to use the monitoring functionalities of Azure without any set up. Now click on **Add**, and find **PowerShell Desired State Configuration**: ![PShell DSC](http://jamesgriff.in/wp-content/uploads/2018/01/PShell-DSC-1024x449.png) Click on **Create** down the bottom and then fill out the 'Install extension' configuration parameters like so:

1.  **Configuration Modules or Script:** click on the folder icon and navigate to the **IISInstall.zip** file that we created
2.  **Module-qualified Name of Configuration**: this needs to consist of the name of our config file with '\'and then the name of our configuration within the file, so in our case, **IISInstall.ps1\IISInstall**
3.  **Version:** let's use the latest, so **2.72**
4.  **Auto Upgrade Minor Version:** set this to **Yes**
5.  Leave everything else blank. There's lots to DSC that we don't have time to cover in this lab!

So it should look a little (well, exactly) like this: ![DSC portal](http://jamesgriff.in/wp-content/uploads/2018/01/DSC-portal-1024x557.png) Click **Ok** when done.

##### **Step 4 - Check the configuration has been successfully applied**

Now you'll be taken to the following screen to watch in awe as the extension is applied. It should go from this: ![DSC transitioning](http://jamesgriff.in/wp-content/uploads/2018/01/DSC-transitioning-1024x331.png) To this: ![DSC provisioned](http://jamesgriff.in/wp-content/uploads/2018/01/DSC-provisioned-1024x337.png) Once that happens, let's check if we've successfully transformed our bog standard Windows VM to an IIS web server. Firstly, we need to modify the settings of the VMs network to allow internet traffic over HTTP/port 80\. We can do this nice and easily in the portal - so click on **Networking** under the Settings submenu: ![Allow HTTP](http://jamesgriff.in/wp-content/uploads/2018/01/Allow-HTTP-1024x398.png) And then click **Add inbound port rule** and select **HTTP** from the **Service** dropdown: ![HTTP inbound](http://jamesgriff.in/wp-content/uploads/2018/01/HTTP-inbound-1024x429.png) This will auto-populate the other fields for you to allow inbound internet traffic on port 80\. So all you need to do is click **Ok.** Once that's applied you'll see the rule appear in the inbound port rules table: ![HTTP rule applied](http://jamesgriff.in/wp-content/uploads/2018/01/HTTP-rule-applied-1024x331.png) We're all set. Now all you need to do is copy the **public IP address** of your VM (handily available just above the Inbound Port Rules table, or back in the Overview pane), and paste it into a new tab. If all's well, this is what you should see: ![IIS webpage](http://jamesgriff.in/wp-content/uploads/2018/01/IIS-webpage-1024x439.png) So there we go. We've installed the PowerShell DSC extension to our VM, then we've used it to apply a configuration that we've created, which has forced it to transform into a web server. Not bad for a day's work.  

## **That's all folks...**

Well that's it for this lab, well done and I hope you enjoyed it. Let's go over all the things we covered:

1.  Used the Cloud Shell in both Bash and PowerShell modes
2.  Understood some basic Azure CLI commands and Azure PowerShell cmdlets
3.  Identified various marketplace images available for deployment in both PowerShell and the Azure portal
4.  Understood the different basic Azure VM sizes and the differences in availability across regions
5.  Deployed a Linux VM using an Ubuntu marketplace image from the portal and configured unmanaged storage
6.  Learned how to access our Linux VM through SSH
7.  Exported an automation template to our machine to re-use for quick deployments
8.  Modified the template to change the image from Ubuntu to Windows and unmanaged storage to managed, and selected a different region
9.  Deployed a template to Azure through local PowerShell
10.  Learned how to RDP into our new Windows machine through PowerShell
11.  Added an extension to our VM to allow Desired State Configuration
12.  Created and enforced a configuration on our VM to make it a web server
13.  Modified networking port rules to allow internet traffic and tested our configuration had been applied

Trying saying all of that without taking a breath! We've covered a lot of key concepts there and I've tried to weave them together into several actions you might perform regularly in creating and administering VMs in Azure, so hopefully this has served as a good introduction to some of the things you can do. Please let me know what you thought of the lab in the comments below, anything I can help with and any improvements you think I should make, it'd be greatly appreciated. I'll be creating more labs in this series to take you further into the world of Azure, so any requests are more than welcome. Goodbye for now, and good luck!
