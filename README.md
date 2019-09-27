# Mercurial to git migration and repository synchronization using git-remote-hg
Configuration of git-remote-hg to transform a mercurial repository and mirror it in git, also explains how to maintain it synchronized with the mercurial original repository.
## Install python, mercurial and git previously
git-remote-hg depends on python2.7, mercurial and git, so those must be on the system and accessible.
```bash
sudo apt-get install -y python2.7 mercurial git
```
## Download and install git-remote-hg
```bash
wget https://raw.github.com/felipec/git-remote-hg/master/git-remote-hg -O ~/bin/git-remote-hg
chmod +x ~/bin/git-remote-hg
```

## Modify .profile and add git-remote-hg to $PATH
Edit .profile with your favourite text editor.
```bash
vim ~/.profile
```
Add this to the end of the file:
```bash
# set PATH so it includes scripts binaries
if [ -d "$HOME/bin" ] ; then
   PATH="$HOME/bin:$PATH"
fi
```

Execute ` source ~/.profile ` for it to work on the terminal in use. On reboot it will be accessible on all terminals.


## Before cloning we should activate the hg-git compatibility layer
That way git-remote-hg will generate commits that are compatible with hg-git and viceversa. We won't be tracking branches from the git repository, so we mark that option to false too.
```bash
git config --global remote-hg.hg-git-compat true
git config --global remote-hg.track-branches false
```

## Clone the mercurial repository using git-remote-hg
```bash
mkdir git-repo
cd git-repo
git clone hg::<link-to-hg-repo> .
```
This will take a while and will generate a git repository identical to the mercurial repo provided. Its size will be very big and will probably need to pass git gc to reduce its footprint.

## Add options to not use all memory on git gc --aggressive
```bash
git config core.packedGitWindowSize 32m
git config core.packedGitLimit 256m
git config pack.packSizeLimit 4g
git config pack.threads 8
git config pack.deltaCacheSize 4
git config pack.windowMemory 512m
```

## Execute Garbage Cleaner to reduce git repo footprint on big repositories
```bash
git gc --aggressive
```
Note: This will take a while depending on the size of the repository

## Push to git mirror but first set up the url to Push
```bash
git remote set-url --push origin <git-repo-url>
git push
git push --tags
```

## Cron process for synchronizing remote git repository with mercurial
I use this script to handle the logging of every synchronization attempt.
```bash
#!/usr/bin/env bash
{
echo $(date -u) " Executing git synchronization cron job."
cd /home/ubuntu/git-pi-cron-repo
git pull
git push
git push --tags
echo $(date -u) " Synchronization job has finished correctly." 
} >> /home/ubuntu/logs/git-sync-cron.log 2>>/home/ubuntu/logs/git-sync-cron.log
```

After saving that script somewhere in the system and giving it execution permissions a cron job can be created to point to it.
```bash
crontab -e

## Add this at the end of the file
*/5 * * * * /path/to/script
```
