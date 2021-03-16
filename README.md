# local-wordpress-bedrock-trellis-project-setup

Project setup for examplewebsite.com based on Wordpress, founded on Roots.io/Bedrock to provide a modern LEMP Stack and Roots.io/Trellis to provide Development and Production servers done right.

## Local Trellis Project Setup

<details>
<summary>Create the Bedrock/Trellis Project Folder Structures</summary>

Trellis relies on a few other software tools. Make sure all dependencies have been installed before moving on:

+ [Virtualbox](https://www.virtualbox.org/wiki/Downloads)
+ [Vagrant](https://www.vagrantup.com/downloads.html)
+ *Recommended* [trellis-cli](https://github.com/roots/trellis-cli)

0. Create the examplewebsite.com project directory and `cd` into it:
```
mkdir examplewebsite.com && cd examplewebsite.com
```

1. Create a Git Repo and flavor to taste:
```
git init
touch README.md
git commit -m "Hello examplewebsite.com"
```

2. Install Trellis:
```
cd examplewebsite.com
```

```
git clone --depth=1 git@github.com:roots/trellis.git && rm -rf trellis/.git
```

3. Install Bedrock:
```
cd examplewebsite.com
```

```
composer create-project roots/bedrock bedrock
```

the folder structure for a Trellis project will look like this:

```
examplewebsite.com/ # → Root folder for the project
├── bedrock/        # → A Bedrock-based WordPress site
│   └── web/        # → You web root directory
│       ├── app/    # → WordPress content directory (themes, plugins, etc.)
│       └── wp/     # → WordPress core (don't touch!)
└── trellis/        # → Your clone of this repository
├── README.md       # → This README file
```

For further details, read Trellis [docs/installation](https://roots.io/docs/trellis/master/installation/#create-a-project)

</details>

## Propagate Trellis with Website Data

> **WARNING**: Don't do any git commits until completing step: **Password Encryption with Ansible Vault** below.

<details>
<summary>Propagate Trellis Files with Actual Website Data</summary>

Edit these files with real website data and plaintext passwords as needed:

+ trellis/group_vars/all/users.yml
+ trellis/group_vars/all/vault.yml
+ trellis/group_vars/development/vault.yml
+ trellis/group_vars/development/wordpress_sites.yml
+ trellis/group_vars/production/vault.yml
+ trellis/group_vars/production/wordpress_sites.yml
+ trellis/group_vars/staging/vault.yml
+ trellis/group_vars/staging/wordpress_sites.yml

If your primary git repo branch is called `main` not `master`:

+ trellis/roles/deploy/defaults/main.yml
  change this:
  ```
  project_version: "{{ branch is defined | ternary(branch, project.branch) | default('master') }}"
  ```
  to this:
  ```
  project_version: "{{ branch is defined | ternary(branch, project.branch) | default('main') }}"
  ```

If on MacOS 10.14~ to 10.15~, open `trellis/vagrant.default.yml` and change `vagrant_mount_type: 'nfs'` to `vagrant_mount_type: 'virtualbox'` to avoid many nfs errors.

</details>

## Setup Ansible Vault for Repo Password Encryption

<details>
<summary>Setup Ansible Vault files Needed</summary>

+ Create `trellis/.vault_pass` (is already ignored in `trellis/.gitignore`) and add a random string that Ansible will use to encrypt the passwords.
+ Edit `trellis/ansible.cfg` and add at the bottom of the `[defaults]` section: 
`vault_password_file = .vault_pass`.

</details>

## Create Vagrant Virtual Machine

<details>
<summary>Use Vagrant to create the virtual machine</summary>

> **NOTE**: This step takes a while to complete (10-15min).

```
cd trellis
```

Run:
```
vagrant up
```

If things don't go correctly, just run the following to take down the vagrant VM:
```
vagrant halt
```

If you make changes and want to start/restart the VM run:
```
vagrant up --provision
```

### Troubleshoot `vagrant up`

#### Message regarding `vagrant up` must be re-run now that plugins are installed

Common if run for the first time. Re-run `vagrant up`.

#### Errors regarding "Administrator privileges" or NFS error timeouts

Common on Mac systems. Read these:

+ [Vagrant Up and annoying NFS password asking](https://askubuntu.com/questions/412525/vagrant-up-and-annoying-nfs-password-asking)
+ [and its valid solution here](https://askubuntu.com/questions/412525/vagrant-up-and-annoying-nfs-password-asking#comment1519152_519841).

#### Message: The nfsd service does not appear to be running.

accompanying message: mount.nfs: Connection timed out
accompanying message: /etc/exports: Operation not permitted

Common on Mac systems.

check to see if /etc/exports exists:
```
ls /etc/ | grep exports
```

If doesn't exist, create with:
```
sudo touch /etc/exports
```

Check the status of nfsd:
```
nfsd status
```

If not running, usually it is because /etc/exports didn't exist and launchd didn't start it at startup. enable and start with: 
```
sudo nfsd enable
sudo nfsd start
```

Re-run vagrant:
```
vagrant up
```

</details>

## Password Encryption with Ansible Vault

<details>
<summary>Use Ansible Vault to encrypt the passwords</summary>

The files containing passwords can now been encrypted:

```
cd trellis
```

Run:
```
ansible-vault encrypt \
group_vars/all/vault.yml \
group_vars/development/vault.yml \
group_vars/production/vault.yml \
group_vars/staging/vault.yml
```

</details>

The files containing passwords have now been encrypted.

**NOTE**: Git commits can now resume as passwords are now encrypted.
