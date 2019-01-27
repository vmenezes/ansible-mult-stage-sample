# ansible-mult-stage-sample

This is a sample of hot to organiza Vagrant/Ansible to have 1 local
environment for development and 2 cloud ones for staging/production.

Local environment is a Ubuntu 18.04 VirtualBox created by Vagrant and setup
by Ansible;

Staging/Production are also Ubuntu 18.04 but those are supposed to be on a
cloud provider(Azuer, AWS, DigitalOcean, etc);

# Getting started

Install VirtualBox and Vagrant on your computer (tested on Ubuntu18.04 with
VirtualBox 5.2.18 and Vagrant 2.0.2). Your project will have 3 servers.
One local Virtual Machine for development, and two cloud servers, for
staging/production. This is a faily common number/naming of environments.

## Getting dev environment running

Start by creating a folder for this project called `my_proj`.
Inside the folder `my_proj` create a folder called `provisioner`.
Finally, inside the folder `provisioner` create a folder called `host_vars`.

Now create a file called `Vagrantfile` inside `my_proj` folder and paste
the content in it:

```
Vagrant.require_version ">= 1.7.0"

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.box_version =  "20181214.0.0"

  config.vm.define "dev-vm" do |machine|
    config.vm.network "forwarded_port", guest: 8000, host: 8000
    config.vm.network "forwarded_port", guest: 5000, host: 5000
    config.vm.network "forwarded_port", guest: 80, host: 8080
    config.vm.hostname = "dev-vm"
  end

  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "provisioner/playbook.yml"
    ansible.install_mode = "pip"
    ansible.compatibility_mode = "2.0"
    ansible.limit = "dev-vm"
  end
end
```

Inside the `provisioner` folder, create a file called `playbook.yml` and
place the following content in it:

```
---
- name: All envs playbook
  hosts: all
  vars:
    ansible_python_interpreter: "/usr/bin/python3"
    become: yes
  tasks:
    - name: "Global task"
      debug:
        msg: "all envs MUST log this {{ my_env }}"

- name: Dev VM only playbook
  hosts: dev-vm
  vars:
    ansible_python_interpreter: "/usr/bin/python3"
    become: yes
  tasks:
    - name: "DEV VM TASK"
      debug:
        msg: "JUST dev-vm MUST log this {{ my_env }}"
```

Inside the `host_vars` folder, create a file called `dev-vm.yml` and
place the following content in it:

```
---
my_env: "DEVELOPMENT_VARIABLE"
```

This should be enough to get it working but to make it a bit nicer, lets
add just 2 more files. First, within `my_proj` folder, create a file
called `.gitignore` and paste the following content to prevent some
temp files of being tracked by your Git:

```
.vagrant/
provisioner/playbook.retry
ubuntu-bionic-18.04-cloudimg-console.log
```

To end, create a `README.md` within `my_proj` folder. This file should be
for you to write your notes about how to use this project, etc.

Your project by now should look like this:

```
my_proj/
    provisioner/
        host_vars/
            dev-vm.yml
        playbook.yml
    .gitignore
    README.md
    Vagrantfile
```

Now to use it, open `my_proj` folder on the terminal and check the
status of your Dev VM using the command `vagrant status`.

> As we have a Vagrantfile with instructions to create a VM withing the
> folder, Vagrant will check the status of the VM with VirtualBox
> and report as `not created`

Lets turn-on our Dev VM by running the command `vagrant up`

> This command will open Vagrantfile, and create a Ubuntu 18.04
> machine with ports 80, 5000 and 8000 forwarded to 8080, 5000
> and 8000 of the host computer. It also will have `/vagrant`
> shared folder inside the VM that is to `my_proj` folder.
> Finally, it woull use the Vagrant module `ansible_local`
> to run the Ansible playbook `my_proj/provisioner/playbook.yml`
> on the Dev VM.

You can see again the status of the Dev VM with `vagrant status`,
that now should report `running` and case you make changes on
the Ansible playbook and need to re-apply to the Dev VM you can
simply run `vagrant provision`. Case you want to "delete" it
just run `vagrant destroy` or to just turn-off `vagrant halt`.

To shell-in your Dev VM just run `vagrant ssh`.

Notes about development environment:

- Variables used by Development environment must be set on `dev-vm.yml`
- Ansible tasks/roles that must be applied on ALL environments must be on `All envs playbook` section on `playbook.yml`
- Ansible tasks/roles that must be applied only on Dev VM must be on `Dev VM only playbook` section on `playbook.yml`
