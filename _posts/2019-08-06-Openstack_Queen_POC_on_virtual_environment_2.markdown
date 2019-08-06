---
layout: post
title:  "Openstack Queens POC on virtual env with VirtualBMC - Part 2"
date:   2019-08-04 20:09:54 +0200
categories: Openstack how-to
---

### Part 2 - Import and introspect baremetal node

Official TripleO install guide [link](https://docs.openstack.org/tripleo-docs/latest/install/index.html){:target="_blank"}

#### Network Isolation diagram

(still in draft)

![](/assets/openstack-poc.png)


#### [Director] Create instackenv.json

It's a JSON file describing your Overcloud baremetal nodes, you need to insert the mac-address of the NICs on the Provisioning network and the VirtualBMC ipmi ports

```
{
    "nodes":[
        {
            "mac":[
                "put_here_controller-0_mac_address"
            ],
            "cpu":"2",
            "memory":"4096",
            "disk":"10",
            "arch":"x86_64",
            "name":"controller-0",
            "pm_type":"pxe_ipmitool",
            "pm_user":"admin",
            "pm_password":"ooo-poc",
            "pm_addr":"192.168.122.1",
            "pm_port":"6230"
        },
        {
            "mac":[
                "put_here_controller-1_mac_address"
            ],
            "cpu":"2",
            "memory":"4096",
            "disk":"100",
            "arch":"x86_64",
            "name":"controller-1",
            "pm_type":"pxe_ipmitool",
            "pm_user":"admin",
            "pm_password":"ooo-poc",
            "pm_addr":"192.168.122.1",
            "pm_port":"6231"
        },
        {
            "mac":[
                "put_here_controller-2_mac_address"
            ],
            "cpu":"2",
            "memory":"4096",
            "disk":"100",
            "arch":"x86_64",
            "name":"controller-2",
            "pm_type":"pxe_ipmitool",
            "pm_user":"admin",
            "pm_password":"ooo-poc",
            "pm_addr":"192.168.122.1",
            "pm_port":"6232"
        },
        {
            "mac":[
                "put_here_compute-0_mac_address"
            ],
            "cpu":"2",
            "memory":"4096",
            "disk":"100",
            "arch":"x86_64",
            "name":"compute-0",
            "pm_type":"pxe_ipmitool",
            "pm_user":"admin",
            "pm_password":"ooo-poc",
            "pm_addr":"192.168.122.1",
            "pm_port":"6233"
        },
        {
            "mac":[
                "put_here_compute-1_mac_address"
            ],
            "cpu":"2",
            "memory":"4096",
            "disk":"100",
            "arch":"x86_64",
            "name":"compute-1",
            "pm_type":"pxe_ipmitool",
            "pm_user":"admin",
            "pm_password":"ooo-poc",
            "pm_addr":"192.168.122.1",
            "pm_port":"6234"
        },
        {
            "mac":[
                "put_here_compute-2_mac_address"
            ],
            "cpu":"2",
            "memory":"4096",
            "disk":"100",
            "arch":"x86_64",
            "name":"compute-2",
            "pm_type":"pxe_ipmitool",
            "pm_user":"admin",
            "pm_password":"ooo-poc",
            "pm_addr":"192.168.122.1",
            "pm_port":"6235"
        }
    ]
}
```

#### [Director] Import introspect and set available state in one shot

```
(undercloud) [stack@director ~]$ openstack overcloud node import --introspect --provide instackenv.json
Started Mistral Workflow tripleo.baremetal.v1.register_or_update. Execution ID: 0916f156-ea41-4bb2-a301-ca8f0175ed39
Waiting for messages on queue 'tripleo' with no timeout.

0 node(s) successfully moved to the "manageable" state.
Successfully registered node UUID 585d214d-6614-4bb3-9c71-9ed4a191054a
Successfully registered node UUID bc364f14-bc5e-47da-9167-46876109f1c6
Successfully registered node UUID 854923d1-a35b-4114-b2f9-9cef8a6b5d90
Successfully registered node UUID 5c6db24c-567e-41f3-bc31-98a85433122b
Successfully registered node UUID d11577d4-259c-4364-8741-b384b0f9f5fa
Successfully registered node UUID 7a4b9cf8-4b34-4cbe-b3ec-a6b6de4a0d84
Waiting for introspection to finish...
Started Mistral Workflow tripleo.baremetal.v1.introspect. Execution ID: d14e1578-30fa-4e74-afc5-b2a043e9f66c
Waiting for messages on queue 'tripleo' with no timeout.
Introspection of node 585d214d-6614-4bb3-9c71-9ed4a191054a completed. Status:SUCCESS. Errors:None
Introspection of node 585d214d-6614-4bb3-9c71-9ed4a191054a completed. Status:SUCCESS. Errors:None
Introspection of node 585d214d-6614-4bb3-9c71-9ed4a191054a completed. Status:SUCCESS. Errors:None
Introspection of node bc364f14-bc5e-47da-9167-46876109f1c6 completed. Status:SUCCESS. Errors:None
Introspection of node bc364f14-bc5e-47da-9167-46876109f1c6 completed. Status:SUCCESS. Errors:None
Introspection of node bc364f14-bc5e-47da-9167-46876109f1c6 completed. Status:SUCCESS. Errors:None
Introspection of node d11577d4-259c-4364-8741-b384b0f9f5fa completed. Status:SUCCESS. Errors:None
Introspection of node d11577d4-259c-4364-8741-b384b0f9f5fa completed. Status:SUCCESS. Errors:None
Introspection of node 854923d1-a35b-4114-b2f9-9cef8a6b5d90 completed. Status:SUCCESS. Errors:None
Introspection of node 7a4b9cf8-4b34-4cbe-b3ec-a6b6de4a0d84 completed. Status:SUCCESS. Errors:None
Introspection of node 854923d1-a35b-4114-b2f9-9cef8a6b5d90 completed. Status:SUCCESS. Errors:None
Introspection of node 7a4b9cf8-4b34-4cbe-b3ec-a6b6de4a0d84 completed. Status:SUCCESS. Errors:None
Introspection of node 854923d1-a35b-4114-b2f9-9cef8a6b5d90 completed. Status:SUCCESS. Errors:None
Introspection of node d11577d4-259c-4364-8741-b384b0f9f5fa completed. Status:SUCCESS. Errors:None
Introspection of node 7a4b9cf8-4b34-4cbe-b3ec-a6b6de4a0d84 completed. Status:SUCCESS. Errors:None
Introspection of node 5c6db24c-567e-41f3-bc31-98a85433122b completed. Status:SUCCESS. Errors:None
Introspection of node 5c6db24c-567e-41f3-bc31-98a85433122b completed. Status:SUCCESS. Errors:None
Introspection of node 5c6db24c-567e-41f3-bc31-98a85433122b completed. Status:SUCCESS. Errors:None
Successfully introspected 6 node(s).
Started Mistral Workflow tripleo.baremetal.v1.provide. Execution ID: abb3c798-d347-45f4-abcb-c18788d42b0e
Waiting for messages on queue 'tripleo' with no timeout.
6 node(s) successfully moved to the "available" state.
```

list imported baremetal node

```
(undercloud) [stack@director ~]$ openstack baremetal node list
+--------------------------------------+--------------+---------------+-------------+--------------------+-------------+
| UUID                                 | Name         | Instance UUID | Power State | Provisioning State | Maintenance |
+--------------------------------------+--------------+---------------+-------------+--------------------+-------------+
| 585d214d-6614-4bb3-9c71-9ed4a191054a | controller-0 | None          | power off   | available          | False       |
| bc364f14-bc5e-47da-9167-46876109f1c6 | controller-1 | None          | power off   | available          | False       |
| 854923d1-a35b-4114-b2f9-9cef8a6b5d90 | controller-2 | None          | power off   | available          | False       |
| 5c6db24c-567e-41f3-bc31-98a85433122b | compute-0    | None          | power off   | available          | False       |
| d11577d4-259c-4364-8741-b384b0f9f5fa | compute-1    | None          | power off   | available          | False       |
| 7a4b9cf8-4b34-4cbe-b3ec-a6b6de4a0d84 | compute-2    | None          | power off   | available          | False       |
+--------------------------------------+--------------+---------------+-------------+--------------------+-------------+
```

Next step: Deploy the Overcloud
