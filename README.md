# ansible_tricks
Tricks and Tips I've written down while using Ansible so as not to forget

### Leave Debug Tasks Behind + Leave Behind Commented-out Tasks

Old debug tasks can be invaluable learning experiences!  Leave them in the task lists!

I find it good practice to leave in debug tasks and commented out tasks created during role creation+debugging.  A lot of the time those are there because something didnt work right the first time.  These little debug tasks can be handy for the next person to come along and read the task to figure out what you were trying to do and why you did it in a particular way.


### Making shell commands idempotent
If you must use command or shell modules you should always make them idempotent  

eg: When modifying root's fsize ulimit, make sure it isnt already set to -1:
```ansible
shell: >
    ( lsuser -c -a fsize root|xargs|egrep -q '.name:fsize root:-1' ) && echo 'nochange' || chuser fsize=-1 root
register: chuser_out
changed_when: chuser_out.stdout != 'nochange'
```

### Colon in a When statement
Taken from: https://github.com/ansible/ansible/issues/38133#issuecomment-483230561

This will not work: `when: cmd_out.stdout == "lslpp: Fileset some_files* not installed."`

eg:
```ansible
  - name: Check if some files already installed
    command: lslpp -l some_files*
    register: cmd_out
    failed_when: no
    changed_when: no  # Dont fail, dont change.
  
  - name: Install Some Files
    debug: 
      var: cmd_out
    when: cmd_out.stdout == "lslpp: Fileset some_files* not installed."
```

This will generate both a YAML Linting error and an Ansible error along the lines of:
* YAML Lint: `incomplete explicit mapping pair; a key node is missed`
* Ansible: `The error was: template error while templating string: unexpected ':'.`

You cannot do a simple `{{':'}}` replacement because Ansible will again complain:
* ` [WARNING]: conditional statements should not include jinja2 templating delimiters such as {{ }} or {% %}. Found: cmd_out.stdout == "lslpp: Fileset some_files* not installed."`
 
You need to concatinate the string using `~` in the when statement, this passes all checks:
* `when: cmd_out.stdout == "lslpp:" ~" Fileset some_files* not installed."

So;

```ansible
  - name: Check if some files already installed
    command: lslpp -l some_files*
    register: cmd_out
    failed_when: no
    changed_when: no  # Dont fail, dont change.
  
  - name: Install Some Files
    debug: 
      var: cmd_out
    when: cmd_out.stdout == "lslpp:" ~" Fileset some_files* not installed."
```

### Merge multiple lists 
```ansible
- name: merge every var that starts with 'start_of_list*'
  set_fact:
    lists_merged: "{{ lists_merged | union(vars[item]) }}"
  loop: "{{ lookup('varnames', 'start_of_list.+').split(',') }}"
```
