# Web-Server-Set-up-with-Azure-Load-Balancer
Azure Load Balancer Setup Guide
This guide will walk you through the process of setting up a basic Azure Load Balancer to distribute web traffic to two virtual machines (VMs) running Internet Information Services (IIS).
Prerequisites
	• An active Azure subscription.
**Step 1: Build a Virtual Network (VNet)
**A Virtual Network is the fundamental building block for your private network in Azure. It enables many types of Azure resources, such as Azure Virtual Machines, to securely communicate with each other, the internet, and on-premises networks.
	1. Go to the Azure Portal: Log in to portal.azure.com.
	2. Create a Resource Group:
		○ From the Azure portal menu, select Resource groups or search for it.
		○ Click + Create.
		○ Enter a Resource group name (e.g., LoadBalanced).
		○ Select a Region (e.g., Central India).
		○ Click Review + create, then Create.
<img width="1158" height="578" alt="image" src="https://github.com/user-attachments/assets/eecd086d-ab10-441b-a747-25d0b8977dcc" />
	3. Create a Virtual Network:
		○ From the Azure portal menu, select Virtual networks or search for it.
		○ Click + Create.
		○ Basics tab:
			§ Subscription: Your Azure subscription.
			§ Resource group: Select the LoadBalanced you just created.
			§ Name: Enter vnetforloadbalancer.
			§ Region: Select the same region as your resource group.
		○ IP Addresses tab:
			§ Leave the default IP address space (e.g., 10.0.0.0/16).
			§ Subnet: The default subnet default (e.g., 10.0.0.0/24) is usually sufficient for this basic setup. If not present, click + Add subnet and create one (e.g., Name: backend-subnet, Address range: 10.0.0.0/24).
		○ Security, Tags, Review + create: Leave defaults, then click Review + create and finally Create.
<img width="1289" height="634" alt="image" src="https://github.com/user-attachments/assets/8b75a52e-6e6f-4557-a2ef-396d2eae1d94" />

**Step 2: Establish Virtual Machines
**You need at least two virtual machines to demonstrate load balancing. We'll create two Windows Server VMs and place them in the VNet.
	1. Create VM 1:
		○ From the Azure portal menu, select Virtual machines or search for it.
		○ Click + Create > Azure virtual machine.
		○ Basics tab:
			§ Subscription: Your Azure subscription.
			§ Resource group: LoadBalanced .
			§ Virtual machine name: VM-01.
			§ Region: Same as your VNet.
			§ Availability options: Select Availability set.
				□ Create new:
					® Name: MyAvailabilitySet.
					® Fault domains: 2 (default).
					® Update domains: 5 (default).
			§ Image: Windows Server 2022 Datacenter GEN2 (or a similar Windows Server image).
			§ Size: Choose a suitable size (e.g., Standard_B2s for testing).
			§ Administrator account: Create a Username and Password. Remember these credentials.
			§ Inbound port rules: Select Allow selected ports and choose RDP (3389).
