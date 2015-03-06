Demo for a bug in ansible
=========================

When using the same handler in two different role inclusions with different
variables results in running the handler different times with the same
variable.

Considering this handler:

    - name: touch myvar
      shell: touch /tmp/{{ myvar }}

And such a role tasks:

    - debug: var=myvar
      changed_when: true
      notify: touch myvar

    - meta: flush_handlers

Used by these roles:

    - hosts: all
      roles:
      - role: .
        myvar: foo

      - role: .
        myvar: bar

You'd expect:

- debug var=foo
- touch /tmp/foo
- debug var=bar
- touch /tmp/bar

But you'll get:

- debug var=foo
- touch /tmp/foo
- debug var=bar
- touch /tmp/**foo**

Full actual output:

    $ vagrant provision
    ==> arch: Configuring proxy environment variables...
    ==> arch: Running provisioner: ansible...
    PYTHONUNBUFFERED=1 ANSIBLE_FORCE_COLOR=true ANSIBLE_HOST_KEY_CHECKING=false ANSIBLE_SSH_ARGS='-o UserKnownHostsFile=/dev/null -o ControlMaster=auto -o ControlPersist=60s' ansible-playbook --private-key=/home/jpic/.vagrant.d/insecure_private_key --user=vagrant --connection=ssh --limit='arch' --inventory-file=/home/jpic/ansible/roles/handlervarbug/.vagrant/provisioners/ansible/inventory --extra-vars={"ansible_python_interpreter":"/usr/bin/python2"} -v Vagrantplaybook.yml

    PLAY [all] ******************************************************************** 

    GATHERING FACTS *************************************************************** 
    ok: [arch]

    TASK: [. | debug var=myvar] *************************************************** 
    changed: [arch] => {
        "changed": true, 
        "myvar": "foo"
    }

    NOTIFIED: [. | touch myvar] *************************************************** 
    changed: [arch] => {"changed": true, "cmd": "touch /tmp/foo", "delta": "0:00:00.003481", "end": "2015-03-07 03:41:13.454568", "rc": 0, "start": "2015-03-07 03:41:13.451087", "stderr": "", "stdout": "", "warnings": ["Consider using file module with state=touch rather than running touch"]}

    TASK: [. | debug var=myvar] *************************************************** 
    changed: [arch] => {
        "changed": true, 
        "myvar": "bar"
    }

    NOTIFIED: [. | touch myvar] *************************************************** 
    changed: [arch] => {"changed": true, "cmd": "touch /tmp/foo", "delta": "0:00:00.003922", "end": "2015-03-07 03:41:13.594109", "rc": 0, "start": "2015-03-07 03:41:13.590187", "stderr": "", "stdout": "", "warnings": ["Consider using file module with state=touch rather than running touch"]}

    PLAY RECAP ******************************************************************** 
    arch                       : ok=5    changed=4    unreachable=0    failed=0   
