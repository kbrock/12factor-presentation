
# 12 Factor App

DISPOSABLE Disposable Infrastructure
---
When||What|Who
---|---|---|---
2002|JAVA|<cite>*[Patterns of Enterprise Application Architecture]*</cite>|@martinfowler
2007||*[heroku.com]*
2011|RUBY|*[12factor.net]*|@adamwiggins

***
```notes
Defines building blocks of distributed applications
Describes best way to put these blocks together to form components

DISPOSABLE - CHEF

one side: Immutable / disposable
all focus on same building blocks of the system
```
```notes
to make stateless need to cut it up
separate code from configuration from data
separate build from configure from running services

TODO: talk about define configure: service urls, # runners, 

service, runner, build, deploy has own (docker), config (machines), data (# runners)
```
SEPARATE|MANY|STATELESS|SERVICE
---|---|---|---
SEPARATE|CODE|CONFIG|DATA
SEPARATE|BUILD|DEPLOY|RUNNER
SERVICE|URL|VERSION_NUMBER|GIT
CONFIG|SCALE|URL|
***
***
# I. CODE Codebase 
One codebase tracked in revision control, many deploys
---
ONE|SERVICE|&nbsp;
---|---|---
ONE|CODE
ONE|GIT
ONE|VERSION_NUMBER|
---
&nbsp;|build|`console`|vmdb|MANY
---|---|---|---|---
upstream|GIT|GIT|GIT|GIT
downstream|GIT|GIT|GIT|GIT

