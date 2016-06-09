***

|||
|---|---|
|Ruby|slow|
|DB|fast<small>*</small>|
|use|Rails|

<small>* except when it's not</small>

---

I love ruby.
It is expressive.
It is easy to write code.
But we send all this data over the wire. And this makes it slow.

Now, the database is fast
And, it is very powerful
But not all of us know how SQL
And not all of us like convoluted "optimized" code

And even with this convoluted code, you have to be so vigilent to try and avoid pitfalls
but we all fall into them.

Lets extend rails.
And lets demand more from rails.
The code and performance we have been getting up until now is atrocious
But while we need to improve rails
We really also need to clean up our code
***

||
|---|
|Keenan|
|Performance team|
|@kbrock|
***
DB Layout

```dot
graph [ fontname="helvetica-bold" bgcolor="transparent"]
node  [ id="\N" shape="Mrecord" style="filled" fontname="helvetica" color="#d4d4d4" fillcolor="transparent" penwidth="1.2" fontcolor="#d4d4d4"]
edge  [ arrowsize="0.5" fontname="helvetica" color="#d4d4d4"]

ems   [ label="Ems (1)" shape="box"]
hosts [ label="Hosts (200)" shape="box"]
vms   [ label="VMs (10,000)" shape="box"]
ems -> hosts
hosts -> vms [ constraint="false"]
{ rank=same ; vms hosts}

hosts -> hardwares
hardwares [ label = "{Hardwares (10,200)|cores*|cpu*|(aggregate_cpu_speed)*}"]

opt [ label = "{* is optional|() is calculated}" penwidth="0.2" shape="record"]
hardwares -> opt [ color="transparent"]
```

---

Db Layout<small>*</small>

```ruby
class ExtManagementSystem < ActiveRecord::Base
  has_many :hosts
end

class Host < ActiveRecord::Base
  has_one :hardware
end

class Hardware < ActiveRecord::Base
  def aggregate_cpu_speed
    cpu_total_cores * cpu_speed if cpu_total_cores && cpu_speed
  end
end
```
<small>* Ruby version</small>
***
***

Not using `:through`

```ruby
class ExtManagementSystem
  def aggregate_cpu_total_cores
    hosts.map(&:hardware).map(&:cpu_total_cores).compact.sum
  end
end
```
<!--
  ems = ExtManagementSystem.first ; bookend("query") { ems.aggregate_cpu_total_cores }
-->
|     ms | bytes | objects |queries | query(ms) |     rows
|    ---:|   ---:|   ---:|  ---:|   ---:|      ---:
|  257.5 | 14,990,187 | 172,227 |  201 |  32.3 |      400

---

Optimizing not using `:through`

```ruby
class ExtManagementSystem
  def aggregate_cpu_total_cores
    MiqPreloader.preload(hosts, :hardware)
    hosts.map(&:hardware).map(&:cpu_total_cores).sum
  end
end
```
<!--
  ems = ExtManagementSystem.first ; bookend("query") { ems.aggregate_cpu_total_cores }
-->

|     ms | bytes | objects |queries | query(ms) |     rows |
|    ---:|   ---:|   ---:|  ---:|   ---:|      ---:|
|   63.5 | 2,223,793 | 17,048 |    2 |   9.9 |      400 |
---

Flipping not using `:through`

```ruby
class ExtManagementSystem
  def aggregate_cpu_total_cores
    Hardware.where(:host_id => host_ids).pluck(:cpu_total_cores).sum
  end
end
```
<!--
  ems = ExtManagementSystem.first ; bookend("query") { ems.aggregate_cpu_total_cores }
-->

|     ms | bytes | objects |queries | query(ms) |     rows |
|    ---:|   ---:|   ---:|  ---:|   ---:|      ---:|
|   50.3 | 225,818 | 4,176 |    2 |   2.1 |     400   |
***

Using `:through`

```ruby
class ExtManagementSystem
  has_many :hardwares, :through => :hosts

  def aggregate_cpu_total_cores
    hardwares.map(&:cpu_total_cores).sum
  end
end
```
<!--
  ems = ExtManagementSystem.first ; bookend("query") { ems.aggregate_cpu_total_cores }
