
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

First, try out:

```
ldapsearch -H ldap://localhost -LL -b ou=Users,dc=openstack,dc=org -x
```

This command returns the list of LDAP users:


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




