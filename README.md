# git-merge-vs-rebase

What happens when master is merged to feature branch? Why is it bad to merge from master to feature branch? How can rebase help avoid the problems due to merging master to feature branch. Lets learn by example.

## Setup repo

#### Create a git repo

`$ git init`  
```bash
Initialized empty Git repository in /data/learn/git/MergeVsRebase/.git/
```

#### Add a line to a new file  
`$ printf "Line 1 (Added by master)\n" >> file.txt`  
>>```bash
Line 1 (Added by master)
```

#### Commit to master
`$ git add file.txt`  
`$ git commit -m "Commit 1 on master"`  
`$ git log`  
```bash
commit ee569e588696fed3c6db9eaf5b9873a5f0fc4107
Time: 14:41:37
    Commit 1 on master
```

#### Create a feature branch
`$ git checkout -b featureBranch`  
> `master` and `feature branch` are ready.

##Rebase Scenario 1:  
feature branch gets new changes after master gets some changes.

#### Add line 2 in master

>>```bash
Line 1 (Added by master)
Line 2 (Added by master)
```

`$ git commit -am "Commit 2 on master"`  
```bash
commit c6ab6ef0a45c84e7d623812ee798410e6ca072b3
Time: 14:45:26
    Commit 2 on master

commit ee569e588696fed3c6db9eaf5b9873a5f0fc4107
Time: 14:41:37
    Commit 1 on master
```

#### Add line 2 in feature branch
>>```bash
Line 1 (Added by master)
Line 2 (Added by featureBranch)
```

```bash
commit bbcf40c01df790f1565e388473da60de25d377af
Time: 14:47:42
    Commit 2 on feature branch

commit ee569e588696fed3c6db9eaf5b9873a5f0fc4107
Time: 14:41:37
    Commit 1 on master
```

### Rebase (Pull new changes from master into feature branch)

`$ git rebase master`  
```bash
First, rewinding head to replay your work on top of it.
.
.
Applying: Commit 2 on feature branch
Using index info to reconstruct a base tree.
.
.
M file.txt
Falling back to patching base and 3-way merge.
.
.
Auto-merging file.txt
CONFLICT (content): Merge conflict in file.txt
error: Failed to merge in the changes.
Patch failed at 0001 Commit 2 on feature branch
The copy of the patch that failed is found in: .git/rebase-apply/patch

When you have resolved this problem, run "git rebase --continue".
If you prefer to skip this patch, run "git rebase --skip" instead.
To check out the original branch and stop rebasing, run "git rebase --abort".

```
#### Conflict?!

>>```bash
Line 1 (Added by master)
<<<<<<< c6ab6ef0a45c84e7d623812ee798410e6ca072b3
Line 2 (Added by master)
=======
Line 2 (Added by featureBranch)
>>>>>>> Commit 2 on feature branch
```

>Master changes are inserted above feature branch changes, which is what we need.  

#### Resolve conflict
>>```bash
Line 1 (Added by master)
Line 2 (Added by master)
Line 2 (Added by featureBranch)
``` 

#### Continue rebase  
`$ git rebase --continue`  
```bash
Applying: Commit 2 on feature branch
```

```bash
commit 300d92083bd1d31d42b85fac704146cca13e7663
Time: 14:47:42
    Commit 2 on feature branch

commit c6ab6ef0a45c84e7d623812ee798410e6ca072b3
Time: 14:45:26
    Commit 2 on master

commit ee569e588696fed3c6db9eaf5b9873a5f0fc4107
Time: 14:41:37
    Commit 1 on master
```

>'Commit 2 on master' is added before 'Commit 2 on feature branch'

#### Deliver (Merge feature branch to master)
`$ git checkout master`  
`$ git merge featureBranch`  
```bash
commit 300d92083bd1d31d42b85fac704146cca13e7663
Time: 14:47:42
    Commit 2 on feature branch

commit c6ab6ef0a45c84e7d623812ee798410e6ca072b3
Time: 14:45:26
    Commit 2 on master

commit ee569e588696fed3c6db9eaf5b9873a5f0fc4107
Time: 14:41:37
    Commit 1 on master
```