-->

|     ms | bytes | objects |queries | query(ms) |     rows |
|    ---:|   ---:|   ---:|  ---:|   ---:|      ---:|
|   54.0 | 767,267 | 4,541 |    1 |   5.9 |      200 |

---

Not using `sum`

```ruby
class ExtManagementSystem
  has_many :hardwares, :through => :hosts
  def aggregate_cpu_total_cores
    hardwares.map(&:cpu_total_cores).sum
  end
end
```
<!--
  ems = ExtManagementSystem.first ; bookend("cores-third") { ems.aggregate_cpu_total_cores }
-->

|       ms |   bytes | objects |queries | query(ms) |     rows |
|      ---:|     ---:|     ---:|  ---:|     ---:|      ---:|
|     34.3 | 767,267 |   4,541 |    1 |     4.2 |      200 |

---

Using `sum`

```ruby
class ExtManagementSystem
  has_many :hardwares, :through => :hosts
  def aggregate_cpu_total_cores
    hardwares.sum(:cpu_total_cores)
  end
end
```
<!--
  ems = ExtManagementSystem.first ; bookend("cores-third") { ems.aggregate_cpu_total_cores }
-->

|       ms |   bytes | objects |queries | query(ms) |     rows |
|      ---:|     ---:|     ---:|  ---:|     ---:|      ---:|
|     30.1 |  96,355 |   1,242 |    1 |     0.6 |          |
---
Not using `sum`<small>*</small>

```ruby
class ExtManagementSystem
  def aggregate_cpu_speed
    hardwares.map(&:aggregate_cpu_speed).sum
  end
end

class Hardware
  def aggregate_cpu_speed
    (cpu_total_cores * cpu_speed) if cpu_total_cores && cpu_speed
  end
end
```
<!--
  ems = ExtManagementSystem.first ; bookend("cores-third") { ems.aggregate_cpu_speed }
-->
<small>* Virtual Attributes</small>

|       ms |   bytes | objects |queries | query(ms) |     rows |
|      ---:|     ---:|     ---:|  ---:|     ---:|      ---:|
|     41.5 | 783,499 |   4,744 |    1 |     3.7 |      200 |
***

Using `:sum`<small>*<small>

```ruby
class ExtManagementSystem
  def aggregate_cpu_speed
    hardwares.sum(:aggregate_cpu_speed)
  end
end

class Hardware
  virtual_attribute :aggregate_cpu_speed, :integer,
    :arel => ->(t) {t.grouping(t[:cpu_total_cores] * t[:cpu_speed])}

  def aggregate_cpu_speed
    (cpu_total_cores * cpu_speed) if cpu_total_cores && cpu_speed
  end
end
```
<!--
  ems = ExtManagementSystem.first ; bookend("cores-third") { ems.aggregate_cpu_speed }
-->
<small>* Virtual Attributes</small>

|       ms |   bytes | objects |queries | query(ms) |     rows |
|      ---:|     ---:|     ---:|  ---:|     ---:|      ---:|
|     31.0 |  97,147 |   1,251 |    1 |     0.5 |        1 |

---
Actual<small>*</small>

```ruby
class ExtManagementSystem
  def aggregate_cpu_speed
    hardwares.sum(Hardware.arel_attribute(:aggregate_cpu_speed))
  end
end
```
<small>* outstanding PR</small>

|       ms |   bytes | objects |queries | query(ms) |     rows |
|      ---:|     ---:|     ---:|  ---:|     ---:|      ---:|
|     31.0 |  97,147 |   1,251 |    1 |     0.5 |        1 |

---

Not using `where`, `count`

```ruby
class ExtManagementSystem
  def vm_count_by_state(state)
    vms.inject(0) { |t, vm| vm.power_state == state ? t + 1 : t }
  end
end
class VmOrTemplate
  def power_state
    POWER_STATES[raw_power_state] || 'unknown'
  end
end
```
<!--
  ems = ExtManagementSystem.first ; bookend("cores-third") { ems.vm_count_by_state("off") }
