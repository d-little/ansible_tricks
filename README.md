# ansible_tricks
Tricks and Tips I've written down while using Ansible so as not to forget

### Making shell commands idempotent
If you must use command or shell modules you should always make them idempotent  

eg: When modifying root's fsize ulimit, make sure it isnt already set to -1:
```ansible
shell: >
    ( lsuser -c -a fsize root|xargs|egrep -q '.name:fsize root:-1' ) && echo 'nochange' || chuser fsize=-1 root
changed_when: move_files.stdout != 'nochange'
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
