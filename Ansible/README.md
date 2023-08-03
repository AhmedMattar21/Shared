# Ansible Demo

**What is Ansible ?**  

Ansible is a radically simple IT automation engine that automate cloud provisioning, configuration managemnet, application deployment, infra-service orchestration, and many other IT needs.


## Inventory File  


inventory.txt Contains targets ip or hostname under a tag.  

```sh
[webServers]
192.168.1.1  
212.15.77.66

[DB]
122.11.12.44
```



**Simple Ad Hoc Command**  


```bash
$ ansible webServers -m ping -i inventory.txt
$ ansible DB -m ping -i inventory.txt
$ ansible localhost -m ping
$ ansible all -m ping -i inventroy.txt
``` 



## Disable SSH Key Checking



**To Disable SSH Key Checking:**  


- Using Environmnet Variables

```bash
$ export ANSIBLE_HOST_KEY_CHECKING=False
```


- Using ansible.cfg
```bash
$ cat /etc/ansible/ansible.cfg
```

> uncomment 'host_key_checking = False'



## Ansible Playbook

	
**playbook.yml:** It is a metaphor representing the configuration files of Ansible.  It contains a list of tasks  (plays) in an order they should get executed against a set of hosts or a single host based on the configuration specified.  Playbooks are written in YAML, in an easy human-readable syntax. 
>You can consider ansible ad-hoc commands as shell commands and a playbook as a shell script.  
 
![AD HOC Command vs Playbook.](images/playbook.png)

## Ansible Playbook Example
Let us consider you want to install an Apache Web server against multiple hosts. The following playbook would get it done for you.  

![AD HOC Command vs Playbook.](images/playbook-ex001.png)

**become_user** the user name that we want to switch to like compare it with **sudo su - user**  
tasks set of tasks to execute, All tasks would be defined below this

and then we have two tasks with two modules, the first module is yum and the second module is service

**in the first task with** yum  the state latest represents that the forementioned package **httpd** should be installed if it is not installed (or) if it is already installed it should be upgraded to the latest version available. If you do not want it to be upgraded if present, You can change it to **state: present**

**On the Second task with** service module, we are making sure that the service named **httpd** is started and running using the **state: started**     Ansible would not restart the service if it is already started and running.

A **play** is an ordered set of tasks which should be run against hosts selected from your inventory.

A **playbook** is a text file that contains a list of one or more plays to run in order.

## Ansible Playbook Example 2
Example Ansible Playbook with multiple hosts and multiple plays.
```yaml

---
  # Play1 - WebServer related tasks
  - name: Play Web - Create apache directories and username in web servers
    hosts: webservers
    become: yes
    become_user: root
    tasks:
      - name: create username apacheadm
        user:
          name: apacheadm
          group: users,admin
          shell: /bin/bash
          home: /home/weblogic

      - name: install httpd
        yum:
          name: httpd
          state: installed
        
  # Play2 - Application Server related tasks
  - name: Play app - Create tomcat directories and username in app servers
    hosts: appservers
    become: yes
    become_user: root
    tasks:
      - name: Create a username for tomcat
        user:
          name: tomcatadm
          group: users
          shell: /bin/bash
          home: /home/tomcat

      - name: create a directory for apache tomcat
        file:
          path: /opt/oracle
          owner: tomcatadm
          group: users
          state: present
          mode: 0755
```

## Run a Playbook on localhost

```yaml
---

- name: "Ansible Demo Play." 
  hosts: localhost
  connection: local
  become: yes

```


## Execute an Ansible Playbook
```bash
$ ansible-playbook sampleplaybook.yml -i ansible_hosts
```
If you have mentioned all the host groups in your default inventory file **/etc/ansible/hosts**  then you do not have use **-i argument**.  this is only when you have a customized inventory file like I do.

## How to Dry Run the playbook without making Actual Changes
ou can actually run the playbook with Dry Run feature to see what changes would be made to the server without having to perform the actual changes.

to do that. you just have to add **-C** to your ansible-playbook startup command
```bash
$ ansible-playbook -C sampleplaybook.yml -i ansible_hosts
```

## Perform a Syntax Check on the Playbook
If you quickly want to verify if everything is ok with the playbook. You can perform a Syntax check.

Here is the ansible command line example on how to perform Syntax check on ansible playbook. Refer the video for the practical idea.
```bash
$ ansible-playbook – syntax-check sampleplaybook.yml -i ansible_hosts
```

## How to use Variables in Ansible Playbook

Ansible playbook supports defining the variable in two forms, Either as a separate file with full of variables and values like a properties file. or a Single liner variable declaration like we do in any common programming languages

**vars** to define inline variables within the playbook

**vars_files** to import files with variables

Let's suppose we want to add a few variables for our webserver like the server name and SSL key file and cert file etc.

it can be done with vars like this

```yaml
---
  - name: Playbook
    hosts: webservers
    become: yes
    become_user: root
    vars:
       key_file:  /etc/apache2/ssl/mywebsite.key
       cert_file: /etc/apache2/ssl/mywebsite.cert
       server_name: www.mywebsite.com
    tasks:
      - name: ensure apache is at the latest version
        yum:
          name: httpd
          state: latest
      ### SOME MORE TASKS WOULD COME HERE ###
      # you can refer the variable you have defined earlier like this #
      # "{{key_file}}"  (or) "{{cert_file}}" (or) "{{server_name}}" #
      ##
      - name: ensure apache is running
        service:
          name: httpd
          state: started
```
and the variables can be referred with the **jinja2** template later **"{{ variable name }}"**