-->

|       ms |   bytes | objects |queries | query(ms) |     rows |
|      ---:|     ---:|     ---:|  ---:|     ---:|      ---:|
|  1,000.1 | 17,704,609+| 622,669+ |    1 |   772.7 |   10,000 |

<small>+ 143,533 freed objects</small>
---

Using `count`

```ruby
class ExtManagementSystem
  POWER_STATES= {"poweredOff"=>"off", "poweredOn"=>"on"}
  def vm_count_by_state(state)
    raw_power_state = POWER_STATES.invert[state]
    vms.where(:raw_power_state => raw_power_state).count
  end
end
class VmOrTemplate
  def power_state
    POWER_STATES[raw_power_state] || 'unknown'
  end
end
```
<!--
  ems = ExtManagementSystem.first ; bookend("cores-third") { ems.vm_count_by_state("off") }
-->

|       ms |   bytes | objects |queries | query(ms) |     rows |
|      ---:|     ---:|     ---:|  ---:|     ---:|      ---:|
|     43.3 | 137,607 |   1,774 |    1 |     6.1 |        1 |

***

Using RBAC<br>
Recipies to avoid

---
RBAC implementation<small>*</small>

```ruby
class Rbac
  def self.search(options)
    targets = options[:targets]
    if targets.kind_of(Array)
      targets = targets.first.class.where(:id => targets) # <= HERE
    end
    targets.where(magic)
  end

  def self.filtered(targets, options)
    search(options.merge(:targets => target))
  end
end
```
<small>* Not really</small>
---
Base Code

```ruby
class Code
  def calling_method
    do_stuff(VmOrTemplate.where(:boot_time => nil)).to_a ; nil
  end

  def do_stuff(targets)
    Rbac.filtered(targets)
  end
end
```
<!--
  bookend { Code.new.calling_method }
-->

|       ms |   bytes | objects |queries | query(ms) |     rows |
|      ---:|     ---:|     ---:|  ---:|     ---:|      ---:|
|    834.8 | 66,396,630 | 482,710 |    4 |   748.9 |   10,000 |

---
"optimized" code calls *

```ruby
class Code
  def calling_method
    do_stuff(VmOrTemplate.where(:boot_time => nil)).to_a ; nil
  end

  def do_stuff(targets)
    return if targets.blank?
    Rbac.filtered(targets)
  end
end
```
<!--
  bookend { Code.new.calling_method }
-->

|       ms |   bytes | objects |queries | query(ms) |     rows |
|      ---:|     ---:|     ---:|  ---:|     ---:|      ---:|
|  1,536.3 | 66,416,957 | 963,491+ |    5 | 1,443.3 |   20,000 |

<small>+ 278,052 freed objects</small>
<small>* for some definition of optimized</small>
---

***
this includes:

```ruby
class Code
  def calling_method
    do_stuff(VmOrTemplate.where(:boot_time => nil)).to_a ; nil
  end

  def do_stuff(targets)
    targets = Array.wrap(targets)
    Rbac.search(:targets => targets)
  end
end
```
<!--
  bookend { Code.new.calling_method }
-->

|       ms |   bytes | objects |queries | query(ms) |     rows |
|      ---:|     ---:|     ---:|    ---:|     ---:|      ---:|
|  2,031.3 | 123,483,752 | 1,013,499+ |5 | 1,549.4 |   20,000 |

<small>+ 252,994 freed objects</small>

---

- `to_a`
- `compact`
- `blank?`, `any?`, `empty?`, `count`
- `Array.wrap()`
- `first.class`

---

Not using `OR`

```ruby
class Code
  def calling_method
    targets = Host.where(:smart => 1)
    targets += Host.where(:power_state => "on")
    targets = targets.uniq.compact
    Rbac.filtered(targets)
  end
end
```
<!--
  bookend { Code.new.calling_method }
-->

|     ms | bytes | objects |queries | query (ms) |     rows |
|    ---:|   ---:|   ---:|  ---:|   ---:|      ---:|
|   53.4 | 3,034,615 | 22,888 |    6 |  20.3 |      600 |
---
Using `OR`

