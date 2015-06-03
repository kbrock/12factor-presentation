# MULTI-TENANT Multi-tenancy
***
# MULTI-TENANT Goals
---
| | |providers and tenants| | |
---|---|---|---|---
DATA|CONSTRAINED|(prevent mistakes)    |UNCONSTRAINED|share
CONFIG|SEPARATE|infrastructure vs app  |DELETE|FILE
CODE|CONSTRAINED|automate              |CONSTRAINED|gems
LOGS|SEPARATE|aggregate                |CONSTRAINED|ROOT
---
|  |   |Extras
---|---|---
UX|APPLIANCE|add appliances
UX|APPLIANCE|Easier upgrades
UX|APPLIANCE|diagnose problems
SCALE||MEASURE MESSAGE BUS
DEBT|CODE|SEPARATE
***
***
# 12 Factor Goals
Guidelines for Enterprise Apps
---
When||What|Who
---|---|---|---
2002|JAVA|<cite>[Patterns of Enterprise Application Architecture]</cite>|@martinfowler
2007||*[heroku.com]*
2011|:gem:|<cite>[12factor.net]</cite> <small>[git][12factor-gh]</small>|@adamwiggins
2013|DOCKER|@docker|@shykes
---
| ||Decrease costs, Increase revenue
---|---|---
LOGS|MEASURE|Easy to diagnose problems
CODE|MULTI-TENANT|separate infrastructure and apps
DOGFOOD||maintain 1 set of infrastructure
VERSIONED||Upgradeable apps
|||
|SCALE||Easy to add services and scale
|CUSTOMER||Easy to add/privilege users
***
***
# Concepts
---
|MIQ    |concepts
------- |---
EC2     |run servers
CUSTOMER|monitor and lifecycle workers / servers
WORKER  |light weight workers
CODE    |User code (automate) runs in worker
DOGFOOD |Manageiq code runs in workers
---
MANY|DISPOSABLE|STATELESS|SERVICE|
---|---|---|---
CODE|CONFIG|DATA|MEASURE
BUILD|DEPLOY|RUNNER|SCALE
|||
DOGFOOD|VERSIONED|URL
---
```dot
digraph factor_flow {
  graph [ fontname="helvetica-bold" ]
  node  [ id="\N" shape="Mrecord" style="filled" fontname="helvetica" fillcolor="#ffffff" penwidth="2" ]
  edge  [ arrowsize="0.5" fontname="helvetica"]

  subgraph xcluster_vmware {
    vm0          [ label="{template|OS}" ]
    vm1          [ label="{template|APPLIANCE}" ]
    vm2          [ label="{template|CUSTOMER}" ]
    {
      rank="SAME"
      service1     [ label="{vm|SERVICE}"]
      service3     [ label="{vm|SERVICE}"]
    }
    metrics1     [ label="{vm|MEASURE}"]
    service2     [ label="{vm|DATA}"]
    service2 -> service1 [ dir="back" ]
    service2 -> service3 [ dir="back" ]
    metrics1 -> service1 [ dir="back" ]
  }

  subgraph cluster_build {
    build [ label="BUILD" ]
    build_custom1 [ label="{CODE|VERSIONED}" ]
    build_custom  [ label="{reports.yml|CUSTOMER}" ]
    vm_build_config [ label="{CONFIG CODE|CUSTOMER}" ]
    build       -> build_custom1 [dir="back" constraint="false"]
    build       -> build_custom [dir="back" ]
    build       -> vm_build_config [ dir="back" ]
  }
    vm0         -> build
    vm1         -> build [ dir="back" ]
    vm1         -> build
    vm2         -> build [ dir="back" label="create"]
  subgraph cluster_deploy {
    deploy_config [ label="{CONFIG|CUSTOMER}" ]
    deploy        [ label="DEPLOY" ]
    deploy   -> deploy_config [dir="back"]
  }
    vm2      -> deploy
    service1 -> deploy     [ dir="back" label="create" ]
  subgraph cluster_runner {
    runner_data [ label="{DATA|{SERVICE|SCALE}|{SERVICE|SCALE}|{MEASURE|SCALE}}"]
    runner                [ label="{RUNNER}" ]
    runner   -> runner_data [ dir="back" ]
  }
    metrics1 -> runner    [ dir="back" constraint="false" style="invis"]
    service2 -> runner    [ dir="back" style="invis" ]
    service1 -> runner    [ dir="back" ]
    service3 -> runner    [ dir="back" ]
}
```
---
|      |      |translation
---    |---   |---
BUILD  |John  |docker file, vmware templates
DEPLOY |automate|kickstart, app console
RUNNER |APPLIANCE|Runner runs containers (docker)
SERVICE|WORKER|container (custom or miq code)
---
```notes
- code, on disk changes at build time. set by us and unique per version.
  (column row, blob keys count. migrations to help, REST response schema)
- config changes at deploy time, put into the env
  unique per customer. url has versioning information in it.
- data, in the database changes at runtime. set by many services and unique by time.

Interesting:
- What does not match changing model (or versioning)
- When CONFIG considered DATA (SEPARATE) e.g.: providers
- When CODE is considered DATA e.g.: automate
- When CODE is not stored on FILE

TODO: CODE (comment on automate)
TODO: CODE (url has server, code determines version / to match own api)

```
CONSTRAINED|   |CHANGING|         |e.g.                    |VERSIONED
------|--------|--------|---------|------------------------|--------
CODE  |FILE    |BUILD   |MIQ      |`schema.rb`, api, keys  |YES
CONFIG|`ENV*`  |DEPLOY  |ROOT     |URL                     |UNKNOWN
DATA  |DATABASE|RUNNER  |CUSTOMER |vms, hosts, providers   |NO
LOGS  |MEASURE |RUNNER  |ANYTIME  |logs, events            |NO
***
***
```notes
move these elsewhere?
```
Rants
---
TURTLE| |
---|---
RUNNER|EC2/vmware is runner
RUNNER|appliance/docker is runner
---
SCALE||DISPOSABLE| |
---|---|---|---
CONFIG|`GUID`|PET
CONFIG|FILE|PET
RUNNER|WORKER|DISPOSABLE
DISCOVERY|WORKER|DISPOSABLE|remove cleanup
---
```notes:
- constantly testing
- meet business needs (which should be same as customers)
- fewer surprises
```
DOGFOOD|QE/Dev/prod parity
-------|---
SERVICE|all use message queue
DEPLOY |appliance console, not [qe scripts], [dev scripts]
DATA   |Correct Postgres version
WORKER |Use workers not simulate
DEV    |use apache, workers, replication, ipa

