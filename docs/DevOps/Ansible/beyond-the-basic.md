# Beyond the basic
Cover how to run plays with more granularity, how to organize tasks and playbooks for simplicity and usability, and other advanced playbook topics that will help to manage infrastructure with even more confidence.

## Handlers
### In some circumstances, notify multiple handlers, or even have handlers notify additional handlers are necessary.
Use a list for the `notify` option:
```yaml
- name: Rebuild application configuration.
  command: /opt/app/rebuild.sh
  notify:
    - restart apache
    - restart memcached
```

### To have one handlers notify another
Add a `notify` option onto the handlers - handlers are basically glorified tasks that can be called by the `notify` option, but since they act as tasks themselves, they can chain themselves to other handlers:
```yaml
handlers:
  - name: restart apache
    service: name=apache2 state=started
    notify: restart memcached

  - name: restart memcached
    service: name=memcached state=restarted
```

!!! warning
    *   Handlers will only be **run** if a task notifies the handlers; if a task that would've notified the handlers is skpped due to a `**when**` condition or something of the like, the handlers will not be run.
    *   Handlers will run <span class="rouge">once</span>, and only once, at the end of a play. If you absolutely need to <span class="rouge">override</span> this behavior and run handlers in the middle of a playbook, you can use the <span class="jade">meta</span> module to do so.
        ```yaml
            - meta: flush_handlers
        ```
    *   If the play `fails` on a particular host(or all hosts) before handlers are notified, the handlers will never be run. If it's desirable to always run handlers, even after the playbook has `failed`, you can use the `meta` module as describled above as a seperate task in the playbook, or you use the command line flag `--force-handlers` when running your playbook. Handlers won't run on any hosts that became unreachable during the playbook's run.

## Enviornment variables
### Set some environment variables for your remote user account
Do by adding lines to the remote user's `.bash_profile`.
```yaml
    - name: Add an environment variable to the remote user's shell.
      lineinfile:
        dest: ~/.bash_profile
        regexp: '^ENV_VAR='
        line: "ENV_VAR=value"
```

### To use an environment variable in further Per-tasks
It's **recommended** to use a task's `register` option to store the environment variable in a variable Ansible can use later, for example:
```yaml hl_lines="8"
    - name: Add an environment variable to the remote user's shell
      lineinfile:
        dest: ~/.bash_profile
        regexp: '^ENV_VAR='
        line: "ENV_VAR=value"

    - name: Get the value of the environment variable just added.
      shell: 'source ~/.bash_profile && echo $ENV_VAR'
      register: foo

    - name: Print the value of the environment variable.
      debug:
        msg: "The variable is {{ foo.stdout }}"
```

!!! bug
    In some situations, the tasks all run over a _persisitent or quasi-cached SSH session_, over which `$ENV_VAR` would't yet be defined.

    Use `source ~/.bash_profile` because to make sure Ansible using the latest environment configuration for the remote user.

