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
   
   Only we need to do is to build ssh connections with **managed machines** and control the process from Control Node.
   
   Let's set up our Control Node with Linux Machine **(VM)**. To do this, we need to add the private ip of Linux Machine to the host file.
   Then create a user on the same machine because we will take our action on user level. 
   Next, update the sudoer file with running **"visudo"** command and give password authentication to the user on root level not user level.
   
   -  user1    ALL=(ALL)     NOPASSWD=ALL

   Next, generate public and private keys by executing **"ssh-keygen"** command
   
   We update the **sshd_config** file under correct directory (/etc/ssh/sshd_config) to **PasswordAuthentication  YES**
   
   Start the sshd service with following command:
   -  service sshd start
   
   Now, we configured our **Control Machine** successfully.
   
   Secondly, create your one of **managed nodes** as **RedHat machine**. Therefore, launch instance for it on EC2 service. 
   
   Do the above steps for this managed machine as well.
   
      Note: We always have to create host file on user level not root level
      
**Then**, its time to add private ip of managed machine to host file of control machine, so those machines can communicate to each other.

   For that we can run:  
   -  ssh-copy-id {managed-machine-private-ip}

   We also can verify connection between control machine and managed machines...
   -  ansible all -i hosts -m ping -w


  If we get **SUCCESS** message, it means we can ping remote machines. From now, you can do all configuration against the ansible machine.
  
  Let's go to operations folder and create our first **playbook** under **"/opt/"** directory
  
  Be carefull, **owner** of the ansible file should be the userID you added for that machine:
  
  If it's not you may need to change the ownerby running:
  
   -  sudo chown userID:GroupID ansible/  

**_First Playbook File:_**
     
     create_user.yml 
      ---
      - hosts: all
        become: true
        tasks:
        - name: create user
          user:
            name: Shark
    
   -  Above, we used user module from Ansible.
   -  All hosts corresponds all VMs available
   -  become: true defines we are running this file as a root user.
   
   Now, lets set up our third machine **(second managed node)**.
   
   This will be an Amazon Linux machine as well. Then do the same above step for first managed machine and copy of private ip of third machine to host file of          Control Node.
   
   Check the connection by running below command:
   -  ansible all -i home/userid/hosts -m ping -w
   
           Note: You may want to change the hostname of your machine by simply runnnig:
                 vim etc/hostname (using vim editor)    
                 
                 
  Now, lets run our playbook file and see what happens:
    
   -  ansible-playbook -i /home/userID/hosts create_user.yml

 **TASK - create_user**
 
 OK         (managed machine-1)
 
 changed    (managed machine-2)

   We are able to create a user named **Shark** with one of the machines. 

   Verify the user creation by checking this directory:
   **cat /home/passwd**

   **shark .... home/shark**
   
   For other machine, there is no change so there will **Item Potency** for that.
   This is another part of the reason why Ansible is so powerfull.
   Ansible is **ITEM POTENT**.
   
   Lets create another playbook file called **install-packages.yml** file and add all below tasks:
   -  ---
      - name: Install packages
        hosts: vmware
        become: true
        tasks:
        - name: Install git
          yum:
            name: git
            state: present
        - name: Install tree
          yum:
            name: tree
            state: present
        - name: Create a file
          file:
            path: /home/userID/loserfile
            state: touch

        - name: Create a directory
          file:
            path: /home/userID/loserpath
            state: directory
            owner: ec2-user

        - name: Copy file to remote host
          copy:
            src: /opt/ansible/index.html
            dest: /home/userID
            mode: 0600
            owner: userID
            group: groupID 
   
   In above example, our hosts are VMs that are in the same group called **"vmware"**.
   
        As you can see, you can have multiple taks with different modules in one ansible playbook.
   
   
   **TO DEPLOY CHANGES:**
     
   -  ansible-playbook -i hosts install-packages.yml

   **TO VERIFY INSTALATION:**

You may just go to second machine and run **"git"** command. 

If you see flags, it means git is **installed.**

    Note: If you would like to remove git from the machine, you just need to update the state of the install package task 
          to absent. Then git will be removed from your machine.
          
  In order to install apache and start the apache service, you may need to run below playbook:
  
      --- 
      - name: Install apache packages 
        hosts: all
        become: true 
        gather_facts: true
        tasks:

        - name: install apache 
          yum:
            name: httpd 
            state: installed
          when: ansible_os_family == "RedHat"
        - name: Start apache service 
          service:
            name: httpd 
            state: started 
          when: ansible_os_family == "RedHat"

        - name: Install apache ubuntu
          apt: 
            name: apache2
            state: present
          when: ansible_os_family == "Debian" 

        - name: Start apache ubuntu
          service:
            name: apache2
            state: started 
          when: ansible_os_family == "Debian"
   
   
  As you can see from the above playbook file, we leveraged yum, service, apt modules from ansible. Addittonally, we added **gather_facts**
  attribute in order to gather facts whenever we run this file. 
  
   -  **yum** - amazon linux install package manager
   -  **apt** - Ubuntu install package manager
   -  **service** -  ansible module for apache
   -  **when** keyword allow us to match with Operating System (OS). If there is correct OS is running then do the task. Otherwise, just skip the task. 
  
   
   
   
   
   
   
   
   
   
   
   
   
