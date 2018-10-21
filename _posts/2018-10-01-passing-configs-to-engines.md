Often times you want to be able to pass configs to your gem/engines at run time. Below is how you can set that up


### In the engine
```ruby
module MyApp
  class Engine < ::Rails::Engine
    isolate_namespace MyApp
  end

    def self.setup(&block)
      @config ||= MyApp::Engine::Configuration.new
      yield @config if block
      @config
    end

    def self.config
      Rails.application.config
    end
end
```

### In the main app
`config/initializers/my_app.rb`

```
MyApp.setup do |config|
  config.testingvar = "asdfasdf"
end
```


It can then be accessed in the engine with

`MyApp.config.testingvar`
