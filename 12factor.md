<!-- https://github.com/jedcn/reveal-ck -->
# 12 Factor App

Immutable Infrastructure
---
- Description
- Relevence
- Factors
- Implications
***
## Set of Best Practices

Adam Wiggins @hirodusk

<small>inspiration:</small>

*Patterns of Enterprise Application Architecture*

@martinfowler
---
Popular for SaaS / PaaS

provider|like|strict
---|---|---
@heroku|:+1:|YES
@cloudfoundary|:+1:|:ballot_box_with_check:
@openshift|:+1:|:white_medium_square:
@docker|:+1:|:white_medium_square:

*[:ballot_box_with_check:]: YES
---
<!--
separate code from configuration from data
create many stateless services
separte build from running services to get performance

NOTE: I mixup git and versioning
-->
:symbols:|:abcd:|:floppy_disk:
---|---|---
:100:|:earth_americas:|:cloud:
:octocat:|:globe_with_meridians:|:hash:
:construction:|:runner:|:car:
***
***
We are a big enterprise application

forget @docker

shift **towards** service and components

follow an approperiate methodology
---
:person_with_blond_hair:||:+1:
---|---|---
:abcd:|:earth_americas:|install / upgrade <small>U/X</small>
:abcd:|:floppy_disk:|multi tenancy
:100:|:cloud:|simplify <small>technical debt</small>
:cloud:|:lock:|security
:girl:|:grey_question:|:+1:
---
### Just add Salt

Ask|&nbsp;
---|---
*Follow 100%?*|:white_medium_square:
*Why deviate?*|:ballot_box_with_check:
***
***
# I. :symbols: Codebase 
One codebase tracked in revision control, many deploys
---
<!-- one app, code, repo, version
upgrading service
:two:|:hash:|<small>upgrades</small>
-->
:one:|:cloud:|&nbsp;
---|---|---
:one:|:symbols:
:one:|:octocat:
:one:|:hash:|
---
&nbsp;|build|`console`|vmdb|:100:
---|---|---|---|---
upstream|:octocat:|:octocat:|:octocat:|:octocat:
downstream|:octocat:|:octocat:|:octocat:|:octocat:

