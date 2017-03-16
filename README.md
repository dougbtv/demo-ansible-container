## demo-ansible-container

a demo ansible container setup.

What follows are my raw notes while I got this working.

## ansible-container hello world

Guess you're gonna need docker, we'll just use the one packaged in centos repos.

```
yum install -y docker
systemctl enable docker
systemctl start docker
docker -v
```

So, first I went and ./spinup.sh a new host.

And then generally following [the installation directions](https://github.com/ansible/ansible-container#installing)

All as root...

```
yum install -y epel-release
yum install -y python-pip
pip install --upgrade pip
pip install -U setuptools
pip install ansible-container
```

Which gives you:

```
[root@anscontainer anscontainer]# ansible-container version
Ansible Container, version 0.2.0
```

Now using the [getting started guide](http://docs.ansible.com/ansible-container/getting_started.html).

```
mkdir anscontainer
cd anscontainer/
ansible-container init
```

Looks like...

```
[root@anscontainer anscontainer]# tree
.
└── ansible
    ├── ansible.cfg
    ├── container.yml
    ├── main.yml
    ├── meta.yml
    ├── requirements.txt
    └── requirements.yml
```

Ummm, might not be new enough.

```
[root@anscontainer anscontainer]# head -n 1 ansible/container.yml 
version: "1"
```

# Installing a newer version.

So that's not new enough, we want at least docker-compose version 2. So... Let's [follow this installation guide](https://docs.ansible.com/ansible-container/installation.html)

Uninstall the old one.

    [root@anscontainer ~]# pip uninstall ansible-container

Now... 

```
yum install -y git
git clone https://github.com/ansible/ansible-container.git
cd ansible-container/
python ./setup.py install
```

That's more like it.

```
[root@anscontainer ~]# ansible-container version
Ansible Container, version 0.3.0-pre
[root@anscontainer ~]# ansible-container init
Ansible Container initialized.
[root@anscontainer ~]# tree ansible
ansible
├── ansible.cfg
├── container.yml
├── main.yml
├── meta.yml
├── requirements.txt
└── requirements.yml

0 directories, 6 files
[root@anscontainer ~]# cat ansible/container.yml 
version: "2"
services:
  # Add your containers here, specifying the base image you want to build from
  # For example:
  #
  # web:
  #   image: ubuntu:trusty
  #   ports:
  #     - "80:80"
  #   command: ['/usr/bin/dumb-init', '/usr/sbin/apache2ctl', '-D', 'FOREGROUND']
  #   dev_overrides:
  #     environment:
  #       - "DEBUG=1"
  #
registries: {}
  # Add optional registries used for deployment. For example:
  #  google:
  #    url: https://gcr.io
  #    namespace: my-cool-project-xxxxxx  
```

## On the pieces.

Let's look at that tree again.

```
[root@anscontainer ~]# tree ansible
ansible
├── ansible.cfg
├── container.yml
├── main.yml
├── meta.yml
├── requirements.txt
└── requirements.yml
```

So...

* `container.yml` is sorta like your "docker-compose"
* `main.yml` is what to do to your containers.
* `requirements.{txt,yml}` is your python & role deps respectively
* `meta.yml` is for ansible galaxy.
* `ansible.cfg` is your... ansible config.

## Trying an install

For fun I tried...

```
[root@anscontainer container]# ansible-container install j00bar.nginx-container
```

So yeah, it did things. Didn't run a container, fwiw.

But it did modify my `main.yml`, so it looks like:

```
[root@anscontainer container]# cat ansible/main.yml 
- hosts: all
  gather_facts: false
- hosts: nginx
  roles:
  - role: j00bar.nginx-container
    STATIC_ROOT: /static
    PIDFILE_DIR: /pid
    ASSET_PATHS:
    - /tmp/dist
```

## Now let's do something semi-realistic with it.

Let's try two nginx services. Each one with a different HTML file that it serves, woo, advanced! ...not.

We'll have two containers "ren" and "stimpy" -- each an nginx install. But, each with a different HTML file, each one saying which one they are, ren or stimpy.

We're going to build the `container.yml` according to [Ansible container compose specification](http://docs.ansible.com/ansible-container/container_yml/reference.html)

created a repo as [demo-ansible-container](https://github.com/dougbtv/demo-ansible-container)

Get all of that stuff in one place...

And build it.

```
ansible-container build
```

And run it -- in production mode. This confused me for a good minute, missing production mode, you can read the "Development vs Production" section in the [ansible-container demo docs](https://ansible.github.io/ansible-container-demo/)

```
ansible-container run --production
```


Even better, run detached.

```
ansible-container run --detached --production
```

And you can stop it, too

```
ansible-container stop
```

