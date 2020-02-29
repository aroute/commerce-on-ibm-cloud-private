# archive-commerce-on-ibm-cloud-private
# Cookbook: Installation and Setup of IBM Cloud Private & WebSphere Commerce 9

*This is a cookbook of multiple recipes which demonstrates the  setup and installation of IBM Cloud Private (ICP version 2.1.0.3) and Commerce 9.*

*[Cloud Private](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0.3/getting_started/whats_new.html) is an IBM's Kubernetes implementation - a  cloud-native platform for container orchestration and more. Kubernetes  is the most popular open-source container orchestration platform designed to automate the deployment, scaling, and operation of containers-based applications.*

WebSphere Commerce is a single, unified e-commerce platform that offers the ability to do business directly with consumers (B2C) or directly with businesses (B2B).

------


### Requirements

The recepie uses three server class virtual machines; one desktop class virtual machine running Ubuntu 16.04.4, and one pre-configured server running CentOS with DB2 installed  and setup. In total, the recepie uses five virtual machines to stand up the cluster.

The cluster employs RFC1918 private Internet IP address scheme of 172.16.x.x for routing. The DNS is resolved via Google  servers.

| Role     | IP Address  |
| -------- | ----------- |
| Master   | 172.16.1.1  |
| Worker 1 | 172.16.1.2  |
| Worker 2 | 172.16.1.3  |
| DB       | 172.16.1.4  |
| Desktop  | 172.16.1.10 |




### 1. Configure Network

   1. Turn ON all the machines at once. Start with master. 

   2. Log in as root.  Verify IP address and look for the adapter.

      ```shell
      root@master:~$ ifconfig
      ens160
      ```

   3. Verify by pinging external website.

      ```shell
      root@master:~$ ping ibm.com
      ```

   4. Verify DNS. Ensure you see 8.8.8.8

      ```shell
      root@master:~$ nslookup ibm.com
      ```

   5. Go to worker1 and worker2. Repeat above mentioned steps to verify the network connectivity.

   6. Change host name of worker1 and worker2

      ```shell
      root@worker1:~$ vi /etc/hostname
      worker1
      ```
      ```shell
      root@worker2:~$ vi /etc/hostname
      worker2
      ```

   7. Go to master, worker1 and worker2 to verify/update the hosts file

      ```shell
      root@master:~$ vi /etc/hosts
      172.16.1.1 master
      172.16.1.2 worker1
      172.16.1.3 worker2
      ```
      ```shell
      root@worker1:~$ vi /etc/hosts
      172.16.1.1 master
      172.16.1.2 worker1
      172.16.1.3 worker2
      ```
      ```shell
      root@worker2:~$ vi /etc/hosts
      172.16.1.1 master
      172.16.1.2 worker1
      172.16.1.3 worker2
      ```

### 2. Verify Network

   1. Verify by pinging.
      ```shell
      root@master:~$ ping master
      ```
      ```shell
      root@master:~$ ping worker1
      ```
      ```shell
      root@master:~$ ping worker2
      ```
      ```shell
      root@master:~$ ping worker3
      ```
      ```shell
      root@master:~$ ping 172.16.1.1
      ```
      ```shell
      root@master:~$ ping 172.16.1.2
      ```
      ```shell
      root@master:~$ ping 172.16.1.3
      ```

### 3. Update Operating System

   1. Update Master
       ```shell
       root@master:~$ apt-get update && apt-get upgrade -y
       ```
   2. Update Worker1
       ```shell
       root@master:~$ apt-get update && apt-get upgrade -y
       ```
   3. Update Worker2
       ```shell
       root@master:~$ apt-get update && apt-get upgrade -y
       ```

### 4. Reboot servers

   1. Reboot Master
       ```shell
       root@master:~$ shutdown -r now
       ```
   2. Reboot Worker1
      ```shell
      root@worker1:~$ shutdown -r now
      ```
   3. Reboot Worker2
      ```shell
      root@worker2:~$ shutdown -r now
      ```

