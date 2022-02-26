# automation-playbooks
Infrastructure Automation Playbooks 

Some assorted Ansible and Terraform infrastructure automation playbooks and shell scripts I find useful for my daily activities as a Storage System Engineer.

## Installation
Prior to running any scripts I recommend you install ansible and terraform. If you require an ansible control node I hightly recommend installing the following container https://github.com/bluxmit/alnoda-workspaces/tree/main/workspaces/ansible-terraform-workspace.

### Modules
All ansible modules required can be installed by running the below playbook on your ansible control node.

```bash
linux/all-linux-ansible-install.yml
```

