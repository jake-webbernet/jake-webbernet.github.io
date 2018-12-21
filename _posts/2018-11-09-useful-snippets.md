## Bash
**Continually check if a website is up**
```shell
$ while ! curl http://download.redis.io/ -m 5 ; do sleep 1 ; done
```

**Check if a command runs successfully**
```shell
$ while docker info ; ret=$? ; [ $ret -ne 0 ]; do echo "waiting for docker to start" && sleep 5; done
```

**Output to HTML**
* Use aha to output terminal output to a HTML file preserving colours. I used this to output a git diff to a HTML file which was really helpful

```shell
$ apt-get install aha
$ git diff master remotes/b/master --colour-words | aha --black > output_file.htm
```

## ROR
**Add Postgres unix date default**

```ruby
change_column :jobs, :status_updated_at, :integer, default: -> { 'extract(epoch from now())' }, null: false
```

## Emacs
**Search and replace in a directory**

Using Dired.

* Navigate to a directory with Dired
* Mark the directory/files you would like to replace with `m`
* Use `u` to to unmark files accidently marked
* Use `Q` to executed `dired-find-and-replace-regexp` and add search terms and replace terms and hit enter
* Cycle through and press `y` or `SPACE` to replace the current one


**Running SQLi**

* Create a buffer with `sql-mode` switched on, or navigate to an SQL file
* Startup a `sql-postgres` session, and supply all necessary credentials to login
* Go back to the first buffer and run `sql-send-region` (C-c C-r) to send the selected SQL statement to postgres
* This makes it really fast for profiling and testing.
