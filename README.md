
# Generate dynamic Ansible inventory off Terraform .tfstate files for TerraDSE

This dynamic inventory is intended to be used by the [TerraDSE project](https://github.com/thompson42/terradse)

This small module will generate a dynamic inventory (hosts) for TerraDSE from tfstate files.

The inventory is controlled by two AWS tags on each instance:

```
tags.DSEDataCenterName
tags.DSENodeType

```

And the following paths in the inventory_generator.py file:

```python
tfstate_path        = 'test_data/tf_state.json'
tfstate_latest_path = 'test_data/tf_state_latest.json' 
```

With these two tags correctly set on existing Terraform tfstate and a new Terraform tfstate we have enough information to generate a dynamic inventory for ansible, avoiding the need to maintain [hosts] files for ansible, we also have the added advantage of being able to version tfstate files in S3 etc.

Case 1: If only a single Terraform tfstate file exists in the first field;tfstate_path and the second field; tfstate_latest_path is empty,  the dynamic inventory script will build the Ansible inventory describing the current state, this can be used to generate a new cluster, or used to perform work on the existing cluster.

Case 2: If two Terraform tfstate files exist the dynamic inventory script will difference the two states and work out your intentions:

Case 2 A: If no difference there is no work to do, an empty inventory will be supplied to Ansible and it should exit with no work done.

Case 2 B: If differences found the script will work out your intentions which will be either [add_node] or [add_datacenter]

If you attempt to add more than 1x node the script will exit and fail.

If you attempt to add more than 1x datacenter the script will exit and fail.

### Set two EC2 tags on AWS all DSE cluster instances:

```
"tags.DSEDataCenterName": "dse_graph",
"tags.DSENodeType": "dse_graph",

```

### When calling an ansible playbook, explicitly point to the inventory_generator.py file:

```
cmd>ansible-playbook -i /path/to/inventory_generator.py dse_keyspace_replication_configure.yml --private-key=~/.ssh/id_rsa_aws
```

Or, in your ansible.cfg file enter the line:

```
inventory = /path/to/inventory_generator.py
```

And then call a playbook via:

```
cmd>ansible-playbook dse_keyspace_replication_configure.yml --private-key=~/.ssh/id_rsa_aws
```

### To debug inventory_generator.py

To dbug the script or just to see how it works prior to using it in ansible call the script outside of ansible on the command line directly:

```
>python inventory_generator.py
```
You will need to make sure the paths tfstate_path (and optionally tfstate_latest_path)  in the script are correct

Output will be an inventory in Ansible compliant format.

### Initial cluster build example, single Terraform tfstate:

```
instance 1{
    ....
    "tags.DSEDataCenterName": "dse_core",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 2{
    ....
    "tags.DSEDataCenterName": "dse_core",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 3{
    ....
    "tags.DSEDataCenterName": "dse_core",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 4{
    ....
    "tags.DSEDataCenterName": "opsc_dsecore",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 5{
    ....
    "tags.DSEDataCenterName": "opsc_dsecore",
    "tags.DSENodeType": "dse_core",
    ....
}

```

From this single Terraform tfstate the dynamic inventory is the initial spin up of a TerraDSE cluster due to the fact there is no prior state file (i.e. not 2x tfstate files supplied to the script)

### Add a node example [add_node], two Terraform tfstate files:

Original tfstate:

```
instance 1{
    ....
    "tags.DSEDataCenterName": "dse_core",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 2{
    ....
    "tags.DSEDataCenterName": "dse_core",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 3{
    ....
    "tags.DSEDataCenterName": "dse_core",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 4{
    ....
    "tags.DSEDataCenterName": "opsc_dsecore",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 5{
    ....
    "tags.DSEDataCenterName": "opsc_dsecore",
    "tags.DSENodeType": "dse_core",
    ....
}

```

New tfstate:

```
instance 1{
    ....
    "tags.DSEDataCenterName": "dse_core",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 2{
    ....
    "tags.DSEDataCenterName": "dse_core",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 3{
    ....
    "tags.DSEDataCenterName": "dse_core",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 4{
    ....
    "tags.DSEDataCenterName": "opsc_dsecore",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 5{
    ....
    "tags.DSEDataCenterName": "opsc_dsecore",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 6{
    ....
    "tags.DSEDataCenterName": "dse_core",
    "tags.DSENodeType": "dse_core",
    ....
}

```

Note that "instance 6" is the difference between the two Terraform tfstate files, it is an additional node going into an existing datacenter called dse_core

### Add a datacenter example [add_datacenter], two Terraform tfstate files:

Original tfstate:

```
instance 1{
    ....
    "tags.DSEDataCenterName": "dse_core",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 2{
    ....
    "tags.DSEDataCenterName": "dse_core",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 3{
    ....
    "tags.DSEDataCenterName": "dse_core",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 4{
    ....
    "tags.DSEDataCenterName": "opsc_dsecore",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 5{
    ....
    "tags.DSEDataCenterName": "opsc_dsecore",
    "tags.DSENodeType": "dse_core",
    ....
}

```

New tfstate:

```
instance 1{
    ....
    "tags.DSEDataCenterName": "dse_core",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 2{
    ....
    "tags.DSEDataCenterName": "dse_core",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 3{
    ....
    "tags.DSEDataCenterName": "dse_core",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 4{
    ....
    "tags.DSEDataCenterName": "opsc_dsecore",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 5{
    ....
    "tags.DSEDataCenterName": "opsc_dsecore",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 6{
    ....
    "tags.DSEDataCenterName": "dse_core_2",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 7{
    ....
    "tags.DSEDataCenterName": "dse_core_2",
    "tags.DSENodeType": "dse_core",
    ....
}
instance 8{
    ....
    "tags.DSEDataCenterName": "dse_core_2",
    "tags.DSENodeType": "dse_core",
    ....
}

```

Note that the difference is instances 6,7,8 and that they are configured to go into a new datacenter called: dse_core_2 of type dse_core (Cassandra only)