If you want to keep the variables in a separate file and import it with **vars_files**

You have to first save the variables and values in the same format you have written in the playbook and the file can later be imported using **vars_files** like this

```yaml
---
  - name: Playbook
    hosts: webservers
    become: yes
    become_user: root
    vars_files:
        - apacheconf.yml
    tasks:
      - name: ensure apache is at the latest version
        yum:
          name: httpd
          state: latest
      ### SOME MORE TASKS WOULD COME HERE ###
      # you can refer the variable you have defined earlier like this 	 #
      # "{{key_file}}"  (or) "{{cert_file}}" (or) "{{server_name}}" #
      ##
      - name: ensure apache is running
        service:
          name: httpd
          state: started
```

the content of the **apacheconf.yml** would like like this

```yaml
key_file: /etc/apache2/ssl/mywebsite.key  
cert_file: /etc/apache2/ssl/mywebsite.cert  
server_name: www.mywebsite.com
```

## Group_var:
The group_vars in Ansible are a convenient way to apply variables to multiple hosts at once. Group_vars is an Ansible-specific folder as part of the repository structure. This folder contains YAML files created to have data models, and these data models apply to all the devices listed in the hosts.ini file. We can apply group variables to logical functions and hardware platforms. The variables should include a common identifier as the naming convention, such as platform_ and group_. A few examples of group_vars candidates include Netflow, Domain name, Multicast, Access control lists, Banners, and QoS policies.

### What is Ansible group_vars?
	
It uses hosts file and group_vars directory to set variables for host groups and deploying Ansible plays/tasks against each host/group. **Files under group_var directory are named after the host group’s name or all**, accordingly, the variables will be assigned to that host group or all the hosts.

Although you can store the variables in inventory files and playbooks, having stored and defined variables in separate files and that too on a host group basis, adds simplicity and makes it easy to recognize, then use.

Files under group_vars are generically used to store basic identifying variables and then leverage these with other desirable variables by using include_vars and var_files.

### How Does ANSIBLE group_vars work?

In this, group_vars directory is under Ansible base directory, which is by default /etc/ansible/. The files under group_vars can have extensions including ‘.yaml’, ‘.yml’, ‘.json’ or no file extension.

It loads hosts and group variable files by searching path which is relative to inventory file or playbook file. So which means if you have your hosts file at location /etc/ansible/hosts and there are three groups names grp1, grp2, and grp3 in the hosts’ file. Then below three files will be loaded for a host, which belongs to all three groups, is targeted for playbook execution.

Similarly, other combinations of these files are used, based on the host’s mapping to groups in the host’s file.

    /etc/ansible/group_vars/grp1.yaml
    /etc/ansible/group_vars/grp2.yml
    /etc/ansible/group_vars/grp3

If there is file /etc/ansible/group_vars/all present **then variables in this file will be pulled for every execution.** The definition of variables in these files is simple key: value pair format. An example is like below:
```yaml
---
# file location is
/etc/ansible/group_vars/grp1 fruit: apple
vegetable: potato

```

Also, there are a few points which should be noted when using group_vars:

- We can create directories under the group named directory, it will read all the files under these directories in lexicographical.  

- If there is some value which is true for your environment and for every server, the variable containing that value should be defined under /etc/ansible/group_vars/all file.

- In AWS case, when we have a dynamic inventory where host groups are created and removed automatically, we need to tag the EC2 instances like “class:webserver”. Then variables defined under the /etc/ansible/group_vars/ec2_tag_class_webserver will be located file.

- If in hosts file, groups are organized in such an order that one group is a child of another, then the variables defined for children will have higher precedence over the variable with the same name defined for the parent.

- When the same host is defined for several groups on the same level of the parent-children hierarchy, the variable file precedence with being on the name of groups in alphabetic order. That means if a host is mapped under groups alpha, beta, and gamma. Then for this host variables under /etc/ansible/group_vars/gamma will be pulled.

-  We can use Ansible Vault for these files under group_vars, to protect the confidential data.


### Examples
Now by using examples, we will try to learn about Ansible group_vars, which you might have to use in day to day operations. We will take some examples, but before going there, we will first understand our lab, we will use for testing purpose.  


Here it control server named **ansible-controller** and two remotes hosts named **host-one** and **host-two**. We will create playbooks and run commands on the ansible-controller node and see the results on remote hosts.  


Also, **group_vars** directory is defined as **/etc/ansible/group_vars**. The inventory file is **/etc/ansible/hosts**.  


We created three files named as below: –

- /etc/ansible/group_vars/alpha
- /etc/ansible/group_vars/beta
- /etc/ansible/group_vars/gamma

The hosts file /etc/ansible/hosts have same groups named and the hosts mapping is like below:  
```
[alpha] host-one
[beta] 	host-two
[gamma] host-one host-two 127.0.0.1
```

In this example, we put contents like below under /etc/ansible/group_vars/alpha to define variables and other two files:

- /etc/ansible/group_vars/beta     
- /etc/ansible/group_vars/gamma   

are empty.

```
fruit: apple
vegetable: tomato
```

Now running debug module in ad-hoc command like below:  
```
$ ansible beta -m debug -a "var=fruit"
$ ansible gamma -m debug -a "var=fruit"
```
We get the following output where we can see that only variable defined in the file /etc/ansible/group_vars/alpha for alpha group means the host-one host will be pulled.  


In this example, we have a playbook with below  

```yaml
hosts: host-one tasks:
name: Here we copy the variable value to remote
copy:
content: "fruit variable valus is {{ fruit }}\n" dest: /tmp/sample.ini
```
Also, group_vars directory is defined as /etc/ansible/group_vars and the files with content under this directory are listed as below:

- cat /etc/ansible/group_vars/alpha
- cat /etc/ansible/group_vars/beta
- cat /etc/ansible/group_vars/gamma

```
$ ansible-playbook ansible_group_vars.yaml
```

On checking, we will see that the value of the variable is pulled from.

```
$ ssh host-one "cat /tmp/sample.ini"
fruit varaible value is banana
```



## Ansible Modules
There are many Modules for Ansible but we will just most used modules for DevOps.

### ping module:
ping module is used for checking up on the remote host, specifically for connectivity to remote host and for checking python on the remote host. It can also be used to check user login and privilages.  

```YAML
---
- name: ping module 
  hosts: all
  become: false
  tasks:
    - name: test connection
      ping:
```

- This do not work on ICMP, but work by default on SSH or any other defined connection method.
- One acceptable parameter is data which accepts values as either pong, crash or some other string.
- Default is pong. This represents the return value on success.
If data is provided with value crash, then this module will return an exception.


### setup module:
setup module helps us in gathering data about the target nodes like hostname, ip, os etc. This module is automatically called by playbook to gather useful variable about remote host. . All the information is presented in a friendly t JSON format, with all values being preceded with “ansible_”.  

```bash
$ ansible host-1 -m setup -u ec2-user
```

You can also filter the result by using the **filter** parameter.
```bash
$ ansible all -m setup -u ec2-user -a 'filter=ansible_os_family'
```


### yum module:
yum module is used to install different packages/software on Redhat based linux flavours.
```yaml
---
- name: Update Package on linux system
  hosts: all
  become: yes
  tasks:
  - name: Uninstall Apache
    yum:
      name: nginx
      state: absent
  - name: Install a list of packages with a list variable
    yum:
      name: "{{ packages }}"
    vars:
      packages:
      - httpd
      - httpd-tools
```

Whether to install **(present or installed, latest)**, or remove **(absent or removed)** a package.

**present and installed** will simply ensure that a desired package is installed.

**latest** will update the specified package if it’s not of the latest available version.

**absent and removed** will remove the specified package.

**Default is None**, however in effect the default action is present unless the autoremove option is enabled for this module, then absent is inferred.

**Choices**:

- "absent"
- "installed"
- "latest"
- "present"
- "removed"


### apt module:
apt package module is used for managing software on debian based linux os.

```yaml
---
- name: Update Package on debian system
  hosts: all
  become: yes
  tasks:
  - name: Run the equivalent of "apt-get update" as a separate step
    apt:
      update_cache: yes
  - name: Install latest version of "openjdk-6-jdk" 
    apt:
      name: openjdk-6-jdk
      state: latest
      install_recommends: no
```

install multiple packages
```yaml
 - name: "install packages to allow apt to use a repository over HTTPS:"
    apt:
      update_cache: yes
      name: 
      - ca-certificates 
      - curl 
      - gnupg
      state: latest
```

> To remove a package with Ansible apt. All you have to do use **state: absent** along with the package name of your choice.


Install .deb file or package using apt

```yaml
---
 - name: Ansible apt module examples
   hosts: web
   become: true
   tasks: 

    - name: Ansible install filebeat deb file from local
      apt:
        deb: /tmp/filebeat-7.6.0-amd64.deb

    - name: Ansible install filebeat deb file from url
      apt: 
        deb: https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.0-amd64.deb

```


Debeg Installed Packages: show 

```
---
 - name: Ansible apt module examples
   hosts: web
   become: true
   tasks: 
    - name: Ansible Update Cache and Upgrade all Packages
      register: updatesys
      apt:
        name: "*"
        state: latest
        update_cache: yes

    - name: Display the last line of the previous task to check the stats
      debug:
        msg:  "{{updatesys.stdout_lines|last}}"

```



### package module:
This modules manages packages on a target without specifying a package manager module (lyum, apt, …). It is convenient to use in an heterogeneous environment of machines without having to create a specific task for each package manager. This module acts as a proxy to the underlying package manager module. While all arguments will be passed to the underlying module,

```yaml
---
- name: Update Packages on target system
  hosts: all
  become: yes
  tasks:
  - name: Install ntpdate
    ansible.builtin.package:
      name: ntpdate
      state: present
   
  - name: Install the latest version of Apache and MariaDB
    ansible.builtin.package:
      name:
        - httpd
        - mariadb-server
      state: latest
```


### pip module:
Pip module is used for managing python packages on the target system.
```yaml
---
- name: Update python packages
  hosts: all
  become: yes
  tasks:
  - name: Install multi python packages with version specifiers
    pip:
      name:
        - django>1.11.0,<1.12.0
        - bottle>0.10,<0.20,!=0.11
  - name: Install specified python requirements
    pip:
      requirements: /my_app/requirements.txt
```


### npm module:
Npm module is used for managing of node.js packages. This module is not part of ansible-core modules. To use it in a playbook, specify: **community.general.npm.**

```yaml
---
- name: npm module
  hosts: all
  tasks:
  - name: Install "coffee-script" node.js package.
    community.general.npm:
      name: coffee-script
      path: /app/location
  - name: Install packages based on package.json.
    community.general.npm:
      path: /app/location
```

### raw module:
Raw module is used to executes a low-down and dirty SSH command, It is s useful and should only be done in a few cases. A common case is installing python on a system which doesnot have python. Another is speaking to any devices such as routers that do not have Python installed on it.  