### 5. Server Setup (utilities, tools, docker)

   Repeat the following 3 steps on worker 1 and worker 2

   1. Configuration
      ```shell
      root@master:~$ sysctl -w vm.max_map_count=262144
      ```
      ```shell
      root@master:~$ echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
      ```
   2. Utilities
      ```shell
      root@master:~$ apt-get install -y linux-image-extra-$(uname -r) linux-image-extra-virtual
      ```
      ```shell
      root@master:~$ apt install ntp -y && systemctl restart ntp && ntpq -p
      ```
      ```shell
      root@master:~$ apt install socat -y && apt install python2.7 python-pip -y
      ```
   3. Docker
      ```shell
      root@master:~$ cat << EOF > docker.sh
      #!/bin/bash
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
      apt-key fingerprint 0EBFCD88
      add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
      apt-get update
      apt install docker-ce=17.12.1~ce-0~ubuntu -y
      docker run hello-world
      EOF
      root@master:~$ chmod +x docker.sh
      root@master:~$ ./docker.sh
      root@master:~$ rm docker.sh
      ```

### 6. SSH keys generation

   1. Master
      ```shell
      root@master:~$ cd ~/
      root@master:~$ ssh-keygen -t rsa -P ""
      root@master:~$ cd .ssh
      root@master:~$ cat id_rsa.pub >> authorized_keys
      root@master:~$ ls -l
      root@master:~$ service ssh restart
      root@master:~$ ssh localhost
      root@master:~$ yes
      root@master:~$ exit
      root@master:~$ ssh master
      root@master:~$ yes
      root@master:~$ exit
      root@master:~$ ssh 172.16.1.1
      root@master:~$ exit
      ```
   2. Worker1
      ```shell
      root@worker1:~$ cd ~/
      root@worker1:~$ ssh-keygen -t rsa -P ""
      root@worker1:~$ cd .ssh
      root@worker1:~$ cat id_rsa.pub >> authorized_keys
      root@worker1:~$ ls -l
      root@worker1:~$ service ssh restart
      root@worker1:~$ ssh localhost
      root@worker1:~$ yes
      root@worker1:~$ exit
      root@worker1:~$ ssh master
      root@worker1:~$ yes
      root@worker1:~$ exit
      root@worker1:~$ ssh 172.16.1.1
      root@worker1:~$ exit
      ```
   3. Worker2
      ```shell
      root@worker2:~$ cd ~/
      root@worker2:~$ ssh-keygen -t rsa -P ""
      root@worker2:~$ cd .ssh
      root@worker2:~$ cat id_rsa.pub >> authorized_keys
      root@worker2:~$ ls -l
      root@worker2:~$ service ssh restart
      root@worker2:~$ ssh localhost
      root@worker2:~$ yes
      root@worker2:~$ exit
      root@worker2:~$ ssh master
      root@worker2:~$ yes
      root@worker2:~$ exit
      root@worker2:~$ ssh 172.16.1.1
      root@worker2:~$ exit
      ```

### 7. SSH keys pairing

   1. Master to Worker 1 and Worker 2
      ```shell
      root@master:~$ ssh-copy-id root@172.16.1.2
      root@master:~$ yes
      root@master:~$ passw0rd
      root@master:~$ ssh-copy-id root@172.16.1.3
      root@master:~$ yes
      root@master:~$ passw0rd
      ```
   2. Worker 1 to Master and Worker 2
      ```shell
      root@worker1:~$ ssh-copy-id root@172.16.1.1
      root@worker1:~$ yes
      root@worker1:~$ passw0rd
      root@worker1:~$ ssh-copy-id root@172.16.1.3
      root@worker1:~$ yes
      root@worker1:~$ passw0rd
      ```
   3. Worker 2 to Master and Worker 1
      ```shell
      root@worker2:~$ ssh-copy-id root@172.16.1.1
      root@worker2:~$ yes
      root@worker2:~$ passw0rd
      root@worker2:~$ ssh-copy-id root@172.16.1.2
      root@worker2:~$ yes
      root@worker2:~$ passw0rd
      ```

