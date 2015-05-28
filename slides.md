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
2011|GEM|<cite>[12factor.net]</cite> <small>[git][12factor-gh]</small>|@adamwiggins
***
```notes
TODO: talk about define configure: service urls, # runners, 
service, runner, build, deploy has own (docker), config (machines), data (# runners)
```
MANY|DISPOSABLE|SERVICE|MEASURE
---|---|---|---
CODE|CONFIG|DATA|STATELESS
BUILD|DEPLOY|RUNNER|SCALE
***
# I. CODE Codebase 
One codebase tracked in revision control, many deploys
---
ONE|SERVICE
---|---
URL|VERSIONED
CODE|GIT
---
||BUILD|DEPLOY|:one:|:two:|:three:|GEM
---|---|---|---|---|---|---
<small>upstream</small>|GIT|GIT|GIT|GIT|GIT|MANY
<small>downstream</small>|GIT|GIT|GIT|GIT|GIT|MANY
---
||BUILD|DEPLOY|:one:|:two:|:three:|GEM
---|---|---|---|---|---|---
ME|NO|YES|YES|NO|NO|:three:
YOU|NO|NO|YES|NO|YES|:two:
OTHERS|YES|YES|YES|YES|YES|MANY
---
|CODE|VERSIONED|VERSIONED
---|---|---
|SERVICE|:one:|:two:|
***
***
# II. VERSIONED Dependencies
Explicitly declare and isolate dependencies
---
```notes
What do we do if our requirements change?
Can others protect us from their changes?

lots of forks (`awesome_spawn`, url?)
a few files - hard to version, not declared

data: changing state of external services
2 versions running
data: schema changes via migrations
data: contents of blobs - migrations (schema)

file/message bus = rpc mechanisms
```
|          |VERSIONED|GIT
-----------|------|---------
URL        |YES   |`/v1/`
FORK       |YES   |`Gemspec`, `rpmspec`
CONFIG     |DEPLOY|`.sample.yml`
DATA       |CODE  |`schema.rb` `blob.yml`
FILE       |CODE  |`config/password`
MESSAGE BUS|CODE  |`version: 1`?
CUSTOM     |BUILD |GEM `reports.yml`
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
CODE  |GEM    |gem install, `rake asset:build`
CONFIG|FILE   |Same all customers
BUILD |SERVICE|DOCKER `"FROM"`
CUSTOM|GEM    |Custom properties
CUSTOM|CONFIG |`custom_report.yml`
---
DEPLOY|SERVICE|  |
------|-------|---
CUSTOM|CONFIG |Cloudinit
CUSTOM|GEM    |Custom properties
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
CONSTRAINT|ROOT
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
CONSTRAINT|CODE
----------|---
CONSTRAINT|CONFIG
CHANGING  |SERVICE
CHANGING  |DATA
CONSTRAINT|APPLIANCE
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
CUSTOM|CUSTOM|CUSTOM|NO|NO
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
*[:symbols:]: CODE
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
*[:gem:]: GEM
*[:hotsprings:]: JAVA
*[:art:]: CUSTOM
*[:lock:]: CONSTRAINT
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
