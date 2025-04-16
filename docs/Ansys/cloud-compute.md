# Cloud Compute

## Overview

For longer simulations of CFD or FEA we can use the VT ARC cluster.
This gives us access to a large number of cores, powerful GPUs, and a large amount of memory.

## Access

You will need to follow the instructions on the [ARC Getting Started](https://www.docs.arc.vt.edu/get_started.html).

You will most likely be using the TinkerCliffs cluster.

To access the cluster you will need to use a terminal and ssh into the cluster.

Before SSH'ing you need to either be on the VT network or use the VPN.

- [Windows VPN Instructions](https://vt4help.service-now.com/sp?id=kb_article&sysparm_article=KB0010740)
- [Mac VPN Instructions](https://vt4help.service-now.com/sp?id=kb_article&sysparm_article=KB0012672)

```shell
ssh <username/PID>@tinkercliffs1.arc.vt.edu # No <> brackets | Use tinkercliffs2.arc.vt.edu if 1 is full/down
```

With X11 forwarding:

```shell
ssh -X <username/PID>@tinkercliffs1.arc.vt.edu
```

Your password will be your PID password. And will be followed by a Duo 2FA push notification.

## Running Jobs

Follow the guide of [Ansys on ARC](https://www.docs.arc.vt.edu/software/ansys.html).

## How to Use Ansys (CFD/FEA) (In Progress/Not Ready)

Do not run Ansys directly on the login node (aka the terminal you ssh'd into earlier). This will slow down the login node and will get you banned from the cluster.
You will need to use the `?` command to submit a job to the cluster.

```shell
? jobfile.sh
```

## Modules Available (10/20/2024)

```shell
module av
```

Small snippet of the output:

```shell
------------------------------------------------------------------------------- /apps/easybuild/modules/tinkercliffs-rome/all --------------------------------------------------------------------------------
   ANSYS/19.5                                                       
   ANSYS/20.1                                                        
   ANSYS/20.2                                                        
   ANSYS/21.1                                                        
   ANSYS/21.2                                                        
   ANSYS/22.1                                              (D)       
   Anaconda3/2020.07                                              
   Anaconda3/2020.11                                           
   Anaconda3/2022.10                                          
   Anaconda3/2024.02-1                                     (D)
   tinkercliffs-rome/ANSYS/23.2
```

## Show Loaded Modules

```shell
module list
```

## Load a Module

```shell
module load tinkercliffs-rome/ANSYS/23.2
```

## Unload a Module

```shell
module unload tinkercliffs-rome/ANSYS/23.2
```

## If on Mac follow the following steps (this includes the X11 forwarding)

1. ```brew install --cask xquartz```
2. ``` ssh -X <username/PID>@tinkercliffs1.arc.vt.edu ```
3. ```touch ~/.Xauthority```
4. ```module load ANSYS/22.1```
5. ```fluent --help```

You should see the help menu for Fluent.

![Fluent Help Menu](./images/Fluent%20Help%20Menu%20X11.jpg)

If that doesn't work install XQuartz from [here](https://www.xquartz.org/).

## If on Windows, if you want X11 forwarding, follow the following steps

1. Setup WSL2 with Ubuntu 24.04 LTS (or any other distro).
2. Login the same way as you would on Mac/Linux.
3. ``` ssh -X <username/PID>@tinkercliffs1.arc.vt.edu ```

## Get added to Globus

Globus shares a users directory with others through the Globus website.

Head to the [Globus website](https://www.globus.org) and login by selecting Virginia Tech as your organization.

You can get added to Globus by contacting the person on the team who is the POC for ARC related things.

## Relevant Links

- <https://github.com/AdvancedResearchComputing/examples/tree/master/ansys>
- <https://www.docs.arc.vt.edu/resources/storage.html>
- <https://www.docs.arc.vt.edu/software/globus.html#globus>
- <https://docs.globus.org/globus-connect-personal/install/linux/>
- <https://app.globus.org/activity>
