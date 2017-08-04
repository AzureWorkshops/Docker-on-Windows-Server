## Overview
The final part of this workshop is to expose an IIS container outside of Azure. We're going to create a simple web server and access it from our local machine.

## IIS
We are going to deploy a basic container hosting IIS and then expose the sample IIS website outside of Azure.

From PowerShell, type the following:
```ps
docker run -d -p 80:80 microsoft/iis
```

This will download and run IIS in the background.  As stated earlier in this workshop, we often run services in _detached_ mode (`-d`).  As new parameter that you see here is mapping, or publishing (`-p`), ports  - very similar to a NAT, if you are familiar with the concept. There are two ports specified here separated by a colon.  The first number is the host's port while the second number is the container's port.  So, in essence, we are mapping the host's port 80 to the container's port 80.  If our container runs multiple services or a service requiring multiple ports, we can also specify a port range.

  1. Let's make sure that IIS is running successfully. Open a web browser on the virtual server and try to navigate to `http://localhost`. Oops. It seems we received an error. What did we do wrong? Let's investigate.

  2. If one is not already open, open a PowerShell window and type in `docker ps`.

  3. You should see something like the following:  
  ```ps
  CONTAINER ID        IMAGE               COMMAND                   CREATED             STATUS              PORTS                NAMES
c21c24a24027        microsoft/iis       "C:\\ServiceMonitor..."   23 seconds ago      Up 14 seconds       0.0.0.0:80->80/tcp   kickass_raman
  ```

  4. Notice the _Ports_ column. Our external port is not mapped to the loopback address (e.g. `127.0.0.1` or `localhost`).  Long story short, this is due to a way Windows maps its network interfaces.
  
  5. We need to get the actual, virtual IP address of the container.  To do this, type the following at the PowerShell prompt (change the container id to your container's id):
  ```ps
  docker inspect --format '{{ .NetworkSettings.Networks.nat.IPAddress }}' c21
  ```

   7. So, let's use the returned IP instead of the `localhost` to load our website.  In the browser change the URL to `http://<your container's virtual IP address>:8080` (e.g. http://192.168.150.195).

## Windows Firewall
Before we can access the IIS server from outside of Azure, we need to open the port in Windows Firewall.

  1. On the remote server, click the **Start Menu** and begin typing **Firewall**. This should provide a menu option for **Windows Firewall with Advanced Security**. Click on it.

  2. Select **Inbound Rules** in the left pane and click **New Rule...** in the right pane.

  3. For **Rule Type**, select **Port** and click **Next**.

  4. On the **Protocol and Ports** page, select **TCP**, **Specific local ports**, and enter **80** in the input box. Click **Next**.

  5. For **Action**, choose **Allow the connection** and click **Next**.

  6. For **Profile**, leave all three profiles checked and click **Next**.

  7. Finally, name the rule **Allow-HTTP** and click **Finish**.

## Network Security Group (NSG)
Now that our web server is running, let's make it available outside of Azure.

When we created our Windows virtual machine, we accepted the defaults, including the default settings for our NSG.  The default settings only allowed RDP (port 3389) access. We need to add a rule to our NSG to allow HTTP traffic over port 80.

  1. If you are not still there, go back to the Azure portal and navigate to the settings of your CentOS virtual machine.

  2. In the left menu, click on **Network interfaces** <img src="https://raw.githubusercontent.com/AzureWorkshops/images/master/icons_network_interfaces.jpg" class="inline"/>.

  3. This will open the _Network Interfaces_ blade for your CentOS virtual machine. Click on the singular, listed interface.

  4. In the left menu, click on **Network security group** <img src="https://raw.githubusercontent.com/AzureWorkshops/images/master/icons_network_security_group.jpg" class="inline"/>.

  5. This will list the currently active NSG.  In our case, it should be the NSG that was created with our virtual machine - **docker-centos-nsg**.  Click on the NSG (**NOTE:** Click on the actual NSG link, **NOT** on **Edit**).

  6. In the left menu, click on **Inbound security roles** <img src="https://raw.githubusercontent.com/AzureWorkshops/images/master/icons_inbound_security_rules.jpg" class="inline"/>.

  7. At the top of the blade, click **Add** <img src="https://raw.githubusercontent.com/AzureWorkshops/images/master/icons_add.jpg" class="inline"/>.

  8. Enter the following configuration:

      * Name: **allow-http**
      * Priority: **1010**
      * Source: **Any**
      * Service: **HTTP**
      * Action: **Allow**

  9. Click **OK**.

This should only take a couple of seconds.  Once you see the rule added, open a new browser and navigate to the IP address of your Windows virtual machine, including the port number.  The IP address used in this workshop's screen shots is **40.121.223.152** (your IP address will be different).  Using the aforementioned IP address, I would direct my browser to **http://40.121.223.152/**.  Doing so, you should see the IIS landing page.