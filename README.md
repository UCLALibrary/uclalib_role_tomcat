# UCLALib Ansible Role: Tomcat

Installs Tomcat on RHEL servers

A top-level Tomcat CATALINA_HOME is installed with separate CATALINA_BASEs for each application

This role creates the following directory structure:

```
CATALINA_HOME:
/usr/local/tomcat7
├── bin
├── conf
├── lib
├── logs
├── temp
├── webapps
└── work

CATALINA_BASEs inherit bin and lib from CATALINA_HOME:
/usr/local/app1
├── conf
├── lib
├── logs
├── temp
├── webapps
└── work
```
For each CATALINA_BASE tomcat app:
* directory structure created in /usr/local/ (optionally using LVM for filesystem)
* environment variable file create in `/etc/sysconfig`
* init script created in `/etc/init.d`
* apache httpd virtual host config added to `/etc/httpd/vhosts.d/vhosts.conf`
* logrotate configuration added to `/etc/logrotate.d/tomcat`

## Dependencies

* [ansible_uclalib_role_java](https://github.com/UCLALibrary/ansible_uclalib_role_java)
* [ansible_uclalib_role_apache](https://github.com/UCLALibrary/ansible_uclalib_role_apache)

## Variables

  * `tomcat_major_version` - defines the major version of Tomcat to use (e.g. 6, 7, 8, etc.)

  * `tomcat_version` - defines the full version of tomcat to use (e.g. 7.0.67)

  * `tomcat_download_url` - defines the URL to download Tomcat binaries

  * `tomcat_applications` - variable that instantiates the tomcat application list
    * `app_name` - name of tomcat web app (usually matches name of .war file)
    * `shut_port`- port number to use for tomcat shutdown port in server.xml
    * `conn_port` - port number to use for tomcat connection port in server.xml
    * `rproxy_path` - path to configure the apache reverse proxy to use to access the tomcat app
    * `proxypass_extra_params` - (optional) defines a list of extra parameters to append to the ProxyPass line in the tomcat vhost file
        * examples include: `nocanon` or `keepalive=on` - separate multiple parameters with a single space all enclosed in quotes (e.g. `"nocanon keepalive=on"`)

  * `tomcat_server_name` - defines the FQDN for the server the tomcat apps will run on. This is used when creating the HTTPD vhost configuration

  * `tomcat_subdirs` - defines the sub-directories to create in each CATALINE_BASE app directory

  * `default_webapps` - defines the default tomcat webapps that will be removed

  * `use_lvm` - determines if the role should provision the tomcat web app within a filesystem managed by Logical Volume Management (LVM)

  * `vg_name` - defines the name of the LVM volume group

  * `lv_size` - defines the size of the LVM logical volume

  * `lvm_base_path` - defines the path where the logical volumes are managed by the operating system

  * `lvm_filesystem_type` - defines the filesystem type to create when provisioning the lvm mount point

Variables with default values that define if this deployment should use SSL
For a local dev deployment, default values enable SSL and install self-signed certificates
For a production install, defaults values are overriden by variables defined in host_vars
  * `tomcat_enable_ssl` - defines if this deloyment should use SSL (`yes` or `no` - default is `yes`)
  * `ssl_cert_base_path` - defines the base path to the SSL certs and key
  * `ssl_cert_file_path` - defines the path to the SSL certs
  * `ssl_key_file_path` - defines the path to the SSL private key
  * `ssl_files`
      * `self_signed` - defines if the certificates are self-signed (`yes` or `no` - default is `yes`)
      * `crt` - contains the contents of the SSL public certificate
      * `interm` - contains the contents of the SSL intermediate chain certificate (only needed if using a trusted cert)
      * `key` - contains the contents of the SSL private key

## Tomcat Download URL Note

The default value for the `tomcat_download_url` variable is:

`http://archive.apache.org/dist/tomcat/tomcat-{{ tomcat_major_version }}/v{{ tomcat_version }}/bin/apache-tomcat-{{ tomcat_version }}.tar.gz`

If you are affiliated with UCLA, you have the option of overriding this default url value with:

`http://pkgs.library.ucla.edu/apache-tomcat/apache-tomcat-{{ tomcat_version }}.tar.gz`

Versions of Tomcat available via the UCLA URL are:

* `7.0.67`
* `7.0.92`
* `8.5.34`
* `8.5.35`
* `8.5.37`

## Sample format for defining tomcat web apps:
  ```
  tomcat_applications:
    - app_name: app1
      shut_port: 8006
      conn_port: 8081
      rproxy_path: path1
    - app_name: app2
      shut_port: 8007
      conn_port: 8082
      rproxy_path: path2
      proxypass_extra_params: "nocanon"
  ```

## Example playbook file:
```
  ---
  - name: uclalib_tomcat_param.yml
    sudo: true
    hosts: test
    vars:
      tomcat_applications:
        - app_name: app1
          shut_port: 8006
          conn_port: 8081
          rproxy_path: path1
        - app_name: app2
          shut_port: 8007
          conn_port: 8082
          rproxy_path: path2
          proxypass_extra_params: "nocanon"
      use_lvm: "yes"

    roles:
      - { role: uclalib_role_java }
      - { role: uclalib_role_apache }
      - { role: uclalib_role_tomcat }
```

  The tomcat_applications variable definitions can be placed in an external vars files and included in a playbook using the vars_files statement:
  Example:
  ```
  ---
  - name: uclalib_tomcat_param.yml
    sudo: true
    hosts: test
    vars_files:
      - plays_vars/tomcat_app_vars.yml

    roles:
      - { role: uclalib_role_java }
      - { role: uclalib_role_apache }
      - { role: uclalib_role_tomcat }
  ```
