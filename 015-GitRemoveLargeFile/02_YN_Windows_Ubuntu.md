# Lab 015: Remove Large File from Git Commit History

Windows + Ubuntu

## Lab goal

This lab specifically focuses on the process of removing large file objects from the Git history.

**Be warned: this technique is destructive to our commit history.**

We have to notify all contributors that they must rebase their work onto our new commits.

## Prerequisites

### Install VirtualBox for Windows

### Install Vagrant for Windows

### Start Vagrant with the Ubuntu file

```dos
vagrant up

vagrant ssh
```

### Install git-filter-repo

<!--
<https://github.com/newren/git-filter-repo>

<https://github.com/briansu2004/git-filter-repo/blob/main/INSTALL.md>

<https://github.com/briansu2004/git-filter-repo/blob/main/git-filter-repo>

<https://raw.githubusercontent.com/newren/git-filter-repo/main/git-filter-repo>
-->

```bash
sudo apt install python3-pip
python3 -m pip install --user git-filter-repo
export PATH=$PATH:/home/vagrant/.local/bin
```

## Steps

### 1. **Initiate** the test environment

Run below command to initiate a git repo

```bash
mkdir git-test
cd git-test
git init
echo "Init setup" > file1
git add file1
git commit -m "Version 1"
du -sh ./

curl -L https://www.kernel.org/pub/software/scm/git/git-2.1.0.tar.gz > git.tgz
du -sh ./

git add git.tgz
git commit -m 'Add git tarball'
du -sh ./
```

<!--
```bash
...
```
-->

### 2. **Remove** the Large File

```dos
git rm git.tgz
git commit -m "Remove large tarball"
du -sh ./
```

### 3. Clean the Database

```dos
git gc
du -sh
git count-objects -v
```

<!--
```bash
...
```
-->

### 4. **Find** Out the Location of The Large File in packfiles

We can find the SHA for the largest file in packfile

```bash
ls .git/objects/pack/*.idx
git verify-pack -v .git/objects/pack/<index name>.idx | sort -k 3 -n | tail -3
```

Note: Please replace **<index name>** with actual index name under `.git/objects/pack` folder.

e.g.

```bash
git verify-pack -v .git/objects/pack/pack-180fe43703f733dfd1a44cffc10190f35a141f90.idx | sort -k 3 -n | tail -3
```

<!--
```bash
...
```
-->

Run below command to find out what the file name of the blob:

```dos
git rev-list --objects --all | grep 82c99a3
```

<!--
```bash
...
```
-->

Note: `82c99a3` is the SHA we found in previous command. It may be different in your scenario.

### 5. **Remove** the File from History

We are going to use `git-filter-repo` command to rewrite the git history. This doesn't come with the original setup and we have to install it seperately. We can follow [this document](https://github.com/newren/git-filter-repo/blob/main/INSTALL.md) to download the tool.

Since the tool has to be used in a repo which has clean clone, we may need to below step to clone it to another repo in order to use it:

```dos
cd ..
git clone git-test git-test-new
```

<!--
```bash
...
```
-->

Then run below command to remove the file from the git history

```dos
cd git-test-new
git filter-repo --invert-paths --path git.tgz
git gc
```

<!--
```bash
...
```
-->

Then we should be able to see the size is reduced

```dos
du -sh
git count-objects -v
```

<!--
```bash
...
```
-->

Lastly, we can push the change to the remote repo to finish this lab.

```dos
git push origin --force --all
```

<!--
```bash
...
```
-->
