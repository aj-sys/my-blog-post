
# Building a VPC on Linux with Network Namespaces

## Introduction

By Ajara Amadu

In this project, I demonstrate how to simulate a Virtual Private Cloud (VPC) on Linux using network namespaces and virtual Ethernet (veth) interfaces.  
This setup helps learners and cloud enthusiasts understand subnetting, routing, and connectivity in a hands-on way — without needing an actual cloud environment.

---

## Project Architecture

The architecture consists of:

- **Router namespace (router2)** – simulates a VPC router.  
- **Public subnet namespace (pub_subnet2)** – hosts a simple HTTP server.  
- **Private subnet namespace (priv_subnet2)** – client namespace to test connectivity.  
- **veth pairs** – connect namespaces and simulate network links.  

### Architectural Diagram

![Architecture](screenshots/vpc_architetural_diagram.png)

---

## Implementation Steps

### Step 1: Create Network Namespace

```bash
sudo ip netns add router2
sudo ip netns add pub_subnet2
sudo ip netns add priv_subnet2


This sets up isolated network environments for our router and subnets.
________________________________________

### Step 2: Create Virtual Ethernet Interface

sudo ip link add veth-pub2 type veth peer name veth-r2-pub
sudo ip link add veth-priv2 type veth peer name veth-r2-priv

veth pairs act like physical network cables connecting namespaces.
________________________________________

sudo ip link set veth-pub2 netns pub_subnet2
sudo ip link set veth-r2-pub netns router2
sudo ip link set veth-priv2 netns priv_subnet2
sudo ip link set veth-r2-priv netns router2

Moves each interface into the correct namespace.
________________________________________

### Step 4: Configure IP Addresses

sudo ip netns exec pub_subnet2 ip addr add 10.0.1.2/24 dev veth-pub2
sudo ip netns exec priv_subnet2 ip addr add 10.0.2.2/24 dev veth-priv2
sudo ip netns exec router2 ip addr add 10.0.1.1/24 dev veth-r2-pub
sudo ip netns exec router2 ip addr add 10.0.2.1/24 dev veth-r2-priv

Assigns IP addresses so subnets can communicate via the router.
________________________________________

### Step 5: Bring Up Interfaces

sudo ip netns exec pub_subnet2 ip link set veth-pub2 up
sudo ip netns exec router2 ip link set veth-r2-pub up
sudo ip netns exec priv_subnet2 ip link set veth-priv2 up
sudo ip netns exec router2 ip link set veth-r2-priv up

Activates all interfaces including loopback for proper network functionality.
________________________________________

### Step 6: Enable Packet Forwarding

sudo ip netns exec router2 sysctl -w net.ipv4.ip_forward=1

Lets the router namespace forward traffic between subnets.
________________________________________

### Step 7: Add Default Routes

sudo ip netns exec pub_subnet2 ip route add default via 10.0.1.1
sudo ip netns exec priv_subnet2 ip route add default via 10.0.2.1

Ensures packets know where to go outside each namespace.
________________________________________

### Step 8: Test Connectivity

sudo ip netns exec pub_subnet2 ping -c 3 10.0.1.1
sudo ip netns exec priv_subnet2 ping -c 3 10.0.1.2


Confirms that private and public subnets can reach each other via the router.
________________________________________

### Step 9: Run a Simple HTTP Server

sudo ip netns exec pub_subnet2 python3 -m http.server 80


Hosts a test web server in the public subnet.
________________________________________

### Step 10: Access from Private Subnet

sudo ip netns exec priv_subnet2 curl http://10.0.1.2


If you see HTML output, routing and connectivity are working correctly.
________________________________________

## Screenshots and Videos

- ### Isolation Testing 
  
 ![Isolation Testing](screenshots/isolation-testing.png)
  
- ### HTTP Server Running 
  
 ![HTTP Server Running](screenshots/http-server-running.png)
  
  
-  ### Private Subnet Access
  

 ![ private subnet access](screenshots/access-from-private-subnet.png)
  

 - ### Cleanup Success 
  
  ![Cleanup Success](screenshots/cleanup-success.png)
  
**Live screen recording:*

- ### Demo Video: [Watch Demo Video](screenshots/vpc-demo.mp4)
  
  
### Project Demo Video

```html <video width="640" height="360" controls>
  <source src="screenshots/vpc-demo.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>```


## Conclusion

This project demonstrates a hands-on, local simulation of a VPC using Linux namespaces. It’s useful for:
- Learning networking fundamentals.
- Practicing subnetting and routing.
- Understanding cloud network concepts without needing AWS.
GitHub Repository
You can find the full project here: [VPC on Linux Project]( https://github.com/aj-sys/vpc_setup_on_linux_project.git)


