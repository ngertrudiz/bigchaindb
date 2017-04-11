# Set Up and Run a Cluster Node

This is a page of general guidelines for setting up a production BigchainDB node. Before continuing, please review the pages about production node [assumptions](node-assumptions.html), [components](node-components.html) and [requirements](node-requirements.html).

Note: These are just guidelines. You can modify them to suit your needs. For example, if you want to initialize the MongoDB replica set before installing BigchainDB, you _can_ do that. We don't cover all possible setup procedures here.


## Security Guidelines

There are many articles, websites and books about securing servers, virtual machines, networks, etc. Consult those.
There are some [notes on BigchainDB-specific firewall setup](../appendices/firewall-notes.html) in the Appendices.


## Sync Your System Clock

A BigchainDB node uses its system clock to generate timestamps for blocks and votes, so that clock should be kept in sync with some standard clock(s). The standard way to do that is to run an NTP daemon (Network Time Protocol daemon) on the node. (You could also use tlsdate, which uses TLS timestamps rather than NTP, but don't: it's not very accurate and it will break with TLS 1.3, which removes the timestamp.)

NTP is a standard protocol. There are many NTP daemons implementing it. We don't recommend a particular one. On the contrary, we recommend that different nodes in a cluster run different NTP daemons, so that a problem with one daemon won't affect all nodes.

Please see the [notes on NTP daemon setup](../appendices/ntp-notes.html) in the Appendices.


## Set Up Storage for MongoDB

We suggest you set up a separate storage device (partition, RAID array, or logical volume) to store the data in the MongoDB database. Here are some questions to ask:

* How easy will it be to add storage in the future? Will I have to shut down my server?
* How big can the storage get? (Remember that [RAID](https://en.wikipedia.org/wiki/RAID) can be used to make several physical drives look like one.)
* How fast can it read & write data? How many input/output operations per second (IOPS)?
* How does IOPS scale as more physical hard drives are added?
* What's the latency?
* What's the reliability? Is there replication?
* What's in the Service Level Agreement (SLA), if applicable?
* What's the cost?

There are many options and tradeoffs.


## Install and Run MongoDB

* [Install MongoDB](https://docs.mongodb.com/manual/installation/)
* [Run MongoDB (mongod)](https://docs.mongodb.com/manual/reference/program/mongod/)


## Install BigchainDB Server

### Install BigchainDB Server Dependencies

* [Install OS-level dependencies](../appendices/install-os-level-deps.html)
* [Install Python 3.4+](https://www.python.org/downloads/)

### How to Install BigchainDB Server with pip

BigchainDB (i.e. both the Server and the officially-supported drivers) is distributed as a Python package on PyPI so you can install it using `pip`. First, make sure you have an up-to-date Python 3.4+ version of `pip` installed:
```text
pip -V
```

If it says that `pip` isn't installed, or it says `pip` is associated with a Python version less than 3.4, then you must install a `pip` version associated with Python 3.4+. In the following instructions, we call it `pip3` but you may be able to use `pip` if that refers to the same thing. See [the `pip` installation instructions](https://pip.pypa.io/en/stable/installing/).

On Ubuntu 16.04, we found that this works:
```text
sudo apt-get install python3-pip
```

That should install a Python 3 version of `pip` named `pip3`. If that didn't work, then another way to get `pip3` is to do `sudo apt-get install python3-setuptools` followed by `sudo easy_install3 pip`.

You can upgrade `pip` (`pip3`) and `setuptools` to the latest versions using:
```text
pip3 install --upgrade pip setuptools
pip3 -V
```

Now you can install BigchainDB Server (and officially-supported BigchainDB drivers) using:
```text
pip3 install bigchaindb
```

(If you're not in a virtualenv and you want to install bigchaindb system-wide, then put `sudo` in front.)

Note: You can use `pip3` to upgrade the `bigchaindb` package to the latest version using `pip3 install --upgrade bigchaindb`.


### How to Install BigchainDB Server from Source

If you want to install BitchainDB from source because you want to use the very latest bleeding-edge code, clone the public repository:
```text
git clone git@github.com:bigchaindb/bigchaindb.git
python setup.py install
```


## Configure BigchainDB Server

Start by creating a default BigchainDB config file for a MongoDB backend:
```text
bigchaindb -y configure mongodb
```

(There's documentation for the `bigchaindb` command is in the section on [the BigchainDB Command Line Interface (CLI)](../server-reference/bigchaindb-cli.html).)

Edit the created config file by opening `$HOME/.bigchaindb` (the created config file) in your text editor:

* Change `"server": {"bind": "localhost:9984", ... }` to `"server": {"bind": "0.0.0.0:9984", ... }`. This makes it so traffic can come from any IP address to port 9984 (the HTTP Client-Server API port).
* Change `"keyring": []` to `"keyring": ["public_key_of_other_node_A", "public_key_of_other_node_B", "..."]` i.e. a list of the public keys of all the other nodes in the cluster. The keyring should _not_ include your node's public key.
* Ensure that `database.host` and `database.port` are set to the hostname and port of your MongoDB instance. (The port is usually 27017, unless you changed it.)

For more information about the BigchainDB config file, see the page about the [BigchainDB configuration settings](../server-reference/configuration.html).


## Maybe Update the MongoDB Replica Set

**If this isn't the first node in the BigchainDB cluster**, then you must add your MongoDB instance to the MongoDB replica set. You can do so using:
```text
bigchaindb add-replicas your-mongod-hostname:27017
```

where you must replace `your-mongod-hostname` with the actual hostname of your MongoDB instance, and you may have to replace `27017` with the actual port.


## Start BigchainDB

```text
bigchaindb start
```