### 8. Reboot servers

   1. Reboot Master
       ```shell
       root@master:~$ shutdown -r now
       ```
   2. Reboot Worker1
      ```shell
      root@worker1:~$ shutdown -r now
      ```
   3. Reboot Worker2
      ```shell
      root@worker2:~$ shutdown -r now
      ```

### 9. SSH keys verification (password-less login)

  1. Master logging in to Worker 1 and Worker 2

      ```shell
      root@master:~$ ssh root@worker1
      root@master:~$ exit
      root@master:~$ ssh root@worker2
      root@master:~$ exit
      ```
      
  2. Worker 1 logging in to Master and Worker 2

      ```shell
      root@worker1:~$ ssh root@master
      root@worker1:~$ exit
      root@worker1:~$ ssh root@worker2
      root@worker1:~$ exit
      ```
      
  3. Worker 2 logging in to Master and Worker 1

      ```shell
      root@worker2:~$ ssh root@master
      root@worker2:~$ exit
      root@worker2:~$ ssh root@worker1
      root@worker2:~$ exit
      ```

### 10. Setup IBM Cloud Private

   1. Docker pull

      ```shell
      root@master:~$ cd ~/
      root@master:~$ docker pull ibmcom/icp-inception:2.1.0.3
      root@master:~$ mkdir /opt/ibm-cloud-private-ce-2.1.0.3
      root@master:~$ cd /opt/ibm-cloud-private-ce-2.1.0.3
      root@master:~$ docker run -e LICENSE=accept -v "$(pwd)":/data ibmcom/icp-inception:2.1.0.3 cp -r cluster /data
      ```
      
   2. Setup inventory (host) file

      ```shell
      root@master:~$ cat << EOF > /opt/ibm-cloud-private-ce-2.1.0.3/cluster/hosts
      [master]
      172.16.1.1
      [worker]
      172.16.1.2
      172.16.1.3
      [proxy]
      172.16.1.1
      #[management]
      # 127.0.0.1
      #[va]
      #5.5.5.5
      EOF
      ```
      
   3. Setup SSH key file

      ```shell
      root@master:~$ cd /opt/ibm-cloud-private-ce-2.1.0.3/cluster
      root@master:~$ cp ~/.ssh/id_rsa ssh_key
      ```

### 11. Deploy IBM Cloud Private

   1. The following installation script may take >30 minutes.

      ```shell
      root@master:~$ cd /opt/ibm-cloud-private-ce-2.1.0.3/cluster
      root@master:~$ cp docker run --net=host -t -e LICENSE=accept \-v "$(pwd)":/installer/cluster ibmcom/icp-inception:2.1.0.3 install
      ```

### 12. Setup Desktop

   1. Setup Utilities

      ```shell
      demo@desktop:~$ sudo apt install ntp -y
      demo@desktop:~$ systemctl restart ntp
      demo@desktop:~$ ntpq -p
      demo@desktop:~$ sudo apt install socat
      demo@desktop:~$ sudo apt install curl
      ```
      
   2. Setup Docker
   
      ```shell
      demo@desktop:~$ sudo apt-get update
      demo@desktop:~$ curl -fsSL get.docker.com -o get-docker.sh
      demo@desktop:~$ sudo sh get-docker.sh
      demo@desktop:~$ sudo usermod -aG docker $USER
      demo@desktop:~$ sudo docker run hello-world
      ```
      
   3. Log out from the desktop, and log back in

   4. Run docker without sudo

      ```shell
      demo@desktop:~$ docker run hello-world
      ```
      
