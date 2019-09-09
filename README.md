# ansible_tricks
Tricks and Tips I've written down while using Ansible so as not to forget

### Colon in a When statement
Taken from: https://github.com/ansible/ansible/issues/38133#issuecomment-483230561

eg, this will not work:
```
  - name: Check if some files already installed
    command: lslpp -l some_files*
    register: krb5_output
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
* `    when: cmd_out.stdout == "lslpp:" ~" Fileset some_files* not installed."

So;

```
  - name: Check if some files already installed
    command: lslpp -l some_files*
    register: krb5_output
    failed_when: no
    changed_when: no  # Dont fail, dont change.
  
  - name: Install Some Files
    debug: 
      var: cmd_out
    when: cmd_out.stdout == "lslpp:" ~" Fileset some_files* not installed."
```
