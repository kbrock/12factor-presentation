# What is Multi-tenancy?
---
# Overview

- high level business cases.
- transle to something we can “do”.
- high level list of actionable items.
***
John Hardy
---
> The CF product architecture needs to accommodate the multi-tenancy *use cases* of today’s service providers and enterprises.
> 
> Product Multi-Tenancy refers to the CF product supporting more than one organization with *separate authentication*, *branding* and *security access controls*, though at the same time supporting the notion of *sharing* between tenants.
---
```notes
I originally assume tenancy means thick wall. Sharring suggests it is different than that.
```
- many LDAPs
- easier branding (get into the database)
- many branding
- better security access control
    + tigher
    + sharing (with holes)
---
> Allow to start managing who can access and the scope of what they can manage in a normalized way
> -- trello
***
***
Loic
---
> Tenant = A container used to *group* or *isolate* *resources* and *identity*.
---
*resources* and *identity* is an rbac statement:

> I want improved rbac.

- *grouping resources* so you can more easily put rbac around them.
- *isolate resources* is the result of applying rbac.
- *grouping identity* so you can apply the other side of rbac.
---
> Enterprise and Cloud Service Providers need a simple way to *isolate environments*, and provide *more granular permissions* including specifics delegations to users and admins.

> Specific groups of people, subdivision of a company or external customer (CSP case), will have access to *“cloud” environments* and will be able to manage it.
---
- *isolate enviroments* for better faith in rbac. (harder to screw up)
- *more granular permissions* by better rbac.
- *cloud* means rbac is tight enough that we can have multiple customers in one install.
***
***
# Translation
(mine)
---
Support Cloud Service Providers and Enterprise

> Once you roll CFME out to larger customers,
a procedure that was once a minor inconvenience, now
becomes a burden.
>
> We need to implement these missing or incomplete features.
---
```notes

You can not get "multi-tenancy."
You can add features that make multi-tenancy work better.
```
Multi-tenancy is a collection of usability features:

- all features can be implemented on their own.
- all features benifit all customers. <small>(some more than others)</small>
- all features have been on our radar for a while.
---
There are a few features that are not strictly usability:

- support more than 1 dynamic brand
- add reserves and quotas 
- admins for a group.
---
Now it is a priority to nail these down.
***
# Separating feature requests into separate projects
***
# Branding
---
- branding, customizations from `vmdb.yml`, `less`, and `png`s.
- get brand into the database
- allow multiple brands
- allow a session id (`brand_id` or `tenant_it`) to switch between them.
***
***
# groups
---
- a tenant is a container use to group or isolate resources and identity.
- request: user can be in multiple tenants
- request: have a tenant drop down in upper right corner
- All group functionality and a bit more
- Add that extra functionality to groups and sub-tenants
---
- groups support and at top level and or at second layer.
- groups support same use cases we currently use tagging.
- do we need to introduce a new tenant concept?
***
***
# authentication
---
# authentication known

- a (sub)tenant can share parent's LDAP (via Organizational Unit)
- a tenant can have own LDAP
- everything that a "tenant" gets, "group" gets too.
- also want another layer, "business group"
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
