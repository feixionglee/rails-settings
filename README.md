# Settings Plugin for Rails

[![Build Status](http://travis-ci.org/ledermann/rails-settings.png)](http://travis-ci.org/ledermann/rails-settings)

Settings is a plugin that makes managing a table of global key, value pairs easy.
Think of it like a global Hash stored in you database, that uses simple ActiveRecord
like methods for manipulation.  Keep track of any global setting that you dont want
to hard code into your rails app.  You can store any kind of object.  Strings, numbers,
arrays, or any object.


## Setup

Requirements:
  ActiveRecord 2.3.x or 3.0.x

You have to create the table used by the Settings model by using this migration:

    class CreateSettingsTable < ActiveRecord::Migration
      def self.up
        create_table :settings, :force => true do |t|
          t.string  :var,         :null => false
          t.text    :value
          t.integer :target_id
          t.string  :target_type, :limit => 30
          t.timestamps
        end

        add_index :settings, [ :target_type, :target_id, :var ], :unique => true
      end

      def self.down
        drop_table :settings
      end
    end
    
Now put update your database with:

    rake db:migrate

## Usage

The syntax is easy.  First, lets create some settings to keep track of:

    Settings.admin_password = 'supersecret'
    Settings.date_format    = '%m %d, %Y'
    Settings.cocktails      = ['Martini', 'Screwdriver', 'White Russian']
    Settings.foo            = 123
    Settings.credentials    = { :username => 'tom', :password => 'secret' }

Now lets read them back:

    Settings.foo            # returns 123

Changing an existing setting is the same as creating a new setting:

    Settings.foo = 'super duper bar'

For changing an existing setting which is a Hash, you can merge new values with existing ones:

    Settings.merge! :credentials, :password => 'topsecret'
    Settings.credentials    # returns { :username => 'tom', :password => 'topsecret' }

Decide you dont want to track a particular setting anymore?

    Settings.destroy :foo
    Settings.foo            # returns nil

Want a list of all the settings?

    Settings.all            # returns {'admin_password' => 'super_secret', 'date_format' => '%m %d, %Y'}

You need name spaces and want a list of settings for a give name space? Just choose your prefered named space delimiter and use Settings.all like this:

    Settings['preferences.color'] = :blue
    Settings['preferences.size'] = :large
    Settings['license.key'] = 'ABC-DEF'
    Settings.all('preferences.')   # returns { 'preferences.color' => :blue, 'preferences.size' => :large }

Set defaults for certain settings of your app.  This will cause the defined settings to return with the
Specified value even if they are not in the database.  Make a new file in config/initializers/settings.rb
with the following:

    Settings.defaults[:some_setting] = 'footastic'
  
Now even if the database is completely empty, you app will have some intelligent defaults:

    Settings.some_setting   # returns 'footastic'

Settings may be bound to any existing ActiveRecord object. Define this association like this:

    class User < ActiveRecord::Base
      has_settings
    end

Then you can set/get a setting for a given user instance just by doing this:

    user = User.find(123)
    user.settings.color = :red
    user.settings.color # returns :red
    user.settings.all # { "color" => :red }

I you want to find users having or not having some settings, there are named scopes for this:

    User.with_settings # => returns a scope of users having any setting
    User.with_settings_for('color') # => returns a scope of users having a 'color' setting
  
    User.without_settings # returns a scope of users having no setting at all (means user.settings.all == {})
    User.without_settings('color') # returns a scope of users having no 'color' setting (means user.settings.color == nil)

That's all there is to it! Enjoy!