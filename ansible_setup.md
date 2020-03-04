1. Download the RPM of Ansible engine from [releases.ansible.com](https://releases.ansible.com/ansible/rpm/release/epel-7-x86_64/)
```
#  curl -O https://releases.ansible.com/ansible/rpm/release/epel-7-x86_64/ansible-2.9.5-1.el7.ans.noarch.rpm
```
2. Install the PRM of Ansible enginee
```
# yum localinstall ansible-2.9.5-1.el7.ans.noarch.rpm
```
4. Install the required libraries in Ansible Control node
```
# yum install -y net-tools iproute zip unzip openssh-clients
```
5. Install the required libraries on Ansible Managed node 
```
# yum install -y net-tools iproute zip unzip openssh-server
```
6. Enable the sshd service on Ansible Managed node
```
# systemctl enable sshd.service && systemctl start sshd.service
```
7. Make a inventory file on Ansible Control node
```
# echo "ansible.mgt ansible_host=172.17.0.3 ansible_port=22 ansible_user=root ansible_password=123456" > ~/ansible_hosts
```
  You can read more about the inventory file at [How to build your inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#inventory)

8. Test
```
# ansible -i ~/ansible_hosts all -m ping 
```
