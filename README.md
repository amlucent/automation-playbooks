# automation-playbooks
Infrastructure Automation Playbooks 

Some assorted Ansible and Terraform infrastructure automation playbooks and shell scripts I wrote and find useful for my daily activities as a Storage System Engineer.  May tend to be geared towards SAN automation.

## Installation
Prior to running any scripts I recommend you install ansible and terraform. If you require an ansible control node I hightly recommend installing the following container https://github.com/bluxmit/alnoda-workspaces/tree/main/workspaces/ansible-terraform-workspace.

### Modules
All ansible modules required can be installed by running the below playbook on your ansible control node.

```bash
linux/all-linux-ansible-install.yml
```

### Variables
all my ansible playbook variables called out in these scripts should be defined and stored in vars file or even in your ansible inventory file.  
