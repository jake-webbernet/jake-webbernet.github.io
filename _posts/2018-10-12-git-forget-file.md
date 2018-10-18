I use WSL to develop, which means I have a non-standard way of maitaning my local development database.

I needed to make a change to my local database.yml, but I never wanted to commit it. Adding it to the `.git_ignore` would delete the file and cause issues for others.

I needed a way to maintain the modified the database.yml, but not have it come up as a change 

`git update-index --assume-unchanged <file>`

This worked.
