the .zip file provided has a source code, a config.json and a .git directory in it

reading the source code we see the token is being pulled from config.json
in config.json the token is a placeholder for security reasons

in the commit history on .git folder there is a commit message "removed unique token as it was a security risk"

it is possible then to check the commit history with 

```git
git log \\ to get the commit ids for the git diff command
git diff [commit-id1] [commit-id2]
```