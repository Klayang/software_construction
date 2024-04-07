Q: how does merge exactly work? What counts as a conflict?

###### Inventing Version Control

Say we have backup files on laptop & desktop machines, with cloud as interchange

- We download the `version5` on laptop, make some changes (now as`version6`)

- However, we forgot it & download `version5` on desktop, make changes & upload
  
  - The changes we made are different. So call these 2 `version6L` & `version6D`

- A fews hours later, we realize we also made changes on laptop, so we upload `6L`
  
  - Bad! Changes in `6D` are overwritten, as they share the file name (`version6`)

    

###### Copy an Object Graph with git clone

The following command actually does 3 things for us:

```shell
git clone ssh://mit.edu/.../ps0-bit.git ps0
```

- Create an empty local directory `ps0`, and `ps0/.git`

- Connect to `mit.edu` & copy the object graph from `ps0-bit.git` into `ps0/.git`

- **Check out** the current version of the `main` branch

    

###### Head Count

Run corresponding commands to answer the following question of head count:

- How many commits are there in a repo?
  
  ```shell
  git log
  ```

- How many different versions of <file_A> are there?
  
  ```shell
  git log <file_A>
  ```

- How many files have been added / modified / deleted in the repo?
  
  ```shell
  git log --name-status
  ```

    

###### Merge

See how `git pull` merges the remote repo to the local one if changes occur: [link](http://web.mit.edu/6.031/www/sp21/classes/05-version-control/#merging)

    

# 