In any other case, using **shell** or **command** module is much more appropriate, Arguments given to raw are run directly through the configured remote shell.  


```yaml
---
- name: raw module usage
  hosts: all
  become: yes
  tasks:
  - name: Bootstrap a host without python2 installed
    raw: dnf install -y python2 python2-dnf libselinux-python
  
  - name: Run a command that uses non-posix shell-isms (in this example /bin/sh doesn't handle redirection and wildcards together but bash does)
    raw: cat < /tmp/*txt
    args:
      executable: /bin/bash
```

### command module:
Command module is used to execute commands on a remote node. It is used mostly to run simple Linux commands.  
The command(s) will not be processed through the shell, so variables like **$HOSTNAME** and operations like **“*”, “<”, “>”, “|”, “;” and “&”** will not work.  
For Windows targets, use the **win_command** module instead. For windows based target **win_command** can be used in place of command module.

```yaml
---
- name: command module
  hosts: all
  tasks:
  - name: Run command if /path/to/database does not exist (without 'args')
    command: /usr/bin/make_database.sh db_user db_name creates=/path/to/database
  - name: Run command if /path/to/database does not exist (with 'args' keyword)
    ansible.builtin.command: /usr/bin/make_database.sh db_user db_name
    args:
      creates: /path/to/database
```

### shell module:
Shell module is designed to execute shell commands against the target unix based hosts. Unlike the Ansible **command** module, Ansible **Shell** would accept highly complexed commands with pipes, redirection etc, you can also execute shell scripts using this module.  
This module is designed to work only with Linux based Machines and not Windows, For windows you should use **win_powershell** module.

```yaml
---
- name: shell module
  hosts: all
  tasks:
  - name: This command will change the working directory to somedir/ and will only run when somedir/somelog.txt doesn't exist
    shell: somescript.sh >> somelog.txt
    args:
      chdir: somedir/
      creates: somelog.txt
  
  - name: This command will change the working directory to somedir/
    shell:
      cmd: ls -l | grep log
      chdir: somedir/
  
  - name: Run a command that uses non-posix shell-isms (in this example /bin/sh doesn't handle redirection and wildcards together but bash does)
    shell: cat < /tmp/*txt
    args:
      executable: /bin/bash
```

### script module:
Script module can be used to run a local script on a remote node. When we invoke the **shell** module and passes a local shell script to it, what it does at the **backend** is that first it transfers the script to the remote node and than executes it.  
  
Two important things to keep in mind about this module is that **it does not require python on the remote node** and the script is executed through shell.  
 **script module is also supported for windows based nodes.**
```yaml
---
- name: script module
  hosts: all
  tasks:
  -  name: Run a script with arguments (free form)
     ansible.builtin.script: /some/local/script.sh --some-argument 1234
   
  -  name: Run a script with arguments (using 'cmd' parameter)
     ansible.builtin.script:
       cmd: /some/local/script.sh --some-argument 1234
```


### copy module:
copy module can be used to copy a file from local/remote machine to remote machine. For windows machine you can use **win_copy** module.

```yaml
---
- name: copy module
  hosts: localhost
  tasks:
  - name: copy a file from local machine to local machine
    copy:
      src: files/src.txt
      dest: files/dest.txt
---
- name: copy module
  hosts: all
  tasks:
  - name: copy a file from remote machine to remote machine
    copy:
      src: /etc/src.txt
      dest: /etc/dest.txt

  - name: copy a file from local machine to remote machine with owner and permissions
    copy:
      src: files/src.txt
      dest: /etc/dest.txt
      owner: foo
      group: foo
      mode: '0644'
```

 

### fetch module:
fetch module can be used whenever we want to fetch a file from remote machine to a local machine. Files are stored in local machine in a directory with the name of the hostname.

```yaml
---
- name: fetch module
  hosts: all
  tasks:
  - name: copy a file from remote machine to local machine
    fetch:
      src: /var/log/access.log
      dest: /var/log/fetched

  - name: copy a file from local machine to remote machine with owner and permissions
    copy:
      src: files/src.txt
      dest: /etc/dest.txt
      owner: foo
      group: foo
      mode: '0644'
```

### get_url module:

**get_url** module can be used to download files form **HTTPS/HTTP/FTP** servers. By default this module uses the proxy configured for the node.  


Custom proxy can be used by setting environment variable or by using **use_proxy** option.


```yaml
---
- name: get_url module
  hosts: all
  tasks:
  - name: download tomcat from apache 
    get_url:
      url: https://downloads.apache.org/tomcat/tomcat-8/v8.5.81/bin/apache-tomcat-8.5.81-deployer.tar.gz
      dest: /tmp/download/tomcat
      mode: 0755
      owner: tomcat
      group: tomcat
```



### archive module:

archive module is used to create a compressed file package of the format of zip, tar, gz, bz2 and xz. By default it assumes that the source file you are trying to compress does exists and It does not copy source file to target node before compressing. One thing to keep in mind is that It requires the package type that we are using as compressed file output should already be installed on the target node.


```yaml
---
- name: archive module
  hosts: all
  tasks:
  - name: Compress directory /path/to/foo/ into /path/to/foo.tgz
    archive:
      path: /path/to/foo
      dest: /path/to/foo.tgz
  - name: Compress regular file /path/to/foo into /path/to/foo.gz and remove it
    archive:
      path: /path/to/foo
      remove: yes
  - name: Create a bz2 archive of multiple files, rooted at /path
    archive:
      path:
      - /path/to/foo
      - /path/wong/foo
      dest: /path/file.tar.bz2
      format: bz2
  - name: Create a gz archive of a globbed path, while excluding specific dirnames
    archive:
      path:
      - /path/to/foo/*
      dest: /path/file.tar.bz2
      exclude_path:
      - /path/to/foo/bar
      - /path/to/foo/baz
      format: gz
```


