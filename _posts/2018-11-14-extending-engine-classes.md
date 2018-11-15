Further on to all the engine work I have been doing lately, I have found problens dealing with models.

It's common to create models in the engine so that they can be used inside the host application.

There is a problem though when you want to associate Engine models with your Host models. You lose the ability to create useful relationships such as `has_one` and `belongs_to`. This is because obviously you cannot add these relationships in your engine as you would violate the loose coupling principles.

For example.

* An engine I have made exposes a `MyEngine::Stream` model
* In one of my hosts, I have a `Staff` model, that I want to be associated with a `MyEngine::Stream`
* Ultimately, it would be really beneficial to setup a relationship between these two entities so that we can setup a `stream.staff` method
* Of course, the `MyEngine::Stream` method cannot have a relationship added directly.

Answer - A little bit of meta programming.

Setup an initializer in the host application for the engine. Lets say `config/initializers/myengine.rb`

```ruby
Rails.application.config.to_prepare do
  MyEngine::Stream.class_eval do
    has_one :staff, foreign_key: 'my_engine_stream_id'
  end
end
```

This uses the `class_eval` metaprogramming method that allows us to dynamically add code to the class at runtime. The `Rails.application.config.to_prepare` block was really important because without it, the `class_eval` was not working when developing locally after a change. I believe this would be because Rails reloads the classes after every change, but doesn't run the initializers. The result would be `missing 'staff' relationship on the stream class. The `to_prepare` seemed to fix this.

Now we can run `stream.staff` and that returns the correct staff model as usual.