!!! question "Why `~/.bash_profile`?"
    There're many different places can store environment variables, including `.bashrc, .profile, and .bash_login` in a user's home folder.

    In some case, since want the environment variable to be available to Ansible, which runs a `pseudo-TTY` shell session, in which case `.bash_profile` is used to configure the environment.

    :material-google-downasaur: [Configuring your login sessions with dot files](http://mywiki.wooledge.org/DotFiles)

Linux will also read **global** environment variables added to `/etc/environment`, so add variable there:
```yaml
    - name: Add a global environment variable.
      lineinfile:
        dest: /etc/environment
        regexp: '^ENV_VAR='
        line: "ENV_VAR=value"
      become: yes
```

!!! tip
    If application requires **many environment variables** (as is the case in many Jave applications), might consider using `copy` or `template` with a local file instead of using `lineinfile` with a large list of itmes.

## Per-tasks environment variables
### Set the environment for just **one** taks.
```yaml
vars:
  proxy_vars:
    http_proxy: http://example-proxy:80/
    https_proxy: https://example-proxy:80/
    [etc...]

tasks:
- name: Download a file, using example-proxy as a proxy.
  get_url:
    url: http://www.example.com/file.tar.org
    dest: ~/Downloads/
  environment: proxy_vars
# In the 'var' section of the playbook (set to 'absent' to remvoe):
proxy_state: present

# In the 'tasks' section of the playbook:
- name: Configure the proxy.
  lineinfile:
    dest: /etc/environment
    regexp: "{{ item.regexp }}"
    line: "{{ item.line }}"
    state: "{{ proxy_state }}"
  with_items:
    - regexp: "^http_proxy="
      line: "http_proxy=http_proxy://example-proxy:80"
    - regexp: "^https_proxy="
      line: "http_proxy=http_proxy://example-proxy:443"
    - regexp: "^https_proxy="
      line: "ftp_proxy=ftp_proxy://example-proxy:80"
```

## Variables
Works with other systems. Usually begin with a letter([A-za-z]), but also start with an underscore(_). It can include any number of letters, underscores, or numbers.

!!! warning
    While it's not explictly stated in Ansible's documentation, <span class="carrot">starting with a variable</span> with an **underscore** (e.g. _my_variable) is uncommon, but sometimes is used as a way of indicating a **'private'** or **'internal use only'** variable.
    For example, maintain <span class="jade">prefix variables</span> with an underscore if they're meant for internal role use only, and should not be override by playbooks using the role.

### Playbook Variables
Variable file can also be imported conditionally.
```yaml
---
- hosts: example

pre_task:
  - include_vars: "{{ item }}"
    with_first_found:
      - "apache_{{ ansible_os_family  }}.yml"
      - "apache_defuault.yml"

tasks:
  - name: Ensure Apache is running.
    service:
      name: "{{ apache_service_name }}"
      state: running
```
!!! note
    Add two files in the same fodler as the example playbook, `apache_RedHat.yml`, and `apache_defuault.yml`.
    Define the variable `apache_service_name: httpd` in the CentOS file, and `apache_service_name: apache2` in the default file.

### Inventory Variables
Define inline
```yaml
# Host-specific variables
[washington]
app1.example.com proxy_state=present
app2.example.com proxy_state=absent
```

Define for the entire group
```yaml
[washington:vars]
cdn_host=washington.static.example.com
api_version=3.0.1
```

For more than a few variables, espcially that apply to more than one or two hosts. Ansible recommends **NOT** storing variables within the inventory.
Instead, use <span class="jade">group_vars</span> and <span class="jade">host_vars</span>YAML files within a specific path, and Ansible will assign them to individual hosts and groups defined in your inventory.
```yaml
# Ansible playbook
---
- hosts: all
  become: true

  vars_files:
    - vars.yml
```
```yaml
# vars.yml
---
foo: bar
baz: qux
```

### Registered Variables
If to determine for the return code, stderr, or stdout to run a later task. Ansible allow to use `register` to store the output of a particular command in a variable at runtime.

### Accessing Variables
Define a list variable:
```yaml
foo_list:
  - one
  - two
  - three
```

Stand Python array syntax
```yaml
foo[0]
```

Convenient filter provided by Jinja
```yaml
foo|first
```

**For larger and more structured arrays(like IP adress), access any part of the array by drilling through the array keys, either using `bracket([])` or `dot(.)`**
```yaml
---
- hosts: all
  # become: true

  vars:
    ip_address: "{{ ansible_enp5s0.ipv4.address }}"
    ip_network: "{{ ansible_enp5s0.ipv4.network }}"

  tasks:
    - debug:
        # var: ip_address
        msg:
          - ip_address is {{ ip_address }}.
          - ip_network is {{ ip_network }}.
```

### Host and Group Variables
Define variables on a per-host or per-group basis within inventory file:
!!! note
    In following case, Ansible will use the group **default** variable <span class="jade">'john'</span> for {{ admin_user }}, but for `host1` and `host2`, the admin users define alongside the hostname will be used.
        ```py hl_lines="7"
        [group]
        host1 admin_user=jane
        host2 admin_user=jack
        host3

        [group:vars]
        admin_user=john
        ```
#### Automatically-loaded `group_vars` and `host_vars`
Ansible will search within the **same director** as your inventory file( or inside `/etc/ansible` if using the default inventory file at `/etc/ansible/hosts`) for two specific dicrecotories: `group_vars` and `host_vars`

### Facts(Vairiables derived from system information)
By default, whenever running an Ansible playbook, firstly it gathers information("factc") about each host in the play.
Facts can be extremely helpful when running playbooks. It can use gathered information like `host IP address, CPU type, disk space, operating system information, and network interface inforamtion` to change when certain tasks are run, or to chagne certain inforamtion used in configuration file.
```bash
ansible [HOST] -m setup
rocky8 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "123.123.123.123"
        ],
        "ansible_all_ipv6_addresses": [
            "fd42:de7d:cge9:c468:6a0d:4b2f:fai6:d76d",
            "re08::527e:4bec:d2g3:e80m"
[...]
```

!!! tip
    If don't need to use facts, and would like to save a few seconds per-host when running playbooks(this can be espcially helpful when running an Ansible playbook angainst dozens or hundreds of servers), you can set `gather_facts: no` in playbook.

#### Local Facts(Facts.d)
Another way of defining host-specific facts is to place a `.fact` file in a special directory on remote hosts `/etc/ansible/facts.d`
```yaml
# /etc/ansible/facts.d/settings.fact
[users]
admin=jane,john
normal=jim
```

```bash
ansible hostname -m setup -a "filter=ansible_local"
rocky8 | SUCCESS => {
    "ansible_facts": {
        "ansible_local": {
            "settings": {
                "users": {
                    "admin": "jane,john",
                    "normal": "jim"
                }
            }
        },
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
```

### Ansible Vault - Keeping secrets secret
Works much like a real-world vault:

*   Take any YAML file would normally have in the playbook <span class="jade"> e.g. a variables file, host vars, group vars, role default vars, or even task includes!</span>
*   Ansible encrypt the vault('closes the door'), using a key(a password that set).
*   Store the key(vault's password) seperately from the playbook in a location only to control or can access.
*   Use the key to let Ansible decrypt the encrypted vault whenever run the playbook.

Let's see how it works in practice.
```yaml
---
- hosts: localhost
  connection: local
  gather_facts: no

  vars_files:
    - vars/api_key.yml

  tasks:
    - name: Echo and API key which was injected into the env.
      shell: echo $API_KEY
      environment:
        API_KEY: "{{ myapp_api_key }}"
      register: echo_result

    - name: Show the result
      debug: var=echo_result.stdout
```

```yaml
# vars/api_key.yml
---
myapp_api_key: "l9bTqfBlbXTQiDaJMqgPJ1VdeFLfId98"
```

To encrypt the file with Ansible Vault.
```bash
ansible-vault encrypt vars/api_key.yml
```

Providing the password at playbook runtime works when running a playbook interactively:
```bash
# Use --ask-vault-pass to supply the vault password at runtime
ansible-playbook main.yml --ask-vault-pass
```

Supply vault passwords via a password file. Set strict permission(e.g.600) so **only** can be read and write the file.
```bash
# Use --vault-password-file to supply the password via file or script.
ansible-playbook main.yml --vault-password-file PATH/OF/PASSWORD/FILE
```
!!! info
    **AES-256 encryption** is extremely secure. it would take billions of billions of years to decrypt a single file, even if all of today's fastest supercomupter cluster were all put to the task 24*7.

### Varible Precedence
Ansible's documentation provides the following ranking:

| <span class="rouge">Parameter</span>  | <span class="jade">Description</span> |
| -------------- | -------------------------------- |
| `--extra-vars`       |  Passed in via the commandline  |
| Task-level vars      |  Task block  |
| Block-level vars     |  All tasks in a block  |
| `[role]/vars/main.yml`  |  Role vars  |
| `include_vars`       |  Module  |
| `set_facts`          |  Module  |
| `register`           |  In a task  |
| `vars_files, vars_prompt, vars`  |  individual play-level  |
| HOST                 |  Host facts  |
| `host-vars`          |  Playbook    |
| `group-vars`          |  Playbook    |
| `host_vars, group_vars, vars`  |  Inventory  |
| `[role]/vars/main.yml`  |  Role default vars  |

:material-google-downasaur: [Official Ansible: Understanding Variable Preceduce](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence)

### If/then/when - Conditions
It's worthwhile to cover a small part of Jinja(the syntax Ansible uses both for tempaltes and for conditionals). Jinja allows the definition of literals like `strings, integers, floats, lists, tuples, dictionaries and bealoons.`

:material-google-downasaur: [Official Jinja: Templates](https://jinja.palletsprojects.com/en/3.1.x/templates/)

#### register
"Register" a variable, and once registered, the variable will be avaiblable to all subsequent tasks.
!!! tip
    If want to see the difference properties of a particular registered variable, run a playbook with <span class="carrot">**-v**</span> to inspect play output.

#### when
`when` is even more powerful if used in conjunction with variables registered by previous tasks.
```yaml
- command: my-app --status
  register: myapp_result

- command: do-something-to-my-app
  when: "'ready' in myapp_result.stdout"
```

####  changed-when and failed_when
If use the `command` or `shell` module when using `changed_when`, Ansible will always report a change. Most modules report whether resulted in changes correctly, but you can override this behavior by invoking `changed_when`.

Example when using PHP Composer as a command to install project dependencies.
```yaml
- name: Install dependencies via Composer
  command: "/usr/local/bin/composer global requrie phpunit/phpunit --prefer-dist"
  register: composer
  changed_when: "'Nothing to install' not in composer.stdout"
```

Example to parse the stderr of a Jenkins CLI command to see if it did, in fact, faild to perform the command that requested:
!!! note
    ```yaml
    - name: Import a Jenkins job via CLI
      shell: >
        java -jar /opt/jenkins-cli.jar -s http://localhost:8000/
        create-job "My job" < /usr/local/my-job.xml
      register: import
      failed_when: "import.stderr and 'exists' not in import.stderr"
    ```
    In this case, only want Ansible to report a failure when the command returns an error, **and** that error doesn't contain 'exists'.
    It's debatable whether the command should report a job already exists via stderr, or just print the result to stdout... but it's easy to account for whatever the command does with Ansible!

#### ingore_errors
When errors don't actually indicate a problem, but it's anonnying. Add `ignore_errors: ture` to the task. Ansible will remain blissfully unaware of any problems running a particular task.

### Deltegation, Local Actions, and Pauses
Ansible allows any task to be delegated to a particular host using `delegate_to:`
```yaml
- name: Add server to Munin monitoring configuration
  command: monitor-server webservers {{ inventory_hostname }}
  deltegate_to: "{{ monitoring_master }}"
```
Delegation is often used to mamage a server's participation in a load balancer or replication pool; might either run particular command locally(as in the example below), or could be used one of Ansible's build-in load balancer modules and `delegate_to` a specific load balancer host directly:
```yaml
- name: Remove server from load balancer
  command: remove-from-lb {{ inventory_hostname }}
  deltegate_to: 127.0.0.1
```
If delegating a task to localhost, Ansible has a convenient shorthand can use, `local_action`, instead of adding the entire `delegate_to` line:
```yaml
- name: Remove server from load balancer
  local_action: command remove-from-lb {{ inventory_hostname }}
```

#### Pausing playbook execution with `wait_for`
You might also use `local_action` in the middle of a playbook to wait for a freshlybooted server or application to start listening on a particular port:
```yaml
- name: Wait for web server to start
  local_action:
    module: wait_for
    host: "{{ inventory_hostname }}"
    port: "{{ webserver_port }}"
    delay: 10
    timeout: 300
    state: started
```

??? note "More things do for `wait_for`"
    *   Using `host` and `port`, wait a maximum of `timeout` seconds of the port to be avaible.
    *   Using `path` (and `search_regex` if desired), wait a maximum of `timeout` seconds for the file to be present(or absent).
    *   Using `host` and `port` and `drained` for the `state` parameter, check if given port has drained all it's active connections.
    *   Using `delay`, can simply pause playbook execution for a given amount of time(in seconds).

#### Running an entire playbook locally
To speed up playbook execution by avoiding the SSH connection overhead, use `--connection=local`
Quick example to run the command `ansible-play test.yml --connection=local:`
```yaml
---
- hosts: 127.0.0.1
  gather_facts: no

  tasks:
    - name: Check the current system date.
      command: date
      register: date

    - name: Print the current system date.
      debug:
        var: date.stdout
```

### Prompts
Under rare circumstances, may require the user to enter the value of a variable that will be used in the playbook.
If the playbook requires a user's personal login information, or if pormpt for a version or other values that may change depending on who is running the playbook, or where it's being run, and if there's no other way this information can be configured (e.g. using environment variables, inventory variables, etc), use `var_prompt`.
```yaml
---
- hosts: all

  vars_prompt:
    - name: share_user
      prompt: "What's your network username?"
    - name: share_pass
      prompt: "What's your network password?"
      private: yes
```
!!! note "a few special options add for prompts"
    *   `private`: if set to `yes`, the user's input will be hidden on the command line.
    *   `defaulAt`: can set a default value for the prompt, to save time for the end user.
    *   `encrypt / confirm / salt_size`: set for password so can verify the entry (the user will have to enter the password twice if confirm is set to yes), and encrypt it using salt (with the specified size and crypt scheme).
!!! warning "avoid unless absoutely necessary"
    It's prefer to use role or playbook variables, inventory variables, or even local environment variables, to maintain complete automation of the playbook run.

### Tags
Tags allow to run (or exclude) subsets of a playbook's tasks.
Tag roles, included files, individual tasks, and even entire plays.
```yaml
---
# Apply tags to an entire play.
- hosts: webservers
  tags: deploy

  roles:
    # Tags applied to a role will applied to tasks in the role
    - role: tomcat
      tags: ['tomcat', 'app']

  tasks:
    - name: Notify on completion
      local_action:
        module: osx_say
        msg: "{{inventory_hostname}} is finished!"
        voice: Zarvox
      tags:
        - notifications
        - say

    - import_tasks: foo.yml
      tags: foo
```

Exclude anything tagged with `notifications`, can use `--skip-tags`.
```bash
ansible-playbook tags.yml --skip-tags "notifications"
```
!!! tip "handy tips"
    Incredibly handy if have a decent tagging structure.
    When only want to run a particular portion of a playbook, or one play in a series.
    There is one caveat when adding one or multiple tags using the `tags` option in a playbook:
    ```yaml
    # Shorthand list syntax
    tags: ['one', 'two', 'three']

    # Explicit list syntax
    tags:
      - one
      - two
      - three

    # Non-working example
    tags: one, two, three
    ```

### Blocks
Group related tasks together and apply particular task parameters on the block level. Also allow to handle errors inside the blocks in a way similar to most programming languages' exception handling.
```yaml
---
- hosts: web
  tasks:
    # Install and configure Apache on RHEL/CentOS hosts
    - block:
        - yum: name=httpd state=present
        - template: src=httpd.conf.j2 dest=/etc/httpd/conf/httpd.conf
        - service: name=httpd state= started enabled=yes
      when: ansible_os_family == 'RedHat'
      become: yes

    # Install and configure Apache on Debian/Ubuntu hosts
    - block:
        - yum: name=apache2 state=present
        - template: src=httpd.conf.j2 dest=/etc/apache2/apache2.conf
        - service: name=apache2 state= started enabled=yes
      when: ansible_os_family == 'Debian'
      become: yes
```
If want to perform a series of tasks with one set of task parameter (e.g. `with_items, when, or become`) applied, blocks are quite handdy.
Blocks are also useful if want to be able to gracefully handle failures in certain tasks.
```yaml
tasks:
  - block:
      - name: Script to connect the app to a monitoring service
        script: monitoring-connect.sh
    rescue:
      - name: This will only run in case of an error in the block
        debug: msg="There was an error in the block"
    always:
      - name: This will always run, no matter what
        debug: msg="This always executes."
```
Blocks can be very helpful for building reliable playbook, but just like exceptions in programming languages, `block/rescue/always` failure handling can over-complicate things.
`failed_when` per-task to define acceptable failure conditions, or to structure playbook in a different way is easier to maintain idempotence, rather than to use `block/rescure/always`.
