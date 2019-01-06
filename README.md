[![Build Status](https://travis-ci.org/FlorianKempenich/TOOCS-nodejs.svg?branch=master)](https://travis-ci.org/FlorianKempenich/TOOCS-nodejs) [![Ansible Role](https://img.shields.io/ansible/role/23202.svg)](https://galaxy.ansible.com/FlorianKempenich/toocs_nodejs)


# TOOCS / Ansible Role: `toocs_nodejs`
> #### /!\ This role has been renamed - Old name: `nvm-node-npm` /!\

Install `NodeJs` & `npm` using `nvm` on Ubuntu/Debian and OSX.  
Also provide a utility to enable the use of the `npm` module from `ansible` on a machine with `node` installed via `nvm`: `set-node-path-fact.yml`

> ### TOOCS?
> TOOCS - The Opinionated One-Click Setups are a set of tools / ansible roles designed to setup a system in one click. They are a simple, reliable, way to setup a given tool. You can use them as is, or, inspecting their code, as a tutorial to follow step by step.
> 
> They are, as their name suggests, opinionated: while they guarantee to setup the given tool in one click, they do **not** guarantee consistency in _how_ they achieve it, new releases might introduce breaking changes.  
> Read the code and make sure you understand what's happening!

## How to use the `npm` module from `ansible` when installing `node` via `npm`?

Once `node` is installed with `nvm`, it requires an activation script in the `.bash_profile` / `.zlogin` to be usable.

```
export NVM_DIR="PATH TO YOUR NVM DIR" (default: "$HOME/.nvm")
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

Since `ansible` is running in non-interactive mode, it does not execute the `.bash_profile` / `.zlogin`.
Which means it will throw a `command not found: npm`, when trying to use the `npm` module from `ansible`.

To avoid that, it is possible to explicitely add the path of `node` to `PATH`.
This is usually automatically done when sourcing `nvm.sh` in the `.bash_profile` / `.zlogin`, but needs to be done manually if here.

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
- include_role:
    name: FlorianKempenich.toocs_nodejs
    tasks_from: set-node-path-fact.yml
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

* `nvm_version`: Optional - Default: '0.33.5'
* `node_version`: Optional - Default: 'lts/*' - Can be specific version number Eg. '6.11.4'

Also you can specify where `nvm` should be installed:
* `nvm_dir`: Optional - Default: "{{ ansible_env.HOME }}/.nvm"


## Example Playbook
Basic installation:
```
- hosts: sandbox
  roles:
      - FlorianKempenich.toocs_nodejs
```

With custom versions:
```
- hosts: sandbox
  tasks:
    - include_role:
        name: FlorianKempenich.toocs_nodejs
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

    - include_role:
        name: FlorianKempenich.toocs_nodejs
        tasks_from: set-node-path-fact.yml
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
