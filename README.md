# .gitignore

**Problem:**

Git doesn't automatically track files because we don't want to record generated files (logs, compiled modules, etc). 

But, seeing these files under the "Untracked files" list in `git status` would be confusing for large repositories.

And inconvenient because you would never be able to use `git add .`

**Solution:**

A `gitignore` file specifies intentionally untracked files or directories that Git should ignore when a user makes commits to their remote Git repository. 

  * They will be essentially invisible to Git.

`gitignore` itself should be committed to the remote repo in order to share the ignore rules with any other users that may clone it.

<br>

Common Patterns for Node/Rails:

  * _note: styled for visual convenience_

```Julia

Ignore NPM
===================================================================

node_modules/       # npm install refetches modules specified in package.json & creates new node_modules/ to store them in locally
bundle.js           # ignore compiled code & regenerates when run `webpack`
bundle.js.map       # ignore compiled code & regenerates when run `webpack`


Files Generated at Runtime
===================================================================

log/                # matches any dirs (and contents) named 'log' anywhere in path
/tmp/               # matches any dirs (and contents) named 'tmp' only from repo root
*.db                # matches any files (or dirs) ending in .db anywhere in path
*.sqlite
pids                # matches any files or dirs named 'pids' anywhere in path  
*.pid


Hidden System Files   
==================================================================

.DS_Store            # often see files starting with '.'


Environment Normalization  
==================================================================

/.bundle/           # but dirs can start with '.' too
/vendor/bundle      # chained paths are always relative to repo root, prepended '/' not neccessary but explicit


Rails
==================================================================

.byebug_history
/public/system

```

---

[More comprehensive .gitignore](../master/.gitignore.md)

---
## Fundamentals: 

- `$ touch .gitignore` - file can only be created manually

- Each line specifies a pattern for the path to ignore.

<hr>

>**Pattern**: a string with a special format designed to match filenames, or to check, classify or validate data strings.

<hr>


**Globbing patterns**: `gitignore` uses **wildmatch** for pattern matching: a combination of  globbing patterns + some additioanl rules to match against file / directory names. 

  * We can construct the patterns using normal characters and various **metacharaters**. 

  * In Github, these patterns are referred to as a **`<pathspec>`** :a pattern used to limit paths in Git commands.

    * Certain Git commands accept `<pathspec>`s that follow wildmatch convention, others only accept standard shell glob pattern convention  

  * Usage rules for most common ones are outlined in a table below
    

<br>

Git commands such as `git status` and `git add` then use these constructed patterns. 
- Note the `.gitignore` file is actually 1 of multiple sources where Git checks gitignore patterns from. 

- http://web.mit.edu/git/www/gitignore.html


## Metacharacters and Common Patterns

_Assumptions_:

1. Directory matches include the directory at **that point** in the path and any **paths underneath** it (so any contained subdir and files)

2. The table / formatting notes below assume the user's `.gitignore` is in the root directory, so top-level and "relative to the directory containing `.gitignore`" are interchangable.

    > "The convention, and simplest approach, is to define a single `.gitignore` file in the root. However, you can choose to define multiple `.gitignore` files in different directories in your repo. Each pattern in a particular `.gitignore` is tested relative to the directory containing that file."


<br>


_Pattern format:_

1.  <kbd>#</kbd> - comment

2. Any path matches itself

3. Standard shell **glob patterns** work, but they will be applied **recursively** throught the _entire_ working tree (see edge cases in Wildmatch section).

    * Also worth noting <kbd>.</kbd> has no special meaning (can't do <kbd>./</kbd> for ex. and don't really need to).

4. You can start patterns with a forward slash (<kbd>/</kbd>) to avoid recursivity (pins the patterns to only match from the top of the working tree).

5. You can end patterns with a forward slash (<kbd>/</kbd>) to specify a directory. 

    * Patterns match directories with or without this. It's used mainly for excluding files from being matched.

6. You can negate a pattern by starting it with an exclamation point (<kbd>!</kbd>).

    * But not if its parent directory has been excluded.
  
    * > Git doesn’t list excluded directories for performance reasons, so any patterns on contained files have no effect, no matter where they are defined.

--- 

**Glob patterns** 

  * Are like simplified regular expressions that shells use.
    
    * One use is to quickly navigate around files: for ex. from Desktop, running `$ cd **/week\ 6/` bypasses typing the name of the folder `week 6` is in first. 
    
    * Note that in terminal, <kbd>**</kbd> only lets you nest 2 levels deep, but in `.gitignore` it's a little different. (see below). 

