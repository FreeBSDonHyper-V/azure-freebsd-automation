This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

# azure-freebsd-automation
Automation tools for testing FreeBSD images on Microsoft Azure
## Overview
Azure automation is the project for primarily running the Test Suite in the Azure environment. Azure automation project is a collection of PowerShell, BASH and python scripts. The test ensures the functionality of Azure Agent and Azure support for different FreeBSD/Linux distributions. This test suite focuses on the Build Verification Tests (BVTs), Azure VNET Tests and Network tests. The test environment is composed of a Windows host machine (with Azure PowerShell SDK) and the Virtual Machines on Azure that perform the actual tests.
## <a id="prepare"></a>Prepare Your Machine for Automation Cycle
### Prerequisite
1.  You must have a Windows Machine with PowerShell. Tested Platforms:

          a.  Server 2012 R2

2.  You must be connected to Internet.
3.  You must have a valid Azure Subscription.

          a.  Subscription Name
          b.  Subscription ID

### Download Latest Automation Code
1.  Checkout from https://github.com/FreeBSDonHyper-V/azure-freebsd-automation.git

### Download Latest Azure PowerShell
1.	Download Web Platform Installer from : http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409
2.	Start Web Platform Installer and select Azure PowerShell and proceed for Azure PowerShell Installation.

### Authenticate Your Machine with Your Azure Subscription
There are two ways to authenticate your machine with your subscription.

1.	Azure AD method

      This creates a 12 Hours temporary session in PowerShell, in that session, you are allowed to run Azure Cmdlets to control / use your subscription. After 12 hours you will be asked to enter username and password of your subscription. This may create problems long running automations, hence we use certificate method.

