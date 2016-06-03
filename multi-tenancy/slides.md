# Multi-tenancy Group POC
---
# Overview

Groups are a partitioning mechanism, but are lacking in a number of ways.

- What minimal demo will prove that groups are viable?
- Do we think we can change groups in this way without breaking existing?

---

easy
brand
    custom_logo => custom_logo_name
    company, name
url -> session[:tenant_group]
    # extra: don't allow wrong user in
user.current_group -> brand
don't allow someone in when !user.groups.include?(url_group)
hard
sub-group -> brand
- pass user_id everywhere and assume User.find().current_group is good
- admin privs within a group

- use groups everywhere tags are used
***
# Branding
---
- stored in `vmdb.yml`, `less`, and `png`s. Move from disk to database.
- we call it branding, customizations, and graphics.
- allow multiple brands. (new)
- use brand_id ("session") to switch between brands. (new)
***
***
# groups
---
- "a tenant is a container use to group or isolate resources and identity."
- request: user can be in multiple tenants.
- request: have a tenant drop down in upper right corner.
- All group functionality and a bit more.
- Want multiple layers of sub-tenant and business group.
---
- everything that a "tenant" gets, "group" get too.
- Do we need to introduce a new "tenant" group concept?
- How many levels do we need?
- Can we have hierarchical groups?
- Can we have groups support same use cases as tagging?
***
***
# authentication
---
# authentication known

- a (sub)tenant can share parent's LDAP (via Organizational Unit)
- a tenant can have own LDAP
- move LDAP configuration off "appliance" and into database.
---
# authentication unknown

- How much user management belongs is in CFME?
- The implications of adding another grouping mechanism.
- What is the problem with existing grouping mechanism?
- How to create first tenant (and ldap link)?
- Customer support concerns: "suport all existing setups"
***
# quota and reserves

TODO
---
# reports

Similar to branding, reports are stored on disk. These need to move into the database and rbac needs to be improved. Also like branding, there need to exist multiple versions of each report, one per tenant (but probably per 
---
# Automate

This is not mentioned, but this needs to have better versioning and locked down for security for a multi-tenant setup

*[OU]: Organizational Unit
