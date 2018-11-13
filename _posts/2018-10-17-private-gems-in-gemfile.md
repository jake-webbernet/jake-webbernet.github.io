So we have started setting up quite a few engines and normal rails gems that are private repos. Because these are private repositories, I knew it was going to be a different setup

It's pretty easy to setup.

The `Gemfile` stays the same, reference the git repo as usual.

`gem 'frefnet', git: "https://github.com/Webbernet/frefnet"`

If you run bundle install with this, you will receive an error because its a private repo.

To fix this we need to add an oauth key to the bundler config.

Generate a personal outh key from your github account, with private repo access

Add it to your config with, and thats it!

`bundle config github.com <GITHUB-USERNAME>:93921321939MYAUTHTOKEN2131231`

### Docker Setup

I needed to also set this up on our CI for our production boxes. 

I had to change
* Dockerfile 
  * This is because the Dockerfile is resposible for installing all the gems. I need to enable this to see the private gem.
* Our CI file
  * This is because I need to pass the Dockerfile the Github OAuth Token

These are the changes I made:
1. Added an `ARG github_token` inside the Dockerfile. This allows me to pass in a github token when building the dockerfile
2. Changed the usual `bundle install` line in the Dockerfile to:
```ruby
RUN bundle config github.com deploywebbernet:$github_token && (bundle install --without test development) && bundle config --delete github.com
```
This does a bundle install with the token, and then removes the token so it doesn't sit in the Container.
3. In the CI build service, I added a `--build-arg` flag to the `docker build` command.
```ruby
docker build -f ci-Dockerfile-Chromium --build-arg github_token=$PRIVATE_GEM_GITHUB_TOKEN -t tester .
```