### unarchive module:
unarchive module is used to unpacks an archive file such as tar, gz, zip. It can also copy the file to the remote server before uncompressing them. The module use unzip and tar -xzf command to unpack the compressed file so these commands should be installed on target nodes. For windows node **win_unzip** can be used.

```yaml
---
- name: unarchive module
  hosts: all
  tasks:
  - name: Extract foo.tgz into /var/lib/foo
    unarchive:
      src: foo.tgz
      dest: /var/lib/foo

  - name: Unarchive a file that is already on the remote machine
    unarchive:
      src: /tmp/foo.zip
      dest: /usr/local/bin
      remote_src: yes

  - name: Unarchive a file that needs to be downloaded 
    unarchive:
      src: https://example.com/example.zip
      dest: /usr/local/bin
      remote_src: yes
```



### file module:
file module is responsible for performing tasks such as creating files and directories, deleting files and directories, creating soft and hard symbolic links, adding and modifying file and directory permissions, and more. For windows machine you could use **win_file** module.

```yaml
---
- name: file module
  hosts: all
  tasks:
  - name: Create a file
    file:
      path: /etc/foo.conf
      state: touch
      mode: u=rw,g=r,o=r
  - name: Create a directory if it does not exist
    file:
      path: /etc/some_directory
      state: directory
      mode: '0755'
  - name: Remove file (delete file)
    file:
      path: /etc/foo.txt
      state: absent
  - name: Change file ownership, group and permissions
    file:
      path: /etc/foo.conf
      owner: foo
      group: foo
      mode: '0644'
  - name: Create a symbolic link
    file:
      src: /file/to/link/to
      dest: /path/to/symlink
      owner: foo
      group: foo
      state: link
```


### acl module:
acl module is used to create and modify access control list entries, similar to the getfacl and setfacl commands. This module requires that acls are enabled on the target filesystem and that the setfacl and getfacl binaries are installed. For windows machine you can use **win_acl** module

```yaml
---
- name: acl module
  hosts: all
  tasks:
  - name: Grant user Joe read access to a file
    acl:
      path: /etc/foo.conf
      entity: joe
      etype: user
      permissions: r
      state: present

  - name: Removes the acl for Joe on a specific file
    acl:
      path: /etc/foo.conf
      entity: joe
      etype: user
      state: absent

  - name: Sets default acl for joe on foo.d
    acl:
      path: /etc/foo.d
      entity: joe
      etype: user
      permissions: rw
      default: yes
      state: present
```

### template module:
A template in Ansible is a file which contains all your configuration parameters, but the dynamic values are given as variables. During the playbook execution, depending on the conditions like which cluster you are using, the variables will be replaced with the relevant values with the help of Jinj2 templating engine. The template files will usually have the .j2 extension
template module does two things, first replace the Jinja2 interpolation syntax variables present in the template file with actual values, second copy (scp) the file to the remote server.


```yaml
---
- name: template module
  hosts: all
  tasks:
  - name: Template a file to /etc/file.conf
    template:
      src: /mytemplates/foo.j2
      dest: /etc/file.conf
      owner: bin
      group: wheel
      mode: '0644'	
```

### find module:
find module functions as same as the Linux Find command and helps to find files and directories based on various search criteria. For windows, you should use a **win_find** module instead.


```yaml
---
- name: find module
  hosts: all
  tasks:
  - name: Recursively find /tmp files older than 2 days
    find:
      paths: /tmp
      age: 2d
      recurse: yes

  - name: Recursively find /tmp files older than 4 weeks and equal or greater than 1 megabyte
    find:
      paths: /tmp
      age: 4w
      size: 1m
      recurse: yes

  - name: Recursively find /var/tmp files with last access time greater than 3600 seconds
    find:
      paths: /var/tmp
      age: 3600
      age_stamp: atime
      recurse: yes
```


### replace module:
replace module is used to replace all the instances of a matching string in a file. It also supports regular expression and can also create backup of a file before replacing.

```yaml
---
- name: replace module
  hosts: all
  tasks:
  - name: Ansible replace Unix with Linux
    replace:
      path: /etc/ansible/sample.txt
      regexp: 'Unix'
      replace: 'Linux'
  - name: Replace before the expression till the begin of the file
    replace:
      path: /etc/apache2/sites-available/default.conf
      before: '# live site config'
      regexp: 'Unix'
      replace: 'Linux'
  - name: Replace between the expressions and create a backup
    replace:
      path: /etc/hosts
      after: '<VirtualHost [*]>'
      before: '</VirtualHost>'
      regexp: 'Unix'
      replace: 'Linux'
      backup: yes
```


### lineinfile module:
lineinfile module is helpful when you want to add, remove, modify a single line in a file. You can also use conditions to match the line before modifying or removing using the regular expressions. You can reuse and modify the matched line using the back reference parameter. you can also use insertafter and insertbefore attribute to make changes at specified portion of the file

```yaml
---
- name: lineinfile module
  hosts: all
  tasks:
  - name: adding a line
    lineinfile:
      path: /etc/selinux/config
      regexp: '^SELINUX='
      line: SELINUX=enforcing

  - name: deleting a line
    lineinfile:
      path: /etc/sudoers
      state: absent
      regexp: '^%wheel'

  - name: Replacing a line
    lineinfile:
      path: /etc/hosts
      regexp: '^127\.0\.0\.1'
      line: 127.0.0.1 localhost
  - name: replace a line only after a specified string 
    lineinfile:
      path: /etc/httpd/conf/httpd.conf
      regexp: '^Listen '
      insertafter: '^#Listen '
      line: Listen 8080
```




