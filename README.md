Laptop
======

Laptop is a script to set up a macOS laptop for web and mobile development.

It can be run multiple times on the same machine safely.
It installs, upgrades, or skips packages
based on what is already installed on the machine.

Requirements
------------

Support OS:
* macOS High Sierra (10.13)
* macOS Mojave (10.14)
* macOS Catalina (10.15)

Older versions may work but aren't regularly tested.
Bug reports for older versions are welcome.

Install
-------

Open Terminal and Install xcode select:

`xcode-select --install`

Create you SSH key and link to your Github Account:

`ssh-keygen -C your.name@spark.re`

Enter a location to save the save the key. Press enter
to save it in the default location, or enter a new location.
If you save it in the default location, all ssh connections
will use this key by default. You should only need to save
it in a non default location if you use multiple keys.

`# Enter file in which to save the key (/Users/USERNAME/.ssh/id_rsa):`

Long story short... just press enter when prompted for the above.

Be sure to add a password to your key to ensure the
security of our servers. This won't need to be a password
you remember, as the ssh agent can remember it for you
(see below). So that being said, the best course of action
is to use a generated password through Dashlane.

Set your identity in the SSH Config:
This will fix an issue when running deployments where
you get permission denied from Github, since our local private
keys are used by Capistranto when pulling the code.

```
touch ~/.ssh/config
echo 'UseKeychain yes' >> ~/.ssh/config
echo 'AddKeysToAgent yes' >> ~/.ssh/config
echo 'IdentityFile ~/.ssh/id_rsa' >> ~/.ssh/config
```

Add the new SSH key to your terminal instance to prevent need to enter SSH password:

`ssh-add -K`

Link your SSH key with your Github Account. First, copy the
SSH key to your clipboard.

`cat ~/.ssh/id_rsa.pub | pbcopy`

Profile > Settings > SSH and GPG Keys > New SSH key.

Download the computer setup repository and move the mac script to your root directory:

```
git clone git@github.com:conorhalloran/comp-setup.git
cp ~/comp-setup/mac ~/mac
```