2.	Certificate Method.

      To learn more about how to configure your PowerShell with your subscription, please visit [here](http://azure.microsoft.com/en-us/documentation/articles/powershell-install-configure/#Connect).

### Download Public Utilities
Download Putty executables from http://www.putty.org and keep them in `.\automation_root_folder\tools`. You should have the following utilities:

        •	plink.exe
        •	pscp.exe
        •	putty.exe
        •	puttygen.exe

Download dos2unix executables from http://sourceforge.net/projects/dos2unix/ and keep them in `.\automation_root_folder\tools`. You should have the following utilities:

        •	dos2unix.exe
		
Download 7-zip executable from http://www.7-zip.org/ ( Direct Download Link : http://www.7-zip.org/a/7za920.zip ) and keep them in `.\automation_root_folder\tools`. You should have the following utility:

        •	7za.exe
		
### Update Azure_ICA_all.xml file
1.	Setup Subscription details.

      Go to Config > Azure > General and update following fields :

        a.	SubscriptionID
        b.	SubscriptionName
        c.	CertificateThumbprint (Make sure you have installed a management certificate and can access it via the Azure Management Portal (SETTINGS->MANAGEMENT CERTIFICATES). )
        d.	StorageAccount
        e.	Location
        f.	AffinityGroup (Make sure that you either use <Location> or <AffinityGroup>. Means, if you want to use Location, then AffinityGroup should be blank and vice versa )

  Example :
  ```xml
  <General>
    <SubscriptionID>Your Subscription ID</SubscriptionID>
    <SubscriptionName>Your Subscription Name</SubscriptionName>
    <CertificateThumbprint>Certificate associated with your subscription</CertificateThumbprint>
    <ManagementEndpoint>https://management.core.windows.net</ManagementEndpoint>
    <StorageAccount>your current storage account</StorageAccount>
    <Location>Your preferred location</Location>
    <AffinityGroup></AffinityGroup>
  </General>
  ```

2.	Add VHD details in XML File.

      Go to Config > Azure > Deployment > Data. Make sure that your "VHD under test" should be present here in one of <Distro>..</Distro> entries. If your VHD is not listed here. Create a new Distro element and add your VHD details.

  Example for ASM:
  ```xml
  <Distro>
    <Name>Distro_Name</Name>
    <OsImage>Distro_OS_Image_Name_As_Appearing_under_Azure_OS_Images</OsImage>
  </Distro>
  ```

  Example for ARM:
  ```xml
  <Distro>
    <Name>Distro_Name</Name>
    <OsVHD>Distro_OS_VHD_Name_As_Appearing_under_Azure_storage.vhd</OsVHD>
  </Distro>
  ```

3.  Save file.

### Prepare VHD to work in Azure
`Applicable if you are uploading your own VHD with FreeBSD/Linux OS to Azure.`

A VHD with FreeBSD/Linux OS must be made compatible to work in Azure environment. This includes –

        1.	Installation of FreeBSD/Linux Integration Services to FreeBSD/Linux VM (if not present)
        2.	Installation of Azure Agent to FreeBSD/Linux VM (if not installed.)
        3.	Installation of minimum required packages. (Applicable if you want to run Tests using Automation code)

Please follow the steps mentioned at:
http://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-create-upload-vhd/

### Prepare VHD to work with Automation code.
`Applicable if you are using an existing uploaded VHD / Platform Image to run automation.`

To run automation code successfully, you need to install the following packages in your FreeBSD/Linux VHD:

	pkg install python27
	ln -s /usr/local/bin/python2.7 /usr/bin/python
	echo "y" | pkg install Security/py-paramiko
	echo "y" | pkg install Py27-setuptools27
	
	
	echo "y" | pkg install bash
	ln -s /usr/local/bin/bash /bin/bash
	echo "y" | pkg install sudo
	echo "y" | pkg install unix2dos
	echo "y" | pkg install wget
	echo "y" | pkg install iperf
	echo "y" | pkg install bind-tools
	echo "y" | pkg install base64
	echo "y" | pkg install gcc
	echo "y" | pkg install psmisc
	echo "y" | pkg install tcpdump
	echo "y" | pkg install git
	echo "y" | pkg install ca_root_nss
	mv /etc/ssl/cert.pem /etc/ssl/cert.pem.old
	ln -s /usr/local/share/certs/ca-root-nss.crt /etc/ssl/cert.pem


### Some useful notes to install the Azure agent
	git clone https://github.com/karataliu/WALinuxAgent.git
	cd WALinuxAgent
	git branch --remote
	git checkout 2.1 
	python setup.py install
	ln -sf /usr/local/sbin/waagent /usr/sbin/waagent
	ln -sf /usr/local/sbin/waagent2.0 /usr/sbin/waagent2.0
	chmod a+x  /usr/sbin/waagent
	chmod a+x  /usr/sbin/waagent2.0
	chmod a+x  /etc/rc.d/waagent
	/usr/sbin/waagent -version
	echo "y" | waagent -deprovision+user
	echo  'waagent_enable="YES"' >> /etc/rc.conf 
 

### Some useful notes to create the user 'bsduser'
	#To add a user called bsduser, run
	pw useradd -n bsduser -s /bin/csh -m
    	echo "the_passwd_of_bsduser" | pw mod user bsduser  -h 0

	#next we should add 'bsduser' into the group 'wheel' and enable the full sudo permission for it (e.g., uncomment out "%wheel ALL=(ALL) NOPASSWD: ALL" in /usr/local/etc/sudoers)


### Create SSH Key Pair
`PublicKey.cer – PrivateKey.ppk`

A FreeBSD/Linux Virtual machine login can be done with Password authentication or SSH key pair authentication. You must create a Public Key and Private key to run automation successfully. To learn more about how to create SSH key pair, please visit [here](http://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-use-ssh-key/).

After creating Public Key (.cer) and putty compatible private key (.ppk), you must put it in your `automation_root_folder\ssh\` folder and mention their names in Azure XML file.

### VNET Preparation
`Required for executing Virtual Network Tests`

#### Create a Virtual Network in Azure
A virtual network should be created and connected to Customer Network before running VNET test cases. To learn about how to create a virtual network on Azure, please visit [here](https://azure.microsoft.com/documentation/articles/vpn-gateway-site-to-site-create/).

#### Create A customer site using RRAS
Apart from Virtual Network in Azure, you also need a network (composed of Subnets and DNS server) to work as Customer Network. If you don’t have separate network to run VNET, you can create a virtual customer network using RRAS. To learn more, please visit [here](https://msdn.microsoft.com/en-us/library/dn636917.aspx).

## How to Start Automation
Before starting Automation, make sure that you have completed steps in chapter [Prepare Your Machine for Automation Cycle](#prepare)

        1.	Start PowerShell with Administrator privileges
        2.	Navigate to folder where automation code exists
        3.	Issue automation command

#### Automation Cycles Available
        1.	BVT
        2.	NETWORK
        3.	VNET
        4.	E2E
        5.	DEPLOYMENT
        6.	EXTENSION


#### Command to Start any of the Automation Cycle
    For ASM:
        .\AzureAutomationManager.ps1 -xmlConfigFile .\Azure_ICA_ALL.xml -runtests –Distro <DistroName> -cycleName <TestCycleToExecute>

    For ARM:
        .\AzureAutomationManager.ps1 -xmlConfigFile .\Azure_ICA_ALL.xml -runtests –Distro <DistroName> -cycleName <TestCycleToExecute>  -UseAzureResourceManager
#### More Information
For more details, please refer to the documents [here](https://github.com/FreeBSDonHyper-V/azure-freebsd-automation/tree/master/Documentation).

#### Known issues
	The below cases can fail:

	BVT-VERIFY-NO-ERROR-IN-LOGS: expected on 11-CURRENT -- we have a few warning/error message in the logs.
	BVT-VERIFY-BOOT-ERROR-WARNINGS : expected on 11-CURRENT -- we have a few warning/error message in 'dmesg'.
	BVT-HVMODULES-CHECK: expected on 11-CURRENT. We're fixing the test case.

	(we're analyzing the below cases:)
	BVT-MTAB-ENTRY-CHECK
	BVT-RESOURCE-DISK-FILESYSTEM
	BVT-VERIFY-MNT-RESOURCE-WRITABLE
	NETWORK-LB-CUSTOMPROBE-SAME-PORT-IPERF-PARALLEL-TCP
	NETWORK-LB-CUSTOMPROBE-SAME-PORT-ONE-NODE-DOWN-TCP
	NETWORK-LB-CUSTOMPROBE-SAME-PORT-TWO-NODES-DOWN-TCP
	NETWORK-LB-NO-PROBE-PORT-IPERF-TCP-PARALLEL
	NETWORK-LB-CUSTOMPROBE-OTHER-PORT-IPERF-TCP
	NETWORK-LB-CUSTOMPROBE-OTHER-PORT-IPERF-PARALLEL-TCP
	NETWORK-LB-CUSTOMPROBE-OTHER-PORT-ONE-NODE-DOWN-TCP
	NETWORK-LB-CUSTOMPROBE-OTHER-PORT-TWO-NODES-DOWN-TCP
	NETWORK-IDNS-SINGLEHS-CHANGED-HOSTNAME
