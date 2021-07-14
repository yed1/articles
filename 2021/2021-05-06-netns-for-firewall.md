# Use Linux network namespace to test firewall rules
a quick howto

## Why
* Simple: need one Linux box
* Safe and clean: play with firewall rules in a disposable sandbox: no worry of being locked out, no worry of polluting host

## How (the steps were performed in a Ubuntu 18.04)
* Setup: connection and baseline of no firewall
    ```
    $ sudo ip netns add test_ns # create a new network namespace
    $ sudo ip link add name veth0 type veth peer name veth1 netns test_ns # add a pair of VETH devices, veth0 in host, veth1 in the  namespace
    $ sudo ip addr 192.168.100.10/24 dev veth0
    $ sudo ip link set veth0 up
    $ sudo ip netns exec test_ns ip addr add 192.168.100.11/24 dev veth1
    $ sudo ip netns exec test_ns ip link set veth1 up
    $ ping 192.168.100.11 # verify icmp available
    $ sudo ip netns exec test_ns nc -l -p 23 & # start tcp 23 in the namespace
    $ sudo ip netns exec test_ns netstat -tlpn # verify tcp 23 started in the namespace
    $ nc -vz 192.168.100 # verify tcp 23 available
    ```
* Experiment: update, test, cycle
    ```
    $ sudo ip netns exec test_ns /bin/sh
        # iptables [-t filter] -L --line-numbers -n # list chains and rules in the (default) filter table
        # iptables -N test_chain # add a custom chain named test_chain to the (default) filter table
        # iptables -A INPUPT -j test_chain> # append a rule in the chain
        # iptables -D INPUT -j <custom_chain_i> # delete the rule
        # iptables -I <test_chain> 2 ... # insert a rule before the rule 2 in the chain from 1st command
    $ ping 192.168.100.11 # verify client can icmp to the namespace
    $ sudo ip netns exec test_ns nc -l -p 23 & # listen to tcp port 23 in the namespace
    $ sudo ip netns exec test_ns netstat -tlpn # verify tcp port 23 in the namespace is being listened
    $ nc -vz 192.168.100 # verify client can tcp to port 23 in the namespace
    ```
* Cleanup
    ```
    $ sudo ip netns delete test_ns
    $ sudo ip link del veth0 # unneeded as it is also done as part of 'ip netns delete test_ns'
    ```

## Reference
* https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking/
    * `$ ip link help`
