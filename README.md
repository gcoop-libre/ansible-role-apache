Apache 2.x
==========

An Ansible Role that installs Apache 2.x on RHEL/CentOS, Debian/Ubuntu, SLES and Solaris.

Requirements
------------

The role does not manage the certificate and key files for the sites using SSL/TLS.

If you need Apache with PHP, you can add the PHP packages to the `apache_packages` variable. Or you can use another role, like the `geerlingguy.php` role or `geerlingguy.apache-php-fpm` if you prefer use PHP as FPM instead of an Apache module.

Role Variables
--------------

Available variables are listed below, along with default values (see `defaults/main.yml`):

    apache_enablerepo: ''

The repository to use when installing Apache (only used on RHEL/CentOS systems). If you'd like later versions of Apache than are available in the OS's core repositories, use a repository like EPEL.

    apache_listen_ip: '*'
    apache_listen_port: 80
    apache_listen_port_ssl: 443

The IP address and ports on which apache should be listening. Useful if you have another service (like a reverse proxy) listening on port 80 or 443 and need to change the defaults.

    apache_keepalive: true
    apache_keepalive_requests: 500
    apache_keepalive_timeout: 5

This properties enables HTTP persistent connections and its timeout and max requests can be customized.

    apache_prefork_server_limit: 256
    apache_prefork_start_servers: 5
    apache_prefork_min_spare_servers: 5
    apache_prefork_max_spare_servers: 10
    apache_prefork_max_clients: 256
    apache_prefork_max_requests_per_child: 0

The configuration of the server MPM module. This properties will apply for the MPM Prefork module and the MPM Worker module. Only Apache 2.4 on Debian you can configure the MPM modes with different values, because it has separated files.

    apache_worker_server_limit: 16
    apache_worker_start_servers: 3
    apache_worker_min_spare_threads: 75
    apache_worker_max_spare_threads: 250
    apache_worker_max_clients: 400
    apache_worker_max_requests_per_child: 0
    apache_worker_thread_limit: 64
    apache_worker_threads_per_child: 25

The configuration of the MPM Worker module. This is only available for Apache 2.4 on Debian GNU/Linux.

    apache_event_server_limit: 16
    apache_event_start_servers: 3
    apache_event_min_spare_threads: 75
    apache_event_max_spare_threads: 250
    apache_event_max_clients: 400
    apache_event_max_requests_per_child: 0
    apache_event_thread_limit: 64
    apache_event_threads_per_child: 25

The configuration of the MPM Event module. This is only available for Apache 2.4 on Debian GNU/Linux.

    apache_remove_default_aliases: True

Remove the default aliases that comes with default Apache configuration.

    apache_base_dir: '/var/www'

The base directory where the web sites would be allocated. This value is used with the next three to generate the Document Root for the Virtual Hosts that hasn't an explicit Document Root.

    apache_web_subdir: 'web'
    apache_ssl_subdir: 'ssl'
    apache_private_subdir: 'private'

The subdirectory for the HTTP web files, the one for the HTTPS web files and the subdirectory for htpasswd files. If a Virtual Host has no Document Root, the role generate three directories:

- apache_base_dir/SITENAME/apache_web_subdir
- apache_base_dir/SITENAME/apache_ssl_subdir
- apache_base_dir/SITENAME/apache_private_subdir

    apache_create_vhosts: true
    apache_vhosts_filename: 'vhosts.conf'

If set to true, a global vhosts configuration file and one file per Virtual Host, managed by this role's variables (see below), will be created and placed in the Apache configuration folder. If set to false, you can place your own vhosts files into Apache's configuration folder and skip the convenient (but more basic) ones added by this role.

    apache_remove_default_vhost: false

On Debian/Ubuntu, a default virtualhost is included in Apache's configuration. Set this to `true` to remove that default virtualhost configuration file.

    apache_global_vhost_settings: |
      DirectoryIndex index.php index.html
      # Add other global settings on subsequent lines.

You can add or override global Apache configuration settings in the role-provided vhosts file (assuming `apache_create_vhosts` is true) using this variable. By default it only sets the DirectoryIndex configuration.

    apache_global_vhost_access_log_format_combined: '%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"'
    apache_global_vhost_access_log_format_proxy: '%{X-Forwarded-For}i %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"'

With this properties you can customize the Combined log format and the log format used to log the requests made through a proxy.

    apache_global_vhost_aliases:
      - name: /images
        dest: /var/www/images

List of aliases to add on vhost.conf. There will be available to all the Virtual Hosts. `apache_create_vhosts` should be enabled.

    apache_global_vhost_custom_errors:
      - code: error-code
        action: error-action

