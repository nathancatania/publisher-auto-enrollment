# Automated Netskope Publisher Enrollment
**Version: 0.1.0**

## What is this?
This is a configuration file that will automatically run the enrollment command on the Netskope Publisher when deployed using the image from the AWS or Azure marketplaces.

## Why is this useful?
This allows you to deploy a Netskope Publisher into AWS/Azure/GCP without requiring inbound (SSH) access to access to configure the Publisher itself, and means that in a closed off IaaS environment (as long as you have outbound access on TCP 443 towards the Netskope cloud), by simply deploying a Publisher from Marketplace, you can have instant access to internal resources with no CLI-config needed.

# How do I use this?
1. [Access the configuration file here](https://github.com/nathancatania/publisher-auto-enrollment/blob/main/user-data.yml) and copy the text. Note where it says to paste in the Publisher enrollment token.
2. Access your Netskope Admin Console and create a new Publisher (Settings > Security Cloud Platform > Publishers). Generate an enrollment token and paste it into the appropriate field in the configuration file. Make sure you paste the token *between* the double quotes!
4. During the VM creation workflow, paste in the configuration text (including your Publisher enrollment token) into a field marked as cloud-init/custom-data/user-data. This is different for every platform (see specific instructions below).
5. Wait a few minutes. After some time, you should see the Publisher be marked as `Connected` in the Netskope Admin Console.

## AWS / EC2
* For AWS EC2, when creating a VM instance, scroll down and under **Advanced Details**, simply paste the configuration into the **User-Data**.

![aws](https://i.imgur.com/vc7POtl.png)

## Azure
* For Azure, when creating a VM instance, select the **Advanced** tab, and paste the configuration into the **Custom data** field under the **Custom data and cloud init** sub-heading.
![azure](https://i.imgur.com/2GetPUq.png)

---

# Who are you?
I'm a Netskope Solutions Engineer that created this to make everyone's lives easier! Any of my own thoughts posted here are my own and do not reflect those of Netskope.

# Validation / Troubleshooting Steps
If you're not sure whether this has worked properly, do the following:

## Connect to the VM
SSH to the VM using the `ubuntu` user, eg: `ssh -i <ssh-key> ubuntu@<vm-ipaddress>`

* Note: You'll only be able to SSH to the VM if you also specified an SSH key or identity in the configuration.
  
## Check the status of cloud-init
Check the status of the cloud-init service: `cloud-init status`

```
ubuntu@i-097b1b0c65d4ef774:~$ cloud-init status
status: done
```

If the status shows `status: running`, cloud-init is still configuring the VM. Either wait a few more minutes or proceed below.

## Check the cloud-init logs

  * `/var/log/cloud-init-output.log` is the live output as the configuration is being applied. Very useful to observe this file if your cloud-init status is still `status: running`. You can also run `sudo tail -f /var/log/cloud-init-output.log` if cloud-init is still running to see the live output as this log is written to.
  * `/var/log/cloud-init.log` is the log of the overarching cloud-init process itself.
 
```
ubuntu@i-097b1b0c65d4ef774:~$ sudo cat /var/log/cloud-init-output.log
[...snip...]
Registering with your Netskope address: ns-[REDACTED].us-sjc1.npa.goskope.com
Publisher certificate CN: [REDACTED]
Attempt 1 to register publisher.
Publisher registered successfully.

Verifying connectivity to the Netskope Dataplane...
Connectivity to the Netskope Dataplane was successfully verified.
Cloud-init v. 21.4-0ubuntu1~20.04.1 running 'modules:final' at Tue, 26 Apr 2022 01:57:49 +0000. Up 18.20 seconds.
Cloud-init v. 21.4-0ubuntu1~20.04.1 finished at Tue, 26 Apr 2022 02:01:43 +0000. Datasource DataSourceEc2Local.  Up 252.19 seconds
```

## Check the applied configuration
`/var/lib/cloud/instance/user-data.txt` is a copy of the configuration template you pasted in during VM creation. You can see what this looks like in a rendered/finished state (ie: the *exact* configuration that cloud-init applies) through the following command: `sudo cloud-init devel render /var/lib/cloud/instance/user-data.txt`

Unrendered template example:
```
ubuntu@i-097b1b0c65d4ef774:~$ sudo cat /var/lib/cloud/instance/user-data.txt
## template: jinja
#cloud-config

# {# REQUIRED - Provide the Publisher enrolment key/token. Ensure to paste it within the double quotes! #}
{% set token = "1234567890" %}

# Enroll the publisher using the provided token
runcmd:
 - "/home/ubuntu/npa_publisher_wizard -token {{ token }}"
```

Rendered template example:
```
ubuntu@i-097b1b0c65d4ef774:~$ sudo cat /var/lib/cloud/instance/user-data.txt
#cloud-config

#
# Enroll the publisher using the provided token
runcmd:
 - "/home/ubuntu/npa_publisher_wizard -token 1234567890"
```
