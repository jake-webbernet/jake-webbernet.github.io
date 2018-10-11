Rails now has `System` tests, instead of `Feature` tests.

The biggest difference being that we can now switch on `config.use_transactional_fixtures` inside the `rails_helper` and also get rid of `Database Cleaner`. This is a pretty big deal with the older feature specs, as these needed to use DB Cleaners `truncation` method after each test which is quite slow.

With System tests
* You don't need the Database Cleaner gem
* You can switch on transactional fixtures

When I implemented it I needed to dump my test database as it had records that were not cleaned up from the last test run (on account of removing DB Cleaner)

This is a pretty great article about system tests
https://medium.com/table-xi/a-quick-guide-to-rails-system-tests-in-rspec-b6e9e8a8b5f6
