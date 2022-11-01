# Order-Fairness

This repository includes three remote repositories. 
* libhotstuff_tx: 
The main code base is taken from Hotstuff repository [libhotstuff](https://github.com/hot-stuff/libhotstuff.git) and 
added support for small-bank transactions.
* Themis_tx: 
We have implemented Themis based on the algorithms given in the paper [Themis](https://www.cs.cornell.edu/~mahimna/themis.pdf). 
Along with that we have also added support for small-bank transactions.
* Rashnu: 
A high-performance fair ordering protocol designed and implemented by us 
with a small-bank transaction support.

The directory structure of all the three projects is the same.

# Rashnu Motivation

Rashnu is a practical high-performance fair ordering
protocol. Rashnu is motivated by the fact that fair ordering among
two transactions is needed when both transactions access a shared
resource.


# Rashnu Installation

### install from the repo
git clone https://github.com/HeenaNagda/Order-Fairness.git
cd Order-Fairness/
git submodule update --init --recursive
cd Rashnu/

### ensure openssl and libevent are installed on your machine, more specifically, you need:
* CMake >= 3.9 (cmake)
* C++14 (g++)
* libuv >= 1.10.0 (libuv1-dev)
* openssl >= 1.1.0 (libssl-dev)
* on Ubuntu: sudo apt-get install libssl-dev libuv1-dev cmake make

### build project
* cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED=ON -DHOTSTUFF_PROTO_LOG=ON
* make

### Run Rashnu locally
* Make sure in the scripts/run_demo.sh file number of replicas are 0 to 3.
* Make sure the hotstuff.conf file has exactly 4 replica signatures.
* start 4 demo replicas with scripts/run_demo.sh
* start the demo client with scripts/run_demo_client.sh in another terminal
* Use Ctrl-C to terminate the client and replicas

# Reproducing the Results

## Step 1 - Environment and Dependencies

### Local Environment

Note: Following steps need to be done on your work computer (a
  work computer is your laptop/home computer).

* Install the latest [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html).
* If .ssh is not present on the host machine, run following: 
  ``mkdir ~/.ssh; chmod 700 ~/.ssh``
* set PATH variable in ~/.bashrc if not set already 
  ``export PATH="$HOME/usr/bin:$PATH"``
  ``export PATH="$HOME/usr/lib:$PATH"``
  ``source ~/.bashrc``
* Install numpy using following command:
  ``pip3 install numpy``
* Cloned the latest ``Order-Fairness`` repo and
  updated all submodules (if not sure, run ``git submodule update --init
  --recursive``). Finally, you have already built the repo so binaries
  ``hotstuff-keygen`` and ``hotstuff-tls-keygen`` are available in the root
  directory of the repo.
* Right now, you should be at the ``<path-to-your-Order-Fairness-repo>/Rashnu/scripts/deploy`` directory in your shell.

### Remote Environment

Node: In this example, we use a typical Linux image, Ubuntu 18.04, on Cloudlab.
But any machine with Ubuntu 18.04 installed may work, in general.

* Update the remote machine
  ``sudo apt-get update -y; sudo apt-get upgrade;``
* Install libuv1-dev by running following:
  ``sudo apt-get install -y libuv1-dev``
* All machines should be accessible from your work computer given an ssh private key. Generate ssh rsa key pair
  ``cd ~/.ssh/; ssh-keygen; cd``
* properly configured the intra-network for the
  machines that participate in our experiment
  + Replica machines should be able to talk to each other via TCP port ranging
  from 10000 (default value generated by ``gen_conf.py``, which could
  be changed). 
  + Each client machine should be able to talk to all replica machines via TCP
  ranging from 20000.
  + To properly configure the intra-network, turn on the firewall, and open TCP ports 10000, 20000, 22.
  ``sudo ufw enable; sudo ufw allow 10000/tcp; sudo ufw allow 20000/tcp; sudo ufw allow 22/tcp; sudo ufw status verbose;``

## Step 2 - Generate the Deployment Setup

* Edit both ``replicas.txt`` and ``client.txt``:

  + ``replicas.txt``: each line is the external IP and local IP separated by
    one or more spaces. The external IP will be used for control actions
    between your work computer and replica machines, whereas the local IP is
    the address used in your inter-replica network infrastructure, with which
    replicas establish TCP connections with others.
  + ``clients.txt``: each line is a single external IP.
  + The same IP can appear multiple times in both files. In this case, you will
    share the same machine among different processes (not recommended for
    replicas due to performance reasons).
* Update the configuration flags according to the requirement in ``./gen_all.sh``.
* Generate ``node.ini`` and ``hotstuff.gen.*.conf`` by running ``./gen_all.sh``.
* Change the ssh key configuration, user name and paths carefully based on your remote machines home directory paths in ``group_vars/all.yml``.
* Build ``Rashnu`` on all remote machine by ``./run.sh setup``.

## Step 3 - Run the Experiment

* (optional) Change the parameters in ``hotstuff.gen.conf`` to your liking.
* (optional) Change the parameters in ``group_vars/clients.yml`` to your liking.
* (for replicas) Create a new experiment run and start all replica processes by ``./run.sh new myrun1``.
 (wait for a while until all replica processes settle down, for good network like EC2, 10 seconds should be more than enough)
* (for replicas) Create a new experiment run and start all client processes by ``./run_cli.sh new myrun1_cli``.
* (wait until all commands are submitted, or you simply would like to end the experiment)
* To collect the results, run ``./run_cli.sh stop myrun1_cli`` followed by ``./run_cli.sh fetch myrun1_cli``.
* To analyze the results, run ``cat myrun1_cli/remote/*/log/stderr | python ../thr_hist.py``.
* Finally, stop replicas: ``./run.sh stop myrun1``.

## Other Notes

* Each ``./run.sh new`` (same for ``./run_cli.sh``) will create a folder that
  contains everything (chosen parameters, raw results) for the run. A good
  practice is to always move on to a new name for a different run, so you keep
  all of your previous experiments nicely.
* The ``run.sh`` script does NOT detect whether there is some other unfinished
  run (it does, however, prevents you from messing up the state of the same run,
  given the id like "myrun1"), so you need to make sure you always ``stop``
  (gracefully exit and all results are available) or ``reset`` (simply kill all
  processes) any historical runs to start fresh.
* To check whether processes are still alive: ``./run.sh check myrun1``.

