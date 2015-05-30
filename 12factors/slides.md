# 12 Factor App

DISPOSABLE Disposable Infrastructure
***
```notes
In 2002 Martin Fowler write about the building blocks of distributed applications.
In 2011(?) Adam Wiggins tuned it with more opinions and more web focused.
Since Heroku was founded in 2007, I'm questioning the git log
```
When||What|Who
---|---|---|---
2002|JAVA|<cite>[Patterns of Enterprise Application Architecture]</cite>|@martinfowler
2007||*[heroku.com]*
2011|:gem:|<cite>[12factor.net]</cite> <small>[git][12factor-gh]</small>|@adamwiggins
***
***
```notes
TODO: talk about define configure: service urls, # runners, 
service, runner, build, deploy has own (docker), config (machines), data (# runners)
```
MANY|DISPOSABLE|SERVICE|MEASURE
---|---|---|---
CODE|CONFIG|DATA|STATELESS
BUILD|DEPLOY|RUNNER|SCALE
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
    service1 -> runner    [ dir="back" label="clone" ]
    service3 -> runner    [ dir="back" label="clone" ]
}
```
***
***
# I. CODE Codebase 
One codebase tracked in revision control, many deploys
---
ONE|SERVICE
---|---
URL|VERSIONED
CODE|GIT
---
||BUILD|DEPLOY|:one:|:two:|:three:|:gem:
---|---|---|---|---|---|---
<small>upstream</small>|GIT|GIT|GIT|GIT|GIT|MANY
<small>downstream</small>|GIT|GIT|GIT|GIT|GIT|MANY
---
||BUILD|DEPLOY|:one:|:two:|:three:|:gem:
---|---|---|---|---|---|---
ME|NO|YES|YES|NO|NO|:three:
YOU|NO|NO|YES|NO|YES|:two:
OTHERS|YES|YES|YES|YES|YES|MANY
---
DEPLOY|VERSIONED
---|---
SERVICE|:one:
SERVICE|:two:
***
***
# II. VERSIONED Dependencies
Explicitly declare and isolate dependencies
---
|||
|---|---|
DEPLOY|Can we run?
DEPLOY|Rollback?
RUNNER|Can someone break us?
RUNNER|Why is broken?
---
|||
---|---
CODE|MESSAGE BUS|`schema.rb`|
CONFIG|api, `blob schema`
DATA|`schema`
---
```notes
State who we depend upon (and the version)

lots of forks (`awesome_spawn`, url?)
a few files - hard to version, not declared

file/message bus = rpc mechanisms
```
where|VERSIONED|type  |VERSIONED|stable|
-----|---------|------|---------|------|
FILE |YES      |CODE  |YES      |YES   |source
FILE |YES      |CONFIG|NO       |NO    |report.yml
FILE |         |DATA  |         |      |
ENV  |NO       |CONFIG|NO
---
|          |VERSIONED|GIT
-----------|------|---------
FILE       |CODE  |GIT FORK
URL        |YES   |`/v1/`
FORK       |YES   |`Gemspec`, `rpmspec`
CONFIG     |DEPLOY|`.sample.yml`
DATA       |CODE  |`schema.rb` `blob.yml`
FILE       |CODE  |`config/password`
MESSAGE BUS|CODE  |`version: 1`?
CUSTOM     |BUILD |`reports.yml`
---
```notes
tell everyone your requirements
can you control the version you run
e.g. db migreations - no?
```
CODE  |BUILD   |code       |FILE        |GIT|:bug:
------|--------|-----------|------------|--------|---|---
FORK  |BUILD   | `rpmspec` |FILE        |GIT
:gem: |BUILD   | `Gemfile` |FILE        |GIT 
CONFIG|RUNNER  | URL       |FILE        |NO      |:five:  
CONFIG|RUNNER  | `database.yml`         |ENV     |:one:
DATA  |RUNNER  | `schema.rb`, blobs     |DATABASE|:two:
CODE  |RUNNER  | rpc                    |MESSAGE BUS|:three:
CODE  |RUNNER  |automate                |DATABASE|CODE       |:four:
---

|     |CONSTRAINED|MULTI-TENANT|:moneybag:| action
|---  |---|----------|----|-------|----------|
|:five:|NO|NO|NO|Lock down api version in code
|:one:|YES|NO|NO|CONFIG TO DEPLOY of TO DATA
|:two:|||:money_with_wings:|Move data from sql to service
|:three:|||YES|Intent not code
|:four:||
***
***
# III. CONFIG Config
Store config in the environment
---
```notes
re ENV: separation is most important part
perils of overwriting
avoid bulk build time vs runtime
```
CONFIG  |      |
--------|------
SEPARATE|CODE
DEPLOY  |`ENV[]`
BUILD   |FILE
MANY    |avoid `RAILS_ENV`
CHANGING|DATABASE
---
what  |CHANGING|          |e.g.                    |where   |VERSIONED
------|--------|----------|------------------------|--------|--------
CODE  |BUILD   |VERSIONED |`schema.rb`, blobs      |FILE    |YES
CONFIG|DEPLOY  |CUSTOMER  |URL                     |`ENV`   |NO
DATA  |RUNNER  |:alarm_clock:|vms, hosts, providers|DATABASE|NO
CONFIG|WORKER  |CUSTOMER  |provider url (alt)      |`ENV`   |NO
CODE  |WORKER  |CUSTOMER  |automate                |DATABASE|UNKNOWN
***
***
# IV. SERVICE Backing Services
Treat backing services as attached resources
---
```notes
url - easy to make remote / scale
url - has versioning
url/dns discoverable
  they have urls, versioned, and discoverable