### blockinfile module:
blockinfile module will insert/update/remove a block of multi-line text. The block will be surrounded by a marker, like begin and end, to make the task idempotent. By default, the block will be inserted at the end of the file

```yaml
---
- name: blockinfile module
  hosts: all
  tasks:
  - name: insert a block into a file
    blockinfile:
      path: /etc/ssh/sshd_config
      block: |
        Match User ansible-agent
        PasswordAuthentication no
  - name: Insert/Update HTML surrounded by custom markers after <body> line
    blockinfile:
      path: /var/www/html/index.html
      marker: "<!-- {mark} ANSIBLE MANAGED BLOCK -->"
      insertafter: "<body>"
      block: |
        <h1>Welcome to {{ ansible_hostname }}</h1>
        <p>Last updated on {{ ansible_date_time.iso8601 }}</p>
```


### service module:
service module is used to control service on target nodes, we can start/stop/restart/reload a service through this module. For windows based target machine you can use **win_service** module.

```yaml
---
- name: service module
  hosts: all
  tasks:
  - name: Start service httpd, if not started
    service:
      name: httpd
      state: started

  - name: Stop service httpd, if started
    service:
      name: httpd
      state: stopped

  - name: Restart service httpd, in all cases
    service:
      name: httpd
      state: restarted

  - name: Reload service httpd, in all cases
    service:
      name: httpd
      state: reloaded
```


### user module:
user module is used to manage users on the target nodes. At the back user module uses useradd to create, usermod to modify and userdel to remove users. You can define uid, group, shell, password etc. For windows based target machine you can use **win_user** module.

```yaml
---
- name: user module
  hosts: all
  tasks:
  - name: Add the user 'johnd' with a specific uid and group of 'admin'
    user:
      name: johnd
      comment: John Doe
      uid: 1040
      group: admin

  - name: Add the user 'james' with a bash shell, appending the group 'admins' and 'developers' to the user's groups
    user:
      name: james
      shell: /bin/bash
      groups: admins,developers
      append: yes

  - name: Remove the user 'johnd'
    user:
      name: johnd
      state: absent
      remove: yes

  - name: Create a 2048-bit SSH key for user jsmith in ~jsmith/.ssh/id_rsa
    user:
      name: jsmith
      generate_ssh_key: yes
      ssh_key_bits: 2048
      ssh_key_file: .ssh/id_rsa

  - name: Added a consultant whose account you want to expire
    user:
     name: james18
     shell: /bin/zsh
     groups: developers
     expires: 1422403387
```


### group module:
group module is used to manage groups on the target nodes. At the back this module uses groupadd to create, groupmod to modify and groupdel to remove groups. For windows based target machine you can use **win_group** module.

```yaml
---
- name: group module
  hosts: all
  tasks:
  - name: Ensure group "somegroup" exists
    group:
      name: somegroup
      state: present

  - name: Ensure group "docker" exists with correct gid
    group:
      name: docker
      state: present
      gid: 1750
  - name: deleting a group
    group:
      name: groupA
      state: absent
```

### cron module:
cron module is used to manage crontab entries and define environment variables.

```yaml
---
- name: cron module
  hosts: all
  tasks:
  - name: Ensure a job that runs at 2 and 5 exists. Creates an entry like "0 5,2 * * sh script.sh"
    cron:
      name: "check dirs"
      minute: "0"
      hour: "5,2"
      job: "sh script.sh"
  - name: 'Delete a job from the crontab'
    cron:
      name: "an old job"
      state: absent
  - name: Creates an entry like "APP_HOME=/srv/app" and insert it after PATH declaration
    cron:
      name: APP_HOME
      env: yes
      job: /srv/app
      insertafter: PATH
```



### debug module:
debug module is used to print statement during playbook execution and can be useful in debugging variables or expression. You can use msg attribute to display a custom message and var attribute for displaying a variable.

```yaml
---
- name: debug module
  hosts: all
  tasks:
  - name: Print a simple statement
      debug:
      msg: "Hello World! A custom message"
  - name: Get uptime information
    shell: /usr/bin/uptime
    register: result

  - name: Print return information from the previous task
    debug:
      var: result
      verbosity: 2
```


### include_vars module:
include_vars module can be used to dynamically load variable from a file or directory during a task run. One thing to keep in mind is that any varible set using set_fact will not be overwritten with inclide_vars.

```yaml
---
- name: include_vars module
  hosts: all
  tasks:
  - name: include a variable file
    include_vars: 
      file: name_vars.yml
  - name: include a variable file conditionally
    include_vars: 
      file: vars-Debian.yml
    when: ansible_os_family == 'Debian'
  - name: Include all .yaml files in vars directory except bastion.yaml
    include_vars:
      dir: vars
      ignore_files:
        - 'bastion.yaml'
      extensions:
        - 'yaml'
```



### include_role module:
this module dynamically loads and execute a specified role as a task.

```yaml
---
- name: include_roles module
  hosts: all
  tasks:
  - name: include role myrole
    include_role:
      name: myrole
  - name: Run tasks/other.yaml instead of main.yaml
    include_role:
      name: myrole
      tasks_from: other
  - name: Pass variables to role
    include_role:
      name: myrole
    vars:
      rolevar1: value from task
  - name: Conditional role
    include_role:
      name: myrole
    when: not idontwanttorun
```


