[![Build Status](https://travis-ci.org/FlorianKempenich/ansible-role-docker.svg?branch=master)](https://travis-ci.org/FlorianKempenich/ansible-role-docker) [![Ansible Role](https://img.shields.io/ansible/role/22817.svg)](https://galaxy.ansible.com/FlorianKempenich/docker)

# Ansible role: `nvm-node-npm`
Install `NodeJs` & `npm` using `nvm` on Ubuntu/Debian and OSX.  
Also provide a utility to enable the use of the `npm` module from `ansible` on a machine with `node` installed via `nvm`: `set-node-path-fact.yml`

## How to use the `npm` module from `ansible` when installing `node` via `npm`?

Once `node` is installed with `nvm`, it requires an activation script in the `.bashrc` / `.zshrc` to be usable.

```
export NVM_DIR="PATH TO YOUR NVM DIR" (default: "$HOME/.nvm")
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

Since `ansible` is running in non-interactive mode, it does not execute the `.bashrc` / `.zshrc`.
Which means it will throw a `command not found: npm`, when trying to use the `npm` module from `ansible`.

To avoid that, it is possible to explicitely add the path of `node` to `PATH`.
This is usually automatically done when sourcing `nvm.sh` in the `.bashrc` / `.zshrc`, but needs to be done manually if here.

```
- npm:
    name: PACKAGE_TO_INSTALL
    state: latest
    global: yes
  become: no
  environment:
    PATH: "PATH_TO_NODE_EXECUTABLE:{{ ansible_env.PATH }}"
```

This will work.
Do not forget to append the content of the original `PATH` variable: ``{{ ansible_env.PATH }}`

**The only question that remains is, how to get the `PATH_TO_NODE_EXECUTABLE`.**  
This is where the `set-node-path-fact.yml` becomes useful. 
Running this list of tasks will set the path of the `node` executable to an **ansible fact**: `node_path`.
Making it accessible for the rest of the Playbook.

Therefore, to use the `npm` module, the complete sequence is:

```
- include_tasks: set-node-path-fact.yml
                     ^
                     ^--- This sets the `node_path` fact.

...
...
...


- npm:
    name: PACKAGE_TO_INSTALL
    state: latest
    global: yes
  become: no
  environment:
    PATH: "{{ node_path }}:{{ ansible_env.PATH }}"
               ^
               ^--- This is set by the `set-node-path-fact.yml`

```

## Requirements
This role is only working on Ubuntu/Debian & OSX.
No other requirements.

## Role Variables
This playbook will install the latest version `NodeJs` using `nvm`.

You can specify which version of `nvm` to use, as well as which version of `node` to install and set as default.

* `nvm_version`: Optional - Eg. '0.33.5'
* `node_version`: Optional - Eg. '6.11.4'

Also you can specify where `nvm` should be installed:
* `nvm_dir`: Optional - Default: "{{ ansible_env.HOME }}/.nvm"


## Example Playbook
Basic installation:
```
- hosts: sandbox
  roles:
      - FlorianKempenich.nvm-node-npm
```

With custom versions:
```
- hosts: sandbox
  tasks:
    - include_role:
        name: FlorianKempenich.nvm-node-npm
      vars:
        nvm_version: '0.33.5'
        node_version: '6.11.4'
        nvm_dir: "{{ ansible_env.HOME }}/.nvm"
```

---

To use the `npm` module from `ansible` later on:
```
- hosts: sandbox
  tasks:
    - include_tasks: set-node-path-fact.yml
                        ^
                        ^--- This sets the `node_path` fact.

    ...
    ...
    ...


    - npm:
        name: PACKAGE_TO_INSTALL
        state: latest
        global: yes
      become: no
      environment:
        PATH: "{{ node_path }}:{{ ansible_env.PATH }}"
                  ^
                  ^--- This is set by the `set-node-path-fact.yml`
```

## License
MIT

## Author Information
Find out more about my work: [Florian Kempenich](https://floriankempenich.com)
