---
title: 'Handlers'
description: 'Ansible handlers for running tasks later'
showDate: false
---

## Using Handlers 

- A task that is triggered and is executed by a successful task.

#### Working with Handlers

- Define a **notify** statement at the level where the task is defined. 
- The **notify** statement should list the name of the handler that is to be executed
- Handlers are listed at the end of the play. 
- Make sure the name of the handler matches the name of the item that is called in the **notify** statement, because that is what the handler is looking for.
- Handlers can be specified as a list, so one task can call multiple handlers.
### Lab
- Define the file index.html on localhost. Use this file in the second play to set up the web server.
- The handler is triggered from the task where the copy module is used to copy the index.html file. 
- If this task is successful, the **notify** statement calls the handler. 
- A second task is defined, which is intended to fail.

```yml
    ---
    - name: create file on localhost
      hosts: localhost
      tasks:
      - name: create index.html on localhost
        copy:
          content: "welcome to the webserver"
          dest: /tmp/index.html
    
    - name: set up web server
      hosts: all
      tasks:
        - name: install httpd
          yum:
            name: httpd
            state: latest
        - name: copy index.html
          copy:
            src: /tmp/index.html
            dest: /var/www/html/index.html
          notify:
            - restart_web
        - name: copy nothing - intended to fail
          copy:
            src: /tmp/nothing
            dest: /var/www/html/nothing.html
      handlers:
        - name: restart_web
          service:
            name: httpd
            state: restarted
```

- All tasks up to **copy index.html** run successfully. However, the task **copy nothing** fails, which is why the handler does not run. The solution seems easy: the handler doesn't run because the task that copies the file /tmp/nothing fails as the source file doesn't exist.
- Create the source file using `touch /tmp/nothing` on the control host and run the task again.

- After creating the source file and running the playbook again, the handler still doesn't run. 
- Handlers run only if the task that triggers them gives a **changed** status.

Run an ad hoc command to remove the /var/www/html/index.html file on the managed hosts and run the playbook again:
`ansible ansible2 -m file -a "name=/var/www/html/index.html state=absent"`
 
Run the playbook again and you'll see the handler runs.

### Understanding Handler Execution and Exceptions

When a task fails, none of the following tasks run. How does that make handlers different? A handler runs only on the success of a task, but the next task in the list also runs only if the previous task was successful. What, then, is so special about handlers?

The difference is in the nature of the handler. 
- Handlers are meant to perform an extra action when a task makes a change to a host. 
- Handler should be considered an extension to the regular task. 
- A conditional task that runs only upon the success of a previous task.

Two methods to get Handlers to run even if a subsequent task fails:

**force_handlers: true** (More specific and preferred)
- Used in the play header to ensure that the handler will run even if a task fails.

**ignore_errors: true** 
- Used in the play header to accomplish the same thing. 

• Handlers are specified in a handlers section at the end of the play.
• Handlers run in the order they occur in the handlers section and not in the order as triggered.
• Handlers run only if the task calling them generates a changed status.
• Handlers by default will not run if any task in the same play fails, unless **force_handlers** or **ignore_errors** are used.
• Handlers run only after *all* tasks in the play where the handler is activated have been processed. You might want to define multiple plays to avoid this behavior.

### Lab: Working with Handlers

1\. Open a playbook with the name exercise73.yaml.

2\. Define the play header:
``` yml
---
- name: update the kernel
  hosts: all
  force_handlers: true
  tasks:
```

3\. Add a task that updates the current kernel:
``` yml
---
- name: update the kernel
  hosts: all
  force_handlers: true
  tasks:
  - name: update kernel
    yum:
      name: kernel
      state: latest
    notify: reboot_server
```

4\. Add a handler that reboots the server in case the kernel was successfully updated:
```yml
---
- name: update the kernel
  hosts: all
  force_handlers: true
  tasks:
  - name: update kernel
    yum:
      name: kernel
      state: latest
    notify: reboot_server
  handlers:
  - name: reboot_server
    command: reboot
```

5\. Run the playbook using **ansible-playbook exercise73.yaml** and observe its result. Notice that the handler runs only if the kernel was updated. If the kernel already was at the latest version, nothing has changed and the handler does not run. Also notice that it wasn't really necessary to use **force_handlers** in the play header, but by using it anyway, at least you now know where to use it.


### Dealing with Failures

#### Understanding Task Execution

- Tasks in Ansible playbooks are executed in the order they are specified. 
- If a task in the playbook fails to execute on a host, the task generates an error and the play does not further execute on that specific host. 
- This also goes for handlers: if any task that follows the task that triggers a handler fails, the handlers do not run. 
- In both of these cases, it is important to know that the tasks that have run successfully still generate their result. Because this can give an unexpected result, it is important to always restore the original situation if that happens.

**any_errors_fatal** 
- Used in the play header or on a block.
- Stop executing on all hosts when a failing task is encountered

#### Managing Task Errors

Generically, tasks can generate three different types of results. 
ok
- The tasks has run successfully but no changes were applied
changed
- The task has run successfully and changes have been applied
failed
- While running the task, a failure condition was encountered

**ignore_errors: yes** 
- Keep running the playbook even if a task fails

**force_handlers**. If
- can be used to ensure that handlers will be executed, even if a failing task
was encountered. 

### Lab: **ignore_errors**

```yml
    ---
    - name: restart sshd only if crond is running
      hosts: all
      tasks:
        - name: get the crond server status
          command: /usr/bin/systemctl is-active crond
          ignore_errors: yes
          register: result
        - name: restart sshd based on crond status
          service:
            name: sshd
            state: restarted
          when: result.rc == 0
```


