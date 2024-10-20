# Cloud Compute

## Overview

For longer simulations of CFD or FEA we can use the VT ARC cluster.
This gives us access to a large number of cores, powerful GPUs, and a large amount of memory.

## Access

You will need to follow the instructions on the [ARC Getting Started](https://www.docs.arc.vt.edu/get_started.html).

You will most likely be using the TinkerCliffs cluster.

To access the cluster you will need to use a terminal and ssh into the cluster.

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

## If on Mac follow the following steps

1. ```brew install --cask xquartz```

Once ssh'ed into ARC run the following command (make sure to pass the -X flag when ssh'ing into ARC):
2. ```touch ~/.Xauthority```

Test it out with:
3. ```module load ANSYS/22.1```
4. ```fluent --help```

You should see the help menu for Fluent.

![Fluent Help Menu](./images/Fluent%20Help%20Menu%20X11.jpg)

If that doesn't work install XQuartz from [here](https://www.xquartz.org/).