### 13. CLIs

   1. Install IBM CLI

      ```shell
      demo@desktop:~$ curl -fsSL https://clis.ng.bluemix.net/install/linux | sh
      demo@desktop:~$ passw0rd
      demo@desktop:~$ bx --help
      demo@desktop:~$ ICP_ADMIN_HOST=172.16.1.1
      demo@desktop:~$ curl -kLO https://${ICP_ADMIN_HOST}:8443/api/cli/icp-linux-amd64
      demo@desktop:~$ bx plugin install icp-linux-amd64
      demo@desktop:~$ bx plugin show icp
      demo@desktop:~$ bx pr --help
      ```

   2. Install Kubectl CLI

      ```shell
      demo@desktop:~$ sudo snap install kubectl --classic
      demo@desktop:~$ bx pr login -a https://${ICP_ADMIN_HOST}:8443 --skip-ssl-validation
      demo@desktop:~$ admin
      demo@desktop:~$ admin
      demo@desktop:~$ 1
      demo@desktop:~$ bx pr clusters
      demo@desktop:~$ bx pr cluster-config mycluster
      demo@desktop:~$ kubectl version
      ```
      
   3. Install Helm CLI

      ```shell
      demo@desktop:~$ docker run -e LICENSE=accept --net=host -v /usr/local/bin:/data ibmcom/icp-helm-api:1.0.0 cp /usr/src/app/public/cli/linux-amd64/helm /data
      demo@desktop:~$ cd /usr/local/bin
      demo@desktop:~$ ls -l
      helm
      demo@desktop:~$ cd ~
      demo@desktop:~$ bx pr login -a https://${ICP_ADMIN_HOST}:8443 --skip-ssl-validation
      demo@desktop:~$ admin
      demo@desktop:~$ admin
      demo@desktop:~$ 1
      demo@desktop:~$ bx pr clusters
      demo@desktop:~$ bx pr cluster-config mycluster
      demo@desktop:~$ helm init --client-only
      demo@desktop:~$ helm version --tls
      demo@desktop:~$ helm search -l
      demo@desktop:~$ helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
      demo@desktop:~$ helm install --name=my-wordpress stable/wordpress --tls
      demo@desktop:~$ helm list --tls
      demo@desktop:~$ kubectl get pods
      demo@desktop:~$ helm delete my-wordpress --purge --tls
      ```

   4. Configure logon script

      ```shell
      demo@desktop:~$ vi connecticp.sh
      ```
      ```shell
      #!/bin/bash
      ICP_ADMIN_HOST=172.16.1.1
      ADMIN_USER=admin
      ADMIN_PASSWORD=admin
      ICP_CLUSTER=mycluster
      bx pr login -a https://${ICP_ADMIN_HOST}:8443 --skip-ssl-validation -u ${ADMIN_USER} -p ${ADMIN_PASSWORD} -c id-${ICP_CLUSTER}-account
      bx pr cluster-config ${ICP_CLUSTER}
      ```

   5. Verify

      ```shell
      demo@desktop:~$ chmod +x connecticp.sh
      demo@desktop:~$ ./connecticp.sh
      demo@desktop:~$ kubectl get pods --all-namespaces
      ```

### 14. Download v95 docker packages

   1. Locate and save v95 docker tar files to master.

### 15. Docker pull and load

   1. Pull vault and consul.

      ```shell
      root@master:~$ docker pull consul:1.2.0
      root@master:~$ docker pull vault:0.10.4
      ```
      
   2. Load v95 packages

      ```shell
      for n in supportcontainer-1.9.6.tar WC_Ent_9005_utility_MPEN.tgz WC_EntPro_9005_ihs_MPEN.tgz WC_EntPro_9005_search_MPEN.tgz WC_EntPro_9005_store_MPEN.tgz WC_EntPro_9005_tx_MPEN.tgz WC_EntPro_9005_xc_MPEN.tgz 
      do
      docker load -i ${n}
      done
      ```
   
   3. Verify
   
      ```shell
      docker images | grep -E 'commerce|support|vault|consul'
      ```
      
