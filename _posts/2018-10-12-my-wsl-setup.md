Setting up a development environment on Windows was a bit tricky but I was able to get there in the end.

### Linux
* Got WSL running on the machine, and installed Ubuntu
* Placed the code inside the home directory (~/) of the linux machine
  * Rather than placing it on the C:// - which apparently is not cached and results in slower speeds
* Added exceptions to the Windows Defender Program - which apparently vastly slows down the WSL system
  * Added the `ruby`, `dpkg` and `git` processes
  * Added `C:\Users\Jake\AppData` folder

### Terminal
* Because the standard Ubuntu Bash Terminal isn't great - I wanted to make a few tweaks
  * Install `zsh` with `apt-get`
  * Install `oh-my-zsh` the standard way
  * Download and install `Hyper` which is an Electron based terminal which looks great
    * Setup in the hyper preferences to open Ubuntu Bash instead of the Windows Cmd with:
    * `shell: 'C:\\Windows\\System32\\bash.exe'`
  
### Dev Environment
* Setup Emacs
  * Installed Emacs using `apt-get`
  * Installed my `.emacs` file
  * Because I wanted to use the Emacs GUI, and WSL does not support GUI Linux programs - I needed to make use of the Windows X-Server protocol.
    * Installed VcXsrv on the machine
    * Added `DISPLAY=:0.0` to the `.zshrc` file
    * Boot emacs with `emacs --display :0.0` and open VxXsrv
* Setup Postgres
  * Installed Postgres on the windows side, and it can still be accessed inside WSL via localhost
  * Thought it would be faster to access the database if its not on WSL as its well known for having slowing IO
* Installed phantomjs with npm.
  * Global mode doesn't work, so I installed it in my WSL home folder `(~/)`
  * I had initially only added the phantom path to the `zshrc` file, but then I couldn't access it in when running specs in Emacs.
  * I then needed to add this directly to my `bashrc` path (as Emacs Rspec must use bash), so that when i type `phantomjs` it boots
  