APPLIANCE COMPUTER   [X. parity](#/13)
---
&nbsp;|build|`console`|vmdb|MANY
---|---|---|---|---
ME|NO|YES|NO|NO
YOU|NO|NO|YES|NO
OTHERS|YES|YES|YES|YES
***
***
# II. VERSION_NUMBER Dependencies
Explicitly declare and isolate dependencies
---
```notes
 code vs config vs data
 tmpl.yml (customer changes) schema
hard to declare config in db / yml dependency
hard to declare data dependencies
tmpl.yml is changed
```
type||example|VERSION_NUMBER
---|---|---|---
CODE|APPLIANCE|`kickstart`, `rpm`|LIKE
CODE|SERVICE|`Gemfile`|LIKE
CONFIG||`ENV[]`, `/defaults`|LIKE
DATA|VERSION_NUMBER|`schema.rb`|DISLIKE
CONFIG|DATA|`tmpl.yml`<small>(migrations)<small>|DISLIKE
CONFIG|FILE|`tmpl.yml` <small>(change)<small>|DISLIKE
***
***
# III. CONFIG Config

Store config in the environment
---
```notes
separation is most important part
perils of overwriting
avoid bulk build time vs runtime
build time vs runtime
```
CONFIG|CODE|WIN
---|---|---
CONFIG|GIT|<small>migrations too</small>
MANY|FILE|`ENV[]`?
CONFIG|CHANGING|Unchanging
CONFIG|MANY|avoid `RAILS_ENV`
BUILD|APPLIANCE|`APACHE_VER`

***
***
# IV. SERVICE Backing Services

Treat backing services as attached resources
---
```notes
  they have urls, versioned, and discoverable
  cut into many services by functionality
```
SERVICE|URL|&nbsp;
---|---|---
SERVICE|VERSION_NUMBER
SERVICE|DISCOVERY|`DNS`
MANY|SERVICE|CODE
---
```notes
lots of forks - are versionable
more performant to move to url model?
a few files - hard to version / declare dependencies
makes sense to convert to config or data
```
FORK|VERSION_NUMBER|LIKE
---|---|---
&nbsp;|URL|SCALE
FILE|VERSION_NUMBER|DISLIKE
&nbsp;|CONFIG|VERSION_NUMBER
MESSAGE BUS|VERSION_NUMBER|DISLIKE
---
MANY|SERVICE|GROOVE
---|---|---
&nbsp;|GROOVE|core competency?
SERVICE|DISPOSABLE|standards
---
```notes
    What does these services look like?
    not a whole appliance
```
SERVICE|UNKNOWN
---|---
APPLIANCE|DISLIKE
RUNNER|LIKE
WORKER|LIKE
console|LIKE
automate|LIKE

service or component

***
***
### V. BUILD Build, DEPLOY release, RUNNER run

Strictly separate build and run stages
---
BUILD|SERVICE|CONFIG|just works
---|---|---|---
CODE|ASSETS|LIKE|`rake asset:build`
CODE|RUBY|LIKE|gem install
FILE|CUSTOM|LIKE|branding
FILE|CUSTOM|LIKE|`custom_report.yml`
FILE|CONFIG|LIKE|Custom properties
---
```notes
when running, no customization, upgrading

```
RUNNER|SCALE|&nbsp;|&nbsp;
---|---|---|---
CODE|VERSION_NUMBER|DISLIKE|rpm upgrade
CODE|RUBY|DISLIKE| gem install
FILE|CUSTOM|DISLIKE|branding
FILE|CONFIG|DISLIKE|self modifying
SECURITY|ROOT|LIKE


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

SERVICE|STATELESS||CONFIG|DATA
---|---|---|---|---
APPLIANCE|STATELESS||CONFIG|UNKNOWN
---
```notes
    rolling upgrades - multiple versions hard
    when there is state, even harder

    create separate service w/ database
    upgrade service easy
    upgrade database hard (urls may help)

    
```
VERSION_NUMBER|VERSION_NUMBER|UNHAPPY|&nbsp;
---|---|---|---|---
SERVICE|DATA|UNHAPPY|UNHAPPY
SERVICE|STATELESS|HAPPY
SERVICE|DATA|VERSION_NUMBER
DATA|URL|VERSION_NUMBER|VERSION_NUMBER
VERSION_NUMBER|VERSION_NUMBER|HAPPY
---
```notes
What can change?
putting state into database

code is versioned
config var names is versioned by the code that calls it
code drived by config is versioned
data in database is not well versioned migrations are crude
(have to reimplement) - automate
should only boot once
```
STATELESS||change?|VERSION_NUMBER|&nbsp;
---|---|---|---|---
CODE|FILE|BUILD|YES
CONFIG|cloud-init|ONE|YES|CODE
DATA|database|YES|NO|migrations
CONFIG|DATA|YES|NO|etcd
CODE|DATA|YES|NO|automate
***
***
# VII. URL Port binding

Export services via port binding
---
```notes
    prefer dns over mod_rewrite
```
URL|YES|&nbsp;
---|---|---
MESSAGE BUS|NO
FORK|NO
FILE|NO
DISCOVERY|YES|`DNS`
***
***
# VIII. SCALE Concurrency

Scale out via the process model
---
```notes
can fork, but new processes (new boxes) important
```
SCALE|&nbsp;|&nbsp;
---|---|---
SERVICE|FORK
SERVICE|COMPUTER|COMPUTER
---
```notes
separate service from runner
pids are job of runner
```
SERVICE|RUNNER
---|---
`PID`|DISLIKE
***
***
# IX. DISPOSABLE Disposability

Maximize robustness with fast startup and graceful shutdown
---
```notes
easy to discover / add workers (and remove)
capacity planning - react to dynamic load
    Pet vs Cattle
```
&nbsp;|SCALE|&nbps;
---|---|---
&nbsp;|YES|startup instantly
CLEANUP|NO
METRICS|YES|dynamic load
PET|NO
PET2|NO
DISPOSABLE|YES
---
```notes
guid locks an image to work
no one else can come in
want to discover and do work easily
separate runner from worker
simplify specification of work
```
SCALE||DISPOSABLE
---|---|---
CONFIG|`GUID`|PET
CONFIG|FILE|PET
RUNNER|WORKER|DISPOSABLE
DISCOVERY|WORKER|DISPOSABLE

region/zone/capability
***
***
# X. Dev/prod parity

Keep development, staging, and production as similar as possible
---
dev|prod
---|---
postgres|postgres
rack|apache
rails|rake / service
simulate|WORKER
postgres|IdM, AD, LDAP, postgres
GIT|rpm
rake|appliance_console
nope|replication
---
```notes
Use same scripts for build, qa, and dev

```
qe|prod
---|---
APPLIANCE|APPLIANCE
python|`appliance_console`
PET|FOOD
***
***
# XI. LOGS Logs

Treat logs as event streams
---
LOGS|&nbsp;
---|---
`miq_top`|splunk
`/var/log/httpd/`|RHCI
vmware events|event collector
METRICS|define normal

***
***
# XII. Admin processes

Run admin/management tasks as one-off processes
---
Convert one off tasks into code

Not saying 100% polished ruby, do bash

e.g.: sql in production -> migration
***
# Refs
- [12factor.net] from [heroku.com]
- [Immutable Infrastructure]
- [Pivotal podcast](http://www.paasmag.com/2014/11/19/all-things-pivotal-episode-7-a-look-at-12-factor-apps/)
- [docker 12 factor](http://www.slideshare.net/williamyeh/12-factor-app-from-dockers-point-of-view)
- [reveal-ck](https://github.com/jedcn/reveal-ck) slides
---
```notes    
? hear no evil, see no evil, say no evil
? ticket, train station, fuelpump
? scisors vs barber
*[::]: TERMINAL (console)
*[:trollface:]: ENEMY/BAD - :snake: :bomb: (do WIN instead)
*[:checkered_flag:]: end goal?
FIX: dog food (not pet + food)
FIX: runner (not runner)
FIX: deploy (not wrench)
ALT changing:  / wild / joker / slotmachine
DELETE: gem / art / assets / cleanup
DELETE: computer vs developer (maybe use appliance instead?)

MERGE: groove -> win -> 
MERGE: like/dislike -> yes/no OR good/bad?
MERGE: happy/unhappy -> like/dislike -> good/bad?
MERGE: STATE -> DOG ?
MERGE: STATELESS -> CATTLE ?
overloading:
appliance: app, web app, appliance (runner or aggregate service?)
service: component, service, exe
scale: scale, concurrency, performance
ruby: gem, ruby, web tech
constrain: Lock down, secure, constrain
``` 

*[:ballot_box_with_check:]: YES
*[:white_medium_square:]: NO
*[:+1:]: LIKE
*[:-1:]: DISLIKE
*[:construction_worker:]: WORKER
*[:symbols:]: CODE
*[:abcd:]: CONFIG
*[:dvd:]: DATA
*[:earth_americas:]: STATELESS
*[:cloud:]: SERVICE
*[:octocat:]: GIT
*[:hash:]: VERSION\_NUMBER
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
*[:bento:]: FOOD
*[:person_with_blond_hair:]: ME
*[:girl:]: YOU
*[:boy:]: OTHERS
*[:cop:]: ROOT
*[:bus:]: MESSAGE BUS
*[:file_folder:]: FILE
*[:notebook:]: DISCOVERY
*[:chart_with_upwards_trend:]: METRICS
*[:fork\_with\_knife:]: FORK
*[:black_joker:]: CHANGING
*[:metal:]: WIN
*[:computer:]: COMPUTER
*[:frowning:]: UNHAPPY
*[:musical_note:]: GROOVE
*[:gem:]: RUBY
*[:hotsprings:]: JAVA
*[:page_facing_up:]: ASSETS
*[:art:]: CUSTOM
*[:grey_question:]: UNKNOWN
*[:evergreen_tree:]: LOGS
*[:lock:]: CONSTRAINT
*[:grining:]: HAPPY
*[:bathtub:]: CLEANUP
*[:scissors:]: SEPARATE

[Patterns of Enterprise Application Architecture]: http://amzn.com/0321127420
[12factor.net]: http://12factor.net
[heroku.com]: http://heroku.com/
[Immutable Infrastructure]: http://chadfowler.com/blog/2013/06/23/immutable-deployments/
