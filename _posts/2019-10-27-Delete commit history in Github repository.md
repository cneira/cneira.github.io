I needed to remove all my commit history so I found this article in:
How to Delete Commit History in Github

Here are the steps to do it:
```bash
$ git checkout --orphan temp_branch 
$ git add -A 
$ git commit -am "the first commit" 
$ git branch -D master
$ git branch -m master 
$ git push -f origin master
```
