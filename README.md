# Working with LDAP on a docker container. Startin with LDAP command-line utilities

Manuel Parra & José Manuel Benítez, 2016

![imgLDAP](http://www.openldap.org/images/headers/LDAPworm.gif)


## Deploy Docker container with LDAP

Only for root user:

```
docker pull larrycai/openldap

docker run -d -p 389:389 --name ldap -t larrycai/openldap
```

* First step, pull the container

* Second step run the container on port 398


Default password for LDAP admin user is `password`


## Training with LDAP

### Starting: everything is okay

Firstly, with your user try out:

```
ldapsearch -H ldap://localhost -LL -b ou=Users,dc=openstack,dc=org -x
```

This command returns the list of LDAP users (something similar):


```
...
dn: ou=Users,dc=openstack,dc=org
objectClass: organizationalUnit
ou: Users

dn: cn=Robert Smith,ou=Users,dc=openstack,dc=org
objectClass: inetOrgPerson
cn: Robert Smith
cn: Robert J Smith
cn: bob  smith
sn: smith
uid: rjsmith
carLicense: HISCAR 123
homePhone: 555-111-2222
mail: r.smith@example.com
mail: rsmith@example.com
mail: bob.smith@example.com
description: swell guy
ou: Human Resources
...
```

The format for a simple query with ``ldapsearch`` is: 

``ldapsearch -H <host> -LL -b ou=<Organizational Unit> -x``

LDAP utility ``ldaputility`` cointains multiple options and configuration, please review: https://www.centos.org/docs/5/html/CDS/ag/8.0/Finding_Directory_Entries-Using_ldapsearch.html


### Adding an user

To add something to the LDAP directory, you need to first create a LDIF file.

The ldif file should contain definitions for all attributes that are required for the entries that you want to create.

With this ``ldif`` file, you can use ``ldapadd`` command to import the entries into the directory as explained in this tutorial.

Create a file, i.e. ``user.ldif`` and copy this skeleton, modify and include your data (i.e. ``cn=mparra`` to ``cn=<user>``, i.e. ``uid=mparra`` to ``uid=<uid>`` , ``gecos`` etc.)

```
dn: cn=mparra,ou=Users,dc=openstack,dc=org
objectClass: top
objectClass: account
objectClass: posixAccount
objectClass: shadowAccount
cn: mparra
uid: mparra
uidNumber: 16859
gidNumber: 100
homeDirectory: /home/mparra
loginShell: /bin/bash
gecos: mparra
userPassword: {crypt}x
shadowLastChange: 0
shadowMax: 0
shadowWarning: 0
```

To add this user to LDAP directory:

```
ldapadd -x -D cn=admin,dc=openstack,dc=org -w password -c -f user1.ldif
```

Syntax:

```
ldapadd [options] [-f LDIF-filename]
```

If you remove ``-w password`` option, and change by ``-W`` it will ask you about LDAP admin password.


## Changing password of LDAP user

```
ldappasswd -s <new_user_password> -W -D "cn=admin,dc=openstack,dc=org" -x "cn=mparra,ou=Users,dc=openstack,dc=org"
```

In this case, using -W option, ``ldappasswd`` ask for LDAP admin password.

Syntax:

```
ldappasswd [ options ] [ user ]
```

Please check: https://www.centos.org/docs/5/html/CDS/cli/8.0/Configuration_Command_File_Reference-Command_Line_Utilities-ldappasswd.html for more information about this command.















