### Example VMware Kickstart For Non-DHCP Environment

This repo contains a VMware ESXi Kickstart (ks.cfg) file for automated ESXi deployments in a non-DHCP environment.

### Story Behind the Idea
- We are planning to install ESXi (version 8 at that time) on a total of 80 ESXi hosts and due to regulatory measures the environment required all components to have static IPs and no DHCP server to be used.

- To workout this we tried to find a to natively boot the UCS Blades using the ESXi ISO via vMedia Policy. It is a wonderful feature in Cisco UCS that allows to attach an ISO media to a number of blade servers managed by a UCSM.

- The only thing we need was a automated installation script(aka the kickstart) for ESXi, now most kickstart files we encountered assumes a DHCP server to be there already so IP assignment is offloaded to a DHCP server at least for initial boot process and later it can be changed to a static IP. But it helps because the server or more precisely the ESXi bootloader can be told to use a kickstart file that is located somewhere on a remote server.

- Now since we don't have a DHCP server it is not possible for our ESXi ISO to download a kickstart located elsewhere. We need to build a self-contained custom ISO to carry the kickstart.

- Now the other problem was how we are going to distribute all the IPs  we kept aside for ESXi hosts from the IP pool making sure the script assign an IP to only one host and avoid any possibility of duplicates.

- It turns out we can use MAC addresses from the Service Profiles we have assigned to blade servers. So we set out to create a simple CSV file which contains the MAC as unique identifier for each server. The format we used is as follows:

| hostname                 | MAC Address(unique identified, below are examples only) | IP Address  |
| ------------------------ | ------------------------------------------------------- | ----------- |
| esxi1.example.domain.org | ####:##:##:##:5E                                        | 10.10.10.10 |
| esxi2.example.domain.org | ##:##:##:##:##:5F                                       | 10.10.10.11 |

- Next we wrote a python script( FYI, ESXi contains the python interpreter out of box) to query the MAC address of server and then it searches throw rows in CSV to find a match against MAC address of the server to determine what specific parameters like the hostname and IP Address belong to this server.

- Once a match is found it executes a series of commands to set all the relevant network parameters.

- The kickstart file does not copy the script from a location rather we preferred to embed the script directly into kickstart (thanks to shell's heredoc construct).

- Have a look at the script to understand it better.

---

### Credits:
- Thanks to 'The Odd Angry Shot' for there wonderful [article]( https://theoddangryshot.com/2016/09/07/esxi-kickstart-with-python/)
- Thanks to 'Virten' for creating the [esxi_ks_injector](https://github.com/fgrehl/virten-scripts/tree/master/bash/esxi_ks_injector) tool
