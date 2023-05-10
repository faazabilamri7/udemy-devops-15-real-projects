# Lab 015: Remove Large File from Git Commit History

Windows + Ubuntu

## Lab goal

This lab specifically focuses on the process of removing large file objects from the Git history.

**Be warned: this technique is destructive to our commit history.**

We have to notify all contributors that they must rebase their work onto our new commits.

## Prerequisites

<!-- - Ubuntu 20.04 OS (Minimum 2 core CPU/8GB RAM/30GB Disk)

- [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) -->

### Install VirtualBox for Windows

### Install Vagrant for Windows

### Start Vagrant with the Ubuntu file

```dos
vagrant up

vagrant ssh
```

### Install git-filter-repo

<https://github.com/newren/git-filter-repo>

<https://github.com/briansu2004/git-filter-repo/blob/main/INSTALL.md>

<https://github.com/briansu2004/git-filter-repo/blob/main/git-filter-repo>

<https://raw.githubusercontent.com/newren/git-filter-repo/main/git-filter-repo>

```bash
sudo apt install python3-pip
python3 -m pip install --user git-filter-repo
export PATH=$PATH:/home/vagrant/.local/bin
```

-->

<!--
vagrant@vagrant:~/git-test-new$ curl -L https://raw.githubusercontent.com/newren/git-filter-repo/main/git-filter-repo > git-filter-repo
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  161k  100  161k    0     0  1349k      0 --:--:-- --:--:-- --:--:-- 1349k
vagrant@vagrant:~/git-test-new$ ls -l
total 168
-rw-rw-r-- 1 vagrant vagrant     11 May 10 15:43 file1
-rw-rw-r-- 1 vagrant vagrant 165782 May 10 15:52 git-filter-repo
vagrant@vagrant:~/git-test-new$ python3 git-filter-repo --analyze
Processed 2 blob sizes
Processed 3 commits
Writing reports to .git/filter-repo/analysis...done.

vagrant@vagrant:~/git-test-new$ sudo apt-get install git-filter-repo
Reading package lists... Done
Building dependency tree       
Reading state information... Done
E: Unable to locate package git-filter-repo
-->

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
vagrant@vagrant:~$ mkdir git-test
est
git init
echo "Init setup" > file1
git add file1
git commit -m "Version 1"
duvagrant@vagrant:~$ cd git-test
vagrant@vagrant:~/git-test$ git init
Initialized empty Git repository in /home/vagrant/git-test/.git/
vagrant@vagrant:~/git-test$ echo "Init setup" > file1
vagrant@vagrant:~/git-test$ git add file1
vagrant@vagrant:~/git-test$ git commit -m "Version 1"
[master (root-commit) 622a65c] Version 1
 Committer: vagrant <vagrant@vagrant.vm>
our name and email address were configured automatically based
on our username and hostname. Please check that they are accurate.
We can suppress this message by setting them explicitly. Run the
following command and follow the instructions in our editor to edit
our configuration file:

    git config --global --edit

After doing this, we may fix the identity used for this commit with:

    git commit --amend --reset-author

 1 file changed, 1 insertion(+)
 create mode 100644 file1
vagrant@vagrant:~/git-test$ du -sh ./
172K    ./
vagrant@vagrant:~$ 
vagrant@vagrant:~$ curl -L https://www.kernel.org/pub/software/scm/git/git-2.1.0.tar.gz > git.tgz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed  
100   162  100   162    0     0    425      0 --:--:-- --:--:-- --:--:--   425 
100 4859k  100 4859k    0     0   394k      0  0:00:12  0:00:12 --:--:--  553k
vagrant@vagrant:~/git-test$ git add git.tgz
dd git tarball'
du -sh ./
vagrant@vagrant:~/git-test$ git commit -m 'Add git tarball'
[master 8778cc0] Add git tarball
 Committer: vagrant <vagrant@vagrant.vm>
our name and email address were configured automatically based
on our username and hostname. Please check that they are accurate.
We can suppress this message by setting them explicitly. Run the
following command and follow the instructions in our editor to edit
our configuration file:

    git config --global --edit

After doing this, we may fix the identity used for this commit with:

    git commit --amend --reset-author

 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 git.tgz
vagrant@vagrant:~/git-test$ du -sh ./
9.7M    ./

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
vagrant@vagrant:~/git-test$ git gc
-vEnumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (7/7)
Writing objects: 100% (7/7), done.
Total 7 (delta 1), reused 0 (delta 0)
vagrant@vagrant:~/git-test$ du -sh
5.0M    .
vagrant@vagrant:~/git-test$ git count-objects -v
count: 0
size: 0
in-pack: 7
packs: 1
size-pack: 4861
prune-packable: 0
garbage: 0
size-garbage: 0
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
vagrant@vagrant:~/git-test$ git verify-pack -v .git/objects/pack/pack-180fe43703f733dfd1a44cffc10190f35a141f90.idx | sort -k 3 -n | tail -3
8778cc0103ac4e923127eef5012d84322d81e515 commit 220 146 163
94e097352fa20456984e4c603273d27ee65003ff commit 225 151 12
82c99a3e86bb1267b236a4b6eff7868d97489af1 blob   4975916 4976258 496
-->

Run below command to find out what the file name of the blob:

```dos
git rev-list --objects --all | grep 82c99a3
```

<!--
vagrant@vagrant:~/git-test$ git rev-list --objects --all | grep 82c99a3
82c99a3e86bb1267b236a4b6eff7868d97489af1 git.tgz
-->

Note: `82c99a3` is the SHA we found in previous command. It may be different in your scenario.

### 5. **Remove** the File from History

We are going to use `git-filter-repo` command to rewrite the git history. This doesn't come with the original setup and we have to install it seperately. We can follow [this document](https://github.com/newren/git-filter-repo/blob/main/INSTALL.md) to download the tool.

Since the tool has to be used in a repo which has clean clone, we may need to below step to clone it to another repo in order to use it:

```dos
cd ..
git clone git-test git-test-new
```

Then run below command to remove the file from the git history

```dos
cd git-test-new
git filter-repo --invert-paths --path git.tgz
git gc
```

Then we should be able to see the size is reduced

```dos
du -sh
git count-objects -v
```

Lastly, we can push the change to the remote repo to finish this lab.

```dos
git push origin --force --all
```

<!--
## Post Project

Just remove the repo folders

```dos
rm -rf git-test git-test-new
```
-->
