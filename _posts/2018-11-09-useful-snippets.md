### Bash

**Continually check if a website is up**
```shell
$ while ! curl http://download.redis.io/ -m 5 ; do sleep 1 ; done
```

**Output to HTML**
* Use aha to output terminal output to a HTML file preserving colours. I used this to output a git diff to a HTML file which was really helpful

```shell
$ apt-get install aha
$ git diff master remotes/b/master --colour-words | aha --black > output_file.htm
```

### ROR
**Add Postgres unix date default**

```ruby
change_column :jobs, :status_updated_at, :integer, default: -> { 'extract(epoch from now())' }, null: false
```
