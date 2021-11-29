---
title: "LDAP Setup"
date: 2021-11-24T19:08:07+01:00
summary: Why, what and how LDAP 
cover:
  image: imgs/cover.png
slug: ldap-setup
tags:
  - ldap
  - docker
  - authentication
---

# Snippets

If you search only snippets, go [below](#cli-snippets).

# Why LDAP

If you need to use LDAP, I'm really sorry, it simply sucks.

It is a **very old** idea with many limitations, but sadly still widely used.

If you search in Google Images or similar for "LDAP" you will only find
interfaces that reminds me of Windows 95, like the following:

{{<figure src="imgs/ldap_windows95_style.jpg" caption="A very old and ugly interface that I don't want to see in my life" align=center >}}

But as I previously said, it is just used a lot, so sooner or later we will have
to face it.

# What is LDAP

LDAP is a protocol to organize data in a hierarchical structure.
Often these data are "users" and "organizations" of a company.

An implementation of ldap is `openldap`, and a convenient container image of
openldap is [`osixia/openldap`](https://hub.docker.com/r/osixia/openldap/).

# How LDAP

## Basic setup

### LDAP Server setup
The simplest setup, using docker:

```sh
docker run --rm -ti -p 389:389 osixia/openldap:1.5.0
```

or using `docker-compose`:

```yaml
services:
  ldap:
    image: osixia/openldap:1.5.0
    ports:
      - "389:389"
```

You can now play with it using a tool to manage the LDAP server.

### LDAP Client setup

There are a lot of tools, but the only usable and not old style one that I found
is [jxplorer](http://jxplorer.org/downloads/users.html).
It is written in Java so it is pretty portable.
It has two "modes": "HTML View" and "Table Editor". "HTML View" is terrible, so
I always use the "Table Editor".

It looks like this:

{{<figure src="imgs/jxplorer.png" caption="Jxplorer" align=center >}}

You will want to login to the LDAP server. The "login" process in LDAP is called
"binding" process. So you need to "bind" to the server.

The default credentials that the container image provides are currently:

```yaml
User DN:    cn=admin,dc=example,dc=com
Password:   admin
```

{{<figure src="imgs/login.png" caption="Login window" align=center >}}

If they differ, refer to the [documentation of the image on github](https://github.com/osixia/docker-openldap).

## cn? dc? ou?

In LDAP we refer to an object in a tree with its full "path".

So `cn=admin,dc=example,dc=org` means "the object called `cn=admin`, inside the
object called `dc=example`, which is inside the object called `dc=org`"

Why `cn` and `dc`?

Because objects are just a bunch of properties, and thus we need to define a
property that identify the object. Why not using the same property for every
object? Who knows.

Anyway:
- `dc` (domain component) is used for directories
- `cn` (common name) is used for leaf nodes like users
- `ou` (organizational unit) is used for ...yes, organizational unit

It is just convention.

This is an example of structure (with the first three nodes compressed):

{{<figure src="imgs/cover.png" caption="Example of structure" align=center >}}

## How to add an object?

Since an object is just a bunch of properties inside a directory, we need to
understand two things to add an object:
1. What is the "path" of the object?
2. What are the properties of the object?

Let's suppose we want to create a top-level organizational unit, for
example we want to add the organizational unitfor the "Quality" department of
our company that is called example.org.

We will create the object:
1. Inside `dc=example,dc=org`, so that it will be `ou=Quality,dc=example,dc=org`
2. With the properties that an organizational unit should have

Properties are defined in groups by "objectClasses". Each object class has a
name and a collection of properties.

If we try to create a new node, and search inside the available object classes,
we can find the organizationalUnit:

{{<figure src="imgs/ou1.png" caption="objectClasses" align=center >}}

Adding it and submitting the final object, will lead to a new object in the
directory structure

{{<figure src="imgs/ou2.png" caption="Quality organizational unit added" align=center >}}

## CLI ftw... or wtf

Personally I really like UIs since they make it easier to understand what is
going on, but when automation is needed, the CLI is the obvious best option.

But, as I previously told, LDAP is old AF. So CLI is terrible.

These are some tools (pre-installed on osx):

```
ldapadd      ldapdelete   ldapmodify   ldappasswd   ldapurl
ldapcompare  ldapexop     ldapmodrdn   ldapsearch   ldapwhoami
slapacl      slapauth     slapconfig   slapindex    slapschema
slapadd      slapcat      slapdn       slappasswd   slaptest
````

Yes, they are a lot, but the most useful are `ldapadd` and `ldapsearch`
