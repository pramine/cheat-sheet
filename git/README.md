# Git Cheat Sheet

## Tag

Create tag:

`$ git tag -a v1.0.0 -m "version 1.0.0"`

Show existing tags:

`$ git show`
 
Explicity push tags to remote server:

`$ git push origin v1.0.0`

Push several tags:

`$ git push origin --tags`

Delete tag:

`$ git tag -d v1.0.0`

Push deleted tag in remote:

`$ git push origin :refs/tags/v1.0.0`