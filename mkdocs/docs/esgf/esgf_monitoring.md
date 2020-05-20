ESGF MONITORING
============================

* Version: 0.0.1
* Date: 15/05/2020
* Authors: Pierre Logerais
* Keywords: centos esgf monitoring

## Description

This procedure describes how to monitor ESGF, and a few signs to be aware of.

## Pre requisites

A Linux machine with an SSH access to vesg and esgf-node.

## Monitoring : basics, tools to use and metrics to refer to

### Tools to monitor activity

* glances

It is your main tool, as it provides lots of useful information.

Glances lets you know the following :

- CPU activity in percentage of the **total** available CPU (not just one, unlike `top`)

- Memory usage, in percent of the total

- Swap usage

- Load average, over the past 1, 5 and 15 minutes

- Network usage (but there are better tools for that, that will be discussed below)

- iowait, aka percentage of processes that are waiting.

- Memory usage, CPU usage, command, uptime, and other stats, for different processes.

* nload

Nload is a tool designed specifically to monitor network usage. You can use the command `nload` to have a general overview, but you can’t really get more specific than that, unfortunately.

* iftop

iftop allows you to see how many connections are opened to the outside world, with which IP addresses. You can use the following command as root :

```iftop -n # -n prevents from resolving IP addresses to hostnames, which generates traffic``` 

* netstat

Similarly to iftop, it lets you list opened connections to the machine.

### Metrics in ESGF machines

There are a few metrics to consider in ESGF :

- iowait corresponds to the percentage of processes that are waiting but don’t do anything. It should stay relatively low.

- Swap usage should stay as low as possible. If the node is requested a lot, the machine might have a full RAM and swap as a result, but that can become a problem when the swap is full too. In that case, monitor closely and watch network activity.

- network usage is bound to be really high when the ESGF node is requested a lot. This is not, in itself, a problem. However, you can encounter situations where there are timeouts often, and in that case, check if specific IPs are doing a lot of requests (it can happen with bots).

- Load average should stay relatively low. Keep in mind that vesg has 8 cores and esgf-node has 4 only, so seeing a load of 8 or slightly more on vesg isn’t a problem. It can become a problem after that, though.

### Use cases of monitoring

* Example of bot IP addresses

We noticed several cases of the data node being overloaded. There were lots of timeouts when loading vesg.ipsl.umpc.fr on a browser, or the search API of the index node. However, `nload` showed a perfectly normal network activity. 

Running `netstat -puWtn | less` and exploring the results made it clear that we had a few IPs that had opened lots of connections. It was confirmed :

```[root@vesg ~]# netstat -puWtn | grep 159.226.234.13 | wc -l
513
[root@vesg ~]# netstat -puWtn | grep 117.172.173.88 | wc -l
201
[root@vesg ~]# 
```

Resolution of these addresses gave nothing for `117.172.173.88`, but `159.226.234.13` was the address of a chinese telephone operator. That meant it was quite clear that it was a bot.

We didn’t see IPs of the same classes pinging our machines, which told us that we had no reason to impose a wider block than on these IP addresses. Therefore, we added an iptables rule for both of these addresses only :

```
iptables -A INPUT -i ens160 -m state --state NEW -s 117.172.173.88 -j REJECT --reject-with icmp-port-unreachable
iptables -A INPUT -i ens160 -m state --state NEW -s 159.226.234.13 -j REJECT --reject-with icmp-port-unreachable
```

After saving the iptables rules and restarting the node, everything worked fine.