### idrac_os_deployment:
idrac_os_deployment module – Boot to a network ISO image .
 
>Note. This module is part of the dellemc.openmanage collection (version 7.5.0).


```yaml
--- 
- name: Booting to Network Operating System image
  idrac_os_deployment:
    idrac_ip: "{{ idrac_ip }}"
    idrac_password: "{{ idrac_password }}"
    idrac_user: "{{ idrac_user }}"
    share_name: "{{ idrac_share | default('10.3.6.38') }}:/var/lib/jenkins/isos" # SHARED_DIR
    iso_image: "Debian-{{ hostname }}.iso"
    expose_duration: 180
  connection: local
 
```


### idrac_server_config_profile:  
idrac_server_config_profile module – Export or Import iDRAC Server Configuration Profile.  

```yaml
---
- name: Export Server Configuration Profile to a network share
  idrac_server_config_profile:
    idrac_ip: "192.168.0.120"
    idrac_user: "{{ idrac_user }}"
    idrac_password: "{{ idrac_password }}"
    share_name: "/var/lib/jenkins/isos"
    share_user: "share_user_name"
    share_password: "share_user_password"
    scp_components: ALL
    job_wait: True
  connection: local
```

```yaml
---
- name: Generating the gateway from the ip address
  shell: echo "{{ idrac_new_ip}}" | awk -F'.' '{print $1"."$2"."$3}'
  register: network
  connection: local

- set_fact:
    idrac_gw: "{{ network.stdout }}.10"

- name: generating idrac templates
  template:
    src: idrac.yml.j2
    dest: /var/lib/jenkins/isos/idrac-{{ hostname }}.xml
    owner: jenkins
    group: jenkins
    mode: 0644
  connection: local
  
- name: Import Server Configuration Profile from a network share
  idrac_server_config_profile:
    idrac_ip: "{{idrac_ip}}"
    idrac_user: "{{ idrac_user }}"
    idrac_password: "{{ idrac_password }}"
    command: "import"
    share_name: "/var/lib/jenkins/isos"
    share_user: "share_user_name"
    share_password: "share_user_password"
    scp_file: "idrac-{{ hostname }}.xml"
    scp_components: "ALL"
    job_wait: True
  connection: local
```

### dellemc_idrac_storage_volume:
dellemc_idrac_storage_volume module – Configures the RAID configuration attributes

```yaml
---
- name: Create multiple volume
  dellemc_idrac_storage_volume:
    idrac_ip: "{{ idrac_ip }}"
    idrac_user: "{{ idrac_user }}"
    idrac_password: "{{ idrac_password }}"
    raid_reset_config: True
    state: "create"
    controller_id: "RAID.Integrated.1-1"
    volume_type: "RAID 5"
    span_depth: 1
    span_length: 3
#    number_dedicated_hot_spare: 1
    disk_cache_policy: "Default"
    write_cache_policy: "WriteBackForce"
    read_cache_policy: "ReadAhead"
    stripe_size: 128
    raid_init_operation: "Fast"
    volumes:
      - name: "3x4TBSSD"
        drives:
          id: ["Disk.Bay.0:Enclosure.Internal.0-1:RAID.Integrated.1-1",
               "Disk.Bay.1:Enclosure.Internal.0-1:RAID.Integrated.1-1",
               "Disk.Bay.2:Enclosure.Internal.0-1:RAID.Integrated.1-1"]
  connection: local
  register: result_setup_raid

```


### idrac_reset:

```yaml
---
- name: Reset iDRAC
  idrac_reset:
    idrac_ip:   "{{ idrac_ip }}"
    idrac_user: "{{ idrac_user }}"
    idrac_password:  "{{ idrac_password }}"
  connection: local


```

## Ansible Roles
Roles let you automatically load related vars, files, tasks, handlers, and other Ansible artifacts based on a known file structure. After you group your content in roles, you can easily reuse them and share them with other users.

### Role directory structure:
An Ansible role has a defined directory structure with eight main standard directories. You must include at least one of these directories in each role.

```
roles/
    common/               # this hierarchy represents a "role"
        tasks/            #
            main.yml      #  <-- tasks file can include smaller files if warranted
        handlers/         #
            main.yml      #  <-- handlers file
        templates/        #  <-- files for use with the template resource
            ntp.conf.j2   #  <------- templates end in .j2
        files/            #
            bar.txt       #  <-- files for use with the copy resource
            foo.sh        #  <-- script files for use with the script resource
        vars/             #
            main.yml      #  <-- variables associated with this role
        defaults/         #
            main.yml      #  <-- default lower priority variables for this role
        meta/             #
            main.yml      #  <-- role dependencies
        library/          # roles can also include custom modules
        module_utils/     # roles can also include custom module_utils
        lookup_plugins/   # or other types of plugins, like lookup in this case
```

>By default Ansible will look in each directory within a role for a main.yml file for relevant content (also main.yaml and main):


- **tasks/main.yml** - the main list of tasks that the role executes.

-  **handlers/main.yml** - handlers, which may be used within or outside this role.

- **library/my_module.py** - modules, which may be used within this role (see Embedding modules and plugins in roles for more information).

- **defaults/main.yml** - default variables for the role (see Using Variables for more information). These variables have the lowest priority of any variables available, and can be easily overridden by any other variable, including inventory variables.

- **vars/main.yml** - other variables for the role (see Using Variables for more information).

- **files/main.yml** - files that the role deploys.

- **templates/main.yml** - templates that the role deploys.

