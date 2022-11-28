# Edge-Cassandra

This repository is used to analyze Cassandra’s performance in different networks. 
We vary Cassandra’s attributes and the overall conditions in each network to simulate a real-world scenario and study 
the effect on the total time taken for a particular request.

## Setup

### Part 1: Continuum

1. Clone the repository into your host machine.
2. Edit `continuum/configuration/infra_only.cfg` to set the number of nodes and cores according to your machine's capability.
    We have set it as follows:
    ```
    cloud_nodes = 1
    edge_nodes = 5
    endpoint_nodes = 1

    cloud_cores = 4
    edge_cores = 2
    endpoint_cores = 1
    ```
3. Run `python3 main.py configuration/infra_only.cfg` from the `continuum` directory.
4. Wait a few minutes for the VMs to be deployed. Note down the ssh commands for the created VMs.
5. Run `virsh list` to check if the VMs are up and running.

### Part 2: Cassandra

1. Use one of the previous commands to ssh into an edge VM. 
For example: `ssh edge0@192.168.122.11 -i /home/f20190095/.ssh/id_rsa_benchmark`
2. Run `nano cassandra_setup.sh`
3. Paste the contents from `scripts/cassandra_setup` into the previous file.
4. Exit the editor and run `chmod +x cassandra_setup.sh`
5. Run `./cassandra_setup.sh`
6. Wait a few minutes for the installation to complete
7. Edit the `/etc/cassandra/cassandra.yaml` file to change the following fields:

    a) Change the `seeds` field to include the IP addresses of all the edge VMs.
    
    For example: `seeds: "127.0.0.1:7000, 192.168.122.11, 192.168.122.12, 192.168.122.13, 192.168.122.14, 192.168.122.15"`
    
    b) Change the `listen_address` and `rpc_address` fields to match the IP address of the current node.
    
    For example, for edge0: `listen_address: 192.168.122.11`, `rpc_address: 192.168.122.11`
    
    c) Add `auto_bootstrap: false` at the end of the file
    
8. To allow Cassandra communication between nodes, run the following command (replace `<ip_address>` with the IP address of every other node):

    ```
    sudo iptables -A INPUT -p tcp -s <ip_address> -m multiport --dports 7000,9042 -m state --state NEW,ESTABLISHED -j ACCEPT
    ```

    For example, for edge0, we will run:
    ```
    sudo iptables -A INPUT -p tcp -s 192.168.122.12 -m multiport --dports 7000,9042 -m state --state NEW,ESTABLISHED -j ACCEPT
    sudo iptables -A INPUT -p tcp -s 192.168.122.13 -m multiport --dports 7000,9042 -m state --state NEW,ESTABLISHED -j ACCEPT
    sudo iptables -A INPUT -p tcp -s 192.168.122.14 -m multiport --dports 7000,9042 -m state --state NEW,ESTABLISHED -j ACCEPT
    sudo iptables -A INPUT -p tcp -s 192.168.122.15 -m multiport --dports 7000,9042 -m state --state NEW,ESTABLISHED -j ACCEPT
    ```
    
9. Run `sudo service cassandra restart`
10. Repeat steps 1-9 for each edge node
11. Run `nodetool status` to check all the nodes are up and communicating with each other.
12. 

### Part 3: Network