### 16. Docker tag

   1. Docker tag vault, consul and support

      ```shell
      docker tag consul:1.2.0 mycluster.icp:8500/default/consul:latest
      docker tag vault:0.10.4 mycluster.icp:8500/default/vault:latest
      docker tag supportcontainer:1.9.6 mycluster.icp:8500/default/supportcontainer:1.9.6
      ```

   2. Docker tag v9

      ```shell
      DNS1=commerce
      DNS2=mycluster.icp:8500/default/
      TAG=9.0.0.5
      for n in search-app crs-app ts-utils xc-app ts-web ts-app
      do
      docker tag ${DNS1}/${n}:${TAG} ${DNS2}${n}:${TAG}
      done
      ```
   
   3. Verify

      ```shell
      root@master:~$ docker images | grep mycluster.icp
      ```
      
   4. Docker login

      ```shell
      root@master:~$ docker login -u=admin -p=admin mycluster.icp:8500
      ```

### 17. Docker push

   1. Docker push vault, consul and support

      ```shell
      root@master:~$ docker push mycluster.icp:8500/default/consul:latest
      root@master:~$ docker push mycluster.icp:8500/default/vault:latest
      root@master:~$ docker push mycluster.icp:8500/default/supportcontainer:1.9.6
      ```
   
   2. Docker push v9 packages.

      ```shell
      DNS2=mycluster.icp:8500/default/
      TAG=9.0.0.5
      for n in search-app crs-app ts-utils xc-app ts-web ts-app
      do
      docker push ${DNS2}${n}:${TAG}
      done
      ```
   
   3. Change name-space (from the Desktop)

      ```shell
      demo@desktop:~$ kubectl get image --all-namespaces -o yaml | sed 's/scope: namespace/scope: global/g' | kubectl replace -f 
      ```

### 18. Download sample helm chart

   1. Access the desktop built in module 2.

   2. Download the Helm charts as a zip file.

   3. Click OK to open the file with the Archive Manager 

   4. Extract the content to your Home\'s folder, or Home\'s Document folder (your choice). You may have to drag the dialog window to see the extract button. 

   5. Click quit to close the Archive Manager.


### 19. Setup Text Editor and open the Helm charts folder

   1. Open GEdit Text Editor from the left pane menu. Click view menu and select Side Panel (F9)

   2. From the Text Editor\'s side panel, click File Browser and open your Helm charts folder > Home > demo > Documents > HelmCharts v9.5

   3. Double-click on HelmCharts v9.5 folder. You are ready to start editing the files (next step) if your Text Editor look like this:


### 20. Editing the Helm Charts: Vault-Consul

   1. Setup script files for Vault-Consul

   2. Edit the file env.profile. Go to Vault / Vault_Consul and double click on env.profile file to open for editing.

      ```reStructuredText
      Replace the kube_minion_ip address with your ICP worker IP address: 172.16.1.2
      Replace dbHostAuth="169.61.47.125" to "172.16.1.4"
      Replace dbHostLive="169.61.47.125" to "172.16.1.4"
      Replace dbUserAuth="wcsdb" to "wcs"
      Replace dbUserLive="wcsdb" to "wcs"
      Replace dbaUserAuth="wcsdb" to "db2inst1"
      Replace dbaUserLive="wcsdb" to "db2inst1"
      ```
   3. Save (Ctrl+S) the file.


### 21. Editing the Helm Charts: deployment.yaml

   1. Go to Vault / Vault_Consul and double click on deployment.yaml file to open it for editing.

   2. Under specs: containers: image: replace the registry host name with your configured registry: mycluster.icp:8500. 

   3. The lines should read mycluster.icp:8500/default/consul:latest and mycluster.icp:8500/default/vault:latest

   4. Save (Ctrl+S) the file.