```
URL      | |
---------|---
SCALE    |bigger service
VERSIONED|
DISCOVERY|`DNS` (mod_rewrite)
URL|user, pass
***
***
### V. BUILD Build, DEPLOY release, RUNNER run
Strictly separate build and run stages
---
```notes
2 differnt build phases?
custom = Docker `FROM` - to customize base image
automate customization XII VERSIONED, code up one offs

gold image, immutable
```
BUILD |SERVICE| |
------|-------|---
CODE  |       |gem install, `rake asset:build`
CONFIG|FILE   |Same all customers
BUILD |SERVICE|DOCKER `"FROM"`
CUSTOMER|CODE   |Custom properties
CUSTOMER|CONFIG |`custom_report.yml`
---
DEPLOY|SERVICE|  |
------|-------|---
CUSTOMER|CONFIG |Cloudinit
CUSTOMER|CODE   |Custom properties
MULTI-TENANT|
CONFIG|TO|DATA
---
```notes
TODO: huh?
state is not in database
```
RUNNER|SERVICE| |
----  |---|---
CONFIG|URL
DATA|
CONSTRAINED|ROOT
***
***
# VI. STATELESS Processes
Execute the app as one or more stateless processes
---
```notes
service is stateless
state goes to data or config
applinace is stateless (if appliance is a service vs runner)
```
CONSTRAINED|CODE
----------|---
CONSTRAINED|CONFIG
CHANGING  |SERVICE
CHANGING  |DATA
CONSTRAINED|APPLIANCE
***
***
# VII. URL Port binding
Export services via port binding
---
```notes
    prefer dns over mod_rewrite
```
|          |VERSIONED| |
-----------|---|---
URL        |YES|
DATA       |NO |`psql`
MESSAGE BUS|NO
FORK       |NO
FILE       |NO
DISCOVERY  |YES|`DNS`
***
***
# VIII. SCALE Concurrency
Scale out via the process model
---
```notes
can fork, but new processes (new boxes) important
```
SCALE  |   |   |
-------|---|---
SERVICE|FORK
SERVICE|COMPUTER|COMPUTER
---
```notes
separate service from runner
pids are job of runner
```
|      |`PID`|SCALE
-------|-----|---
RUNNER |YES  |YES
SERVICE|NO   |NO
***
***
# IX. DISPOSABLE Disposability
Maximize robustness with fast startup and graceful shutdown
---
```notes
goal: make router (runner) stateless
easy to discover / add workers (and remove)
capacity planning - react to dynamic load
    Pet vs Cattle

    immutable - chef
    same blocks
    config state
    customization state
```
| |SCALE| |
---|---|---
| |YES|startup instantly
CLEANUP|NO
MEASURE|YES|dynamic load
PET|NO|PET2
DISPOSABLE|YES
---
SCALE||DISPOSABLE| |
---|---|---|---
CONFIG|`GUID`|NO|producers determine
CONFIG|FILE|NO|manual
RUNNER|WORKER|YES
DISCOVERY|WORKER|YES|workers discover

***
***
# X. DOGFOOD Dev/prod parity
Keep development, staging, and production as similar as possible
---
- same scripts to setup dev / qa / production
- scripts constantly under test
- ensure scripts at least meet our needs
- devs run ipa, apache, rpm, appliance_console
- devs run replication, actual workers (vs simulate)
***
***
# XI. MEASURE Logs
Treat logs as event streams
---
MEASURE|define normal|
---|---
`miq_top`|
`/var/log/httpd/`|RHCI
vmware events|event collector

***
***
# XII. VERSIONED Admin processes
Run admin/management tasks as one-off processes
---
```notes
get processes into git:
leave it in raw form (bash, sql)
put in `migrations`, `scripts/``

1. may need to do it again
2. team review
3. want to be able to reproduce

provide this for our customers
```
CODE|CONFIG|DATA|STATELESS|VERSIONED
---|---|---|---|---
CUSTOMER|CUSTOMER|CUSTOMER|NO|NO
GIT|GIT|GIT|YES|YES
***
```notes
REVIEW
```
MANY|DISPOSABLE|SERVICE|MEASURE
---|---|---|---
CODE|CONFIG|DATA|STATELESS
BUILD|DEPLOY|RUNNER|SCALE
---
# Refs
- [12factor.net] from [heroku.com]
- <cite>[Immutable Infrastructure]</cite>
- <cite>[Pivotal podcast](http://www.paasmag.com/2014/11/19/all-things-pivotal-episode-7-a-look-at-12-factor-apps/)</cite>
- <cite>[docker 12 factor](http://www.slideshare.net/williamyeh/12-factor-app-from-dockers-point-of-view)</cite>
- slides: [reveal-ck](https://github.com/jedcn/reveal-ck)
[Patterns of Enterprise Application Architecture]: http://amzn.com/0321127420
[12factor.net]: http://12factor.net
[12factor-gh]: https://github.com/heroku/12factor
[heroku.com]: http://heroku.com/
[Immutable Infrastructure]: http://chadfowler.com/blog/2013/06/23/immutable-deployments/
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
*[:art:]: CUSTOM
*[:lock:]: CONSTRAINED
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
