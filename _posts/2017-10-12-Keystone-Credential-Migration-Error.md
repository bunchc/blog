---
title: "Keystone Credential Migration Error"
date: 2017-10-12
layout: "post"
categories: keystone, openstack, openstack-ansible, troubleshooting
---

# Credential migration in progress. Cannot perform writes to credential table.

In openstack-ansible versions 15.1.7 and 15.1.8, there is an issue with the version of shade and the keystone db_sync steps not completing properly. This is fixed in 15.1.9, however, if running one afore mentioned releases, the following may help.

# Symptom:

Keystone reports 500 error when attempting to operate on the credential table.

You will find something similar to this in the keystone.log file

```
./keystone.log:2017-10-04 18:54:43.978 13170 ERROR keystone.common.wsgi [req-19551bfb-c4d5-4582-adc0-6edcbe7585a5 84f7baa50ec34454bdb5d6a2254278b3 98186b853beb47a8bcf94cc7f179bf76 - default default] (pymysql.err.InternalError) (1644, u'Credential migration in progress. Cannot perform writes to credential table.') [SQL: u'INSERT INTO credential (id, user_id, project_id, encrypted_blob, type, key_hash, extra) VALUES (%(id)s, %(user_id)s, %(project_id)s, %(encrypted_blob)s, %(type)s, %(key_hash)s, %(extra)s)'] [parameters: {'user_id': u'84f7baa50ec34454bdb5d6a2254278b3', 'extra': '{}', 'key_hash': '8e3a186ac35259d9c5b952201973dda4dfc1eefe', 'encrypted_blob': 'gAAAAABZ1S5zAOe7DBj5-IoOe3ci1C1QzyLcHFRV3vJvoqpWL3qVjG8EQybUaZJN_-n3vFvoR_uIL2-2Ic2Sug9jImAt-XgM0w==', 'project_id': None, 'type': u'cert', 'id': 'ff09de37ad2a4fce97993da17176e288'}]
```

To validate:

1. Attach to the keystone container and enter the venv

```
lxc-attach --name $(lxc-ls -1| grep key)
cd /openstack/venvs/keystone-15.1.7
source bin/activate
source ~/openrc
```

2. Attempt to create a credential entry:

```
openstack credential create admin my-secret-stuff

An unexpected error prevented the server from fulfilling your request. (HTTP 500) (Request-ID: req-d8814c07-59a6-4a06-80dc-1f46082f0866)
```

## Fix at build time

To fix when building, add shade 1.22.2 to the global-requirements-pins.txt prior to building the environment:

```
echo "shade==1.22.2" | tee -a /opt/openstack-ansible/global-requirement-pins.txt

scripts/bootstrap-ansible.sh \
    && scripts/bootstrap-aio.sh \
    && scripts/run-playbooks.sh
```

## To fix while running

* Pin shade to 1.22.2
* Rerun os-keystone-install.yml
* keystone-manage db_sync expand, migrate, and contract

1. Pin shade:

```
echo "shade==1.22.2" | tee -a /opt/openstack-ansible/global-requirements-pins.txt
```

2. Run os-keystone-install.yml

```
cd /opt/openstack-ansible/playbooks
openstack-ansible -vvv os-keystone-install.yml
```

### keystone db_sync steps

With shade pinned, the following steps should unlock the credential table in the keystone database:

1. Attach to the keystone container and enter the venv

```
lxc-attach --name $(lxc-ls -1| grep key)
cd /openstack/venvs/keystone-15.1.7
source bin/activate
source ~/openrc
```

2. Expand the keystone database

```
keystone-manage db_sync --expand
```

3. Migrate the keystone database

```
keystone-manage db_sync --migrate
```

4. Then, contract the keystone database

```
keystone-manage db_sync --contract
```

> __Note:__ These are the same steps the os_keystone role uses.

5. After this is done, test credential creation:

```
openstack credential create admin my-secret-stuff

+------------+----------------------------------+
| Field      | Value                            |
+------------+----------------------------------+
| blob       | my-secret-stuff                  |
| id         | 4d1f2dd232854dd3b52dc0ea2dd2f451 |
| project_id | None                             |
| type       | cert                             |
| user_id    | 187654e532cb43599159c5ea0be84a68 |
+------------+----------------------------------+
```

## Still didn't work?

1. Dump the keystone database to a file, then make a backup of said file

```
lxc-attach --name $(lsc-ls -1 | grep galera)
mysqldump keystone > keystone.orig
cp keystone.orig keystone.edited
```

2. Edit the file to remove / add the following

```
--- edit out this section ---
BEGIN
  IF NEW.encrypted_blob IS NULL THEN
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Credential migration in progress. Cannot perform writes to credential table.';
  END IF;
  IF NEW.encrypted_blob IS NOT NULL AND OLD.blob IS NULL THEN
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Credential migration in progress. Cannot perform writes to credential table.';
  END IF;
END */;;
--- end edits ---

--- add this to the first line ---
USE keystone;
--- end addition ---
```

3. Then apply the changes

```
mysql < keystone.edited
```

4. After this is done, test credential creation:

```
openstack credential create admin my-secret-stuff

+------------+----------------------------------+
| Field      | Value                            |
+------------+----------------------------------+
| blob       | my-secret-stuff                  |
| id         | 4d1f2dd232854dd3b52dc0ea2dd2f451 |
| project_id | None                             |
| type       | cert                             |
| user_id    | 187654e532cb43599159c5ea0be84a68 |
+------------+----------------------------------+
```

# Resources

The following resources were not harmed during the filming of this blog post:

* [https://gist.github.com/lbragstad/ddfb10f9f9048414d1f781ba006e95d1](https://gist.github.com/lbragstad/ddfb10f9f9048414d1f781ba006e95d1)