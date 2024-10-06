# Cloud Compute

## Overview
For longer simulations of CFD or FEA we can use the VT ARC cluster.
This gives us access to a large number of cores, powerful GPUs, and a large amount of memory.

## Access
You will need to follow the instructions on the [ARC Getting Started](https://www.docs.arc.vt.edu/get_started.html).

You will most likely be using the TinkerCliffs cluster.

To access the cluster you will need to use a terminal and ssh into the cluster.

```shell
ssh <username>@tinkercliffs1.arc.vt.edu # No <> brackets | Use tinkercliffs2.arc.vt.edu if 1 is full/down
```

Your password will be your PID password. And will be followed by a Duo push notification.

## Running Jobs
Follow the guide of [Ansys on ARC](https://www.docs.arc.vt.edu/software/ansys.html).

## How to Use Ansys (CFD/FEA) (In Progress/Not Ready)
Do not run Ansys directly on the login node (aka the terminal you ssh'd into earlier). This will slow down the login node and will get you banned from the cluster.
You will need to use the `?` command to submit a job to the cluster.

```shell
? jobfile.sh
```