### Lab: Forcing Handlers to Run

```yaml
    ---
    - name: create file on localhost
      hosts: localhost
      tasks:
      - name: create index.html on localhost
        copy:
          content: "welcome to the webserver"
          dest: /tmp/index.html
    
    - name: set up web server
      hosts: all
      force_handlers: yes
      tasks:
        - name: install httpd
          yum:
            name: httpd
            state: latest
        - name: copy index.html
          copy:
            src: /tmp/index.html
            dest: /var/www/html/index.html
          notify:
            - restart_web
        - name: copy nothing - intended to fail
          copy:
            src: /tmp/nothing
            dest: /var/www/html/nothing.html
      handlers:
        - name: restart_web
          service:
            name: httpd
            state: restarted
```

#### Specifying Task Failure Conditions

**failed_when** 
- conditional used to evaluate some expression.
- Set a failure condition on a task
### Lab: failed_when

```yml
    ---
    - name: demonstrating failed_when
      hosts: all
      tasks:
      - name: run a script
        command: echo hello world
        ignore_errors: yes
        register: command_result
        failed_when: "’world’ in command_result.stdout"
      - name: see if we get here
        debug:
          msg: second task executed
```

**fail** module
- specify when a task fails.
- Using this module makes sense only if **when** is used to define the exact condition when a failure should occur. 

### Lab: Using the fail Module

```yaml
    ---
    - name: demonstrating the fail module
      hosts: all
      ignore_errors: yes
      tasks:
      - name: run a script
        command: echo hello world
        register: command_result
      - name: report a failure
        fail:
          msg: the command has failed
        when: "’world’ in command_result.stdout"
      - name: see if we get here
        debug:
          msg: second task executed
```

- The **ignore_errors** statement has movedfrom the task definition to the play  header. 
- Without this move, the message "second task executed" would never be  shown because the fail module always generates a failure message.
- The main advantage of using the fail module instead of using **failed_when** is that the fail module can easily be used to set a clear failure message, which is not possible when using **failed_when**.

#### Managing Changed Status

In Ansible, there are commands that change something and commands that don't. Some commands, however, are not very obvious in reporting their
status. 

### Lab:  Change status

```yml
    ---
    - name: demonstrate changed status
      hosts: all
      tasks:
      - name: check local time
        command: date
        register: command_result
    
      - name: print local time
        debug:
          var: command_result.stdout
```

- Reports a changed status, even if nothing really was changed!

- Managing the changed status can be useful in avoiding unexpected results while running a playbook.  

**changed_when**
- If you set **changed_when** to false, the playbook reports only an ok or failed status and never reports a changed status.

### Lab: Using **changed_when**
```yml
---
- name: demonstrate changed status
  hosts: all
  tasks:
  - name: check local time
    command: date
    register: command_result
    changed_when: false
    
  - name: print local time
    debug:
      var: command_result.stdout
```

#### Using Blocks

- Useful when working with conditional statements. 
- A group of tasks to which a **when** statement can be applied. 
- As a result, if a single condition is true, multiple tasks can be executed. 
- To do so, between the **tasks:** statement in the play header and the actual tasks that run the specific modules, you can insert a **block:** statement. 

### Lab: Using Blocks

```yml
---
- name: simple block example
  hosts: all
  tasks:
  - name: setting up http
    block:
    - name: installing http
      yum:
        name: httpd
        state: present
    - name: restart httpod
      service:
        name: httpd
        state: started
    when: ansible_distribution == "CentOS"

```

- The **when** statement is applied at the same level as the **block** definition. 
- When you define it this way, the tasks in the block are executed only if the **when** statement is true.

#### Using Blocks with rescue and always Statements

- Blocks can be used for simple error handling as well, in such a way that if any task that is defined in the **block** statement fails, the tasks that are defined in the **rescue** section are executed. 
- Besides that, an **always** section can be used to define tasks that should always run, regardless of the success or failure of the tasks in the block. 

### Lab: Using Blocks, **rescue**, and **always**

```yml
- name: using blocks
  hosts: all
  tasks: 
  - name: intended to be successful
    block:
    - name: remove a file
      shell:
        cmd: rm /var/www/html/index.html
    - name: printing status
      debug:
        msg: block task was operated
    rescue:
    - name: create a file
      shell:
        cmd: touch /tmp/rescuefile
    - name: printing rescue status
      debug:
        msg: rescue task was operated
    always:
    - name: always write a message to logs
      shell:
        cmd: logger hello
    - name: always printing this message
      debug:
        msg: this message is always printed
```

- Run this twice to see the rescue. (The file is already created so a task in the block fails)

**command_warnings=False** 
- Setting in ansible.cfg to avoid seeing command module warning message. 

- you cannot use a block on a loop. 
- If you need to iterate over a list of values, think of using a different solution.

## More Handlers, testing, and blocks

A handler won't run if a task in the playbook fails. 

Use **force_handlers** to make handlers run even if a tasks fails. **ignore_errors** can also accomplish this.

Handlers run after the play is finished. If you have multiple plays, the handlers for the first play will run before the second play begins. 

Useful doc to search: [Error handling in playbooks](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_error_handling.html)

The **fail** module also exists and let's you specify a clear failure message:  
```
ansisible-doc fail
```

A good blocks [document](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_blocks.html#block-error-handling) exists and covers blocks, rescue, and always. 
