I was really excited to get Chrome silenium running on Capybara, but I had this feeling that there was going to be pain getting this to run in our CI environment.


### Changing the CI Dockerfile
Our CI environment is run using Docker. This makes it really easy to make changes to the test container.

Below is the extra commands needed in the `Dockerfile` to allow for chrome to run headlessly on an Ubuntu box

```shell
# Install chromium dependencies for tests
RUN apt-get install -y \
  gconf-service \
  libasound2 \
  libatk1.0-0 \
  libcairo2 \
  libcups2 \
  libfontconfig1 \
  libgdk-pixbuf2.0-0 \
  libgtk-3-0 \
  libnspr4 \
  libpango-1.0-0 \
  libxss1 \
  fonts-liberation \
  libappindicator1 \
  libnss3 \
  lsb-release \
  xdg-utils

# install chrome
RUN wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
RUN dpkg -i google-chrome-stable_current_amd64.deb; apt-get -fy install
```

### Updating `spec/rails_helper`

I needed to register a new driver with a few config settings. These were apparently imporatant for Docker because without them, the ChromeDriver would report that the browser was continually crashing.

```shell
Capybara.server = :puma, { Silent: true }

Capybara.register_driver :chrome_headless do |app|
  options = ::Selenium::WebDriver::Chrome::Options.new

  options.add_argument('--headless')
  options.add_argument('--no-sandbox')
  options.add_argument('--disable-dev-shm-usage')
  options.add_argument('--window-size=1400,1400')

  Capybara::Selenium::Driver.new(app, browser: :chrome, options: options)
end
```

I also decided that it would be best to allow for `HEADLESS` to be able to toggled. This means that our developers would actually be able to develop with a chrome browser, whilst we could run headlessly on CI.

I added the HEADLESS environment variable.

```shell
if ENV['HEADLESS']
  Capybara.javascript_driver = :chrome_headless
end
```

Inside the rspec `config` block
```shell
config.before(:each, type: :system, js: true) do
  if ENV['HEADLESS']
    driven_by :chrome_headless
  else
    driven_by :selenium
  end
end
```

Lastly I added the `HEADLESS` variable to our CI and was sure to pass this through our to our test docker container with `-e HEADLESS=$HEADLESS`

Easier than I thought
