# How to connect to a CDP host

Once you spin up your data lake environment, and any other data hub clusters or experiences, you will probably need to ssh into one of the hosts.  


## AWS

### Connect as a normal user

```
ssh yourCDPuser@hostname
```
It will prompt you for your password, which is the workload password you set up within CDP.  Note there is no need for keypairs or pem files to ssh into CDP hosts.

### Connect as cloudbreak

If you need to sudo for a task or otherwise be root for some reason, it requires the keypair to authenticate.   

If you created your environment from the CDP console, you will need to use the keypair you selected when you created the environment.  That pem file will need 400 permissions.
```
ssh -i /path/to/keypair.pem cloudbreak@hostname
sudo -i
```

If you created your environment using the Ansible playbook (described in this very repository), unless you configured it to use a specific keypair in the SSH section, the playbook will create a keypair for you.   The default location is in your home directory under `./ssh` with these two files, where namespacePrefix is the namespace you configured when you ran the playbook:
* `namespacePrefix_ssh_rsa.pub`
* `namespacePrefix_ssh_rsa`

```
ssh -i ~/.ssh/namespacePrefix_ssh_rsa cloudbreak@hostname
sudo -i
```