Review the script (avoid running scripts you haven't read!):
```sh
less mac
```

Execute the downloaded script:
```sh
sh mac 2>&1 | tee ~/laptop.log
```

Optionally, review the log:
```sh
less ~/laptop.log
```

Once Script Completes
---------

In iTerm2:
Preferences > Profile > Command:
Send text at start: `ssh-add -K`

In Terminal, let Github know who you are:

```
git config --global user.name "Your Name"
git config --global user.email your.name@gmail.com
```

Your computer has been through a lot since running this script.
Restart it and move onto the next step.

Setup Rbenv, Ruby and Spark
---------

```
rbenv install 2.5.7
rbenv global 2.5.7
rbenv rehash
gem install bundler

cd ~/whatever-parent-directory-you-decide-to-install-spark-into
git clone git@github.com:sprk/spark.git
cd spark
```
Create the master key and copy the variable that has been shared with
you in the Dashlane "Sharing Center" under "Rails - Credentials Master Key"
`touch config/master.key && open config/master.key`

Populate your private variables in the zshrc private file:
`open ~/.files/zshrc/private`
`SPARK_SEED_ADMIN_EMAIL="first.lastname@spark.re"`

Unless you have received an aws access key id and secret key, leave these variables as is.

Setup your local database and put your computers username in the designated field.
`cp config/settings.local.example.yml config/settings.local.yml`
`open config/settings.local.yml`

Ensure yarn is setup correctly.
`yarn install --check-files`

Create and Populate your local database:
`rboot`

Debugging
---------

Your last Laptop run will be saved to `~/laptop.log`.
Read through it to see if you can debug the issue yourself.
If not, copy the lines where the script failed into a
[new GitHub Issue](https://github.com/thoughtbot/laptop/issues/new) for us.
Or, attach the whole log file as an attachment.

What it sets up
---------------

macOS tools:

* [Homebrew] for managing operating system libraries.
[Homebrew]: http://brew.sh/

Unix tools:

* [Exuberant Ctags] for indexing files for vim tab completion
* [Git] for version control
* [OpenSSL] for Transport Layer Security (TLS)
* [RCM] for managing company and personal dotfiles
* [The Silver Searcher] for finding things in files
* [RipGrep] as a faster alternative to the Silver Surfer
* [Tmux] for saving project state and switching between projects
* [Watchman] for watching for filesystem events
* [Neovim] as the one true editor(tm) that you may choose to ignore

[Exuberant Ctags]: http://ctags.sourceforge.net/
[Git]: https://git-scm.com/
[OpenSSL]: https://www.openssl.org/
[RCM]: https://github.com/thoughtbot/rcm
[The Silver Searcher]: https://github.com/ggreer/the_silver_searcher
[Tmux]: http://tmux.github.io/
[Watchman]: https://facebook.github.io/watchman/
[Neovim]: https://neovim.io/
[RipGrep]: https://github.com/BurntSushi/ripgrep
[Zsh]: http://www.zsh.org/

Heroku tools:

* [Heroku CLI] and [Parity] for interacting with the Heroku API

[Heroku CLI]: https://devcenter.heroku.com/articles/heroku-cli
[Parity]: https://github.com/thoughtbot/parity

GitHub tools:
* [Hub] for interacting with the GitHub API

[Hub]: http://hub.github.com/

Image tools:
* [ImageMagick] for cropping and resizing images

Programming languages, package managers, and configuration:

* [Bundler] for managing Ruby libraries
* [Node.js] and [NPM], for running apps and installing JavaScript packages
* [Ruby] stable for writing general-purpose code
* [Yarn] for managing JavaScript packages

[Bundler]: http://bundler.io/
[ImageMagick]: http://www.imagemagick.org/
[Node.js]: http://nodejs.org/
[NPM]: https://www.npmjs.org/
[Ruby]: https://www.ruby-lang.org/en/
[Yarn]: https://yarnpkg.com/en/

Databases:

* [Postgres] for storing relational data
* [Redis] for storing key-value data

[Postgres]: http://www.postgresql.org/
[Redis]: http://redis.io/

It should take less than 15 minutes to install (depends on your machine).

Customize in `~/.laptop.local`
------------------------------

Your `~/.laptop.local` is run at the end of the Laptop script.
Put your customizations there.
For example:

```sh
#!/bin/sh

brew bundle --file=- <<EOF
brew "Caskroom/cask/dockertoolbox"
brew "go"
brew "ngrok"
brew "watch"
EOF

default_docker_machine() {
  docker-machine ls | grep -Fq "default"
}

if ! default_docker_machine; then
  docker-machine create --driver virtualbox default
fi

default_docker_machine_running() {
  default_docker_machine | grep -Fq "Running"
}

if ! default_docker_machine_running; then
  docker-machine start default
fi

fancy_echo "Cleaning up old Homebrew formulae ..."
brew cleanup
brew cask cleanup

if [ -r "$HOME/.rcrc" ]; then
  fancy_echo "Updating dotfiles ..."
  rcup
fi
```

Write your customizations such that they can be run safely more than once.
See the `mac` script for examples.

Laptop functions such as `fancy_echo` and
`gem_install_or_update`
can be used in your `~/.laptop.local`.

See the [wiki](https://github.com/thoughtbot/laptop/wiki)
for more customization examples.

Contributing
------------

Edit the `mac` file.
Document in the `README.md` file.
Follow shell style guidelines by using [ShellCheck] and [Syntastic].

```sh
brew install shellcheck
```

[ShellCheck]: http://www.shellcheck.net/about.html
[Syntastic]: https://github.com/scrooloose/syntastic

Thank you, [contributors]!

[contributors]: https://github.com/thoughtbot/laptop/graphs/contributors

By participating in this project,
you agree to abide by the thoughtbot [code of conduct].

[code of conduct]: https://thoughtbot.com/open-source-code-of-conduct

License
-------

Laptop is © 2011-2018 thoughtbot, inc.
It is free software,
and may be redistributed under the terms specified in the [LICENSE] file.

[LICENSE]: LICENSE

About thoughtbot
----------------

![thoughtbot](http://presskit.thoughtbot.com/images/thoughtbot-logo-for-readmes.svg)

Laptop is maintained and funded by thoughtbot, inc.
The names and logos for thoughtbot are trademarks of thoughtbot, inc.

We are passionate about open source software.
See [our other projects][community].
We are [available for hire][hire].

[community]: https://thoughtbot.com/community?utm_source=github
[hire]: https://thoughtbot.com?utm_source=github
