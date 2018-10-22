So we have started setting up quite a few engines and normal rails gems that are private repos. Because these are private repositories, I knew it was going to be a different setup

It's pretty easy to setup.

The `Gemfile` stays the same, reference the git repo as usual.

`gem 'frefnet', git: "https://github.com/Webbernet/frefnet"`

If you run bundle install with this, you will receive an error because its a private repo.

To fix this we need to add an oauth key to the bundler config.

Generate a personal outh key from your github account, with private repo access

Add it to your config with, and thats it!

`bundle config github.com <GITHUB-USERNAME>:93921321939MYAUTHTOKEN2131231`

+ Bonus this what can be added to the dockerfile for the build, you can pass the github token through with an env variable, install and then remove it again.

`RUN bundle config github.com deploywebbernet:$github_token && (bundle install --without test development) && bundle config --delete github.com`
