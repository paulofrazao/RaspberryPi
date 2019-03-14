## Installing Docker 18.09 on a Raspberry Pi.

What better way to say "Happy Pi Day" (03.14) than by installing Docker 18.09 on Raspberry Pi. This article will walk you through the process of installing Docker 18.09 on a Raspberry Pi. I came across many old articles that showed the process of installing Docker on Pi and many failed due to older versions and some syntax issues. Special shout out to @Stefan Scherer and his monitoring image (stefanscherer/monitor) along with the whoami image (stefanscherer/whoami) that allows Pimoroni Blinkt! LED's to turn on/off when scaling an application within a Swarm Cluster.

### Instructions

For this demo, I used 7 Raspberry Pi's 3 (model B+) and 1 Pimoroni Blinkt! LED for each Pi.

1. download the following Raspian image '2018-11-13-raspbian-stretch-full.img' from https://www.raspberrypi.org/downloads/raspbian/

2. use balenaEtcher (https://www.balena.io/etcher/) to write the image to each of your microusb cards.

3. to make DNS hostname resolution a little easier, I setup local hostnames on each PI. Below is an example.
```markdown
192.168.93.231 pi-mgr1 pi-mgr1.docker.com
192.168.93.232 pi-mgr2 pi-mgr2.docker.com
192.168.93.233 pi-mgr3 pi-mgr3.docker.com
192.168.93.241 pi-node1 pi-node1.docker.com
192.168.93.242 pi-node2 pi-node2.docker.com
192.168.93.243 pi-node3 pi-node3.docker.com
192.168.93.244 pi-node4 pi-node4.docker.com
```

4. On each of your Pi's, install the following:
a. Install the following prerequisites.
```markdown
sudo apt-get install apt-transport-https ca-certificates software-properties-common -y
```
b. Download and install Docker.
```markdown
curl -fsSL get.docker.com -o get-docker.sh && sh get-docker.sh
```
c. Give the 'pi' user the ability to run Docker.
```markdown
sudo usermod -aG docker pi
```
d. Import Docker CPG key.
```markdown
sudo curl https://download.docker.com/linux/raspbian/gpg | sudo apt-key add -
```
e. Setup the Docker Repo.
```markdown
vim /etc/apt/sources.list
add the following line and save:   deb https://download.docker.com/linux/raspbian/ stretch stable
```
f. Patch and update your Pi.
```markdown
sudo apt-get update
sudo apt-get upgrade
```
g. Start the Docker service.
```markdown
systemctl start docker.service
```
h. To verify that Docker is installed and running.
```markdown
docker info
```
i. You should now some information in regards to versioning, runtime,etc.

5. Now that Docker has been installed on all of your Pi's, we can now setup Docker Swarm.

6. On one of your Pi devices that will be a master node, type the following:
```markdown
docker swarm init
```

7.Once Docker initiates the swarm setup, you will be presented with a command to add additional worker nodes. Below is an example.
```markdown
docker swarm join --token <token-key> 192.168.93.231:2377
```
a. on each worker node paste the text in step 7

8.To add additional manager nodes, the token and string will be different than the worker string. In order to discover the correct string to add manager nodes, do the following command on an existing working manager node.
```markdown
docker swarm  join-token manager
```
b. copy and paste the output to each of your manager nodes

9. If you want to add additional worker nodes and don't have the correct syntax, just type the following on any of the working manager nodes to retrieve it.
```markdown
docker swarm join-token worker
```

10. To have a graphical representation of your current cluster, we will install the VIZ application. For more information, go to https://github.com/dockersamples/docker-swarm-visualizer. To install, type the following:
```markdown
docker service create \
--name=viz \
--publish=9090:8080/tcp \
--constraint=node.role==manager \
--mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
alexellis2/visualizer-arm:latest
```

11. Using a browser, connect to one of your master servers on port 9090. You should now see the Visualizer showing your worker and manager nodes.

12. Now we will install the monitor app that will be deployed on both the manager and worker nodes. Type the following on the one of the manager nodes.
```markdown
docker service create --name monitor --mode global \
--restart-condition any --mount type=bind,src=/sys,dst=/sys \
--mount type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
stefanscherer/monitor:1.2.0
```

13. Once the monitor app is install, we will now install the 'whoami' app. The 'whoami' app is a small application that will trigger the LED's on/off by scaling the application up and down. For each running instance, you will get one LED turned on. As we scale the application up to 5, you will have 5 LEDs turn on. As you scale up and down the number of LEDs that turn on will depend on how many containers you have running in your cluster. To install the application, type the following.
```markdown
docker service create --name whoami stefanscherer/whoami:1.1.0
```

14. Once deployed, you should have 1 LED turned on.

15. Now lets scale the application to 5. Type the following.
```markdown
docker service scale whoami=5
```

16. You should now have 5 LEDs on. Please that this will take some time as the Pi devices are not very fast and require some time to properly deploy and bring up.


If you have any suggestions to make this even better and/or easier, please let me know.

Thanks for your time!

--Paulo

