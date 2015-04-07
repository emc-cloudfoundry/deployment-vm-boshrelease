# Deployment VM

The deployment vm is a workstation for who want to use bosh_cli and cf_cli to manage the deployed bosh and cloudfoundry.

  1. The deployment VM should support multi-user work.
  2. The users can login the deployment VM via SSH.
  3. The bosh_cli and cf_cli should be setup automatically for all users.
  4. The users should be specified in deploying manifest, so that once the deployment VM is deployed, it’s ready to work as well.
  5. Create a file in one of the users home directory and if you bosh recreate the deployment VM, then the file created in the home directory should still be there.
  6. If the user changes their password after logging in, then when bosh recreates is done to the deployment VM, then the password should be reset to what is in the bosh manifest.

### Out-of-Box Applications
* bosh_cli 1.2749.0
* bosh_cli_plugin_micro 1.2749.0
* cf_cli 6.6.1
* openjdk 1.7.0_55-b13

# Usage
##### Create & Upload BOSH Release
To use this BOSH release, first create the dev release, then the final release, then upload it to your BOSH director:

```
bosh target BOSH_HOST
git clone https://github.com/emc-cloudfoundry/deployment-vm-boshrelease.git
cd emc-cloudfoundry/deployment-vm-boshrelease
bosh create release --force
bosh create release --force --final
bosh upload release
```
##### Create a BOSH deployment manifest
Please refer to the file example-manifest.yml for an example of the deployment manifest.
##### Deploy using the BOSH deployment manifest

Using the previous created deployment manifest, now we can deploy it:

```
bosh deployment path/to/deployment.yml
bosh -n deploy
```
##### How to Add User Account
You can add user accounts in the properties section of the deploying manifest like this:
```YAML
properties:
  deployment_vm:
    accounts:
    - chris
    - maggie:password
```
Look at this sample, there are two accounts will be created, the user name are “chris” and “maggie”. Because there is no password is specified for “chris”, the default password is assigned, the value is the same as user name. So the password of “chris” is “chris”, and for “maggie” is “password”.
##### How to Make User Files be Persistent
1. Add persistent_disk property to your deployment manifest, for below example, there is a 20G disk for persistence.

```YAML
jobs:
- name: deployment-vm
  templates:
  - name: deployment-vm
  instances: 1
  resource_pool: medium
  persistent_disk: 20000
  networks:
  - name: default
    static_ips:
    - 10.62.51.83
```
2. Once you finished the deploying of deployment-vm, there will be a directory named "persis-store" in each user's home directory.
3. You can put the files what you want to persist while upgrading to your "persis-store" directory.
4. If you're using harden stemcell, then your "persis-store" will not be access by other users.
