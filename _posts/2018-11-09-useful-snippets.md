### Bash
* Continually check if a website is up  
```shell
while ! curl http://download.redis.io/ -m 5 ; do sleep 1 ; done
```

* Use aha `apt-get install aha` to output terminal output to a HTML file preserving colours. I used this to output a git diff to a HTML file.
 * `git diff master remotes/b/master --colour-words | aha --black > output_file.htm`
  
### ROR
* Add Postgres unix date default
  * `change_column :jobs, :status_updated_at, :integer, default: -> { 'extract(epoch from now())' }, null: false`
