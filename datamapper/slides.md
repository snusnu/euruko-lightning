!SLIDE title-slide

# DataMapper 1.0.0.rc3 #

!SLIDE bullets incremental

* Dan Kubb (Ruby Hero 2009)
* Dirkjan Bussink (DataObjects)
* ~210 contributors
* ~40 (No)SQL adapters and ~120 plugins
* 1.8.{6,7}, 1.9.{1,2}, JRuby, Rubinius
* Friendly Community (#datamapper #dm-hacking)

!SLIDE bullets

* 183 projects
* 905 watchers
* 238 forkers
* 131 collaborators
* 210 contributors


!SLIDE center

# DataMapper 1.0.0 #
# Next weekend! #


!SLIDE

    @@@ Ruby
    require 'rubygems'
    require 'dm-core'

    DataMapper.setup(:default, 'sqlite::memory:)

    class Person
      include DataMapper::Resource
      property :id,	  Serial
      property :name, String, :required => true
      property :url,  URI # custom property
      has n, :addresses
    end

    class Address
      include DataMapper::Resource
      property :street, String, :required => true
      belongs_to :person
    end

    DataMapper.auto_migrate!

!SLIDE

    @@@ Ruby
    # Identity Map #
    DataMapper.repository do
      puts Person.get(1).object_id # => 22324270
      puts Person.get(1).object_id # => 22324270
    end

!SLIDE

    @@@ Ruby
    # Strategic Eager Loading
    people = Person.all
    # No query issued
    people.each do |person|
      puts person.addresses
    end

    # SELECT "id", "name" FROM people;
    # SELECT "id", "street"
    #   FROM addresses
    #   WHERE "person_id" IN (1,2,3)

!SLIDE

    @@@ Ruby
    # Multiple repositories
    DataMapper.setup(:repo1, "postgres://postgres@localhost/dm")
    DataMapper.setup(:repo2, "mongo://root@localhost/dm")

    get "/people" do
      # Odd / even sharding
      repo = "repo#{user.id % 2 + 1}".to_sym
      DataMapper.repository(repo) do
        @people = Person.all
      end
      erb :people
    end

!SLIDE

    @@@ Ruby
    # Advanced queries
    Person.all(Person.addresses.street.like => "%street%")

    # SELECT "people"."id", "people"."name", "people"."url"
    #   FROM "people"
    #   INNER JOIN "addresses"
    #   ON "people"."id" = "addresses"."person_id"
    #   WHERE "addresses"."street" LIKE '%street%'
    #   GROUP BY "people"."id", "people"."name", "people"."url"
    #   ORDER BY "people"."id"

!SLIDE

    @@@ Ruby
    # Named scopes
    class Person
      # named scopes, pure ruby, DSL or NoDSL?
      def self.named_like_me
        all(:name.like => "%me%")
      end
    end

!SLIDE

    @@@ Ruby
    # dm-constraints
    class Person
      # ...
      has n, :addresses, :constraint => :destroy!
      # ...
    end

    # ALTER TABLE addresses
    # ADD CONSTRAINT addresses_person_id_fk
    # FOREIGN KEY (person_id)
    # REFERENCES people
    # ON DELETE CASCADE
    # ON UPDATE CASCADE

!SLIDE

    @@@ Ruby
    # Set operations
    Person.all - Person.all(:name => 'Martin')

    # => SELECT * FROM people WHERE NOT(name = 'Martin')

    [1, 2] - [2   ] == [1]       # difference
    [1, 2] & [2, 3] == [2]       # intersection
    [1, 2] | [2, 3] == [1, 2, 3] # union

!SLIDE commandline incremental

	$ gem install rails --pre
    Get rails
	$ rails funky_app -m http://github.com/snusnu/rails-templates/raw/master/dm_rails_master.rb
	  Generates a rails3 app all set up for datamapper
	  The app template will be available at datamapper.org soon
	$ cd funky_app
	  Change into the newly generated application
	$ rake db:setup
	  Create, automigrate and seed the database
	$ rails g scaffold Person name:string hobby:string
	  Create a simple Person model with name and hobby properties
	$ rails g rails_metrics Metric
	  Enable the rails_metrics engine (github.com/engineyard/rails_metrics)
	  You need to add: gem 'rails_metrics' to your Gemfile
	  Currently, my fork at snusnu/rails_metrics is needed
	  I'm pretty confident stuff will get pulled upstream
	$ rails s
	  Start the server

!SLIDE bullets

* datamapper.org
* alfred.datamapper.org
* wiki.github.com/datamapper/dm-core/matrix
* github.com/datamapper
* github.com/dkubb
* github.com/snusnu
* quasipartikel.at

!SLIDE

# Thx Dirkjan! I stole all your code examples :P #
