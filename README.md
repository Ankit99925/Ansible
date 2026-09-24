# ansible

Playbooks for configuring machines. Terraform creates things that do not exist;
Ansible configures things that do. Both are needed and they do not overlap.

| Directory | What                                            |
|-----------|-------------------------------------------------|
| `lab/`    | Host bridges and Pi-hole for the home lab       |

## Setup

    sudo apt install ansible
    cd lab
    cp host_vars/localhost.yml.example host_vars/localhost.yml
    cp host_vars/ubuntu-server.yml.example host_vars/ubuntu-server.yml
    # edit both

Run playbooks from inside `lab/`, so that `host_vars/` and `templates/` are
found — Ansible looks for them relative to the inventory and the playbook.

## Local values

`host_vars/*.yml` is gitignored. Each has an `.example` beside it saying what
to set and how to find it.

## Ideas that matter

**Idempotency.** A task that finds things already correct does nothing and says
`changed=0`. Run a playbook twice; the second run should change nothing. That
is not tidiness — it is what makes a playbook safe to re-run on a live system,
because a task that changes nothing does not trigger its handler.

**Handlers run once, at the end.** Several tasks can notify the same handler
and it fires once, after all of them. That matters when the intermediate state
would be invalid.

**Ansible keeps no state.** Removing a task from a playbook does nothing to the
machine. To remove something you have to say so explicitly, with
`state: absent`. This is the opposite of Terraform, where deleting a resource
from the config destroys it.

**Check mode is not a full dry run.** `--check` skips anything it cannot
predict, so a task that depends on an earlier one having run will report a
failure that is not real. Useful, but read its output knowing that.

## Useful commands

    ansible-playbook -i inventory.ini <playbook>.yml --check --diff
    ansible-playbook -i inventory.ini <playbook>.yml
    ansible-playbook -i inventory.ini <playbook>.yml --syntax-check

    ansible -i inventory.ini <group> -m ping
    ansible -i inventory.ini <group> -b -m command -a "<command>"
    ansible-inventory -i inventory.ini --host <host>

The last one prints every variable resolved for a host — the quickest way to
find out why a template rendered the wrong value.

`-K` asks for the sudo password. `-b` means become, that is, run as root.
