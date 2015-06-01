# MULTI-TENANT Multi-tenancy
***
# MULTI-TENANT Goals
---
| | |providers and tenants| | |
---|---|---|---|---
DATA|CONSTRAINED|(prevent mistakes)    |UNCONSTRAINED|share
CONFIG|SEPARATE|infrastructure vs app  |DELETE|FILE
CODE|CONSTRAINED|automate              |CONSTRAINED|gems
LOGS|SEPARATE|aggregate
---
|  |   |Side Goals
---|---|---
UX|APPLIANCE|add appliances
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
CODE||separate infrastructure and apps
DOGFOOD||maintain 1 set of infrastructure
VERSIONING|Upgradeable apps
|||
|SERVICE||Easy to add services and scale
|CUSTOMER||Easy to add/privilege users
***
***
# Concepts
---
|       ||concepts
------- |---|---
EC2     ||run servers
CUSTOMER||=monitor and lifecycle workers / servers
WORKER  |RUNNER|light weight workers
CODE    |DOCKER|User runs automate code in worker
DOGFOOD ||Manageiq run code in workers
---
MANY|DISPOSABLE|STATELESS|SERVICE|
---|---|---|---
CODE|CONFIG|DATA|MEASURE
BUILD|DEPLOY|RUNNER|SCALE
DOGFOOD|VERSIONING|URL
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
```notes
- code, on disk changes at build time. set by us and unique per version.
  (column row, blob keys count. migrations to help, REST response schema)
- config changes at deploy time, put into the env
  unique per customer. url has versioning information in it.
- data, in the database changes at runtime. set by many services and unique by time.

Interesting:
- What does not match changing model (or versioning)
- When CONFIG considered DATA (SEPARATE) e.g.: privders
- When CODE is considered DATA e.g.: automate
- When CODE is not stored on FILE

TODO: CODE (comment on automate)
TODO: CODE (url has server, code determines version / to match own api)

```
      |        |CHANGING|         |e.g.                    |VERSIONED
------|--------|--------|---------|------------------------|--------
CODE  |FILE    |BUILD   |CFME     |`schema.rb`, api, keys  |YES
CONFIG|`ENV*`  |DEPLOY  |CUSTOMER |URL                     |UNKNOWN
DATA  |DATABASE|RUNNER  |ANYTIME  |vms, hosts, providers   |NO
LOGS  |MEASURE |RUNNER  |ANYTIME  |logs, events            |NO
***
***
DOCKER FTW
---
MULTI-TENANT|SEPARATE|&nbsp;|MULTI-TENANT|CONSTRAINED |&nbsp;|SCALE|
---|---| --- |---|---|---|---
CONFIG|vmdb.yml    ||APPLIANCE|read only? ||YES|
DATA  |hosts, vm   ||ROOT     |groups      ||
CODE  |automate    ||API      |APIs
CUSTOMER|reports.yml ||
---

***
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
