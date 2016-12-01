# Working with LDAP on a docker container. Starting with LDAP command-line utilities

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


### Attributes

The data itself in an LDAP system is mainly stored in elements called attributes. Attributes are basically key-value pairs. Unlike in some other systems, the keys have predefined names which are dictated by the objectClasses selected for entry. Furthermore, the data in an attribute must match the type defined in the attribute's initial definition.

```
...
home: /home/mparra
...
```


### Entries

Attributes by themselves are not very useful. To have meaning, they must be associated with something. Within LDAP, you use attributes within an entry. 

An entry is basically a collection of attributes under a name used to describe something.

```
# LDAP user entry example
dn: cn=mparra,ou=Users,dc=openstack,dc=org
objectClass: top
objectClass: account
objectClass: posixAccount
objectClass: shadowAccount
cn: mparra
...
```


### DIT

A DIT is simply the hierarchy describing the relationship of existing entries. 

Upon creation, each new entry must "hook into" the existing DIT by placing itself as a child of an existing entry.

This creates a tree-like structure that is used to define relationships and assign meaning.

### Distinguished name (dn)

It functions like a full path back to the root of the Data Information Trees, or DITs.

For example:

```
dn: cn=mparra,ou=Users,dc=openstack,dc=org
```

cn: Common name
ou: organizational segment

The direct parent is an entry called ``ou=Users`` which is being used as a container for entries describing users.

The parents of this entry derived from the ``openstack.org` domain name, which functions as the root of our DIT.

These are often used for the general categories under the top-level DIT entry, things like ```ou=people, ou=groups, and ou=inventory``` are common.


### Structure

![LDAPstruct](https://sites.google.com/site/manuparra/home/ou.png)


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

Now, try out:

```
ldapsearch -H ldap://localhost -LL -b ou=Users,dc=openstack,dc=org -x
```

It returns LDAP directory with the last user added.

## Modifying LDAP user account data: DELETE, MODIFY.

The syntax of the ldapmodify tool on the command-line can take any of these forms:

```
ldapmodify [ options ]

ldapmodify [ options ] < LDIFfile

ldapmodify [ options ] -f LDIFfile
```

LDIF text file containing new entries or updates to existing entries on LDAP directory.

When modifying the contents of a directory, you must satisfy several prerequisite conditions. 

First, the bind DN and password used for authentication must have the appropriate permissions for the operations being performed.

Create an example LDIF Modify and save the file as i.e. ``mparra_modify.ldif``

```
dn: cn=mparra,ou=Users,dc=openstack,dc=org
changetype: modify
replace: loginShell
loginShell: /bin/csh
```

Then execute:

```
ldapmodify -x -D "cn=admin,dc=openstack,dc=org" -w password -H ldap:// -f mparra_modify.ldif
```

It will update ``cn=mparra`` with a new ``loginShell``, in this case ``/bin/csh``

Check if the change has been done:

```
...
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
gecos: mparra
shadowMax: 0
shadowWarning: 0
loginShell: /bin/csh
...

```

Adding a entry of LDAP user. Create a new file ``manu_add_descrip.ldif`` and add: 

```
dn: cn=mparra,ou=Users,dc=openstack,dc=org
changetype: modify
add: description
description: Manuel Parra
```

Execute next command:

```
ldapmodify -x -D "cn=admin,dc=openstack,dc=org" -w password -H ldap:// -f manu_add_descrip.ldif
```

Now, check changes:


```
ldapsearch -H ldap://localhost -LL -b ou=Users,dc=openstack,dc=org -x
```

And finally delete description. Create a new file i.e. ``manu_del_descr.ldif``

```
dn: cn=mparra,ou=Users,dc=openstack,dc=org
changetype: modify
delete: description
``` 

Then execute: 

```
ldapmodify -x -D "cn=admin,dc=openstack,dc=org" -w password -H ldap:// -f manu_del_descr.ldif
```

Now, check out:

```
ldapsearch -H ldap://localhost -LL -b ou=Users,dc=openstack,dc=org -x
```

Verify if entity description is not set.

















