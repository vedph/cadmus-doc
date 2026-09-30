---
title: "Accounts" 
layout: default
parent: "Cadmus Tool"
nav_order: 2
---

# Accounts Commands

## Add User Command

🎯 Add a user account.

```sh
./cadmus-tool add-user NAME PASSWORD EMAIL FIRST_NAME LAST_NAME
```

## Add User Roles Command

🎯 Add role(s) to a user account.

```sh
./cadmus-tool add-user-roles NAME ROLE1 ROLEn
```

## Delete User Command

🎯 Delete a user account.

```sh
./cadmus-tool delete-user USER_NAME [-y]
```

## Delete User Roles Command

🎯 Delete role(s) from a user account.

```sh
./cadmus-tool delete-user-roles
```

## List Users Command

🎯 List user accounts.

```sh
./cadmus-tool list-users
```

## Seed Users Command

🎯 Seed user accounts from a list.

```sh
./cadmus-tool seed-users JSON_FILE_PATH DB_NAME [-d]
```

- `-d` / `--dry`: dry run.

The users list format is like:

```json
[
  {
    "UserName": "doe",
    "Password": "P4ss-W0rd!",
    "Email": "john.doe@somewhere.com",
    "Roles": ["admin", "editor", "operator", "visitor"],
    "FirstName": "John",
    "LastName": "Doe"
  },
  // ...
]
```

## Update User Command

🎯 Update a user account.

```sh
./cadmus-tool update-user
```