[qe scripts]: https://github.com/RedHatQE/cfme_tests/blob/master/utils/appliance.py
[dev scripts]: https://github.com/ManageIQ/cfme_tools/tree/master/fusion-dev-appliance-setup
***
***
# SEPARATE CONSTRAINED Road Map
---
MULTI-TENANT|SEPARATE|&nbsp;|MULTI-TENANT|CONSTRAINED |&nbsp;|SCALE|
---|---| --- |---|---|---|---
CONFIG|vmdb.yml    ||APPLIANCE|read only? ||YES|
DATA  |hosts, vm   ||ROOT     |groups      ||
CODE  |automate    ||API      |APIs
CUSTOMER|reports.yml ||
***
***
CONFIG|review
---|---
FILE|URL in ENV
DEPLOY|Changes at deploy only
CODE|same for all users, then it is code
DATA|If it needs to change, then it is data
CUSTOMER|Different user than adding a provider
SEPARATE|Divide by where, when, who changes.
---
```notes
smtp password to database
tenant
```
CONFIG     |appliance console action
---        |---
ROOT       |move appliance configuration to console  |deploy vs runner 
CUSTOMER   |move customizations/passwords to database|file -> db
URL        |miq_auth: add host (use url)             |url / configuration
CONSTRAINED|enable console for admins (no root)
GIT        |Move to own repo / own Gemfile
WORKER     |Move ruby rep out of vmdb.yml to console |root/config
---
```notes
separate # workers from runtime(provider) data
one time password
```
CONFIG    |Wish list
------    |---
DEPLOY    |welcome packet, IdM OTP
DEPLOY    |expose console in cloudinit
DISPOSABLE|Use zone not GUID
RUNNER    |separate runner and worker config
***
***
CODE     |      |review
---------|------|----------
VERSIONED|API   |MESSAGE BUS DATA blob CONFIG
BUILD    |GIT   |Changes at build only
FILE     |      |Another container or data
MIQ      |      |Same for all miq users
SEPARATE |      |Divide config vs data
---
```notes
automate is long living git branches
```
CODE       |    |actions for automate
---        |----|---
CONSTRAINED|ROOT|Run as non root                       |security
CONSTRAINED|DATA|block breaking into database API      |security
VERSIONED  |API |vmdb version                          |upgrade
VERSIONED  |FILE|automate version/monkey patching      |upgrade
CUSTOMER   |   |Process to add endpoints (vs overwrite)|upgrade
||
SEPARATE   |MIQ |Automate yml files are not vmdb       |upgrade
FORK       |URL |long running automate process         |Scaling
---
```notes
every method is public interface (not tested)
```
CODE       |MESSAGE BUS|actions
-----      |----       |----
VERSIONED  |API        |add vmdb version number (rpc)  |upgrade
CONSTRAINED|API        |messages not methods           |security, upgrade
||
DISPOSABLE |           |Use zone not GUID              |SCALING, upgrade, debt
WORKER     |           |generic worker (fewer queues)  |debt
CONSTRAINED|MESSAGE BUS|don't modify queue             |SCALING
---
CODE|   |vmdb actions
----|---|---
CONFIG| |Move config to appliance console
||
RUNNER| |Extract runner/worker to own code base
***
***
# General action items
aka ran out of time to present

