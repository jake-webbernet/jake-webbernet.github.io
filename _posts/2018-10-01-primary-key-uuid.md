Sometimes I need to create a table where I can have a unique identifier that isn't sequential. This is mostly for security reasons as I don't want people guessing a record and possibly accessing something they shouldn't.

Usually I would just create a new indexed string field on a table, and use a model hook to generate a `SecureRandom` string.

Turns out the new rails allows you to specify the ID as a `uuid`, if you are using postgres. Like this

```ruby
  create_table :frefnet_file_references, id: :uuid do |t|
    t.string :content_type, null: false
    t.hstore :meta_data
    t.integer :size
    t.timestamps
  end
```

But *before* I run this, I need to enable the `pgcrypto` postgres extension which is what gives us this function.

This needs to run before all other migrations so I created a file at `db/migrate/0_enable_extensions.rb`

```ruby
class EnableExtensions < ActiveRecord::Migration[5.2]
  def change
    enable_extension 'pgcrypto'
  end
end
```
