Since learning about gems I have done lots of work that involves working with gems locally.

To develop with a gem locally you can reference it in your gemfile like so 

```ruby
gem 'my-gem', path '../my-gem-path'
gem 'my-other-gem', path '../my-other-gem-path'

```
This has heaps of benefits if you are doing integration work, and you want to put a `byebug` in your gem.

The only problem is that you need to revert this change when pushing to master. Master won't pull the gem locally from the path, it will likely need to pull it from an external source such as RubyGems or Github.

So instead of using the `path` parameter, I've found a way to use bundle config in a way that uses the local version of the gem when working locally...and the remote sourced gem in production.

To get this going the first task is to ensure that you have a `branch` specifier for the gem.

```ruby
gem 'mygem, git: "https://github.com/Webbernet/mygem", branch: 'master'
```

Next, we can tell bundle that for `mygem`, you can find it in `~/mygem` on my computer. 

```shell
 $ bundle config local.mygem ~/mygem
```

This is a local bundle setting of course, so it won't affect production at all.


