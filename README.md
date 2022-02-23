   ****HOW TO INSTALL PACKAGES AND APACHE BY LEVERAGING ANSIBLE****

![alt text](https://github.com/tanersa/ansible/blob/feature/ansible-install/Ansible-8.png)


   **ANSIBLE** is an open source automation engine which can automate:
   
   -  Configuration Management
   -  Provisioning
   -  Application Deployment
   -  Orchestration
   -  Deploy Containers


   It does not need to have **Master Node** in order to pull configuration.
   The way we can configure Ansible is to have an **Control Node** and multiple **Managed Nodes** that will be SSH to.
   
   First of all, we have **inventory file** (host file) to define which ansible managed machines we will work with. Then, we have **playbooks** where we determine our 
   tasks as well as modules we may use. Lastly, we have **Control Node** which will be the center of the configuration process.
   
   Only we need to do is to build ssh connections with manages machines and control the process from Control Node.
   
