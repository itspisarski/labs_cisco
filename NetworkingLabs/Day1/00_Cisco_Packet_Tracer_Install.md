# Lab 00 - Cisco Packet Tracer Discovery

**Objective:** Discover the key components of Cisco Packet Tracer


### ⚙️ Steps

1. On your Virtual Machine, open the Cisco Packet Tracer Application
3. On the bottom left of the screen, we will be using different object during the labs, we will be mostly using PCs, Switches and Routers
![Screenshot1](https://github.com/itspisarski/labs_cisco/blob/main/NetworkingLabs/ressources/networking_screenshot_1.png)
> You can drag and drop an object, and press delete to remove it from the canva.

3. In the bottom left section, choose "endpoints" as highlighted in the screenshot, then drag and drop a PC. Finally left click on the PC you just added to reach the same page as the screenshot below.
![Screenshot2](https://github.com/itspisarski/labs_cisco/blob/main/NetworkingLabs/ressources/networking_screenshot_2.png)
> This page is the physical view of our hardware, we won't be using it much
4. Navigate to Config, this is where we can see all the interfaces of the device and configure specific IPs and Subnets
![Screenshot3](https://github.com/itspisarski/labs_cisco/blob/main/NetworkingLabs/ressources/networking_screenshot_3.png)
> We will be using this page to set up specific subnets and ip adresses in our labs
5. Navigate to Desktop view. Here we will have access to some tools associated with the device
![Screenshot4](https://github.com/itspisarski/labs_cisco/blob/main/NetworkingLabs/ressources/networking_screenshot_4.png)
6. Click on the command prompt
![Screenshot5](https://github.com/itspisarski/labs_cisco/blob/main/NetworkingLabs/ressources/networking_screenshot_5.png)
7. Now we will add a switch in our lab, look at the bottom left for the 2960 switch, then drag and drop it. Finally left click on the switch you just added.
![Screenshot6](https://github.com/itspisarski/labs_cisco/blob/main/NetworkingLabs/ressources/networking_screenshot_6.png)
8. You are landing on the physical view, again not very usefull for us, let's look at the config tab
![Screenshot7](https://github.com/itspisarski/labs_cisco/blob/main/NetworkingLabs/ressources/networking_screenshot_7.png)
9. This time we have way more interfaces than our PC, well that's good since our switch role will be to connect multiple host
> The config tab can be used to manually configure the interfaces, without use of a CLI, altough, automation will require us to learn about CLI commands
10. Now let's look at the CLI, this is where we will be using our Cisco IOS commands
![Screenshot8](https://github.com/itspisarski/labs_cisco/blob/main/NetworkingLabs/ressources/networking_screenshot_8.png)
11. Now let's move back to the PC we added and leave the switch. Click on the PC and navigate to Config tab, then FastEthernet0. In this page add an ip address as indicated in the screenshot. The subnet will be added automatically.
![Screenshot9](https://github.com/itspisarski/labs_cisco/blob/main/NetworkingLabs/ressources/networking_screenshot_9.png)
12. Our PC has now a static Ip Adress, next step, let's connect it to the switch. To do so, choose the lightning icon at the bottom left, and choose Straight-Trough cable. Then click on the PC and choose the Ethernet interface to connect with as indicated in the screenshot.
![Screenshot10](https://github.com/itspisarski/labs_cisco/blob/main/NetworkingLabs/ressources/networking_screenshot_10.png)
13. Drag the cable to the switch on an interface.
![Screenshot11](https://github.com/itspisarski/labs_cisco/blob/main/NetworkingLabs/ressources/networking_screenshot_11.png)
14. You will see an orange circle next to the switch, this is the interface status, meaning it's starting up.
![Screenshot12](https://github.com/itspisarski/labs_cisco/blob/main/NetworkingLabs/ressources/networking_screenshot_12.png)
> You can click on the switch and go the the CLI tab to see the startup of the interface, following our new connection !
15. You can use the fastforward button to speed up the process and you should have everything back to green !
![Screenshot13](https://github.com/itspisarski/labs_cisco/blob/main/NetworkingLabs/ressources/networking_screenshot_13.png)

16. Congratulations, you just finished your quick overview of cisco packet tracer, and you are now ready for the labs ! 

