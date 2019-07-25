kerberos_principals
===================

This role, in short, allows to deploy kerberos principals and keytabs of users and/or hosts

Requirements
------------

This role does not depend on any extra modules or other roles.


Role Variables
--------------

All the role vars explained in short, including their defaults, as defined in the `defaults/main.yml`
```
# The (path to the) kadmin.local command
kerberos_principal_kadmin_cmd: kadmin.local

# The default directory for the created keytabs (referenced only in the `kerberos_principal_keytab` default value)
kerberos_principal_keytab_dir: /etc/security/keytabs

kerberos_principal_keytab: "{{ kerberos_principal_keytab_dir }}/{{ kerberos_principal_primary }}.user.keytab"

# The principal's 'instance' part. By default it is set to the host's fqdn, but it can be set empty
kerberos_principal_instance: "{{ ansible_fqdn }}"

# The principal's primary part. By default set to the current user (but usually set to the principal's username)
kerberos_principal_primary: "{{ ansible_user }}"

# The kerberos realm:
kerberos_principal_realm: MYREALM.LOCAL

# Flag if enabled lists all the principals after the deployment
kerberos_principal_listprincs: False

# The principals to create, configured as a list of (principal) dicts
# by default set to a list with 1 element to create a principal for the current user
kerberos_principal_principals: 
  - username: "{{ ansible_user }}"
```

Examples
----------------
Example playbook, to deploy various example configs (below). Set the desired config in `kerberos_principal_principals`

```
- hosts: all
  # gather_facts required for the variable "ansible_fqdn" (used by default in var kerberos_princ_host)
  gather_facts: yes
  become: yes
  tasks:
    - name: deploying the kerberos principals and keytabs
      block:
        - import_role:
            name: kerberos_principals
          vars:
            kerberos_principal_listprincs: True
            kerberos_principal_realm: "{{ kerberos_realm }}"
            #kerberos_principal_principals: "{{ (username is defined) | ternary([ {'username': username} ], kerberos_principals | default([]) ) }}"
            kerberos_principal_principals: "{{ _kerberos_principals_host }}"
            #kerberos_principal_principals: "{{ _kerberos_principals_testusers }}"
      tags: kerberos-principals
```

The playbook is based on following example variables (added in the role defaults for convenience):
```
## Optional Example configs for `kerberos_principal_principals`
# Example principals config to create a host principal in default keytab
_kerberos_principals_host:
  - username: "host"
    keytab: "/etc/krb5.keytab"
    keytab_owner: root

# Example Config for testing and showing the additional optional fields in a principal dict
_kerberos_principals_testusers: 
 - username: "testuser1"
 - username: "testuser2"
   keytab:  "~/my.keytab"
   primary: "testuser2"
   instance: "{{ ansible_fqdn }}"
```
