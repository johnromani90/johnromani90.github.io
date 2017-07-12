---
title:  "Automating Dev Environment with Iterm2 Scripting"
date:   2017-06-22 10:18:00
description: one command and go
---
I love learning about other developer workflows and shortcuts. I recently started playing around with [Iterm2's scripting](https://www.iterm2.com/documentation-scripting.html) ability and have found it simple and very satisfying to use.

I typically use four terminal tabs when developing.

***

1. redis
2. rails server
3. postgres
4. other

***

My goal was to automate this process with just one commmand, here is what I ended up with:

```bash
tell application "iTerm"
  tell current session of current tab of current window
    write text "/Users/johnromani/projects/he-api/"
    write text "redis-server"

    split horizontally with default profile
  end tell

  tell second session of current tab of current window
    write text "/Users/johnromani/projects/he-web/"
    write text "yarn"
    write text "npm run develop"
  end tell

  tell current window
    create tab with default profile
  end tell

  tell current session of current tab of current window
    write text "/Users/johnromani/projects/he-api/"
    write text "sidestart.sh"
  end tell

  tell current window
    create tab with default profile
  end tell

  tell current session of current tab of current window
    write text "/Users/johnromani/projects/he-api/"
    write text "rails s"
  end tell

  tell current window
    create tab with default profile
  end tell

  tell current session of current tab of current window
    write text "/Users/johnromani/projects/he-api/"
    write text "pghe"
  end tell

  tell current window
    create tab with default profile
  end tell

```

The above is pretty self explanatory but here is the gist. First, per Iterm's docs, you are suppose to define a dir called

.```
   library/application\ support/iterm/iterm_scripts/`
```

From testing, I discovered that you do NOT need to put your scripts there, so I decided to put my script dir in a git repo where I keep all my .dotfiles.

Next, create your script with a `.scpt` extension. Unless you are doing something more complex than me, you can find all the commands you need by searching stack over flow. For me, all I need to do is create four tabs, change directories, and execute the command I want. First tab uses a predefined shell script for sidekiq and redis, second typical rails server, third an alias to connect to development psql db, and last the path for the project I'm working in.

To run you iterm script simply:

```
    osascript ../your_script.scpt
```

All in all this saves me about 30 seconds every day, but there is something really satisfying about opening my laptop and having my dev environment loaded with one command.

End result:

![alt text](https://github.com/johnromani90/johnromani90.github.io/blob/master/assets/images/he_dev.gif?raw=true)
