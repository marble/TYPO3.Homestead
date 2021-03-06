TYPO3 Homestead
===============

TYPO3 Homestead is your one-stop [TYPO3](http://typo3.org) development environment. Just run `vagrant up` and a full Linux Ubuntu distribution will be built with all the packages and configuration needed to start development right away.

This environment is inteded as as a local environment. Security-wise it is in no way fit for production.

Features
--------

TYPO3 Homestead comes with the following stack:

* composer
* hhvm
* mariadb
* nginx
* nodejs
* php-apcu
* php-fpm
* self signed ssl certificates
* TYPO3 CMS
* TYPO3 FLOW
* TYPO3 NEOS
* xdebug
* xhprof / blackfire
* zsh

The flexible configuration allows you to create any combination of TYPO3 source and PHP backend with or without SSL.

Requirements
------------

* [Virtualbox](https://www.virtualbox.org/) or another virtualization product - Free!
* [Vagrant](http://www.vagrantup.com/) - Free!
* [Ansible](http://docs.ansible.com/intro_installation.html) - Free! (only the Tower ui will cost you)

Dependencies
------------

TYPO3 Homestead uses several roles from the ansible-galaxy:

* [nbz4live.php-fpm](https://galaxy.ansible.com/list#/roles/304)
* [jdauphant.nginx](https://galaxy.ansible.com/list#/roles/466)
* [geerlingguy.composer](https://galaxy.ansible.com/list#/roles/429)
* [laggyluke.nodejs](https://galaxy.ansible.com/list#/roles/285)

Later you will need to install these roles before you can run the playbook.

```bash
cd TYPO3.Homestead
sudo ansible-galaxy install -r requirements.yml
```

Installation
------------

Installation is pretty straight forward:
```bash
git clone https://github.com/Tuurlijk/TYPO3.Homestead.git
cd TYPO3.Homestead
```

Now setup your shared directory to hold your TYPO3 sources and sites in your `Vagrantfile`:
```ruby
# tune this setting in 'Vagrantfile':
config.vm.synced_folder "~/Projects/TYPO3/Development", "/var/www", create: true, group: "www-data", owner: "vagrant", mount_options: ["dmode=775,fmode=664"]
```

Then install the requirements and boot the machine. A protocol is left in `vagrant-up.log.txt`:
```bash
cd TYPO3.Homestead
sudo ansible-galaxy install -r requirements.yml
vagrant up 2>&1 | tee vagrant-up.log.txt
```

When the installation process has finished, you can visit [http://typo3.homestead](http://typo3.homestead). And also any of the pre-configured sites or any site you configured. The default sites are:

* [4.5.typo3.cms](http://4.5.typo3.cms)
* [4.5.39.typo3.cms](http://4.5.39.typo3.cms)
* [6.2.typo3.cms](http://6.2.typo3.cms)
* [6.2.9.typo3.cms](http://6.2.9.typo3.cms)
* [7.0.typo3.cms](http://7.0.typo3.cms)
* [7.0.2.typo3.cms](http://7.0.2.typo3.cms)
* [1.2.typo3.neos](http://1.2.typo3.neos)
* [dev-master.typo3.neos](http://dev-master.typo3.neos)

Currently the sites are not fully set up yet. You will need to run through the install tools by hand. This will be simplified later on.

The database credentials can be found in `roles/mariadb/vars/main.yml`. The typo3 user has access to all database. The install tool password is the TYPO3 default.

You can pass seveal command line options to Vagrant when booting the machine:
* VAGRANT_CORES - Amount of cpu cores to enable (1)
* VAGRANT_DEBUG - Enable Virtualbox GUI during machine start (false)
* VAGRANT_HOSTNAME - The hostname the system will use (typo3.homestead)
* VAGRANT_MEMORY - Amount of memory in megabytes to use (2048
* VAGRANT_PRIVATE_NETWORK - Private network address to use (192.168.12.12)

You can just set the variables when booting the machine:

```bash
VAGRANT_CORES=2 VAGRANT_PRIVATE_NETWORK=192.66.99.11 vagrant up
```

Making it faster
----------------

When you provision the machine, a ramfs based ramdisk will be created. This is a RAM based filesystem that will grow as needed. Fstab entries will be created for each active website. These will mount-bind your typo3temp folders to a corresponding folder on the ramfs based RAM disk. Unfortunately we can not auto-mount these on the next machine boot as they will need the fileshare from the host system to be active.

This means that the disks wil be active when provisioning for the first time, but when you start up the machine again (after it has already been provisioned) you will need to mount the ramdisks manually by running:
 ```bash
 ANSIBLE_ARGS='--tags=typo3-cms-ramdisk' vagrant provision
 ```

 You should also run that command when you add a website for which you wish to use a ramdisk based typo3temp folder.

Variables
---------

You can override any of the role variables in the configuration files in the `/vars/` directory. The options have been tuned for usage with TYPO3, so the ones you will most likely be changing are the `typo3.yml` and the `websites.yml` files. In the `typo3.yml` file you can configure your typo3.org username and also what versions of TYPO3 source you wish to have available. You can specify an array of tags (`git tag`) and branches (`git branch -r`) to checkout from git:

```yaml
typo3:
  cms:
    sources:
      tags: ['TYPO3_4-5-39', 'TYPO3_6-2-9', 'TYPO3_7-0-2']
      branches: ['TYPO3_4-5', 'TYPO3_6-2', 'TYPO3_7-0']
```

You can then 'map' the available TYPO3 sources to a domain name. TYPO3 Homestead will then know what source to link to what domain name during the provisioning step.

```yaml
typo3:
  cms:
    sources:
      tags: ['TYPO3_4-5-39', 'TYPO3_6-2-9', 'TYPO3_7-0-2']
      branches: ['TYPO3_4-5', 'TYPO3_6-2', 'TYPO3_7-0']
    sites:
      4.5.typo3.cms: 'TYPO3_4-5'
      4.5.39.typo3.cms: 'TYPO3_4-5-39'
      6.2.typo3.cms: 'TYPO3_6-2'
      6.2.9.typo3.cms: 'TYPO3_6-2-9'
      7.0.typo3.cms: 'TYPO3_7-0'
      7.0.2.typo3.cms: 'TYPO3_7-0-2'
```

If you change any typo3 configuration after you have provisioned your server, you will need to re-provision using:

```bash
ANSIBLE_ARGS='--tags=typo3' vagrant provision
```

If you change any nginx configuration after you have provisioned your server, you will need to re-provision using:

```bash
ANSIBLE_ARGS='--tags=nginx' vagrant provision
```

You can also do both at once. Please not that typo3 needs to run first because the ssl certificates and web directories need to be available for nginx to pass the configuration check:

```bash
ANSIBLE_ARGS='--tags=typo3,nginx' vagrant provision
```

The NEOS configuration is WIP. The TYPO3 CMS configuration sets up an empty database and an empty site. So you will need to run through the install tool and set things up.

Configuration Examples
----------------------

You can choose between different PHP backends:
* php-fpm
* php-fpm-xhprof (not enabled yet: wip)
* hhvm

```yaml
nginx_sites:
  4.5.typo3.cms:
    - server_name 4.5.typo3.cms
    - root "{{ typo3_webroot }}4.5.typo3.cms/"
    - include snippets/php-fpm.conf
  4.5.hhvm.typo3.cms:
    - server_name 4.5.hhvm.typo3.cms
    - root "{{ typo3_webroot }}4.5.hhvm.typo3.cms/"
    - include snippets/hhvm.conf
```

Please take care to add a domain-name to source mapping in the typo3.yml for each site you configure.

The `typo3_ssl_certificates` variable is an array of domain names for which self signed ssl certificates will be generated.

TODO
----

* Complete the preconfiguration of TYPO3 CMS instances
* Add installation and configuration of NEOS instances
* Find a proper solution for git deep-clone (cp -pr ?)
* Nginx configuration snippets?
  https://github.com/h5bp/server-configs-nginx/blob/master/h5bp/
* Caching and NFS slowness
  http://www.whitewashing.de/2013/08/19/speedup_symfony2_on_vagrant_boxes.html
  https://gitlab.com/gitlab-org/cookbook-gitlab/blob/master/Vagrantfile
* Setup cronjobs

License
-------

[GNU General Public License version 3](https://www.gnu.org/licenses/gpl-3.0.html)

References
----------

- [konomae/ansible-laravel-settler](https://github.com/konomae/ansible-laravel-settler)
- [laravel/homestead](https://github.com/laravel/homestead)
- [laravel/settler](https://github.com/laravel/settler)
