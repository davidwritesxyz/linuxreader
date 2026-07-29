---
title: 'Using when to Run Tasks Conditionally'
summary: 'Using when to Run Tasks Conditionally'
showDate: false
---

## Using when to Run Tasks Conditionally

- Use a **when** statement to run tasks conditionally.
- you can test whether:
	- a variable has a specific value
	- whether a file exists
	- whether a minimal amount of memory is available
	- etc.

#### Working with when 

Install the right software package for the Apache web server, based on the Linux distribution that was found in the Ansible facts. Notice that 
- when used in **when** statements, the variable that is evaluated is not placed between double curly braces.

```
    ---
    - name: conditional install
      hosts: all
      tasks:
      - name: install apache on Red Hat and family
        yum:
          name: httpd
          state: latest
        when: ansible_facts[’os_family’] == "RedHat"
      - name: install apache on Ubuntu and family
        apt:
          name: apache2
          state: latest
        when: ansible_facts[’os_family’] == "Debian"
```

- not a part of any properties of the modules on which it is used
- must be indented at the same level as the module itself. 

- For a string test, the string itself must be between double quotes.
- Without the double quotes, it would be considered an integer test. 

#### Using Conditional Test Statements

Common conditional tests that you can perform with the **when** statement:

Variable exists
- variable is defined
Variable does not exist
- variable is not defined
First variable is present in list mentioned as second
- ansible_distribution in distributions
Variable is true, 1 or yes
- variable
Variable is false, 0 or no
- not variable
Equal (string)
- key == "value"
Equal (numeric)
- key == value
Less than
- key < value
Less than or equal to
- key <= value
Greater than 
- key > value
Greater than or equal to
- key >= value
Not equal to
- key != value

- Look for "Tests" in the Ansible documentation, and use the item that is found in Templating (Jinja2). 
https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_tests.html#tests

- When referring to variables in **when** statements, you don't have to use curly brackets because items in a **when** statement are considered to be variables by default. 
- So you can write **when: text == "hello"** instead of **when: "{{ text }}" == "hello"**.

There are roughly four types of **when** conditional tests:
• Checks related to variable existence
• Boolean checks
• String comparisons
• Integer comparisons

The first type of test checks whether a variable exists or is a part of another variable, such as a list. 

Checks for the existence of a specific disk device, using **variable is defined** and **variable is not defined**. All failing tests result in the message "skipping."
```yaml
    ---
    - name: check for existence of devices
      hosts: all
      tasks:
      - name: check if /dev/sda exists
        debug:
          msg: a disk device /dev/sda exists
        when: ansible_facts[’devices’][’sda’] is defined
      - name: check if /dev/sdb exists
        debug:
          msg: a disk device /dev/sdb exists
        when: ansible_facts[’devices’][’sdb’] is defined
      - name: dummy test, intended to fail
        debug:
          msg: failing
        when: dummy is defined
      - name: check if /dev/sdc does not exist
        debug:
          msg: there is no /dev/sdc device
        when: ansible_facts[’devices’][’sdc’] is not defined
```

**[match](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/match_test.html)**
- Compare string against regular expression. Regex is anchored to the start of the string.

Match uids starting with 1:
`item.uid | string is match('1'))`

**search**
`url is search("users/.*/resources/.*")`

**regex**
`url is regex("example\.com/\w+/foo")`
### Lab: Check that finds whether the first variable value is present in the second variable's list. 
- executes the **debug** task if the variable **my_answer** is in **supported_packages**. 
- **vars_prompt** is used. This stops the playbook, asks the user for input, and stores the input in a variable with the name my_answer.
```yml
    ---
    - name: test if variable is in another variables list
      hosts: all
      vars_prompt:
      - name: my_answer
        prompt: which package do you want to install
      vars:
        supported_packages:
        - httpd
        - nginx
      tasks:
      - name: something
        debug:
          msg: you are trying to install a supported package
        when: my_answer in supported_packages
```

**Boolean check**
- Works on variables that have a Boolean value (not very common)
- Should not be defined with the check for existence. 
- Used to check whether a variable is defined.

**string comparisons and integer comparisons**
- Ie: Check if more than 1 GB of disk space is available. 
- When doing checks on available disk space and available memory, carefully look at the expected value. 
- Memory is shown in megabytes, by default, whereas disk space is expressed in bytes.

### Lab: integer check, install vsftpd if more than 50 MB of memory is available.
```yml
    ---
    - name: conditionals test
      hosts: all
      tasks:
      - name: install vsftpd if sufficient memory available
        package:
          name: vsftpd
          state: latest
        when: ansible_facts[’memory_mb’][’real’][’free’] > 50
```

#### Testing Multiple Conditions

