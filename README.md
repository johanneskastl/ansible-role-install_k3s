install_k3s
=========

Install k3s (without downloading and executing shell code with root permissions)

Requirements
------------

None.

Role Variables
--------------

None.

Dependencies
------------

None

Example Playbook
----------------

Use a playbook like the following to deploy onto a single node:

```
- hosts: some_hostname
  roles:
    - role: 'johanneskastl.install_k3s'
```

Depending on your host, you might want to set the tls_san variable to contain any external IPs or hostnames or similar.

In case you want to setup k3s with one controlplane node and multiple workers, use the following playbooks:

```
- name: 'Install, configure and start k3s on the first k3s server only'
  hosts: 'k3sservers[0]'
  gather_facts: 'yes'
  become: 'true'

  roles:
    - role: 'johanneskastl.install_k3s'
      vars:
        fetch_token: 'true'
```

```
- name: 'Install, configure and start k3s'
  hosts: 'k3sworkers'
  gather_facts: 'yes'
  become: 'true'
  vars_files:
    - 'configuration.yml'

  pre_tasks:

    - name: 'Create /etc/rancher/k3s directory'
      file:
        path: '/etc/rancher/k3s'
        state: 'directory'
        owner: 'root'
        group: 'root'
        mode: '0755'

    - name: 'Copy token file to /etc/rancher/k3s/token_to_register'
      copy:
        src: 'k3s-token'
        dest: '/etc/rancher/k3s/token_to_register'
        owner: 'root'
        group: 'root'
        mode: '0600'

  roles:
    - role: 'install_k3s'
      vars:
        k3s_node_type: 'agent'
        server_url: "https://{{ hostvars[groups['k3sservers'][0]]['ansible_default_ipv4']['address'] }}:6443"
        token_file: '/etc/rancher/k3s/token_to_register'`
```

The second playbook sets the type to `agent`, the server_url to the IP address of the master node and hands over the path to the token file.

License
-------

BSD-3-Clause

Author Information
------------------

I am Johannes Kastl, reachable via kastl@b1-systems.de.