> Commit logs on master and branch looks exactly the same. No extra merge commits.  

## Rebase Scenario 2:  
master gets new changes after a feature branch gets some changes.

#### Add line 3 in feature branch

>>```bash
Line 1 (Added by master)
Line 2 (Added by master)
Line 2 (Added by featureBranch)
Line 3 (Added by featureBranch)
```

```bash
commit f2cdee65c63c00d9d771b904ecaa58701f529981
Time: 15:01:24
    Commit 3 on feature branch

commit 300d92083bd1d31d42b85fac704146cca13e7663
Time: 14:47:42
    Commit 2 on feature branch
.
.
```

#### Add line 3 in master. 

>>```bash
Line 1 (Added by master)
Line 2 (Added by master)
Line 2 (Added by featureBranch)
Line 3 (Added by master)
```

```bash
commit 19c61e4a3864c5b3a020fcd3b162065ee1854d43
Time: 15:03:18
    Commit 3 on master

commit 300d92083bd1d31d42b85fac704146cca13e7663
Time: 14:47:42
    Commit 2 on feature branch
.
.
```

### Rebase (Pull new changes from master into feature branch)
`$ git checkout featureBranch`  
`$ git rebase master`  
```bash
First, rewinding head to replay your work on top of it.
.
.
Applying: Commit 3 on feature branch
Using index info to reconstruct a base tree.
.
.
M file.txt
Falling back to patching base and 3-way merge.
.
.
Auto-merging file.txt
CONFLICT (content): Merge conflict in file.txt
error: Failed to merge in the changes.
Patch failed at 0001 Commit 3 on feature branch
The copy of the patch that failed is found in: .git/rebase-apply/patch

When you have resolved this problem, run "git rebase --continue".
If you prefer to skip this patch, run "git rebase --skip" instead.
To check out the original branch and stop rebasing, run "git rebase --abort".
```
#### Conflict

>>```bash
Line 1 (Added by master)
Line 2 (Added by master)
Line 2 (Added by featureBranch)
<<<<<<< 19c61e4a3864c5b3a020fcd3b162065ee1854d43
Line 3 (Added by master)
=======
Line 3 (Added by featureBranch)
>>>>>>> Commit 3 on feature branch
```
>Master changes are inserted above feature branch changes.  

#### Resolve conflict

>>```bash
Line 1 (Added by master)
Line 2 (Added by master)
Line 2 (Added by featureBranch)
Line 3 (Added by master)
Line 3 (Added by featureBranch)
```

#### Continue rebase
`$ git rebase --continue`  
```bash
commit f724946335d46b61d258d5a2d9e11bb152c1458c
Time: 15:01:24
    Commit 3 on feature branch

commit 19c61e4a3864c5b3a020fcd3b162065ee1854d43
Time: 15:03:18
    Commit 3 on master

commit 300d92083bd1d31d42b85fac704146cca13e7663
Time: 14:47:42
    Commit 2 on feature branch
.
.
```
>'Commit 3 on master' is added before 'Commit 3 on feature branch'. Note the timestamp and the order of the commits.

#### Deliver (Merge feature branch to master)
`$ git checkout master`  
`$ git merge featureBranch`  
```bash
Updating 19c61e4..f724946
Fast-forward
 file.txt | 1 +
 1 file changed, 1 insertion(+)
```
```bash
commit f724946335d46b61d258d5a2d9e11bb152c1458c
Time: 15:01:24
    Commit 3 on feature branch

commit 19c61e4a3864c5b3a020fcd3b162065ee1854d43
Time: 15:03:18
    Commit 3 on master

commit 300d92083bd1d31d42b85fac704146cca13e7663
Time: 14:47:42
    Commit 2 on feature branch
.
.
```
> Commit logs on master and branch looks exactly the same. No extra merge commits again.  

# Now, lets merge instead of rebase.

## Merge Scenario 1:
master gets new changes after a feature branch gets some changes.

