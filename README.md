![](/Assets/Title1.png)
<br>
<br>
<h1>OpenSSH</h1>
This playbook requires OpenSSH as the connection method from the Unix localhost to the remote Windows Hosts.
<br>
<br>
<h2>Installing OpenSSH</h2>
<br>
To install the OpenSSH components:
<br>
<br>
Open Settings, select Apps > Apps & Features, then select Optional Features.

Scan the list to see if the OpenSSH is already installed. If not, at the top of the page, select Add a feature, then:

Find OpenSSH Client, then click Install
Once setup completes, return to Apps > Apps & Features and Optional Features and you should see OpenSSH listed.

After installation navigate to task manager > services > find openSSH > right click > start
<br>
<br>
<h2>OpenSSH Connections</h2>
<br>
Depending on your organisation's infrastructure firewall rules you may have to manually create a firewall rule to allow port 22 SSH connections.
<br>
<br>
Otherwise a firewall rule is automatically created and enabled on port 22 when OpenSSH is installed called OpenSSH-Server-In-TCP.
<br>
<br>
<h1>Windows Ansible Credentials</h1>

To successfully SSH and authenticate with the remote windows hosts ansible will need to use a windows account on the remote machine.
<br>
<br>
You have the choice to use an existing account on the remote machines but I would highly recommend creating a new windows user for ansible.

