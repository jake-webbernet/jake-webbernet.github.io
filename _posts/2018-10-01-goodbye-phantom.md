So it's common knowledge that you can run your automated tests via Google Chrome instead of PhantomJS. PhantomJS is now officially not supported for obvious reasons.

I've been meaning to start running my feature specs with Chrome rather than PhantomJS but i've always had a bit of trouble getting it running.

But I finally did it, and this is what I had to do:

Added these two gems to my `test` section of my rails `Gemfile`
```ruby
  gem 'selenium-webdriver'
  gem 'chromedriver-helper'
 ```
 
Removed all code mentioning `Poltergeist` in my `rails_helper`

I'm using system specs now rather then feature specs, so I added these two sections to my `rails_helper`. It's pretty handy to be able to run the specs in non-headless mode when developing. It even works on my WSL setup which I really did not expect.

```ruby
  config.before(:each, type: :system, js: true) do
    if ENV['HEADLESS']
      driven_by :selenium_chrome_headless
    else
      driven_by :selenium
    end
  end
```

```ruby
Capybara.register_driver :selenium do |app|
  options = Selenium::WebDriver::Chrome::Options.new(args: ['disable-gpu'])
  Capybara::Selenium::Driver.new(
      app,
      browser: :chrome,
      options: options
  )
end
```

And here is what I needed to do to get chromium and the driver running locally on my computer

```shell
# Update the local chromedriver
$ chromedriver-update 

# Install chromium so that chromedriver has something to drive!
$ sudo apt install chromium-browser

# And finally to test if its working
$ chromedriver 
-> Starting ChromeDriver 70.0.3538.16 (16ed95b41bb05e565b11fb66ac33c660b721f778) on port 9515
-> Only local connections are allowed.
```
