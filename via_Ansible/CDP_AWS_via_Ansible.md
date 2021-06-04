this is a work in progress

# Creating a CDP environment in AWS via Ansible


##  Build Docker Image

This will execute out of a docker container so you will not need to install ansible to your local machine.

1.  create working directory
  ```
  mkdir ~/cdp_ansible
  cd ~/cdp_ansible
  ```

2. Clone the repo
  ```
  git clone https://github.com/cloudera-labs/cloudera-deploy.git
  cd cloudera-deploy
  ```

3.  Build the docker image
  ```
  chmod +x quickstart.sh
  export provider=aws
  ./quickstart.sh ~/cdp_ansible
  ```

  This will build the docker image, spin up a container from that image, and drop you into the shell for that container.
  
---

## Configure the Credentials within the Container

From the prior step you should be inside the container as the root user...

1. `aws configure`
2. `cdp configure`
3. `vi ~/.config/cloudera-deploy/profiles/default`
  * Set your `admin password`, although I'm unsure where this is used.  It is _not_ used in connecting to the CDP hosts; that is your CDP workload password.
  * Set your `namespace`, and keep it to 5 characters or less
    * NOTE:  Azure is expecting `name_prefix`rather than `namespace` 
  * Set your `tags` if desired
  * Set `infra_type: aws`
  * Set your `infra_region` to your desired AWS region
  * All the SSH & license settings can be commented out

---

## Run the Ansible Playbook

`ansible-playbook /runner/project/cloudera-deploy/main.yml -e "definition_path=examples/sandbox" -t plat`

### Estimated time to spin up the environment:  minutes

---

## Test your environment



---

# To Destroy the environment

`ansible-playbook /runner/project/cloudera-deploy/main.yml -e "definition_path=examples/sandbox" -t teardown`