- **when** statements can also be used to evaluate multiple conditions. 
- To do so, you can group the conditions with parentheses and combine them with **and** and **or** keywords.
- **and** runs if both conditionals are ture
- **or** runs if one of the conditions are true
### Lab: **and** is used and  runs the task only if both conditions are true.
```yml
    ---
    - name: testing multiple conditions
      hosts: all
      tasks:
      - name: showing output
        debug:
          msg: using CentOS 8.1
        when: ansible_facts[’distribution_version’] == "8.1" and ansible_facts[’distribution’] == "CentOS"
```

- You can make more complex statements by grouping conditions together in parentheses. 
- Group the **when** statement starts with a \> sign to wrap the statement over the next five lines for readability. 

### Lab: Combining complex statements
```yml
    ---
    - name: using multiple conditions
      hosts: all
      tasks:
      - package:
          name: httpd
          state: removed
        when: >
          ( ansible_facts[’distribution’] == "RedHat" and
            ansible_facts[’memfree_mb’] < 512 )
          or
          ( ansible_facts[’distribution’] == "CentOS" and
            ansible_facts[’memfree_mb’] < 256 )
```

#### Combining loop and when

### Lab: Combining loop and when, Perform a kernel update only if /boot is on a dedicated mount point and at least 200 MBis available in the mount.

```yml
    ---
    - name: conditionals test
      hosts: all
      tasks:
      - name: update the kernel if sufficient space is available in /boot
        package:
          name: kernel
          state: latest
        loop: "{{ ansible_facts[’mounts’] }}"
        when: item.mount == "/boot" and item.size_available > 200000000
```

#### Combining loop and register

### Lab: Combining **register** and **loop**

```yml
    ---
    - name: test register
      hosts: all
      tasks:
        - shell: cat /etc/passwd
          register: passwd_contents
        - debug:
            msg: passwd contains user lisa
          when: passwd_contents.stdout.find('lisa') != -1
```

**passwd_contents.stdout.find**, 
- **passwd_contents.stdout** does not contain any item with the name **find**. 
- Construction that is used here is **variable.find**, which enables a task to search a specific string in a variable. (the**find** function in Python is used)
- When the Python **find** function does not find a string, it returns a value of −1. 
- If the requested string is found, the **find** function returns an integer that returns the position where the string was found. 
- For instance, if the string lisa is found in /etc/passwd, it returns an unexpected value like 2604, which is the position in the file, expressed as a byte offset from the beginning, where the string is found for the first time.
- Because of the behavior of the Python **find** function, **variable.find** needs *not* to be equal to −1 to have the task succeed. So don't write **passwd_contents.stdout.find('lisa') = 0** (because it is not a Boolean), but instead write **passwd_contents.stdout.find('lisa') != -1**. 

### Lab: Practice working with conditionals using **register**.

When using **register**, you might want to define a task that runs a command that will fail, just to capture the return code of that command, after which the playbook should continue. If that is the case, you must ensure that **ignore_errors: yes** is used in the task definition. 

1\. Use your editor to create a new file with the name exercise72.yaml. Start writing the play header as follows:

``` pre1
---
- name: restart sshd service if httpd is running
  hosts: ansible1
  tasks:
```

2\. Add the first task, which checks whether the httpd service is running, using command output that will be registered. Notice the use of **ignore_errors: yes**. This line makes sure that if the service is *not* running, the play is still executed further.

``` pre1
---
- name: restart sshd service if httpd is running
  hosts: ansible1
  tasks:
  - name: get httpd service status
    command: systemctl is-active httpd
    ignore_errors: yes
    register: result
```

3\. Add a debug task that shows the output of the command so that you can analyze what is currently in the registered variable:

``` pre1
---
- name: restart sshd service if httpd is running
  hosts: ansible1
  tasks:
  - name: get httpd service status
    command: systemctl is-active httpd
    ignore_errors: yes
    register: result
  - name: show result variable contents
    debug:
      msg: printing contents of the registered variable {{ result }}
```

4\. Complete the playbook by including the **service** task, which is started only if the value stored in **result.rc** (which is the return code of the command that was registered) contains a 0. This is the case if the previous command executed successfully.

``` pre1
---
- name: restart sshd service if httpd is running
  hosts: ansible1
  tasks:
  - name: get httpd service status
    command: systemctl is-active httpd
    ignore_errors: yes
    register: result
  - name: show result variable contents
    debug:
      msg: printing contents of the registered variable {{ result }}
  - name: restart sshd service
    service:
      name: sshd
      state: restarted
    when: result.rc == 0
```

5\. Use an ad hoc command to make sure the httpd service is installed: `ansible ansible1 -m yum -a "name=httpd state=latest"`.

6\. Use an ad hoc command to make sure the httpd service is stopped: `ansible ansible1 -m service -a "name=httpd state=stopped"`.

7\. Run the playbook using `ansible-playbook exercise72.yaml` and analyze the result. You should see that the playbook skips the **service** task.

8\. Type `ansible ansible1 -m service -a "name=httpd state=started"` and run the playbook again, using `ansible-playbook exercise72.yaml`. Playbook execution at this point should be successful.