The reason for this is just to keep the accounts universal across all the windows hosts while leaving an audit trail.
<br>
<br>
<h2> To Create A New Windows User:</h2>
<br>
1. Select Start > Settings > Accounts and then select Family & other users. (In some versions of Windows you'll see Other users.)

2. Select Add someone else to this PC.

3. Select I don't have this person's sign-in information, and on the next page, select Add a user without a Microsoft account.

4. Enter a user name, password, or password hint—or choose security questions—and then select Next.

5. Open Settings and create another account

6. Select Start >Settings > Accounts .

7. Under Family & other users, select the account owner name (you should see "Local Account" below the name), then select Change account type.

8. Under Account type, select Administrator and then select OK.

9. Sign in with the new administrator account.
<br>
<br>

<h1> Installing Required Packages</h1>

First and foremost you will need to install ansible preferably on a Unix machine as opposed to a windows machine or windows subsystem for Linux (WSL).

Python is also required to run ansible so we will be installing this also.

<h2>Ubuntu</h2>

We will be using PIP to install ansible and its packages so we will need to first install python to install PIP:

1. sudo apt update

2. sudo apt-get install software-properties-common

3. sudo add-apt-repository ppa:deadsnakes/ppa

4. sudo apt-get update

5. sudo apt-get install python3

6. python --version
<br>
<br>

Now that we have Python installed we can install PIP:

1. sudo apt-get -y install python3-pip

2. pip3 --version

<br>

Now finally using PIP we can install Ansible and its windows modules package:

1. sudo python -m pip install ansible

2. ansible-galaxy collection install ansible.windows

<br>

<h2>Centos</h2>

We will first install Python which is required to use ansible:

1. sudo yum update -y

2. yum install -y python3

3. python --version

<br>

We can install Ansible and its windows modules package:

1. sudo yum -y install epel-repo

2. sudo yum -y install ansible

3. ansible-galaxy collection install ansible.windows
<br>
<br>
<br>

![](/Assets/Title2.png)

The inventory is a remote host and ansible variables configuration file.

![](/Assets/Inventory.png)

<br>
The first 5 lines define the remotes host into a group call Test_VM multiple hosts can exist within this group and multiple groups can exist allowing diverse remote host grouping.
<br>
<br>
Lines 9 to 23 are important ansible variables that do the following:
<br>
<br>
ansible_python_interpreter: /usr/bin/python3 = Ansible requires python to operate so this variable defines the python installation location.
<br>
<br>
ansible_user: ansible = This variable defines the user that ansible will use to ssh to remote hosts and perform commands, in this instance, we are using the ansible user with its own generated ssh key.
<br>
<br>

ansible_ssh_private_key_file: '/home/ansible/.ssh/id_rsa' = This variable defines the location of the private ssh key the ansible user will use to ssh to remote hosts to successfully authenticate.
<br>
<br>

ansible_password: !vault |
$ANSIBLE_VAULT;1.1;AES256
31376234663162326432336330396638326636363136663761343832613736356165376334373934
3734616132343466326632316133633233303864383963370a633863633531663438626637316334
61353135643166616234326164646432613532633536303165623963333933616530363734656564
3865383162363366340a613730633166313136363930666563616462303333353038633164343436
3031
<br>
<br>

This variable stores the password of the remote host ansible account in the format of an encrypted ansible vault password.
<br>
<br>
ansible_become_user: ansible = This variable will make sure when connected to the remote windows host it will become the user ansible in the powershell console.
<br>
<br>
ansible_become_method: runas = Escalations the ansibles user to run tasks as root without becoming root to ensure sufficient permissions.
<br>
<br>
ansible_shell_type: powershell = Defines what shell with performs its tasks and commands in, because this is windows and not Linux we are using powershell instead of bash.
<br>
<br>
ansible_connection: ssh = The method ansible will use to connect to remote hosts, ssh is the default connection method but it also does support winrm connections.
<br>
<br>

![](/Assets/Title3.png)

The 7-Zip playbook was developed to automate the uninstall of any old versions of 7zip and install the lasted version from a locally provided file for increased security.
<br>
<br>

![](/Assets/7zip1.png)

This ansible task will create a folder on the remote windows host called 7zip_Update by executing powershell commands.
<br>
<br>

![](/Assets/7zip2.png)
<br>
<br>
This ansible task will copy the locally stored 7zip.exe file located at Ansible-Windows-Java-7zip-Playbook/roles/7zip/files/7zip.exe to the remote windows host within the created 7zip_Update folder.

![](/Assets/7zip3.png)
<br>
<br>
This ansible task will silently (/s) uninstall any existing versions of 7zip by running the uninstall.exe at C:\Program Files\7-Zip\Uninstall.exe.
<br>
<br>


![](/Assets/7zip4.png)
<br>
<br>
This ansible task will instantly (/S) install the latest 7zip version located locally at C:\Users\ansible\7zip_Update\7zip.exe, a product_id is requited to installed programs using the win_package module in this case 7-Zip sill suffice.
<br>
<br>


![](/Assets/7zip5.png)
<br>
This ansible task will remove the 7-zip installation file and 7zip_Update folder which is important to be able to re-run the playbook as it will fail if the folder already exists.
<br>
<br>

![](/Assets/Title4.png)

The Java playbook was developed to automate the uninstall of any old versions of Java and install the lasted version from a locally provided offline installer file for increased security.

![](/Assets/java1.png)

This ansible task will create a folder on the remote windows host called Java_Update by executing powershell commands.
<br>
<br>

![](/Assets/java2.png)
<br>
<br>
This ansible task will copy the locally stored offline installer Java.exe file located at Ansible-Windows-Java-7zip-Playbook/roles/Java/files/Java.exe to the remote windows host within the created Java_Update folder.
<br>
<br>

![](/Assets/java3.png)
<br>
<br>
This ansible task will copy the locally stored config installation file located at Ansible-Windows-Java-7zip-Playbook/roles/Java/files/config to the remote windows host within the created Java_Update folder.
<br>
<br>

![](/Assets/java4.png)
<br>
<br>
This ansible task will silently uninstall any existing versions of "Java 8" using the wmic product uninstall command where any program with the name java 8 in it will be uninstalled.
<br>
<br>

![](/Assets/java5.png)
<br>
<br>
This ansible task will instantly install the latest java 8 version located locally at Ansible-Windows-Java-7zip-Playbook/roles/Java/files/Java.exe, an offline config file is required for successful installation which is located at Ansible-Windows-Java-7zip-Playbook/roles/Java/files/config.
<br>
<br>

![](/Assets/java6.png)
<br>
<br>
This ansible task will remove the offline java.exe installation file and Java_Update folder which is important to be able to re-run the playbook as it will fail if the folder already exists.
<br>
<br>

![](/Assets/Title5.png)
<br>
<br>
Ansible Vault encrypts variables and files so you can protect sensitive content such as passwords or keys rather than leaving it visible as plaintext in playbooks or roles.
<br>
<br>

![](/Assets/vault.png)
<br>
<br>

1. Create a file in your ansible directory e.g. password.txt.

2. Create a vault password within the file e.g. password123.

3. Enter the vm ansible account password into the vault using the command:
ansible-vault encrypt_string --vault-id file_path_to_password_file --name 'VM_Host_Name'

4. Copy and paste the ansible password for the remote windows user account into the command line. 

5. Type CTRL + D

6. A password encryption string will appear copy and paste all of it from !vault to the last number

7. Place that new password string into your ansible_password variable in your host file

8. Run playbooks with the following command to use the vaulted passwords: 
ansible-playbook playbook.yaml -i inventory.yaml --vault-password-file password.txt

