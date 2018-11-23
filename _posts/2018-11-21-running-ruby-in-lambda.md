There comes a time when you need to use a service like AWS Lambda to run code.

Problem is that I'm terrible at NodeJS and know very little about it.

Enter [Rumbda](https://github.com/kleaver/rumbda) a tool that allows you to write ruby in Lambda.

Install as a gem

`gem install rumbda`

Create a directory, enter it, and run 

`rumbda init`

This creates a source/main.rb along with some other files. Add your Ruby files here and add any Gems in the supplied Gemfiles.

Run

`rumbda build`