:tv: :computer:   [X. parity](#/13)
---
&nbsp;|build|`console`|vmdb|:100:
---|---|---|---|---
:person_with_blond_hair:|:ballot_box_with_check:|:white_medium_square:|:white_medium_square:|:white_medium_square:
:girl:|:white_medium_square:|:ballot_box_with_check:|:white_medium_square:|:white_medium_square:
:construction_worker:|:ballot_box_with_check:|:ballot_box_with_check:|:ballot_box_with_check:|:ballot_box_with_check:
***
***
# II. :hash: Dependencies
Explicitly declare and isolate dependencies
---
<!-- code vs config vs data
 tmpl.yml (customer changes) schema
hard to declare config in db / yml dependency
hard to declare data dependencies
tmpl.yml is changed
-->
type||example|:octocat:
---|---|---|---
:symbols:|:tv:|`kickstart`, `rpm`|:+1:
:symbols:|:cloud:|`Gemfile`|:+1:
:abcd:||`ENV[]`, `/defaults`|:+1:
:floppy_disk:|:hash:|`schema.rb`|:-1:
:abcd:|:floppy_disk:|`tmpl.yml`<small>(migrations)<small>|:-1:
:abcd:|:file_folder:|`tmpl.yml` <small>(change)<small>|:-1:
***
***
# III. :abcd: Config

Store config in the environment
---
<!-- separation is most important part
perils of overwriting
avoid bulk build time vs runtime
build time vs runtime -->
:abcd:|:symbols:|:metal:
---|---|---
:abcd:|:octocat:|<small>migrations too</small>
:100:|:file_folder:|`ENV[]`?
:abcd:|:black_joker:|Unchanging
:abcd:|:100:|avoid `RAILS_ENV`
:construction:|:tv:|`APACHE_VER`

***
***
# IV. :cloud: Backing Services

Treat backing services as attached resources
---
<!--
  they have urls, versioned, and discoverable
  cut into many services by functionality
-->
:cloud:|:globe_with_meridians:|&nbsp;
---|---|---
:cloud:|:hash:
:cloud:|:notebook:|`DNS`
:100:|:cloud:|:symbols:
---
<!--
lots of forks - are versionable
more performant to move to url model?
a few files - hard to version / declare dependencies
makes sense to convert to config or data
-->
:fork_and_knife:|:hash:|:+1:
---|---|---
&nbsp;|:globe_with_meridians:|:car:
:file_folder:|:hash:|:-1:
&nbsp;|:abcd:|:hash:
:bus:|:hash:|:-1:
---
:100:|:cloud:|:musical_note:
---|---|---
&nbsp;|:musical_note:|core competency?
:cloud:|:recycle:|standards
---
<!--
    What doe these services look like?
    not a whole appliance
-->
:cloud:|:grey_question:|&nbsp;
---|---|---
:tv:|:-1:|:construction:
:runner:|:+1:
:construction_worker:|:+1:|worker
:wrench:|:+1:|console
:wrench:|:+1:|automate

service or component

***
***
### V. :construction: Build, :octocat: release, :runner: run

Strictly separate build and run stages
---
:construction:|:cloud:|:abcd:|just works
---|---|---|---
:symbols:|:page_facing_up:|:+1:|`rake asset:build`
:symbols:|:gem:|:+1:|gem install
:file_folder:|:art:|:+1:|branding
:file_folder:|:art:|:+1:|`custom_report.yml`
:file_folder:|:abcd:|:+1:|Custom properties
---
<!--
when running, no customization, upgrading

-->
:runner:|:car:|&nbsp;|&nbsp;
---|---|---|---
:symbols:|:hash:|:-1:|rpm upgrade
:symbols:|:gem:|:-1:| gem install
:file_folder:|:art:|:-1:|branding
:file_folder:|:abcd:|:-1:|self modifying
:lock:|:cop:|:+1:


***
***
# VI. :earth_americas: Processes

Execute the app as one or more stateless processes
---
<!--
service is stateless
state goes to data or config
applinace is stateless (if appliance is a service vs runner)
-->

:cloud:|:earth_americas:||:abcd:|:floppy_disk:
---|---|---|---|---
:tv:|:earth_americas:||:abcd:|:grey_question:
---
<!--
    rolling upgrades - multiple versions hard
    when there is state, even harder

    create separate service w/ database
    upgrade service easy
    upgrade database hard (urls may help)

    
-->
:hash:|:hash:|:frowning:|&nbsp;
---|---|---|---|---
:cloud:|:floppy_disk:|:frowning:|:frowning:
:cloud:|:earth_americas:|:grinning:
:cloud:|:floppy_disk:|:hash:
:floppy_disk:|:globe_with_meridians:|:hash:|:hash:
:hash:|:hash:|:grinning:
---
<!--
What can change?
putting state into database

code is versioned
config var names is versioned by the code that calls it
code drived by config is versioned
data in database is not well versioned migrations are crude
(have to reimplement) - automate
should only boot once
-->
:earth_americas:||change?|:hash:|&nbsp;
---|---|---|---|---
:symbols:|:file_folder:|:construction:|:ballot_box_with_check:
:abcd:|cloud-init|:one:|:ballot_box_with_check:|:symbols:
:floppy_disk:|database|:ballot_box_with_check:|:white_medium_square:|migrations
:abcd:|:floppy_disk:|:ballot_box_with_check:|:white_medium_square:|etcd
:symbols:|:floppy_disk:|:ballot_box_with_check:|:white_medium_square:|automate
***
***
# VII. :globe_with_meridians: Port binding

Export services via port binding
---
<!--
    use dns over mod_rewrite
-->
:globe_with_meridians:|:ballot_box_with_check:
---|---
:wrench:|:bus:
:wrench:|:fork_and_knife:
:wrench:|:file_folder:
:notebook:|`DNS`
***
***
# VIII. :car: Concurrency

Scale out via the process model
---
<!--
can fork, but new processes (new boxes) important
-->
:car:|&nbsp;|&nbsp;
---|---|---
:cloud:|:fork_and_knife:
:cloud:|:computer:|:computer:
---
<!--
separate service from runner
pids are job of runner
-->
:cloud:|:runner:
---|---
`PID`|:-1:
***
***
# IX. :recycle: Disposability

Maximize robustness with fast startup and graceful shutdown
---
<!--
easy to discover / add workers (and remove)
capacity planning - react to dynamic load
    Pet vs Cattle
-->
:car:|:+1:|startup instantly
---|---|---
:bathtub:|:-1:|no cleanup
:chart_with_upwards_trend:|:+1:|dynamic load
:dog:|:-1:|:cat:
:cow:|:+1:
---
<!--
guid locks an image to work
no one else can come in
want to discover and do work easily
separate runner from worker
simplify specification of work
-->
:car:||:cow:
---|---|---
:abcd:|`GUID`|:dog:
:abcd:|:file_folder:|:dog:
:runner:|:construction_worker:|:cow:
:notebook:|:construction_worker:|:cow:

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
simulate|:construction_worker:
postgres|IdM, AD, LDAP
git|rpm
rake|appliance_console
nope|replication
---
<!--
Use same scripts for build, qa, and dev

dog food
-->
qe|prod
---|---
:tv:|:tv:
python|`appliance_console`
:dog:|:bento:
***
***
# XI. :evergreen_tree: Logs

Treat logs as event streams
---
:evergreen_tree:|&nbsp;
---|---
`miq_top`|splunk
`/var/log/httpd/`|RHCI
vmware events|event collector
:chart_with_upwards_trend:|define normal

***
***
# XII. Admin processes

Run admin/management tasks as one-off processes
---
Convert one off tasks into code

Not saying 100% polished ruby, do bash

e.g.: sql in production -> migration
***
:person_with_blond_hair:

multi-tenancy: moving more config into data
***
image|term
---|---
:symbols:|code
:abcd:|config
:floppy_disk:|data
:tv:|appliance
:cloud:|service
:file_folder:|file on disk
---
image|term
---|---
:person_with_blond_hair:|developer
:girl:|developer
:construction_worker:|jason
:construction:|build
:cop:|root user
:octocat:|git
---
image|term
---|---
:hash:|version number
:globe_with_meridians:|url
:notebook:|lookup (dns)
:bus:|message queue
:car:|scale
:earth_americas:|stateless
---
image|term
---|---
:construction_worker:|worker
:wrench:|console
:wrench:|tool
:recycle:|disposable
---
# Refs
- [12factor app](http://12factor.net/)
- [immutable infrastructure](http://chadfowler.com/blog/2013/06/23/immutable-deployments/)
- [Pivotal podcast](http://www.paasmag.com/2014/11/19/all-things-pivotal-episode-7-a-look-at-12-factor-apps/)
- [docker 12 factor](http://www.slideshare.net/williamyeh/12-factor-app-from-dockers-point-of-view)
***
