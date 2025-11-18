---
title: "LDAP Snippets"
date: 2021-11-25T10:51:48+01:00
summary: Useful snippet for managing LDAP from CLI
cover:
  image: imgs/cover.png
slug: ldap-snippets
tags:
  - snippet
  - ldap
  - cli
  - authentication
---

{{<alert info>}} ℹ️ Every snippet that uses a file can be onelined using cat, like:
  ```sh
  cat << EOF | ldapcommand
  ldif file content here
  EOF
```
{{</alert>}}

# Commands

In the snippets below we use some arguments.
Here are the arguments per-tool, explained:

## `ldapadd`, `ldapmodify`, `ldapsearch`
- `-D "cn=admin,dc=example,dc=org"` is the username, called "Binding DN"
- `-w admin` is the password
- `-f filename.ldif` specify which LDIF file to use to add/modify/search

## `ldapsearch` only
- `-b "dc=example,dc=org"` is the base address where to start the search

# Snippets

## Create a new user

{{<highlight yaml "linenos=true">}}
# file.ldif
# mandatory:
dn: cn=billy,dc=example,dc=org
cn: billy
objectClass: inetOrgPerson
sn: Jones # sn=Surname
userPassword: billy
# userPassword can be hashed with slappasswd, see next snippet
# userPassword: {SSHA}g1RqOK2kk/nV241xD2BZRdxdVuRqGNnG
# optionally:
uid: 123
givenName: Billy & Mandy
mail: billy@example.org
{{</highlight>}}

```sh
ldapadd -D "cn=admin,dc=example,dc=org" -w admin -f file.ldif
```

## Generate hashed password

```sh
slappasswd -g # generates random non hashed password

slappasswd # hashes password with default hash (like SSHA = Salted SHA)

slappasswd -h '{CRYPT}' # generates hashed password with CRYPT hash algorithm

# oneline:
PASSWORD=$(slappasswd -g) && echo $PASSWORD && slappasswd -s "$PASSWORD"
```

## Export all the readable objects

```sh
ldapsearch -D "cn=admin,dc=example,dc=org" -w admin -b "dc=example,dc=org"
```


## Allow every authenticated user to read the directory

- `dn` is the selector of the database (you could have `hdb` instead of `mdb` if your LDAP is older)
- `olcAccess`
  - `{1}` is the priority between all the other rules (can be negative, rules are applied from the smallest number to the higher one)
  - `to *` is the resource to which the rule applies, could be `to dn.base="dc=example,dc=org"` or similar, [see docs](https://www.openldap.org/doc/admin24/access-control.html) for more info
  - `by anonymous auth` allows every un-authenticated user to authenticate
  - `by users read` allows every authenticated user to read the object specified in the `to` clause

```yaml
# config.ldif
dn: olcDatabase={1}mdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {1}to *
  by anonymous auth
  by users read
```

```sh
ldapmodify -D "cn=admin,cn=config" -w config -f config.ldif
```