7. <kbd>*</kbd>  is a wildcard that matches zero or more characters.

    * Often used to ignore files with a particular extension.

    * Allows partial file / directory names:

      * You cannot search for partial file or directory _names_ (not paths) by default. You have to append a metacharacter that modifies this behavior (for ex. <kbd>*</kbd>).

 
      * ex: the pattern `contro` in .gitignore

        * will _not_ ignore your `controllers` folder 
    
        * but the pattern `controllers` or `contro*` will


  8. `[abc]` matches any character inside the brackets (in this case a, b, or c).

  9. (<kbd>?</kbd>) matches a single character.

  10. Brackets enclosing characters separated by a hyphen (`[0-9]`) matches any character between them (in this case 0 through 9). 

  --- 

11. You can use <kbd>**</kbd> to match nested directories; `a/**/z` would match `a/z`, `a/b/z`, `a/b/c/z`, and so on.

    * Only valid if you use them with a <kbd>/</kbd> like above or one of 2 ways below:

      a) <kbd>**/</kbd> appended to the front: matches in all directories (it matches any arbitrary path of directories and subdir to get to the file/dir).

      * You don't need to write out the entire path to a file or dir. For ex. writing `en.yml` will match the file with that name in `app/config/locales/application_controller/en.yml`.

      b) <kbd>/**</kbd> appended to the back: matches everything inside (all subdirectories / files).

      * Use it to ignore entire folders.
  
--- 

**wildmatch specific**:

12. A <kbd>/</kbd> in a pattern effects how Git uses it to check for a match.

  * **By default (no <kbd>/</kbd>), patterns in `.gitignore` match files in any directory**.

      * meaning `controllers` or `contro*` would match: 
        * `app/foo/bar/controllers/baz.html`.

        * `app/controllers/foo`.
    
  
Following this logic, the below pattern with <kbd>*</kbd> appended to the front and no <kbd>/</kbd>:

  * `*comment_expand.html`, `*ent_expand.html`, `*.html` would all match `/spec/javascripts/fixures/comment_expand.html`.


12b. If the pattern _does_ contain a <kbd>/</kbd> , Git treats it as a standard shell glob. (even if it is just appended/prepended at the front/back).

* **nested paths will be relative to root**: patterns specifying a subdir/file in a specific directory need to start from the root directory. (You can prepend a <kbd>/</kbd> to be explicit but it wouldn't make a difference). 

* It will also use <kbd>*</kbd> following std. convention:

> Paths relative to the directory prefix will be matched against that pattern using fnmatch(3) and the FNM_PATHNAME flag; in particular, <kbd>*</kbd> and <kbd>?</kbd> can match directory separators.


For example, 
  * `"Documentation/*.html"` matches `"Documentation/git.html"` 
  * but not `"Documentation/ppc/ppc.html"`
  * also not `"tools/perf/Documentation/perf.html"`.

---

## Table for quick reference
(non-comprehensive)

Symbol | Pattern | Match exs | Description |  Path Specifcation   |
--- | --- | --- | ---|  --- |  
  | | `foo`  | matches:<ol><li>**dir** `foo`</li><li>**file** `foo`</li></ol>  <ul><li>*`foo`*</li><li>*`foo/debug.bar`*</li> <li>*`foo/debugger/debug.bar`*</li><li>*`assets/foo`*</li> <li>*`assets/foo/debug.bar`*</li><li> ~~*but not `foo.bar`*~~</li><li>~~*but not `fooing`*~~</li></ul> | By default, patterns match anywhere. So w/ no <kbd> /</kbd> , any **dir** or **file** named `foo` will be ignored. | **None** - _(matches anywhere in repo)_ |
 <kbd> /</kbd> | `foo/` |matches: <ol><li>**dir** `foo`</li></ol> <ul><li>*examples will be same as above, but assuming foo must always be a dir.*</li></ul>|  **Appending** <kbd> /</kbd>  indicates to match a **dir** `foo`, ignored anywhere | **None** - _(matches anywhere in repo)_
 <kbd> /</kbd> |   `/foo`| matches: <ol><li>**dir** `foo`</li><li>**file** `foo`</li></ol> <ul><li>*matches `foo`*</li><li>~~*but not `assets/foo`*~~</li> | **Prepending** <kbd> /</kbd> matches **dirs**/**files** only in **root**. (Std. shell pattern <kbd> /</kbd> means `cwd`; should be root here.)  | **Root** - _(root of repository only)_
 <kbd> /</kbd> |  `foo/debug.bar`  | <ul><li>*matches `foo/debug.bar`*</li> <li>~~*but not `debug.bar`*~~</li> <li>~~*not`assets/foo/debug.bar`*~~ </li>| Patterns specifying a **subdir**/**file** **in specific** **dir** are relative to **root**. (_Can prepend <kbd> /</kbd> to be explicit, but don't need it now_)| **Root** - _(path must begin from root of repository only)_ |
 | <kbd>*</kbd>  |   `*.bar`  | matches: <ol><li>**dir** ending in _`.bar`_ </li><li>**file** ending in _`.bar`_</li></ol> <ul><li>*`.bar`*</li><li>*`debug.bar`*</li><li>*`foo/debug.bar`*</li> | <kbd>*</kbd> is a wildcard that matches 0 or more chars. (Anything except <kbd> /</kbd>) No specificity for **dir** / **file**.   | **None** - _(doesn't effect path, so anywhere in repo)_</li></ul> 
 |  <kbd>*</kbd>  |   `foo*`   |  matches: <ol><li>**dir** beginning w/ _`foo`_ </li><li>**file** beginning w/ _`foo`_</li></ol> <ul><li>*`fooolder/debug.bar`*</li><li>*`foo`*</li><li>*`foo.txt`*</li><li>*`foobar.txt`*</li><li>`app/assets/foobar.txt`</li></ul>  |  Same description as above. When matching a pattern w/out a <kbd>/</kbd>, can match it anywhere |   **None** - _(doesn't effect path, so anywhere in repo)_ |
 | <kbd>**</kbd>+<kbd>/</kbd>   | `**/foo`  | matches: <ul><li> **file** or **dir** `foo` anywhere</li><li>`**/foo` === `foo`</li></ul> |**Leading** <kbd>**/</kbd>matches a **dir** anywhere in repo. (same as plain `foo`, just more explicit )  | **None** - _(matches anywhere in repo)_ |
 | <kbd> /</kbd>+<kbd>**</kbd>  |  `foo/**`  |  matches: <ul><li>all **files** / **dirs** inside **dir** `foo` w/ infinite depth</li><li>`foo/**` === `foo`</li></ul> | Trailing <kbd>/**</kbd> matches everything inside a **dir**, from anywhere in repo. (same as `foo`, but more explicit )  | **None** - _(matches anywhere in repo)_ |

---

### Ignoring a previously committed file

Files already tracked by Git are not effected by the gitignore file. If you want to ignore a file that you've committed in the past, you'll need to remove / cache it, then add a `.gitignore` rule for it. (Simply adding it to `.gitignore` after you've committed it appears to not do anything).

Use **`git rm --cached`**

> Using the `--cached` option with `git rm` means that the file will be deleted from your remote repository, but will remain in your working directory as an ignored file. 

>You can omit the `--cached` option if you want to delete the file from _both_ the remote repository and your local file system.

_below:_ the bash command **`echo`** accepts a string as standard input and redirects, or echoes, it back to the terminal as standard output. If we use it with **`>>`(double redirection arrow)**, it appends it to the file on the right instead of the terminal. (this would be the same as just typing `debug.log` into `.gitignore`). The **`git rm --cached`** is the important part. Then commit.

```bash

$ echo debug.log >> .gitignore
$ git rm --cached debug.log
rm 'debug.log'
$ git commit -m "Start ignoring debug.log"

```

- **note:** to the `rm --cached` you **need to pass the actual relative path to the file**; not the same pattern conventions as `.gitignore` file.

<hr>

### Debugging .gitignore files
To track down why a particular file is being ignored - use **`git check-ignore `** with the **`-v`** (or **`--verbose`**) option to determine which pattern is causing a particular file / directory to be ignored:

```bash

$ git check-ignore -v debug.log
.gitignore:3:*.log debug.log

```
The output shows:

```erb

<file containing the pattern> : <line number of the pattern> : <pattern> <file name>

```

<hr>

### Github Templates

Github provides us with a collection of `.gitignore` templates for specific programming languages, frameworks, tools, environments, etc. 

These templates are available in the Github.com interface:
1. When creating new repositories and files. (Dropdown at the bottom <kbd>`Add .gitignore: None`</kbd>) 
2. On the main page of the repo, if you select <kbd>`Create new file`</kbd> and type _.gitignore_ in the input for file name, template selector will appear again.
3. Links below


<hr>

### References

a. <cite>[Git Globbing Pattern reference](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository)</cite>

b. <cite>[Git Globbing Pattern reference #2](https://git-scm.com/docs/gitignore/2.19.0)</cite>

c. <cite>[Alternative Globbing Pattern reference](https://www.atlassian.com/git/tutorials/saving-changes/gitignore)</cite>

d. <cite>[Github .gitignores master repo](https://github.com/github/gitignore)</cite>

 - <cite>[Github .gitignore Node](https://github.com/github/gitignore/blob/master/Node.gitignore)</cite>
  - <cite>[Github .gitignore Rails](https://github.com/github/gitignore/blob/master/Rails.gitignore)</cite>
  - <cite>[Github .gitignore Ruby](https://github.com/github/gitignore/blob/master/Ruby.gitignore)</cite>