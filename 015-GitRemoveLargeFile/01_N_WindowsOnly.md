# Lab 015: Remove Large File from Git Commit History

Windows only

## Lab goal

This lab specifically focuses on the process of removing large file objects from the Git history.

**Be warned: this technique is destructive to your commit history.**

We have to notify all contributors that they must rebase their work onto our new commits.

## Prerequisites

- Ubuntu 20.04 OS (Minimum 2 core CPU/8GB RAM/30GB Disk)

- [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

## Steps

### 1. **Initiate** the test environment

Run below command to initiate a git repo

```dos
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

### 4. **Find** Out the Location of The Large File in packfiles

You can find the SHA for the largest file in packfile

```dos
git verify-pack -v .git/objects/pack/<index name>.idx | sort -k 3 -n | tail -3
```

Note: Please replace **<index name>** with actual index name under `.git/objects/pack` folder. <br/>

Run below command to find out what the file name of the blob:

```dos
git rev-list --objects --all|grep 82c99a3
```

Note: `82c99a3` is the SHA you found in previous command. It may be different in your scenario.

### 5. **Remove** the File from History

We are going to use `git-filter-repo` command to rewrite the git history. This doesn't come with the original setup and you have to install it seperately. You can follow [this document](https://github.com/newren/git-filter-repo/blob/main/INSTALL.md) to download the tool. <br/>

Since the tool has to be used in a repo which has clean clone, you may need to below step to clone it to another repo in order to use it:

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

Then you should be able to see the size is reduced

```dos
du -sh
git count-objects -v
```

Last, you can push the change to the remote repo to finish this lab

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