<img width="1266" height="647" alt="image" src="https://github.com/user-attachments/assets/9cd2309a-8f1a-46e9-8c74-929f5e1df80f" />
			○ Disks tab: Leave defaults.
			○ Networking tab:
			§ Virtual network: Select MyVNet.
			§ Subnet: Select loadbalanced (or default if that's what you named it).
			§ Public IP: Select Create new.
			§ NIC network security group: Basic.
			§ Load balancing: Select No. (We'll add it later).
		○ Management, Advanced, Tags, Review + create: Leave defaults, then click Review + create and finally Create.
	2. Create VM 2:
		○ Repeat the steps for VM 1, but make the following changes:
			§ Virtual machine name: VM-02.
			§ Availability options: Select Availability set and choose the existing MyAvailabilitySet.
			§ Ensure all other settings (Resource group, VNet, Subnet, Image, Size, Admin account) are the same as VM-01.
<img width="1267" height="638" alt="image" src="https://github.com/user-attachments/assets/61b3f475-36cc-4ba6-be47-2d58efbe49d3" />

Wait for both VMs to be deployed. This may take a few minutes.
**Step 3: Install IIS on Both Virtual Machines
**Internet Information Services (IIS) is a flexible, secure, and manageable Web server for hosting web content.
	1. Connect to VM-01 via RDP:
		○ In the Azure portal, navigate to VM-01.
		○ On the Overview blade, click Connect > RDP.
		○ Click Download RDP File and open it.
		○ Enter the username and password you set during VM creation.
	2. Install IIS on VM-01:
		○ Once connected, open Server Manager.
		○ Click Add roles and features.
		○ Click Next until you reach Server Roles.
		○ Select Web Server (IIS). When prompted, click Add Features.
		○ Click Next until Confirmation, then click Install.
		○ Once installation is complete, close Server Manager.
		○ Create a simple HTML page:
			§ Open File Explorer and navigate to C:\inetpub\wwwroot.
			§ Create a new file named index.html.
			§ Open index.html with Notepad and add some simple HTML content to identify this VM, e.g.:
<!DOCTYPE html>
<html>
<head>
    <title>VM-01</title>
</head>
<body>
    <h1>Welcome to VM-01!</h1>
    <p>This page is served from the first virtual machine.</p>
</body>
</html>
			§ Save the file.
		○ Test IIS locally: Open a web browser on VM-01 and go to http://localhost. You should see your VM-01 page.
<img width="531" height="281" alt="image" src="https://github.com/user-attachments/assets/a5637016-61e0-4121-9338-c4b0ab6198f1" />
		○ Allow HTTP traffic through Windows Firewall:
			§ Open Windows Defender Firewall with Advanced Security.
			§ In the left pane, select Inbound Rules.
			§ In the right pane, click New Rule....
			§ Select Port, then Next.
			§ Select TCP, and enter 80 for Specific local ports, then Next.
			§ Select Allow the connection, then Next.
			§ Leave profiles as default, then Next.
			§ Give the rule a Name (e.g., Allow HTTP) and click Finish.
	1. Repeat for VM-02:
		○ Connect to VM-02 via RDP.
		○ Install IIS on VM-02 using the same steps.
		○ Create an index.html file in C:\inetpub\wwwroot with content identifying it as VM-02, e.g.:
<!DOCTYPE html>
<html>
<head>
    <title>VM-02</title>
</head>
<body>
    <h1>Welcome to VM-02!</h1>
    <p>This page is served from the second virtual machine.</p>
</body>
</html>
		○ Allow HTTP traffic through Windows Firewall on VM-02.
<img width="425" height="277" alt="image" src="https://github.com/user-attachments/assets/525b1817-262b-45e8-95bb-c66d281e5991" />

Step 4: Set up inbound port rule in NSG for each VM
Without this step performed, the VMs won't be able to receive incoming requesting from public internet , therefore won't be able to serve the web page to the outside world, public internet.
This is for VM 1 
<img width="1287" height="629" alt="image" src="https://github.com/user-attachments/assets/d1f9a72b-a4b5-46c4-8b55-e32d2de4ff51" />

This is for VM2
<img width="1288" height="653" alt="image" src="https://github.com/user-attachments/assets/f8396ea3-5e17-408f-a1c2-1f79271b12ba" />


Step 5: Construct a Load Balancer
Now, you'll create the Azure Load Balancer and configure its components to distribute traffic to your VMs.
	1. Create a Load Balancer:
		○ From the Azure portal menu, select Load balancers or search for it.
		○ Click + Create.
		○ Basics tab:
			§ Subscription: Your subscription.
			§ Resource group: LoadBalanced.
			§ Name: loadBL.
			§ Region: Same as your VNet and VMs.
			§ SKU: Select Standard. (Basic SKU has limitations for Availability Sets).
			§ Type: Public.
			§ Tier: Regional.
		○ Frontend IP configuration tab:
			§ Click + Add a frontend IP configuration.
			§ Name: FrontLoadConfig.
			§ IP version: IPv4.
			§ IP type: IP address.
			§ Public IP address: Click Create new.
				□ Name: frontendLBIP.
				□ SKU: Standard.
				□ Availability zone: No zone.
				□ Click OK.
			§ Click Add.
<img width="1231" height="141" alt="image" src="https://github.com/user-attachments/assets/d1872e19-3492-43d0-a377-6f9c12f65661" />
			○ Backend pools tab:
			§ Click + Add a backend pool.
			§ Name: MyBackendPool.
			§ Virtual network: MyVNet.
			§ IP version: IPv4.
			§ Backend Pool Configuration:
				□ Add: Select IP address.
				□ Associate: Select Virtual machine.
				□ Virtual machine: Select VM-01.
				□ IP address: Select the private IP of VM-01.
				□ Click Add.
				□ Repeat for VM-02.
			§ Click Add.
<img width="1276" height="217" alt="image" src="https://github.com/user-attachments/assets/63a63142-72bb-42e6-b034-647202608496" />
			○ Inbound rules tab:
			§ Health probes:
				□ Click + Add a health probe.
				□ Name: MyHealthProbe.
				□ Protocol: TCP.
				□ Port: 80 (for IIS).
				□ Interval: 5 (seconds).
				□ Unhealthy threshold: 2 (consecutive failures).
				□ Click Add.
<img width="1263" height="585" alt="image" src="https://github.com/user-attachments/assets/9984f126-2eaa-4ca5-a3a6-75f924d83e8f" />

§ Load balancing rules:
				□ Click + Add a load balancing rule.
				□ Name: MyHTTPRule.
				□ Frontend IP address: FrontLoadConfig.
				□ Backend pool: MyBackendPool.
				□ Protocol: TCP.
				□ Port: 80.
				□ Backend port: 80.
				□ Health probe: MyHealthProbe.
				□ Session persistence: None (for basic round-robin).
				□ Idle timeout (minutes): 4.
				□ Floating IP (direct server return): Disabled.
				□ Click Add.
		○ Outbound rules, Inbound NAT rules, Tags, Review + create: Leave defaults, then click Review + create and finally Create.
Wait for the Load Balancer to be deployed.
**Step 5: Run a Load Balancer Test
**Now it's time to test if your load balancer is correctly distributing traffic.
	1. Get the Load Balancer's Public IP:
		○ In the Azure portal, navigate to your loadBL.
		○ On the Overview blade, locate the Frontend IP configuration and note down the Public IP address.
	2. Access the Web Application:
		○ Open a web browser on your local machine (not on the VMs).
		○ Enter the Public IP address of your Load Balancer into the browser's address bar (e.g., http://20.xxx.xxx.xxx).
		○ You should see either the "Welcome to VM-01!" or "Welcome to VM-02!" page.
<img width="338" height="199" alt="image" src="https://github.com/user-attachments/assets/eafc9686-2d71-4b2d-9eed-a56eafff8591" /><img width="343" height="206" alt="image" src="https://github.com/user-attachments/assets/c9afb384-30a0-4bd7-9a65-0f837bc741b7" />
	3. Test Traffic Distribution:
		○ Refresh your web browser multiple times.
		○ Due to the round-robin nature of the load balancer (with Session persistence set to None), you should see the page switch between "Welcome to VM-01!" and "Welcome to VM-02!" as the load balancer distributes requests to different VMs.
		○ Simulate a VM failure:
			§ In the Azure portal, navigate to VM-01.
			§ Click Stop. Confirm the stop.
			§ Wait a minute or two for the health probe to detect the VM is unhealthy.
			§ Refresh your web browser accessing the Load Balancer's Public IP. You should now consistently see only the "Welcome to VM-02!" page. This demonstrates that the load balancer has detected the unhealthy VM and is routing all traffic to the healthy one.
<img width="338" height="199" alt="image" src="https://github.com/user-attachments/assets/be201310-11dd-4385-90e1-69643075e486" />
		§ Start VM-01 again and observe that traffic is again distributed.


**Conclusion**
You have successfully set up a basic Azure Load Balancer to distribute HTTP traffic to two virtual machines. This setup provides high availability for your web application by ensuring that if one VM becomes unavailable, traffic is automatically routed to the healthy VM(s).