### 21. Deploying the Helm Charts: Vault-Consul

   1. Open the Terminal and run ./connecticp.sh to log in to the cluster.

   2. Change directory

   3. ```shell
      demo@desktop:~$ cd /home/demo/Documents/HelmCharts_v9.5
      ```
   4. Grant execution privileges to the Vault directory.

   5. ```shell
      demo@desktop:~$ chmod -R +x Vault
      ```
   6. Change directory and deploy

   7. ```shell
      demo@desktop:~$ cd /home/demo/Documents/HelmCharts_v9.5/Vault
      demo@desktop:~$ ./deploy_vault.sh
      ```
      > Note: The deploy_vault.sh script takes few minutes to run and will show lots of information on the terminal. The most important information you need to see is at the end: The Vault Token (see screen shot below - your token will be different).

   8. Open up a new and empty Text file in the Text Editor, and copy/paste the vault token. You will need this token later. Save the file in your Helm folder, as "commands.txt" but keep it open.

### 22. Deploying the Helm Charts: Verify Vault-Consul

   1. Run the following two commands to verify. Replace < > with your vault token. The best way to do this is to copy/paste the commands in the new text file you just created in your text editor, edit/replace, and then copy/paste in terminal.

   2. ```shell
      curl -X GET -H "X-Vault-Token:<my vault token>" http://172.16.1.1:30552/v1/demo/qa/domainName
      ```

   3. ```shell
      curl -X GET -H "X-Vault-Token:<my vault token>" http://172.16.1.1:30552/v1/demo/qa/auth/dbName
      ```


### 23. Editing the Helm Charts: Setup v9

   1. Go back to the text editor and browse to WCV9 folder. Double click to open the values.yaml file for editing.

   2. Replace vault token with your vault token retrieved from the above completed steps (which you copied to a new text file for yourself).

   3. Replace image_repo storage with your registry: mycluster.icp:8500/default/ (note that there should be a "/" at the end).

   4. Save (Ctrl+S) the file.


### 24. Editing the Helm Charts: Preparing the Helm commands for execution

   1. Prepare and execute two Helm commands for auth and live deployments.  Use the same new text file you have earlier created to record the vault token for yourself. Note that these are two commands which will be executed one after the other.

   2. Replace <my vault token> with with your token

   3. ```shell
      helm install --name demoauth \
      --set Common.Vault_token=<my vault token> \
      --set Common.Image_repo=mycluster.icp:8500/default/ \
      --set Tsapp.Merchantkey=eZrWIdOyaDv5FCOTK32Uni288jgIHDv/P9wxhzKmHdiGZ+n8WJ+Ah56uPbfZ9yJWtjQlGczlmr6OgvArFHCgZQ== \
      --set Common.SPIUser_PWD_Base64=c3BpdXNlcjpwYXNzdzByZA== \
      --set Common.SPIUser_PWD_AES=eNdqdvMAUGRUbiuqadvrQfMELjNScudSp5CBWQ8L6aw= \
      --set Common.SPIUser_Name=spiuser \
      --set Common.Environment_Type=auth \
      ./WCSV9 --tls
      ```
      
   4. ```shell
      helm install --name demolive \
      --set Common.Vault_token=<my vault token> \
      --set Common.Image_repo=mycluster.icp:8500/default/ \
      --set Tsapp.Merchantkey=eZrWIdOyaDv5FCOTK32Uni288jgIHDv/P9wxhzKmHdiGZ+n8WJ+Ah56uPbfZ9yJWtjQlGczlmr6OgvArFHCgZQ== \
      --set Common.SPIUser_PWD_Base64=c3BpdXNlcjpwYXNzdzByZA== \
      --set Common.SPIUser_PWD_AES=eNdqdvMAUGRUbiuqadvrQfMELjNScudSp5CBWQ8L6aw= \
      --set Common.SPIUser_Name=spiuser \
      --set Common.Environment_Type=auth \
      ./WCSV9 --tls
      ```

### 25. Deploying v9 Helm charts

   1. After you have prepared the above two commands (replacing the needed elements), go to Terminal and change directory
cd /home/demo/Documents/HelmCharts_v9.5

   2. Copy/paste the first block to the terminal (deploying auth).

   3. Within few seconds, you will see the deployment (see the screen shot below).

   4. Repeat the same. Copy/paste the second block to the terminal (deploying live).

   5. Within few seconds, you will see the deployment.


