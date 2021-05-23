install_k3s
=========

Install k3s (without downloading and executing shell code with root permissions)

Requirements
------------

None.

Role Variables
--------------

Many.

Basically most options that can be handed over to k3s when installing via the script being curled and piped to bash, can be set as variables that will be put into the `config.yaml` file.
The only special one is `disable_something` (the CLI option is called `disable`, but that makes a horrible variable name).

*Variables translated to config.yaml settings*
- `disable_something`
- `bind_address`
- `https_listen_port`
- `advertise_address`
- `advertise_port`
- `tls_san`
- `data_dir`
- `cluster_cidr`
- `service_cidr`
- `cluster_dns`
- `cluster_domain`
- `flannel_backend`
- `token`
- `token_file`
- `write_kubeconfig`
- `write_kubeconfig_mode`
- `kube_apiserver_arg`
- `kube_scheduler_arg`
- `kube_controller_manager_arg`
- `kube_cloud_controller_manager_arg`
- `datastore_endpoint`
- `datastore_cafile`
- `datastore_certfile`
- `datastore_keyfile`
- `default_local_storage_path`
- `disable_scheduler` (true/false)
- `disable_cloud_controller` (true/false)
- `disable_network_policy` (true/false)
- `node_name`
- `with_node_id` (true/false)
- `node_label`
- `node_taint`
- `docker` (true/false)
- `container_runtime_endpoint`
- `pause_image`
- `private_registry`
- `node_ip`
- `node_external_ip`
- `resolv_conf`
- `flannel_iface`
- `flannel_conf`
- `kubelet_arg`
- `kube_proxy_arg`
- `rootless` (true/false)
- `agent_token`
- `agent_token_file`
- `server_url`
- `secrets_encryption` (true/false)

If you want to have a specific version of k3s installed, settings `k3s_version` will do that 8see the example playbook below).

In case you want to override the default locations, these two variables can be set:
- `path_to_install_to`: defaults to `/usr/local/bin`
- `systemd_directory_path`: defaults to `/etc/systemd/system`

If you know why you would want that, feel free to use a different URL or channel. Here are the defaults:
- `k3s_installation_url`: defaults to `https://update.k3s.io/v1-release/channels`
- `k3s_installation_channel`: defaults to `stable`

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

If you want to have a different k3s version, use something like this:
```
- hosts: some_hostname
  roles:
    - role: 'johanneskastl.install_k3s'
      vars:
        k3s_version: 'v1.19.10+k3s1'
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