---
DATA|actions
---|---
DATA|Introduce tenant models
||
DISPOSABLE|pull postgres out of appliance
---
MEASURE|actions
---|---
LOG|RHCI agregate logs / partition by tenant
LOG|miq_top -> standard events / metrics
---
DEBT|wishlist
--- |---
CODE|remove code that is not core competency
***
***
# FAQ
---
## This looks like it will take a while.
Why put everything on hold for 2 years?

No, only change what will make your current project work better.
At the very least, just don't introduce state into the wrong place.

---
# Can you explain the steps necessary to get us there?
For Multi-tenancy,  [high level](#/5/1), see [slides](#/6/1)

---
# Is the goal to introduce Docker?
No, while docker and all PaaS use 12factor principles, that is not the goal.

The goal is to put state in the correct place. This makes it easier to host the appliance on all platforms, including Docker.

---
# Is this really necessary?
Nope, our app works just fine right the way it is.

But most of our pain points are simply fighting the basic principles of state as found on the [grid](#/3/5).

---
# Principles of state?
Every algorithm has assumptions, inputs, and process.
12 factors simply names them and shares how they work together.

[Patterns of Enterprise Application Architecture]: http://amzn.com/0321127420
[12factor.net]: http://12factor.net
[12factor-gh]: https://github.com/heroku/12factor
[heroku.com]: http://heroku.com/
*[:ballot_box_with_check:]: YES
*[:white_medium_square:]: NO
*[:construction_worker:]: WORKER
*[:gem:]: CODE
*[:abcd:]: CONFIG
*[:dvd:]: DATA
*[:innocent:]: STATELESS
*[:cloud:]: SERVICE
*[:octocat:]: GIT
*[:hash:]: VERSIONED
*[:globe_with_meridians:]: URL
*[:runner:]: RUNNER
*[:wrench:]: DEPLOY
*[:one:]: ONE
*[:100:]: MANY
*[:construction:]: BUILD
*[:rocket:]: SCALE
*[:tv:]: APPLIANCE
*[:cow:]: DISPOSABLE
*[:dog:]: PET
*[:cat:]: PET2
*[:bento:]: DOGFOOD
*[:boy:]: ME
*[:girl:]: YOU
*[:person_with_blond_hair:]: OTHERS
*[:cop:]: ROOT
*[:bus:]: MESSAGE BUS
*[:file_folder:]: FILE
*[:notebook:]: DISCOVERY
*[:chart_with_upwards_trend:]: MEASURE
*[:fork_and_knife:]: FORK
*[:black_joker:]: CHANGING
*[:computer:]: COMPUTER
*[:hotsprings:]: JAVA
*[:bust_in_silhouette:]: CUSTOMER
*[:lock:]: CONSTRAINED
*[:unlock:]: UNCONSTRAINED
*[:bathtub:]: CLEANUP
*[:scissors:]: SEPARATE
*[:put_litter_in_its_place:]: DELETE
*[:ship:]: DOCKER
*[:+1:]: LIKE
*[:-1:]: DISLIKE
*[:guitar:]: CFME
*[:grey_question:]: UNKNOWN
*[:arrow_right:]: TO
*[:office:]: MULTI-TENANT
*[:turtle:]: TURTLE
*[:sunflower:]: UX
*[:electric_plug:]: PLUGABLE
*[:mailbox:]: DATABASE
*[:alarm_clock:]: ANYTIME
*[:evergreen_tree:]: LOGS
*[:door:]: API
*[:tophat:]: Red Hat
*[:moneybag:]: DEBT
***
