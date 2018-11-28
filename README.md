# .gitignore

`.gitignore` file in your project's root directory prevents
 listed files from being pushed to and pulled from your remote Git repositories. 

- Git uses it to determine which files and directories to ignore before you make a commit.

- `.gitignore` itself should be committed into your repository, in order to share the ignore rules with any other users that clone the repository.

## Usage:

- `$ touch .gitignore`
- list the files or directories you want to ignore, one per line.
---
*ex .gitignore - things to ignore:*

---
**Note**: Often there are multiple ways to select a file/dir to ignore with a **globbing pattern**.
- *ex*. below, `node_modules` and `node_modules/` will both select the `node_modules` folder we are aiming for (the latter is just more specific and says only dirs not files)

```Scheme

'Ignore NPM'
===================================================================

node_modules/         ;bc when run npm install, it'll refetch all the modules specified in package.json and create a new node_modules/ to store them in locally
bundle.js             ;bc ignore compiled code & regenerates when run `webpack`
bundle.js.map         ;bc ignore compiled code & regenerates when run `webpack` 


'Files Generated at Runtime'
===================================================================

log/                ;ignores any dirs (and contents) named 'log' anywhere in path
/tmp/               ;ignores dirs (and contents) that are in repo root
Gemfile.lock
db                  ;any file or dir (but here obv file) ending in .db anywhere in path
*.sqlite
pids                ;any 'pids' (file or dir but here obv dir) anywhere in path     
*.pid


'Hidden System Files'   
==================================================================

.DS_Store


'Environment Normalisation'   
==================================================================

/.bundle/           ;some dir can actually start with '.'
/vendor/bundle      ;chained paths always relative to repo root, prepended '/' not neccessary but explicit


'Rails'
==================================================================

.byebug_history     ;but more likely leading '.' is a file
/public/system

```

---

[More comprehensive .gitignore](../master/.gitignore)

---

## Rules

`.gitignore` uses **globbing patterns** to match against file names. You can construct your patterns using various symbols:

<kbd>#</kbd> -  comments

<br>

\*** ***dir** matches mean the dir at **that point** in the path and any **paths underneath** it (so any contained subdir and files)* not the whole chain ***

Symbol | Pattern | Match ex's | Description |  Path Specifcation   |
--- | --- | --- | ---|  --- |  
  <kbd> </kbd> | `foo`  | matches:<ol><li>**dir** `foo`</li><li>**file** `foo`</li></ol>  <ul><li>*`foo`*</li><li>*`foo/debug.bar`*</li> <li>*`foo/debugger/debug.bar`*</li><li>*`assets/foo`*</li> <li>*`assets/foo/debug.bar`*</li></ul> | By default, patterns match anywhere. So w/ no slash, any **dir** or **file** named `foo` will be ignored. | **None** - _(matches anywhere in repo)_ |
 <kbd> /</kbd> | `foo/` |matches: <ol><li>**dir** `foo`</li></ol> <ul><li>*examples will be same as above, but assuming foo must always be a dir.*</li></ul>|  **Appending** <kbd> /</kbd>  indicates to match a **dir** `foo`, ignored anywhere | **None** - _(matches anywhere in repo)_
 <kbd> /</kbd> |   `/foo`| matches: <ol><li>**dir** `foo`</li><li>**file** `foo`</li></ol> <ul><li>*matches `foo`*</li><li>~~*but not `assets/foo`*~~</li> | **Prepending** <kbd> /</kbd> matches **dirs** or **files** only in the **repository root**. This is like normal shell pattern  | **Root** - _(root of repository only)_
 <kbd> /</kbd> |  `foo/debug.bar`  | <ul><li>*matches `foo/debug.bar`*</li> <li>~~*but not `debug.bar`*~~</li> <li>~~*not`assets/foo/debug.bar`*~~ </li>| Pattern specifying a **subdir**/**file** **in specific** **dir** are relative to **root**. (_Can prepend <kbd> /</kbd> to be explicit, but don't need it now_)| **Root** - _(path must begin from root of repository only)_ |
 | <kbd>*</kbd>  |   `*.bar`  | matches: <ol><li>**dir** ending in _`.bar`_ _(if that were possible)_</li><li>**file** ending in _`.bar`_</li></ol> <ul><li>*`.bar`*</li><li>*`debug.bar`*</li><li>*`foo/debug.bar`*</li> | <kbd>*</kbd> is a wildcard that matches 0 or more chars. No specificity for **dir** or **file**.   | **None** - _(doesn't effect path, so anywhere in repo)_</li></ul> |
 | <kbd>**</kbd>   | `**/foo`  | `**/foo` === `foo` |  A **leading** <kbd>**</kbd> followed by a <kbd> /</kbd> means match in all **dir**.  | **None** - _(matches anywhere in repo)_ |


---

### **Ignoring a previously committed file**
If you want to ignore a file that you've committed in the past, you'll need to remove / cache it, then add a `.gitignore` rule for it. (Simply adding it to `.gitignore` after you've committed it appears to not do anything)

Use **`git rm --cached`**

> Using the `--cached` option with `git rm` means that the file will be deleted from your remote repository, but will remain in your working directory as an ignored file. 

>You can omit the `--cached` option if you want to delete the file from _both_ the remote repository and your local file system.

_below:_ the bash command **`echo`** accepts a string as standard input and redirects, or echoes, it back to the terminal as standard output. If we use it with **`>>`(double redirection arrow)** , it appends it to the file on the right instead of the terminal. (this would be the same as just typing `debug.log` into `.gitignore`). the **`git rm --cached`** is the important part, line below it is just what it returns. then commit.

```bash

$ echo debug.log >> .gitignore
$ git rm --cached debug.log
rm 'debug.log'
$ git commit -m "Start ignoring debug.log"

```

- **note:** to the `rm --cached` you **need to pass the actual relative path to the file** not the same pattern conventions as `.gitignore` file.

[https://www.atlassian.com/git/tutorials/saving-changes/gitignore](https://www.atlassian.com/git/tutorials/saving-changes/gitignore)