#### Add Line 4 to feature branch
`$ git checkout featureBranch`  
`$ printf "Line 4 (Added by featureBranch)\n" >> file.txt`  
>>```bash
Line 1 (Added by master)
Line 2 (Added by master)
Line 2 (Added by featureBranch)
Line 3 (Added by master)
Line 3 (Added by featureBranch)
Line 4 (Added by featureBranch)
```
`$ git commit -am "Commit 4 on feature branch"`  
```bash
commit af04a4bca56c3d35a58e2e63faa750c4a39f7b87
Time: 15:10:19
    Commit 4 on feature branch

commit f724946335d46b61d258d5a2d9e11bb152c1458c
Time: 15:01:24
    Commit 3 on feature branch
.
.
```

#### Add Line 4 to master
`$ git checkout master`  
`$ printf "Line 4 (Added by master)\n" >> file.txt`  
>>```bash
Line 1 (Added by master)
Line 2 (Added by master)
Line 2 (Added by featureBranch)
Line 3 (Added by master)
Line 3 (Added by featureBranch)
Line 4 (Added by master)
```
`$ git commit -am "Commit 4 on master"`  
```bash
commit f485c9835e069b89a08da2388474ab0a160f9b93
Time: 15:11:55
    Commit 4 on master

commit f724946335d46b61d258d5a2d9e11bb152c1458c
Time: 15:01:24
    Commit 3 on feature branch
.
.
```

`$ git checkout featureBranch`  

#### Merge (Pull master to feature branch)
`$ git merge master`  
```bash
Auto-merging file.txt
CONFLICT (content): Merge conflict in file.txt
Automatic merge failed; fix conflicts and then commit the result.
```

##### Conflict

>>```bash
Line 1 (Added by master)
Line 2 (Added by master)
Line 2 (Added by featureBranch)
Line 3 (Added by master)
Line 3 (Added by featureBranch)
<<<<<<< HEAD
Line 4 (Added by featureBranch)
=======
Line 4 (Added by master)
>>>>>>> master
```

#### Resolve conflict

>>```bash
Line 1 (Added by master)
Line 2 (Added by master)
Line 2 (Added by featureBranch)
Line 3 (Added by master)
Line 3 (Added by featureBranch)
Line 4 (Added by featureBranch)
Line 4 (Added by master)
```
> Note the order of changes. Changes in feature branch is added above master changes. This order 

#### Commit the merge. 
`$ git commit -am "Merge 4 from master to feature branch"`  
> This is an extra commit which could have been avoided by rebasing.

```bash
commit b5db134ca93cdb0629df76b8feb22015d2fd3680
Merge: af04a4b f485c98
Time: 15:15:58

    Merge 4 from master to feature branch

commit f485c9835e069b89a08da2388474ab0a160f9b93
Time: 15:11:55
    Commit 4 on master

commit af04a4bca56c3d35a58e2e63faa750c4a39f7b87
Time: 15:10:19
    Commit 4 on feature branch

commit f724946335d46b61d258d5a2d9e11bb152c1458c
Time: 15:01:24
    Commit 3 on feature branch
.
.
```
> Note: master changes are added after feature branch changes. 

`$ git checkout master`  
`$ git merge featureBranch`  
```bash
Updating f485c98..b5db134
Fast-forward
 file.txt | 1 +
 1 file changed, 1 insertion(+)
```
```bash
commit b5db134ca93cdb0629df76b8feb22015d2fd3680
Merge: af04a4b f485c98
Time: 15:15:58

    Merge 4 from master to feature branch

commit f485c9835e069b89a08da2388474ab0a160f9b93
Time: 15:11:55
    Commit 4 on master

commit af04a4bca56c3d35a58e2e63faa750c4a39f7b87
Time: 15:10:19
    Commit 4 on feature branch

commit f724946335d46b61d258d5a2d9e11bb152c1458c
Time: 15:01:24
    Commit 3 on feature branch
.
.
```

## Merge Scenario 2:
feature branch gets new changes after a master gets some changes.

