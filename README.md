# hivemq-ansible-installation
Unofficial Installation of HiveMQ Cluster with Ansible on Linux Ubuntu

## Install Ansible on your machine
Refer to : https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

```
pip install ansible

or 

pip install ansible-core
```

Install additional module
ansible-galaxy collection install community.postgresql

## Configure passwordless authentication
If you don't already have a SSH Key, you can generate a new one with (Assuming you are on Linux or MacOS)
```
ssh-keygen
```

Then deploy the key on remote hosts by using this command for each target system (Assuming you are on Linux or MacOS) :
```
ssh-copy-id <username>@<IP_address_of_target-host>
```

## Create an inventory
Copy the inventory template inventory.ini.template and rename it to inventory.ini
Once done update the different blocks to have the list of machines for the cluster and if needed the server for the postgresql database.
A line should looks like :
```
hive001 ansible_host=192.168.9.138 ansible_connection=ssh ansible_user=admin ansible_become_pass=admin
```
The propery ansible_become_pass is used when sudo is needed.

## Test connectivity 
To test the connectivity to a host group (ex: cluster) use the following command 
```
ansible cluster -m ping -i inventory.ini
```

## Deploy PostgreSQL
Run the following command to install PostgreSQL
```
ansible-playbook -i inventory.ini postgresql-install.yml
```

Once deployed, you can create a dedicated username and database for ESE.
Connect on the remote host (ssh server_user@server_ip) and execute the following commands
```
sudo su - postgres
createuser --interactive --pwprompt
createdb -O hivemq-ese hivemq-db
psql hivemq-db < /tmp/Full-ESE-Deployment.sql
```

## Deploy HiveMQ
Run the following command to install HiveMQ Cluster
```
ansible-playbook -i inventory.ini hivemq-install.yml
```

If you want to deploy HiveMQ with TLS listeners, you can generate a self signed certificate and the associated jks with the following commands. 
```
keytool -genkey -keyalg RSA -alias hivemq -keystore ./configs/keystore.jks -storepass hivemq -validity 920 -keysize 2048
```
The first question asks about your first and last name. This is the common name of your certificate. Please enter the URL under which you will connect with your MQTT clients. For example, broker.yourdomain.com (for production) or localhost (for development).

In case you want to use your own keystore.jks, just place it into the configs folder. The default password for the private key and the jks is "hivemq", if different, you need to update it in the broker-tls.xml.

## Update HiveMQ Configuration on all servers
If you want to redeploy the configuration on all hosts and restart the service, you can update the files in the configs folder and run :
```
ansible-playbook -i inventory.ini ./playbooks/hivemq-update-configuration.yml
```