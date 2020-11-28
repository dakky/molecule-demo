`ansible-playbook -i production.ini playbooks/dns-dhcp-server.yaml`

# requirements

`pip install ansible molecule[docker,podman] molecule-goss`
goss und dgoss installieren https://github.com/aelsabbahy/goss/releases


# molecule in bestehende rolle installieren
molecule init scenario default --role-name dnsmasq --driver-name docker --verifier-name goss|ansible

# setup molecule
molecule.yaml anpassen:
```
---
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: ubuntu
    image: geerlingguy/docker-ubuntu2004-ansible
    tmpfs:
      - /run
      - /tmp
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:ro
    capabilities:
      #- SYS_ADMIN
      - ALL
    command: "/lib/systemd/systemd"
    pre_build_image: true
provisioner:
  name: ansible
  config_options:
    defaults:
      interpreter_python: auto_silent
      callback_whitelist: profile_tasks, timer, yaml
    ssh_connection:
      pipelining: false
verifier:
  name: goss
  enabled: True
lint: |
  set -e
  yamllint .
  ansible-lint .
```

# what does molecule?
By default, “molecule test” executes these steps in order:

    Install required dependencies
    Lint the project
    Destroy existing instances
    Run a syntax check
    Create instances
    Prepare instances (if required)
    Converge instances by applying the role tasks
    Check the role for idempotence
    Verify the results using the defined verifier
    Destroy the instances


# Pitfalls

## /etc/hosts
ansible tasks mit /etc/hosts => docker mag das nicht
**fix**:
```   
   tags:
    - molecule-notest
```

## dockercapabilties
je nach use case (zB `setcap()`)ist zB `ALL` nötig => security kritisch => dedizierter test host?