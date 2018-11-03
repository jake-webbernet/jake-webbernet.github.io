I made an article about setting up an engine so that you can pass it configuration variables via an initializer.

Needed to do the same thing for a normal gem.


In the main gem file `lib/mygem.rb`, set it up like so.

```ruby
module MyGem
  class << self
    attr_accessor :configuration
  end

  def self.configure
    self.configuration ||= Configuration.new
    yield(configuration)
  end

  class Configuration
    attr_accessor :api_key, :api_secret
  end
end
```
The Configuration object that is stored in the gem module allows us to store config attributes. In the above example I needed to pass an `api_key` and `api_secret` into the gem.

So my host would have an initializer `config/initializers/mygem.rb` to set it up

```ruby
MyGem.configure do |config|
  config.api_key = 'APIKEYHERE'
  config.api_secret = 'APISECRETHERE'
end
```

In the gem, it can be accessed easily with:

`Zoomnet.configuration.api_secret`
