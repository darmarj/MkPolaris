# Playbook Organization - Roles, Includes, and Imports
## Imports
Tasks can easily be included in a similar way. In the `tasks:` section of your playbook, add `import_tasks` directives like so:
```yaml
tasks:
  - import_tasks: import_tasks.yml
```
Just like with variable include files, tasks are formatted in a flat list in the included file. `import_tasks.yml` as the example:
```yaml
---
- name: Add profile info for user.
  copy:
    src: example_profile
    dest: "/home/{{ usrname }}/.profile"
    owner: "{ username }"
    group: "{ username }"
    mode: 0744

- name: Add private keys for user.
  copy:
    src: "{{ item.src }}"
    dest: "/home/{{ username }}/.ssh/{{ item.dest }}"
    owner: "{ username }"
    group: "{ username }"
    mode: 0600
  with_items: "{{ ssh_private_keys }}"

- name: Restart example service.
  service:
    name: example
    state: restarted
```

## Includes
`import_tasks` **_statically_** imports the task file as if it were part of the main playbook, once, before the play is executed.

`include_tasks` **_dynamic_** to do different things depending on how the rest of the playbook runs.

!!! abstract "case"
    ```yaml
    ---
    - name: Check for existing log files in dynamic log_file_paths variable
      find:
        paths: "{{ item }}"
        patterns: '*.log'
      register: found_log_file_paths
      with_items: "{{ log_file_paths }}"
    ```
    the `log_file_paths` variable is set by a task earlier in playbook, so this include file would't be able to know the value of that variable until the playbook has already partly completed.

## Dynamic includes
On Ansible 2.0 and later version, `includes` has been evaluated during playbook exeuction.
!!! abstract "case"
    ```yaml
    # Include extra tasks file, only if it's present at runtime
    - name: Check if extra_tasks.yml is present
      stat:
        path: tasks/extra_tasks.yml
      register: extra_tasks_file
      connection: local

    - include_tasks: tasks/extra_tasks.yml
      when: extra_tasks_file.stat.exists
    ```
    If the file `tasks/extra_tasks_file.yml` is not present, Ansible skips the `include_tasks`. Even use a `with_items` loop (or any other `with_*` loop) with includes.

## Handler imports and includes
Handlers can be imported or included just like tasks, within a playbook's `handlers` section.
!!! tip
    ```yaml
    handlers:
      - import_tasks: handlers.yml
    ```
    This can be helpful in limiting the noise in main playbook, since handlers are usually used for things like restarting services or loading a configuration, and can distract from the playbook's primary purpose.

## Playbook imports
If have two playbooks - one to setup webserver(web.yml), and one to setup database server(db.yml), use the following playbokk to run both at the same time:
!!! abstract "case"
    ```yaml
    - hosts: all
      remote_user: root

      tasks:
        [...]

    - import_playbook: web.yml
    - import_playbook: db.yml
    ```
    In infrastructure, create master playbook that includes each of the individual playbooks. When want to initialize the infrastructure, make changes across entire fleet of servers, or check to make sure the configuration matches playbook definition.

## Complete includes example

