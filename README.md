# Mongo 101
Getting started with the Mongo Shell to query data at doopoll.

## Getting Setup
Before you do anything you'll need to install Mongo onto your Mac. The recommended way to do this is via a package manager called Homebrew.

Open up your terminal and paste in the following commands:
```bash
// Install Homebrew
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
// Update the list of brew packages
brew update
// Install mongoDB
brew install mongodb
```

Once you're at this point you can paste in your personal login shell script. If you don't have one yet you can request one. Once you have it, keep it secret, keep it safe.

It should look something like this:
```bash
mongo candidate.5.mongolayer.com:10832/doopoll -u <username> -p<password>`
```

Once it loads, you should be in the shell!