Customizations for the Apache error pages. You can read about the `action` values on [Apache documentation](https://httpd.apache.org/docs/2.4/custom-error.html).

    apache_vhosts:
      - servername: 'local.dev'
        serveralias:
          - 'alias1.local'
          - 'alias2.local'
        serveradmin: webmaster@localhost
        documentroot: '/var/www/html'
        separate_logs: true
        separate_logs_proxy_format: true
        custom_errors:
          - code: error-code
            action: error-action
        frame_options: SAMEORIGIN
        deflate: true
        deflate_dont_vary: User-Agent
        fileetag: true
        aliases:
          - name: alias-url
            dest: alias-path
        setenvif:
          - attribute: 'X-Forwarded-For'
            pattern: '(.*)'
            var: 'ENV_VAR'
            value: 'true'
        redirect_to_https: false
        allowoverride: 'All'
        rewritebase: '/'
        custom_rewrites:
          - pattern: regex
            substitution: text
            flags: '[R=301,L]'
            conditions:
              - test_string: '%{HTTP_HOST}'
                pattern: '^old\.site\.com$'
                flags: '[NC]'
        redirect_to_servername: True
        exclude_from_redirect:
          - 'valid.alias.com'
        redirect_to_file: '/index.php'
        restricted_access:
          - path: '/secret'
            regex: false
            all: false
            ips:
              - '127.0.0.1'
              - '192.168.0.1'
            hosts:
              - 'www.site.com'
            env_variables:
              - 'ENV_VAR'
            htpasswd: '/.htpasswd'
        extra_parameters: 'Custom VHost configuration'

Add a set of properties per virtualhost. The only one required is `servername`. If there is no documentroot, it will be generated as described before.

All the request to a ServerAlias will be redirected to the ServerName with an 301 code, except those aliases specified in the `exclude_from_redirect` propierty.

The paths inside `restricted_access` should be relative to the Document Root. If the Virtual Host has no explicit Document Root, the htpasswd file will be in the __apache_private_subdir__ directory.

The `|` denotes a multiline scalar block in YAML, so newlines are preserved in the resulting configuration file output.

    apache_vhosts_ssl: []

No SSL vhosts are configured by default, but you can add them using the same pattern as `apache_vhosts`, with a few additional directives:

    apache_vhosts_ssl:
      - servername: 'local.dev',
        certificate_file: '/path/to/certificate.crt'
        certificate_key_file: '/path/to/certificate.key'
        certificate_chain_file: '/path/to/certificate_chain.crt'
        sts: true
        sts_max_age: '63072000'
        sts_subdomains: true
        redirect_to_http: false

These first three properties set the certificates path. The next three allow the override of the global STS configuration. The last one redirects all the requests to the HTTP host.

The are other SSL directives can be managed with other SSL-related role variables.

    apache_ssl_protocol: 'All -SSLv2 -SSLv3'
    apache_ssl_cipher_suite: 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH'

The SSL protocols and cipher suites that are used/allowed when clients make secure connections to your server. These are secure/sane defaults, but for maximum security, performand, and/or compatibility, you may need to adjust these settings. You may find some information in [Cipherli.st: Strong Ciphers for Apache, nginx and Lighttpd](https://cipherli.st/).

    apache_frame_options: SAMEORIGIN

This property set the security policy of the sites when they are loaded within a Frame or IFrame. You can check the valid options in [this Wikipedia Article](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields#Frame-Options). This global configuration can be overrided vhost by vhost.

    apache_sts: true
    apache_sts_max_age: '63072000'
    apache_sts_subdomains: true

HTTP Strict Transport Security is enabled by default, with a Max Age of 1 year and the subdomains are included. This global configurations can be overrided vhost by vhost.

    apache_mods_enabled:
      - rewrite.load
      - ssl.load
    apache_mods_disabled: []

This properties are for Debian and Ubuntu ONLY. Which Apache mods to enable or disable (these will be symlinked into the appropriate location). See the `mods-available` directory inside the apache configuration directory (`/etc/apache2/mods-available` by default) for all the available mods.

    apache_packages:
      - [platform-specific]

The list of packages to be installed. This defaults to a set of platform-specific packages for RedHat or Debian-based systems (see `vars/RedHat.yml` and `vars/Debian.yml` for the default values).

    apache_state: started

Set initial Apache daemon state to be enforced when this role is run. This should generally remain `started`, but you can set it to `stopped` if you need to fix the Apache config during a playbook run or otherwise would not like Apache started at the time this role is run.

    apache_ignore_missing_ssl_certificate: true

If you would like to only create SSL vhosts when the vhost certificate is present (e.g. when using Let’s Encrypt), set `apache_ignore_missing_ssl_certificate` to `false`. When doing this, you might need to run your playbook more than once so all the vhosts are configured (if another part of the playbook generates the SSL certificates).

Dependencies
------------

None.

Example Playbook
----------------

    - hosts: webservers
      vars_files:
        - vars/main.yml
      roles:
         - gcoop-libre.apache

*Inside `vars/main.yml`*:

    apache_listen_port: 8080
    apache_vhosts:
      - servername: example.com

License
-------

GPLv2

Author Information
------------------

This role was created in 2016 by [gcoop Cooperativa de Software Libre](http://gcoop.coop).
