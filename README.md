Role Name
=========

An ansible role to setup a chroot and launch a sshd instance in the chroot via systemd-nspawn to allow for further management via Ansible

Requirements
------------

- Systemd build based system
- Chroot name cannot contain any characters that systemd-nspawn and machinectl don't support, such as hypens
- Allow Ansible to connect to a non-standard SSH port as defined in the inventory for the chroot with ansible_port
- If you run a playbook and limit it to one chroot, you most also include the build_host in the limit.
- Each chroot built on the same host should have unique variables:
  - ansible_port
  - chroot_dir

Role Variables
--------------

    # These two variables must be defined per chroot
    chroot_build_host:
    chroot_dir:

    # Variable to control if the chroot is torndown
    chrootbuild_teardown: false

    # Additional arguments to pass to yum for the initial chroot creation via yum --installroot
    chrootbuild_yum_args: ''

    # Specify where to put the yum.conf and yum.repos.d in the chroot, we use the yumrepos role for this work
    chroot_yumrepo_dir_path: "{{ yumrepo_dir_path|default(chroot_dir + '/etc/yum.repos.d') }}"
    chroot_yumrepo_conf_path: "{{ yumrepo_conf_path|default(chroot_dir + '/etc/yum.conf') }}"
    # Yum repos, we use the yumrepos role for this work
    chroot_yumrepo_repos: "{{ yumrepo_repos|default({}) }}"

    # What base package group should be used to create the chroot
    chroot_base_package_group: "core"
    # A space delimited list of additional packages to install into the chroot, passed directly as an argument to yum
    chrootbuild_extra_pkgs: "kernel"

    chrootbuild_root_add_ssh_pub_key: true

    # Default to using the insecure vagrant pub key, for the lack of a better default.
    chrootbuild_root_ssh_pub_key: 'ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ=='

    # Allow the specification of environment variables like http_proxy and https_proxy if needed to get to the yum repos
    proxy_env: {}

### Example chroot host_vars file

    chroot_build_host: build1
    chroot_dir: /var/chroots/rhel72
    chrootbuild_yum_args: '--exclude=firewalld --exclude="NetworkManager*" --exclude=rdma --exclude=yum-rhn-plugin --exclude=subscription-manager'

    yumrepo_dir_path: "{{ chroot_dir }}/etc/yum.repos.d"
    yumrepo_conf_path: "{{ chroot_dir }}/etc/yum.conf"
    chrootbuild_root_ssh_pub_key: 'ssh-rsa ...'

Dependencies
------------

None

Example Playbook
----------------

### Inventory

    [build_hosts]
    build1

    [chroots]
    rhel72 ansible_ssh_host=build1 ansible_ssh_port=20202 ansible_ssh_user='root' ansible_ssh_private_key_file='~/.ssh/build_key'

### Playbook

    - hosts: chroots
      become: yes
      become_method: sudo
      gather_facts: no
      roles:
        - chrootbuild
        - { role: dns, tags: ["packages", "dns", "repos"] }
        - { role: chrootbuild, chrootbuild_teardown: true, tags: ["packages", "dns", "repos"] }

License
-------

BSD

Author Information
------------------

Ben Allen <bsallen@alcf.anl.gov>
