## ActiveRecord Associations
A study of ActiveRecord's Associations and their generated methods.

#####Questions: What methods are generated for each association type? What is the required database column in the matching table? How do the args passed to the `initialize` method affect the situation?

We'll experiment with four models: Game, World, Character, and PowerUp.

A World `has_many` characters and powerups. We're looking for the following behaviour:
 - `mushroom_kingdom.characters =>[...]`
 - `mushroom_kingdom.power_ups => [...]`
 
Characters and Powerups both `belong_to` a World.
 - `mario.world => #<World ...>`
 - `fire_flower.world => #<World ...>`

A Character `has_many` powerups available `through` the world. 
 - `mario.power_ups => [...]`

A Powerup `belongs_to` _many_ characters. 
 - `fire_flower.characters => [...]`

Additionally, a Game has many characters but a character also has many games.
 - `mario.games => [...]`
 - `game.characters => [...]`
 
##Control
First we need a control model to compare our results against. This will be a blank class that inherits from ActiveRecord. The only column it will have will be its automatically generated `id` column.
```ruby
class Blank < ActiveRecord::Base
end

class CreateBlanksTable < ActiveRecord::Migration
  def change
    create_table :blanks do |t|
    end
  end
end
```

```ruby
control = Blank.new
=> #<Blank id: nil>
control_methods = control.methods.map {|m| m.to_s}.sort!
control_class_methods = Blank.methods.map {|m| m.to_s}.sort!
```
The list of control methods can be found [here](https://github.com/MooseBoost/ActiveRecordAssociations/blob/master/control_instance_methods.md "Instance Methods") and [here](https://github.com/MooseBoost/ActiveRecordAssociations/blob/master/control_class_methods.md "Class Methods").

##World
```ruby
class World < ActiveRecord::Base
  has_many :characters
end
```

Create a corresponding table with a column for the name of the world.
Note that the _plural_ version of the class is used for the table.
```ruby
class CreateWorldTable < ActiveRecord::Migration
  def change
    create_table :worlds do |t|
      t.string :name
    end
  end
end
```

Migrate the change over to the database.
```ruby
rake db:migrate
```

`db/schema.rb` now has the following:
```ruby
ActiveRecord::Schema.define(version: 20160405222201) do

  create_table "worlds", force: :cascade do |t|
    t.string "name"
  end

end
```

Let's now create a world that has no name. It will be a simple instance, with no attributes.
```ruby
>> the_world = World.create
D, [2016-04-05T22:33:05.639088 #3388] DEBUG -- :    (0.2ms)  begin transaction
D, [2016-04-05T22:33:05.644635 #3388] DEBUG -- :   SQL (0.5ms)  INSERT INTO "worlds" DEFAULT VALUES
D, [2016-04-05T22:33:05.657346 #3388] DEBUG -- :    (12.0ms)  commit transaction
=> #<World id: 1, name: nil>
```

We are now ready to make our comparisons.

###A _World_ that `has_many` _characters_
Note: The list of methods was created like this:
```ruby
>> world_methods = the_world.methods.map {|m| m.to_s}.sort!
>> (world_methods - control_methods).each {|m| puts m}

>> world_class_methods = World.methods.map {|m| m.to_s}.sort!
>> (world_class_methods - control_class_methods).each {|m| puts m}
```

Again, here is our model and its table.
```ruby
class World < ActiveRecord::Base
  has_many :characters
end
```
```ruby
class CreateWorldTable < ActiveRecord::Migration
  def change
    create_table :worlds do |t|
      t.string :name
    end
  end
end
```

#####Generated `has_many` Methods
From `has_many(:characters)` | From DB column :name | Class Methods
--- | --- | ---
after_add_for_characters | name | after_add_for_characters
after_add_for_characters= | name= | after_add_for_characters=
after_add_for_characters? | name? | after_add_for_characters?
after_remove_for_characters | name_before_type_cast | after_remove_for_characters
after_remove_for_characters= | name_came_from_user? | after_remove_for_characters=
after_remove_for_characters? | name_change | after_remove_for_characters?
autosave_associated_records_for_characters | name_changed? |
before_add_for_characters | name_was | before_add_for_characters
before_add_for_characters= | name_will_change! | before_add_for_characters=
before_add_for_characters? | reset_name! | before_add_for_characters?
before_remove_for_characters | restore_name! | before_remove_for_characters
before_remove_for_characters= | | before_remove_for_characters=
before_remove_for_characters? | | before_remove_for_characters?
character_ids | |
character_ids= | |
characters | |
characters= | |
validate_associated_records_for_characters | |

As you can see, ActiveRecord generates instance methods based on both names of the columns in the database and the various association macros. There are other methods that can be called on the results of these methods such as `characters.clear`. A list can be found [here](http://api.rubyonrails.org/classes/ActiveRecord/Associations/ClassMethods.html "Generated Methods")



###A _Character_ `belongs_to` a _world_
If a model is a child of another model it gets a `belongs_to` association. If a model has two or more parents it can belong to all of them through multiple `belongs_to` method calls. Another way of looking at the `belongs_to` association is to ask youself if you want this kind of method: `child.dad => #<Dad...>` or `child.mom => #<Mom...>`

If a model is a child of anything, it requires a parent_id column to be added to it's corresponding table. Any column with a `_id` suffix becomes a foreign key that points to the corresponding table's id column. So, if a character is a child of world, it will need a world_id column to point to the `worlds` table `id` column.

Here is our Character model.
```ruby
class Character < ActiveRecord::Base
  belongs_to :world
end
```
```ruby
class CreateCharactersTable < ActiveRecord::Migration
  def change
    create_table :characters do |t|
      t.string :name
      t.integer :world_id
    end
  end
end
```

Here is a nameless character.
```ruby
>> the_character = Character.new
=> #<Character id: nil, name: nil, world_id: nil>
>> character_methods = the_character.methods.map {|method| method.to_s}.sort!
>> (character_methods - control_methods).each {|m| puts m}
```
#####Generated `belongs_to` methods
From `belongs_to(:world)` | From DB column :name| Class Methods
--- | --- | ---
world | name | **none**
world= | name= | 
autosave_associated_records_for_world | name? | 
belongs_to_counter_cache_after_update | name_before_type_cast | 
build_world | name_came_from_user? | 
create_world | name_change | 
create_world! | name_changed? | 
 | name_was | 
 | name_will_change! | 
 | reset_name! | 
 | restore_name! | 

###A _Character_ `has_many` _power_ups_ `through:` _world_ 
At this point, we've looked at the generated methods we end up with when our class has both a `belongs_to` relationship and a `has_many` relationship. A Character model with both of those relationships would have both sets of methods, along with the database-column generated methods. But what about this `has_many through:` association, that we've got going on between our Characters and their many powerups? Do we get any additional methods showing up? The answer to that question is simply this: no.

###It's-a Me....
I've gone ahead and created the world `mushroom_kingdom` with a name of "Mushroom Kingdom". I've also created two characters named Mario and Luigi along with two powerups: `mushroom`, and `fire_flower`.

As stated at the beginning, these are the associations we are after:
```ruby
mushroom_kingdom.characters =>[...]
mushroom_kingdom.power_ups => [...]
mario.world => #<World ...>
fire_flower.world => #<World ...>
mario.power_ups => [...]
fire_flower.characters => [...]
```
Here are updated versions of each of our three classes.
```ruby
class World < ActiveRecord::Base
  has_many :characters
  has_many :power_ups
end
```
```ruby
class Character < ActiveRecord::Base
  belongs_to :world
  has_many :power_ups, through: :world
end
```
```ruby
class PowerUp < ActiveRecord::Base
  belongs_to :world
  belongs_to :character
end
```
And here is our current database schema.
Note: The name of the `power_ups` table has an underscore where the class name had an uppercase letter.
```ruby
ActiveRecord::Schema.define(version: 20160406024221) do

  create_table "characters", force: :cascade do |t|
    t.string  "name"
    t.integer "world_id"
  end

  create_table "power_ups", force: :cascade do |t|
    t.string  "name"
    t.integer "world_id"
  end

  create_table "worlds", force: :cascade do |t|
    t.string "name"
    t.integer "charcter_id"
    t.integer "power_ups_id"    
  end

end
```
So lets test some of our methods. 
If we give Mario a world to live in, like this:
```ruby
mario.world = mushroom_kingdom
```
We expect our `mushroom_kingdom.characters` to return some info about its single inhabitant, Mario.
Instead, however, we get this:
```ruby
>> mushroom_kingdom.characters.first
=> nil
```
Hmmm...
<hr>
The problem is this: 
**Children are not responsible**.
<hr>
Rather than telling a child object (`mario`) something the parent needs to know, we need to tell the parent (`mushroom_kingdom`). The parent is the responsible one in the relationship and will let the child object know what it needs to know.
```ruby
>> goomba = Character.create(name: "Goomba")
=> #<Character id: 3, name: "Goomba", world_id: nil>

>> mushroom_kingdom.characters << goomba
>> goomba.world
=> #<World id: 1, name: "Mushroom Kingdom">

>> goomba
=> #<Character id: 3, name: "Goomba", world_id: 1>

>> mushroom_kingdom.power_ups << mushroom
>> mushroom_kingdom.power_ups << fire_flower
>> fire_flower.world
=> #<World id: 1, name: "Mushroom Kingdom">
```

So our associations are starting to come together - as long as we are giving our information to the parent and not the irresponsible child. So how about the `has_many through:` association between our characters and our powerups?
```ruby
>> mario.power_ups.first.name
=> "Mushroom"
```
Seems to work. The key to making this `has_many through:` association work is in the database. If you look at the `worlds` table above, you'll notice that it has foreign keys (`_id` columns) for both the `characters` table and the `power_ups` table. This is what ties our characters to our powerups. As a general rule, **the `through:` table gets foreign keys for current model and its child**. So what about asking our fire flower which characters have access to it?
```ruby
>>fire_flower.characters.first.name
=> NoMethodError: undefined method `characters' for #<PowerUp id: 2, name: "Fire Flower", world_id: 1>
```
If we look back to our generated methods, we see that it is the `has_many` macro that gives us our `______s` method. Our PowerUp class, however, is using the `belongs_to` macro which does not generate such a method.
<hr>
_When we ask the powerup for a list of characters that own it, it may seems as it `belongs_to` specific characters. However, given the `characters` method that we want, we should view the powerup as though it `has_many` owners._
<hr>
```ruby
class PowerUp < ActiveRecord::Base
  belongs_to :world
  has_many :characters, through: :world
end
```
Our `world` table is already setup with the proper foreign keys.
```ruby
fire_flower.characters.first.name
=> "Mario"
```
The relationship that characters have with powerups would be called a _many to many_ relationship, as a character can have many powerups, and a powerup can have many owning characters.

###Many to Many
You can also set up this kind of association by using the `has_and_belongs_to_many` macro.
To experiment with this, we'll use a `Game` model to interact with our characters. Our desired behaviour looks like this:
```ruby
mario.games => [...]
super_mario_bros.characters => [...]
```
Our classes will need the new macro.
```ruby
class Game < ActiveRecord::Base
  has_and_belongs_to_many :characters
end

class Character < ActiveRecord::Base
  belongs_to :world
  has_many :power_ups, through: :world
  has_and_belongs_to_many :games
end
```
Additionally, we need a special table in our database called a _join table_. This is a table that only has two foreign keys: one for each of our models.

We'll create that, and a table for our `Game` class in two seperate migrations.
```ruby
class CreateGamesTable < ActiveRecord::Migration
  def change
    create_table :games do |t|
      t.string :name
    end
  end
end
```
```ruby
class CreateCharacterGameTable < ActiveRecord::Migration
  def change
    create_table :characters_games do |t|
      t.integer :character_id
      t.integer :game_id
    end
  end
end
```
After migration, our schema now looks like this:
```ruby
ActiveRecord::Schema.define(version: 20160407042123) do

  create_table "characters", force: :cascade do |t|
    t.string  "name"
    t.integer "world_id"
  end

  create_table "characters_games", force: :cascade do |t|
    t.integer "character_id"
    t.integer "game_id"
  end

  create_table "games", force: :cascade do |t|
    t.string "name"
  end

  create_table "power_ups", force: :cascade do |t|
    t.string  "name"
    t.integer "world_id"
  end

  create_table "worlds", force: :cascade do |t|
    t.string  "name"
    t.integer "charcter_id"
    t.integer "power_ups_id"
  end

end
```
After creating two games named "Super Mario Bros" and "Super Mario Bros 3" you can do this:
```ruby
mario.games << super_mario_bros
luigi.games << super_mario_bros

super_mario_bros.characters.first.name
=> "Mario"
super_mario_bros.characters.last.name
=> "Luigi"

super_mario_bros_3.characters << mario
mario.games.first.name
=> "Super Mario Bros"
mario.games.last.name
=> "Super Mario Bros 3"
```

Closing Notes:
While writing this, I am using Sinatra and tux to interact with both Active Record and the database. One very confusing issue that came up for me with this last section was a seeming failure of the 'many to many' relationship to actually operate both ways. 

For example, When running the final command `mario.games.last.name`, I would get `"Super Mario Bros"` instead of `"Super Mario Bros 3"`. I noticed that tux was not querying the database as it did with the `mario.games.first.name` command. Instead, it seemed to simply be pulling from the established local environment where `mario.games`, having already been queried from the database once, had only one game. Upon exiting tux, re-entering, re-establishing our 'mario' variables, and running the command `mario.games.last.name`, the database was queried and now had two games, the last of which was "Super Mario Bros 3".

###Final Thoughts

In summing everything up from the above experiment a few simple rules emerge. Follow these, and you will arrive at Active Record Association bliss.

#####Rules to Live By
 - A Parent `has_many`,  a Child `belongs_to`
 - Give the child table the foreign key (`_id`) columns.
 - Provide the parent with information, not the child.
 - When using a `has_many through` association, the `through` table requires the foreign keys for parent and child.
 - When using a `has_and_belongs_to_many` association an additional join table is required.
  - This table should be named with the plural versions of both tables, joined by an underscore, and in alphbetical order.
  - It's best to remove the primary `id` key. `:id => false`
  - Its two columns are simple foreign keys.
 - Lastly, ask yourself what method/result you want. Refer to the above charts to help you determine which association will give you that method/result.
