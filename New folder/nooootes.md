# Mac Shell Notes

## Change default shell to bash

* To change default shell to bash: `chsh -s /bin/bash`

* To change back to default option: `chsh -s /bin/zsh`

* To see list of available shells: `cat /etc/shells`

## Install Brew using Shell:

* `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

## Install JDK 1.8 using Brew

* `brew tap AdoptOpenJDK/openjdk`
* `brew cask install adoptopenjdk8`

## Install Maven using Brew

* `brew install maven`

## Install NVM using shell - [instructions](https://github.com/nvm-sh/nvm/blob/master/README.md#installing-and-updating):

* `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.2/install.sh | bash`

## Install Node (10.16.0) using NVM in shell:

* `nvm install 10.16.0`
* `nvm use 10.16.0`
* `nvm alias default v10.16.0`

## Create .bash_profile file under the user folder with the following:

    export JAVA_HOME=/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home
    export NVM_DIR="$HOME/.nvm"
    source $(brew --prefix nvm)/nvm.sh
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
    [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm

## Install Nginx:

* `brew tap denji/nginx`
* `brew install nginx-full --with-upload-module`

## Set up SSH key:

1. Open terminal
1. Excute in terminal: `ssh-keygen -t ed25519 -C "Your email id”`
1. Enter
1. Enter
1. pbcopy < ~/.ssh/id_ed25519.pub
1. Open browser:
1. Sign-in to gitlabs
1. Go to SSH keys page
1. Command + v
1. Press “Add key” button
1. It will add the ssh keys.

## Unzipping 7z Files

* `% brew install p7zip`
* `% 7za x myfiles.7z`

# PATCH APPLICATION TO GIT
 
* First, take a look at what changes are in the patch. You can do this easily with git apply: `git apply --stat fix_empty_poster.patch`
	* Note that this command does not apply the patch, but only shows you the stats about what it’ll do. 
* Next, you’re interested in how troublesome the patch is going to be. Git allows you to test the patch before you actually apply it: `git apply --check fix_empty_poster.patch`
* If you don’t get any errors, the patch can be applied cleanly: `git apply fix_empty_poster.patch`

# DBeaver

1. Install DBeaver from https://dbeaver.io/download/ (Latest)
1. Download IBM Data Server Driver for JDBC and SQLJ (JCC Driver) from https://www.ibm.com/support/pages/db2-jdbc-driver-versions-and-downloads
1. To add a DB connection @ DBeaver:
    * New Database Connection > DB2 LUW > Add Database and Security Information
    * Edit Driver Settings > Add File > Select the JDBC Driver File (After unzipping - only unzip downloaded file)
    * Test Connection
