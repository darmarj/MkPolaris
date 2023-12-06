---
date: 2023-06-05
authors: [darmarj]
description: >
  GPG & secret-store
categories:
  - Security
---

# Secure-store

## [GPG](https://gnupg.org/)
GnuPG is a complete and free implementation of the OpenPGP standard as defined by [RFC4880](https://www.ietf.org/rfc/rfc4880.txt) (also known as PGP). GnuPG allows you to encrypt and sign your data and communications; it features a versatile key management system, along with access modules for all kinds of public key directories.

``` bash
gpg --gen-key

# edit expire if needed
gpg --edit-key GPG_PUB_KEY

# export GPG pub key
gpg --output GPG_PUB_KEY --armor --export EMAIL_ID

# export GPG pri key
gpg --output GPG_PRI_KEY --armor --export-secret-key EMAIL_ID

# import keys
gpg --import GPG_PRI_KEY
gpg --import GPG_PUB_KEY
```

The standard unix password manager
## [pass](https://www.passwordstore.org/)

``` bash
pass init GPG_PUB_KEY

# init git repo for pass secret-store
pass git init

pass insert github
pass generate aws
pass show
pass git log

# set it up for sys env
export key=$(pass show KEY_PATH/KEY_ID)
```