```ruby
class Code
  def calling_method
    targets = Host.where(:smart => 1).or(
              Host.where(:power_state => "on"))
    Rbac.filtered(targets)
  end
end
```
<!--
  bookend { Code.new.calling_method }
-->

|     ms | bytes | objects |queries | query(ms) |     rows |
|    ---:|   ---:|   ---:|  ---:|   ---:|      ---:|
|   29.4 | 171,010 | 2,220 |    3 |   0.8 |          |

***

Not using `AND`

```ruby
class Code
  def calling_method
    possible_targets = [Host.where(:smart => 1),
                        Host.where(:power_state => "on")]
    targets = possible_targets.inject(nil) do |a, hosts|
      a.nil? ? hosts : a & hosts
    end
    # Rbac.filtered(targets)
  end
end
```
<!--
  bookend { Code.new.calling_method }
-->

|     ms | bytes | objects |queries | query(ms) |     rows |
|    ---:|   ---:|   ---:|  ---:|   ---:|      ---:|
|   42.0 | 1,886,341 | 13,673 |    2 |  12.6 |      400 |

---
Not using `AND` with RBAC

```ruby
class Code
  def calling_method
    possible_targets = [Host.where(:smart => 1),
                        Host.where(:power_state => "on")]
    targets = possible_targets.inject(nil) do |a, hosts|
      a.nil? ? hosts : a & hosts
    end
    Rbac.filtered(targets)
  end
end
```
<!--
  bookend { Code.new.calling_method }
-->

|     ms | bytes | objects |queries | query(ms) |     rows |
|    ---:|   ---:|   ---:|  ---:|   ---:|      ---:|
|   58.5 | 3,031,237 | 22,895 |    6 |  21.0 |      600 |

***
***

Using `AND`

```ruby
class Code
  def calling_method
    targets = Host.where(:smart => 1, :power_state => "on")
    # Rbac.filtered(targets).to_a
  end
end
```
<!--
  bookend { Code.new.calling_method }
-->

|     ms | bytes | objects |queries | query(ms) |     rows |
|    ---:|   ---:|   ---:|  ---:|   ---:|      ---:|
|   34.9 | 887,308 | 6,668 |    1 |   6.0 |      200 |

---
Using `AND` with RBAC

```ruby
class Code
  def calling_method
    targets = Host.where(:smart => 1, :power_state => "on")
    Rbac.filtered(targets).to_a
  end
end
```
<!--
  bookend { Code.new.calling_method }
-->

|     ms | bytes | objects |queries | query(ms) |     rows |
|    ---:|   ---:|   ---:|  ---:|   ---:|      ---:|
|   34.2 | 1,042,494 | 8,590 |    4 |   6.5 |      200 |

***

- DBMS `id[]`
- DBMS sort by ruby column (w/ limit / offset)
- DBMS `MiqExpression :supported_by_sql => false`

***

Not using subqueries

```ruby
class ExtManagementSystem < ActiveRecord::Base
  def aggregate_cpu_total_cores
    ids = hosts.pluck(:id)
    Hardware.where(:host_id => ids).pluck(:cpu_total_cores).sum
  end
end
```

---

```ruby
class ExtManagementSystem < ActiveRecord::Base
  def aggregate_cpu_total_cores
    ids = hosts.select(:id).collect(&:id)
    Hardware.where(:host_id => ids).pluck(:cpu_total_cores).sum
  end
end
```

***
Using subqueries

```ruby
class ExtManagementSystem < ActiveRecord::Base
  def aggregate_cpu_total_cores
    ids = hosts.select(:id)
    Hardware.where(:host_id => ids).pluck(:cpu_total_cores).sum
  end
end
```

---

- RUBY `MiqPreloader.preload`
- RUBY `sort` (w/o limit)

---

*[:suspect:]: CULPRIT
*[:mortar_board:]: EXPERT
*[:gem:]: RUBY
*[:dvd:]: DBMS
*[:negative_squared_cross_mark:]: NO
*[:white_check_mark:]: YES
