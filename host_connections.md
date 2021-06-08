# How to connect to a CDP host

Once you spin up your data lake environment, and any other data hub clusters or experiences, you will probably need to ssh into one of the hosts.  


## AWS

### Connect as a normal user

```
ssh yourCDPuser@hostname
```
It will prompt you for your password, which is the workload password you set up within CDP.  Note there is no need for keypairs or pem files to ssh into CDP hosts.

### Connect as cloudbreak

If you need to sudo for a task or otherwise be root for some reason, it requires a keypair or to authenticate.   

If you created your environment from the CDP console, you will need to use the keypair (aka pem file) you selected when you created the environment.  That pem file will need 400 permissions.  If you don't have that pem file, you're screwed.  No, really.  Just tear it down because you ain't getting in.
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

---

## Azure

Azure doesn't use pem files like AWS does, so you'll have a keypair you created when you created your CDP environment (or you used a keypair you had previously created).   In either case you should have these two files on your local machine, hopefully you know where they are:

* `namespacePrefix_ssh_rsa.pub`
* `namespacePrefix_ssh_rsa`

