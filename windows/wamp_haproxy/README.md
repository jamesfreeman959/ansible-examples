# Setting up the Ansible host

These playbooks have been tested with Ansible 2.2.1.0 on RHEL 7.3

In addition to Ansible, please install the following packages:
```
yum install python2-boto python2-winrm
```

You will also need a copy of the ec2 dynamic inventory script and configuration file, which should be downloaded from here:

[ec2.py](https://raw.github.com/ansible/ansible/devel/contrib/inventory/ec2.py)
[ec2.ini](https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/ec2.ini)

# Setting up the EC2 instances

1. Set up your `group_vars/all` file to contain your AWS ID and access key, preferred region, number of web server instances and so on. Note that for this demo we assume the existence of a Security Group with access to HTTP (80/tcp) WinRM over HTTPS (5986/tcp) enabled. You may also find it useful to have access to RDP traffic (3389/tcp) to debug any issues should they occur. Place the name of this into this file also.

2. Create an Ansible Vault to store the initial password for your Windows instances:
```
ansible-vault create group_vars/secret.yml
```

The contents of this should look something like this:
```
win_initial_password: myTempPassword123!
```

3. Run the playbook as follows:
```
ansible-playbook demo-aws-wamp-launch.yml --ask-vault-pass
```

4. Now set up the dynamic inventory script to ensure it can find your new EC2 instances. You will need to export your AWS credentials in your terminal session as follows:
```
export AWS_ACCESS_KEY_ID='xxxxxxx'
export AWS_SECRET_ACCESS_KEY='xxxxxx'
```

5. Also check the region setting in `ec2.ini`

6. Now set up two group variables files to match the tags of the two sets of EC2 machines created in step 3 - it is recommended to use ansible-vault to create these as they will contain the Windows Adminstrator password as specified in step 2 above:

```
$ ansible-vault create tag_ansible_group_windows_dbservers.yml
Vault password:
ansible_ssh_user: Administrator
ansible_ssh_pass: myTempPassword123!
ansible_ssh_port: 5986
ansible_connection: winrm
# The following is necessary for Python 2.7.9+ when using default WinRM self-signed certificates:
ansible_winrm_server_cert_validation: ignore
$ ansible-vault create tag_ansible_group_windows_webservers.yml
Vault password:
ansible_ssh_user: Administrator
ansible_ssh_pass: myTempPassword123!
ansible_ssh_port: 5986
ansible_connection: winrm
# The following is necessary for Python 2.7.9+ when using default WinRM self-signed certificates:
ansible_winrm_server_cert_validation: ignore
```

6. Now run the deployment playbook as follows:
```
ansible-playbook -i ec2.py site.yml --ask-vault-pass
```

7. Now if you have the AWS CLI installed, you can find the IP or DNS name of your new load balancer to access the service:
```
aws elb describe-load-balancers
```

8. Now to update the web content on the front-end nodes, edit the `rolling_update.yml` file and set the serial to identify which of the 3 instances you want to update, and then run:
```
ansible-playbook -i ec2.py rolling_update.yml --ask-vault-pass
```