- **meta/main.yml** - metadata for the role, including role dependencies and optional Galaxy metadata such as platforms supported.


### Storing and finding roles:

By default, Ansible looks for roles in the following locations:



- in collections, if you are using them

- in a directory called roles/, relative to the playbook file

- in the configured roles_path. The default search path is ~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles.

- in the directory where the playbook file is located


### Using Roles:

You can use roles in three ways:

- at the play level with the roles option: This is the classic way of using roles in a play.

- at the tasks level with **include_role**: You can reuse roles dynamically anywhere in the tasks section of a play using include_role.

- at the tasks level with **import_role**: You can reuse roles statically anywhere in the tasks section of a play using **import_role**.

#### Using roles at the play level

```yaml
---
- hosts: webservers
  roles:
    - common
    - webservers
```

When you use the **roles** option at the play level, Ansible treats the roles as static imports and processes them during playbook parsing. Ansible executes each play in this order:



- Any **pre_tasks** defined in the play.

- Any handlers triggered by **pre_tasks**.

- Each role listed in **roles:** , in the order listed. Any role dependencies defined in the role’s **meta/main.yml** run first, subject to tag filtering and conditionals. See Using role dependencies for more details.

- Any **tasks** defined in the play.

- Any **handlers** triggered by the roles or tasks.

- Any **post_tasks** defined in the play.

- Any handlers triggered by **post_tasks**.


#### Including roles: dynamic reuse:
You can reuse roles dynamically anywhere in the tasks section of a play using **include_role**. While roles added in a roles section run before any other tasks in a play, included roles run in the order they are defined. If there are other tasks before an **include_role** task, the other tasks will run first.

```yaml

---
- hosts: webservers
  tasks:
    - name: Print a message
      ansible.builtin.debug:
        msg: "this task runs before the example role"

    - name: Include the example role
      include_role:
        name: example

    - name: Print a message
      ansible.builtin.debug:
        msg: "this task runs after the example role"
```

You can pass other keywords, including variables and tags, when including roles:

```yaml
---
- hosts: webservers
  tasks:
    - name: Include the foo_app_instance role
      include_role:
        name: foo_app_instance
      vars:
        dir: '/opt/a'
        app_port: 5000
      tags: typeA
  ...
```

You can conditionally include a role:

```yaml
---
- hosts: webservers
  tasks:
    - name: Include the some_role role
      include_role:
        name: some_role
      when: "ansible_facts['os_family'] == 'RedHat'"
```

#### Importing roles: static reuse:
You can reuse roles statically anywhere in the **tasks** section of a play using **import_role**. The behavior is the same as using the roles keyword. For example:

```yaml
---
- hosts: webservers
  tasks:
    - name: Print a message
      ansible.builtin.debug:
        msg: "before we run our role"

    - name: Import the example role
      import_role:
        name: example

    - name: Print a message
      ansible.builtin.debug:
        msg: "after we ran our role"
```

## Important Notes
Here will list every tip or trick for feature use.

### SSH Login without password
1- on your local machine.
```bash
$ ssh-keygen -t rsa
$ cd ~/.ssh
$ ssh-copy-id -i my_id.pub root@tf2 
```

> where root is the remote user and tf2 is the remote host that u want to login without password.

### Using Passwords with Ansible
There are two methods:
- First Method: Using CLI (not secure)

```bash
ansible-playbook -i inventory my.yml \
--extra-vars 'ansible_become_pass=YOUR-PASSWORD-HERE'
```
or
```bash
ansible-playbook --ask-become-pass -i inventory my.yml
```
or
```bash
ansible-playbook -i inventory my.yml -K
```


- Second Method: Using Ansible vault  

Inventory File

```yaml
[cluster:vars]
k_ver="linux-image-4.13.0-26-generic"
ansible_user=vivek  # ssh login user
ansible_become=yes  # use sudo 
ansible_become_method=sudo 
ansible_become_pass='{{ my_cluser_sudo_pass }}'
 
[cluster]
www1
www2
www3
db1
db2
cache1
cache2
```



create a new encrypted data file named password.yml, run the following command:

```bash
$ ansible-vault create passwd.yml
```
Set the password for vault. After providing a password, the tool will start whatever editor you have defined with $EDITOR. Append the following  
```bash
my_cluser_sudo_pass: your_sudo_password_for_remote_servers
```

Save and close the file in vi/vim. Finally run playbook as follows:
```bash
$ ansible-playbook -i inventory --ask-vault-pass --extra-vars '@passwd.yml' my.yml
```



How to edit my encrypted file again

```bash
$ ansible-vault edit passwd.yml
```

How to change password for my encrypted file

```bash
$ ansible-vault rekey passwd.yml
```

### Connect EC2 Instance

```bash
ansible-playbook ansible/playbook.yml -i ansible/inventory.txt --key-file ~/Downloads/aws.pem --user ubuntu
```


### Checking if a File Exists in Ansible
The easiest way to check if a file exists using Ansible is with the stat module.

The purpose of the stat module is to retrieve facts about files and folders and record them in a register. The stat module uses the following syntax:

```yaml
---
- name: Playbook name
  hosts: all
  tasks:
  - name: Task name
    stat:
      path: [path to the file or directory you want to check]
    register: register_name
...
```

One of the values recorded in the register is exists. Combining this value with the debug module lets you display a message detailing whether a file or folder exists:

```yaml
- name: Task name
    debug:
      msg: "The file or directory exists"
    when: register_name.stat.exists
```
![example](images/ansible-check-if-a-file-exists-02-1.png)

### Share SSH keys between the hosts using ansible