#### Add Line 5 to master
`$ git checkout master`  
`$ printf "Line 5 (Added by master)\n" >> file.txt`  
>>```bash
Line 1 (Added by master)
Line 2 (Added by master)
Line 2 (Added by featureBranch)
Line 3 (Added by master)
Line 3 (Added by featureBranch)
Line 4 (Added by featureBranch)
Line 4 (Added by master)
Line 5 (Added by master)
```
`$ git commit -am "Commit 5 on master"`  
```bash
commit b8ae94db961c27a3612d57a30b26a91c2a242108
Time: 19:16:52

    Commit 5 on master

commit b5db134ca93cdb0629df76b8feb22015d2fd3680
Merge: af04a4b f485c98
Time: 15:15:58

    Merge 4 from master to feature branch
.
.
```


#### Add Line 4 to feature branch

`$ git checkout featureBranch`  
`$ printf "Line 5 (Added by featureBranch)\n" >> file.txt`  
>>```bash
Line 1 (Added by master)
Line 2 (Added by master)
Line 2 (Added by featureBranch)
Line 3 (Added by master)
Line 3 (Added by featureBranch)
Line 4 (Added by featureBranch)
Line 4 (Added by master)
Line 5 (Added by featureBranch)
```
`$ git commit -am "Commit 5 on feature branch"`  
```bash
commit a38b956c67588bbd7ab89dc84d4cf6ff0cb0eddd
Time: 19:22:58

    Commit 5 on feature branch

commit b5db134ca93cdb0629df76b8feb22015d2fd3680
Merge: af04a4b f485c98
Time: 15:15:58

    Merge 4 from master to feature branch
.
.
```

#### Merge (Pull master to feature branch)
`$ git merge master`  
```bash
Auto-merging file.txt
CONFLICT (content): Merge conflict in file.txt
Automatic merge failed; fix conflicts and then commit the result.
```

##### Conflict

>>```bash
Line 1 (Added by master)
Line 2 (Added by master)
Line 2 (Added by featureBranch)
Line 3 (Added by master)
Line 3 (Added by featureBranch)
Line 4 (Added by featureBranch)
Line 4 (Added by master)
<<<<<<< HEAD
Line 5 (Added by featureBranch)
=======
Line 5 (Added by master)
>>>>>>> master
```


#### Resolve conflict

>>```bash
Line 1 (Added by master)
Line 2 (Added by master)
Line 2 (Added by featureBranch)
Line 3 (Added by master)
Line 3 (Added by featureBranch)
Line 4 (Added by featureBranch)
Line 4 (Added by master)
Line 5 (Added by featureBranch)
Line 5 (Added by master)
```

> Note the order of changes. Changes in feature branch is added above master changes. This order 

#### Commit the merge. 
`$ git commit -am "Merge 5 from master to feature branch"`  
> This is an extra commit which could have been avoided by rebasing.

```bash
commit 8d059bac457d09de7771470d316e3d10a142ede1
Merge: a38b956 b8ae94d
Time: 19:31:46

    Merge 5 from master to feature branch

commit a38b956c67588bbd7ab89dc84d4cf6ff0cb0eddd
Time: 19:22:58

    Commit 5 on feature branch

commit b8ae94db961c27a3612d57a30b26a91c2a242108
Time: 19:16:52

    Commit 5 on master

commit b5db134ca93cdb0629df76b8feb22015d2fd3680
Merge: af04a4b f485c98
Time: 15:15:58

    Merge 4 from master to feature branch
.
.
```

> Note: master changes are added after feature branch changes. 

`$ git checkout master`  
`$ git merge featureBranch`  
```bash
Updating b8ae94d..8d059ba
Fast-forward
 file.txt | 1 +
 1 file changed, 1 insertion(+)
```
```bash
commit 8d059bac457d09de7771470d316e3d10a142ede1
Merge: a38b956 b8ae94d
Time: 19:31:46

    Merge 5 from master to feature branch

commit a38b956c67588bbd7ab89dc84d4cf6ff0cb0eddd
Time: 19:22:58

    Commit 5 on feature branch

commit b8ae94db961c27a3612d57a30b26a91c2a242108
Time: 19:16:52

    Commit 5 on master

commit b5db134ca93cdb0629df76b8feb22015d2fd3680
Merge: af04a4b f485c98
Time: 15:15:58

    Merge 4 from master to feature branch
.
.
```


