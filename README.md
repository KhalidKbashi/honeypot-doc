#
# Honeypot Contains SSH and MySQL Pots in Azure

A. Creating a VM on Azure
1. log on your accout at Azure Portal
2. from the main panel hover on Azure Services and Click on Virtual Machines

![Screenshot 2024-04-18 172247](https://github.com/KhalidKbashi/honeypot-doc/assets/54905807/25ecb0c5-b0d8-40a3-9fd3-514f750b16e7)

3. Click on "Create"

![Screenshot 2024-04-18 174254](https://github.com/KhalidKbashi/honeypot-doc/assets/54905807/0ece161b-a900-4887-9264-1d93f1130535)

4. for VM's specification i'll start by creating a new resource group (i named it myDEV)
5. then select the region ,and image (i chose ubuntu as in picture), and VM architecture 
	1. we won't need availability zones options cause what we are doing is experimental and doesn't need that much redundancy
 
 ![Screenshot 2024-04-18 172511](https://github.com/KhalidKbashi/honeypot-doc/assets/54905807/a462f3f0-cab3-4170-ba76-7f11c4ab8cf0)

6. for VM's size i will choose the least cause we won't need that much resources

![Pasted image ٢٠٢٤٠٤١٨١٧٣٠١٩](https://github.com/KhalidKbashi/honeypot-doc/assets/54905807/26e95b5e-1510-4ce6-87a9-8c570fc8d3b3)

7. in Administrator Account Section we will configure the SSH setting to control the VM through terminal
	1. Select Authentication Type as Password , set username and password then in inbound port rules select port 22 (we will change it later)

![Pasted image ٢٠٢٤٠٤١٨١٧٣٦٣٣](https://github.com/KhalidKbashi/honeypot-doc/assets/54905807/9e3df800-b269-4c81-b289-18df2306c618)

8. in Network Tab enable "Delete public IP and NIC when VM is Deleted" 

![Pasted image ٢٠٢٤٠٤١٨١٧٣٩٠٩](https://github.com/KhalidKbashi/honeypot-doc/assets/54905807/1789d7c0-316e-40c6-8c51-39de6af4ba7c)

9. for sake of arrangement hover to tags tab and create a tag to make managing and using VM easier
	1. ![[Pasted image ٢٠٢٤٠٤١٨١٧٤٠٥١.png]]
10. Click on Review + Create
11. open Virtual Machines again and select the VM we've Just created
12. Hover to Networking Section and Copy the Public IP Address 
	1. ![[Pasted image ٢٠٢٤٠٤١٨١٧٤٤٥٤.png]]
13. now let's connect to the VM through VM's Public IP Address
	1. ssh -p 22 20.29.117.249

installing the SSH Pot
* the pot rejects every credentials attacker enters and logs it

1. firstly i'll close the port 22 cause i want to run the honeypot through 
	1. sudo systemctl stop ssh
	2. sudo systemctl stop ssh.socket
2. now let's open another port for SSH connection
	1. sudo sshd -p 55677
	2. now you can safely terminate current session and use the 55677 port if any somthing wrong happened
3. i'll use Honeypot created by kingtuna
4. clone the repo using 
	1. git clone https://github.com/kingtuna/sshpot.git
5. ![[Pasted image ٢٠٢٤٠٤١٨٠٨٠٣١٧.png]]
6. reading the end of README file there will be some prerequisites packages to install 
	1. sudo apt update
	2. sudo apt-get _install libssh_-dev
	3. sudo apt install make
7. next we will generate an RSA public key to be used by the server (the pot)
	1. ssh-keygen -t rsa
	2. next enter the file name that the RSA key will be stored into
	3. ![[Pasted image ٢٠٢٤٠٤١٨٠٨٠٩٢٤.png]]
8. open configuration file config.h
	1. nano config.h
	2. ![[Pasted image ٢٠٢٤٠٤١٨٠٨٠٦٥٩.png]]
	3. we will change the path of the RSA_KEYFILE to the file we've just created (temp.txt)
9. now lets MAKE it
	1. just type "make"
	2. ![[Pasted image ٢٠٢٤٠٤١٨٠٨١٦٢٠.png]]
10. now let's run it
	1. sudo ./sshpot -p 22 &
	2. ![[Pasted image ٢٠٢٤٠٤١٨٠٨١٨٤٥.png]]
11. to make sure it's running
	1. sudo ss -tulp | grep ssh
	2. ![[Pasted image ٢٠٢٤٠٤١٨٠٨١٩٥٥.png]]

now let's install the MySQL Pot
* the pot records all credentials attacker typing and logs it

1. i'll use Honeypot created by qeeqbox
3. Firstly the prerequisites to install 
	1. sudo apt update
	2. sudo apt-get install postgresql
	3. sudo apt-get install python-psycopg2
	4. sudo apt-get install libpq-dev
	5. sudo apt install python3-pip
	6. pip3 install honeypots
4. create the config file for MySQL
	1. nano config.json
	2. past this
		1. {
  "logs": "file,terminal,json",
  "logs_location": "/var/log/honeypots/",
  "syslog_address": "",
  "syslog_facility": 0,
  "postgres": "",
  "sqlite_file":"",
  "db_options": [],
  "sniffer_filter": "",
  "sniffer_interface": "",
  "honeypots": {
    "mysql": {
      "port": 3306,
      "ip": "0.0.0.0",
      "user": "mysql",
      "socket": "/var/lib/mysql/mysql.sock",
      "password": "anonymous",
      "log_file_name": "mysql.log",
      "max_bytes": 10000,
      "backup_count": 10,
      "options":["capture_commands"]
    }
  }
}
	3. save it
5. run it
	1. sudo -E python3 -m honeypots --setup mysql --config config.json &
6. to make sure it's running
	1. sudo ss -tulp | grep mysql
	2. ![[Pasted image ٢٠٢٤٠٤١٨٠٨٥٢٠٣.png]]
