# GIT-Notes
Here you will find my notes on GIT

## Table of Contents

1. [Fundamental](#fundamental)
2. [Install GIT](#install-git)
3. [Clone an existing GIT repository or create a new one](#clone-an-existing-git-repository-or-create-a-new-one)
4. [How to get help on GIT](#how-to-get-help-on-git)
5. [How to know that all is stored on GIT](#how-to-know-that-all-is-stored-on-git)
6. [How to get status of a GIT Repository](#how-to-get-status-of-a-git-repository)
7. [How to add files to a GIT Repository](#how-to-add-files-to-a-git-repository)
8. [How to modify files already tracked](#how-to-modify-files-already-tracked)
9. [How to ignore one or more files from versioning](#how-to-ignore-one-or-more-files-from-versioning)
10. [Viewing your staged and unstaged changes](#viewing-your-staged-and-unstaged-changes)
11. [Commit your changes (git commit)](#commit-your-changes-git-commit)
12. [Removing files (git rm)](#removing-files-git-rm)
13. [Renaming files (git mv)](#renaming-files-git-mv)
14. [Viewing the commit history (git log)](#viewing-the-commit-history-git-log)
15. [Undoing things](#undoing-things)
16. [Showing your remote server (git remote)](#showing-your-remote-server-git-remote)
17. [Adding remote repositories](#adding-remote-repositories)
18. [Pushing to your remotes (git push)](#pushing-to-your-remotes-git-push)
19. [Inspecting a remote](#inspecting-a-remote)
20. [Removing or Renaming remotes](#removing-or-renaming-remotes)
21. [Tagging](#tagging)
22. [Branching - Useful to solve problems without touch the master branch until the resolution](#branching---useful-to-solve-problems-without-touch-the-master-branch-until-the-resolution)
23. [Working with pull requests](#working-with-pull-requests)
24. [How to clean bad data out of your Git repository history](#how-to-clean-bad-data-out-of-your-git-repository-history)

# Fundamental

The Git directory is where Git stores the metadata and object database for your project. This is the most important part of Git, and it is what is copied when you clone a repository from another computer.

The working directory is a single checkout of one version of the project. These files are pulled out of the compressed database in the Git directory and placed on disk for you to use or modify.

The staging area is a file, generally contained in your Git directory, that stores information about what will go into your next commit. It’s sometimes referred to as the “index”, but it’s also common to refer to it as the staging area.

The basic Git workflow goes something like this:

You modify files in your working directory.

You stage the files, adding snapshots of them to your staging area.

You do a commit, which takes the files as they are in the staging area and stores that snapshot permanently to your Git directory.

If a particular version of a file is in the Git directory, it’s considered committed. If it’s modified but has been added to the staging area, it is staged. And if it was changed since it was checked out but has not been staged, it is modified

# Install GIT
https://git-scm.com/book/en/v2/Getting-Started-Installing-Git

# GIT Utilities
## Clone an existing GIT repository or create a new one

1. Clone an existing repository to your "my_grit" working directory:
   * ```git clone git://github.com/schacon/grit.git my_grit```

OR

1. Create a new GIT Repository inside your "my_project" directory:
   * ```mkdir my_project```
   * ```cd my_project```
   * ```git init```
   
2. Customize your GIT project environment (locally to your "my_project" directory):
   * ```cd my_project```
   * ```git config user.name "John Doe"```
   * ```git config user.email johndoe@example.com```
   * ```git config core.editor vim```     (you can set also "emacs" as your prefer editor)
   * ```git config merge.tool vimdiff```  (you can set also "kdeff3", "tkdiff", "meld", "xxdiff", "emerge", "vimdiff", "gvimdiff", "ecmerge" or "opendiff")
   
3. Check your GIT settings:
   * ```git config --list```
   
## How to get help on GIT

   * ```git help <command-name>```  (example: "```git help config```")
   
## How to know that all is stored on GIT

   * ```git status```
   
   result:
     
   ``` 
   On branch master
   nothing to commit, working directory clean
   ```

## How to get status of a GIT Repository

   * Example of GIT Status after have done a change:
   
   ```git status```
   ```
   On branch master
   Untracked files:
      (use "git add <file>..." to include in what will be committed)

         README

   nothing added to commit but untracked files present (use "git add" to track)
   ================================================
   (this command tell you how add the README file to versioning, throught the "git add" command)
   ```
   
## How to add files to a GIT Repository

1. Example on how to add a file to "my_project" repository:
   * ```cd my_project```
   * ```touch README```
   * ```git add README .gitignore```     (<== ```git add <FILE_PATH_1 or DIR_PATH_1> <FILE_PATH_1 or DIR_PATH_1> ...```)
   * ```git status```
   ```
      On branch master
      Changes to be committed:                     (<== this line indicate that we are in the "stage" area)
        (use "git reset HEAD <file>..." to unstage)

              new file:   README
              new file:   .gitignore
   ```
   * ```git commit -m 'initial project version: Added README file'```
   
   NOTE: if you add an entire directory, you add all files inside that directory.

## How to modify files already tracked

1. Change an already tracked file (benchmarks.rb)
   * ```vim benchmarks.rb```
   
2. Show the new status:
   * ```git status```
     ```
     On branch master
     Changes to be committed:
       (use "git reset HEAD <file>..." to unstage)

             new file:   README

     Changes not staged for commit:
       (use "git add <file>..." to update what will be committed)
       (use "git checkout -- <file>..." to discard changes in working directory)

             modified:   benchmarks.rb
     ```
     
3. Add the modified file to the staging area:
   * ```git add benchmarks.rb```

4. View the new status:
   * ```git status```
     ```
     On branch master
     Changes to be committed:
       (use "git reset HEAD <file>..." to unstage)

             new file:   README
             modified:   benchmarks.rb
     ```
              
5. If you modify a file that it is already present on the stage area, you have to add it again to the stage area:
   * ```vim benchmarks.rb```
   * ```git status```
     ```
     On branch master
     Changes to be committed:
       (use "git reset HEAD <file>..." to unstage)

             new file:   README
             modified:   benchmarks.rb

     Changes not staged for commit:
       (use "git add <file>..." to update what will be committed)
       (use "git checkout -- <file>..." to discard changes in working directory)

             modified:   benchmarks.rb
     ```
   * ```git add benchmarks.rb```
   * ```git status```
     ```
      On branch master
      Changes to be committed:
        (use "git reset HEAD <file>..." to unstage)

              new file:   README
              modified:   benchmarks.rb
     ```

## How to ignore one or more files from versioning

   * If you want to ignore some files/directories to git versioning, you have to edit your "```.gitignore```" file by following these rules:

     * Blank lines or lines starting with **#** are ignored.

     * Standard glob patterns work.

     * You can start patterns with a forward slash (**/**) to avoid recursivity.

     * You can end patterns with a forward slash (**/**) to specify a directory.

     * You can negate a pattern by starting it with an exclamation point (**!**).

   Example of a "```.gitignore```" file:
   ```
   # Comment for .gitignore file
   # no .a files
   *.a

   # but do track lib.a, even though you're ignoring .a files above
   !lib.a

   # only ignore the TODO file in the current directory, not subdir/TODO
   /TODO

   # ignore all files in the build/ directory
   build/

   # ignore doc/notes.txt, but not doc/server/arch.txt
   doc/*.txt

   # ignore all .pdf files in the doc/ directory
   doc/**/*.pdf
   ```
   
   Other examples: https://github.com/github/gitignore
   
## Viewing your staged and unstaged changes

   * To see what you’ve changed but not yet staged, type ```git diff``` with no other arguments:

   ```git diff```
   ```
   diff --git a/benchmarks.rb b/benchmarks.rb
   index 3cb747f..da65585 100644
   --- a/benchmarks.rb
   +++ b/benchmarks.rb
   @@ -36,6 +36,10 @@ def main
              @commit.parents[0].parents[0].parents[0]
            end

   +        run_code(x, 'commits 1') do
   +          git.commits.size
   +        end
   +
            run_code(x, 'commits 2') do
              log = git.commits('master', 15)
              log.size
   ```

   * If you want to see what you’ve staged that will go into your next commit, you can use ```git diff --staged```. 
   This command compares your staged changes to your last commit:

   ```git diff --cached```
   ```
   diff --git a/README b/README
   new file mode 100644
   index 0000000..03902a1
   --- /dev/null
   +++ b/README2
   @@ -0,0 +1,5 @@
   +grit
   + by Tom Preston-Werner, Chris Wanstrath
   + http://github.com/mojombo/grit
   +
   +Grit is a Ruby library for extracting information from a Git repository
   ```
      
## Commit your changes (git commit)

   * To add changes to your commit area you have to run the command:
     ```git commit```

     and add an useful message to describe your commit at the first line.
      
     or
      
     ```git commit -m "useful summary message"```
      
## Removing files (git rm)

   * To remove completely a file/directory from GIT versioning and from your working directory you have to use:
      * ```git rm <FILE_PATH_1 or DIR_PATH_1> <FILE_PATH_2 or DIR_PATH_2> ...```
      
   * To keep the file in your working area but remove it from your staging area:
      * ```git rm --cached <FILE_PATH_1 or DIR_PATH_1> <FILE_PATH_2 or DIR_PATH_2> ...```
      
 You can pass files, directories, and file-glob patterns to the ```git rm``` command:
    * ```git rm log/\*.log```   (<== This command removes all files that have the .log extension in the log/ directory.)
    * ```git rm \*~```          (<== This command removes all files whose names end with a ~)

## Renaming files (git mv)

   * ```git mv <original_name> <target_name>```

   Example: ```git mv README README.md```
      
## Viewing the commit history (git log)

1. Full History:
   * ```git log```

2. Last 2 commits with their changes:
   * ```git log -p -2```

3. First line of Full History:
   * ```git log --pretty=oneline```

4. Full History with date of commit creation:
   * ```git log --pretty=fuller```
   
## Undoing things

1. One of the common undos takes place when you commit too early and possibly forget to add some files, or you mess up your commit message.

   * ```git commit --amend```    (<== run this command immediately after your previous commit. This command overwrite your last commit. Useful to add something on the last commit)
   
    Example:
      * ```git commit -m 'initial commit'```
      * ```git add .gitignore```
      * ```git commit --amend```       (and add an useful message that will replace the last commit message)

2. Unstaging a Staged File (you are into the staging area before doing the commit):
   * ```git reset HEAD <file>...```
      
    Example:
      * ```git status```
      ```
      On branch master
      Changes to be committed:
       (use "git reset HEAD <file>..." to unstage)

          new file:   README
          modified:   benchmarks.rb
      ```
              
      * ```git reset HEAD benchmarks.rb```
      ```
      Unstaged changes after reset:
      M       benchmarks.rb
      ```

   NOTE: Calling ```git reset``` without an option is not dangerous - it only touches your staging area.
   
3. Discard changes to a modified file (DANGEROUS):
   * ```git checkout -- README.md```   (Discard all changes you have done on the README.md file after the last commit. Any changes you made to that file will be lost)
   
4. Discard changes of the last commit already pushed by reverting code to a previous commit pushed on the repository:
   * ```git log --pretty=oneline```
   ```
   b5e7effda6e90b1761966fcf305113e78130e37a Add how.txt
   b96b657573d7126c13c4f212f79d9c71d203e20e First Commit
   ```
      
   * ```git revert b96b657573d7126c13c4f212f79d9c71d203e20e```
   * ```git push origin master```

5. Undo the last commit (before push and without lose changes):
   * ```git reset --soft HEAD~1```

## Showing your remote server (git remote)

   * ```git remote -v```
   
   ```
   bakkdoor  https://github.com/bakkdoor/grit (fetch)
   bakkdoor  https://github.com/bakkdoor/grit (push)
   cho45     https://github.com/cho45/grit (fetch)
   cho45     https://github.com/cho45/grit (push)
   defunkt   https://github.com/defunkt/grit (fetch)
   defunkt   https://github.com/defunkt/grit (push)
   origin    git@github.com:mojombo/grit.git (fetch)
   origin    git@github.com:mojombo/grit.git (push)
   ```
   
   This means we can pull contributions from any of these users pretty easily.

## Adding remote repositories

   * ```git remote add pb git://github.com/paulboone/ticgit.git```
   * ```git remote -v```

   ```
   origin  git://github.com/schacon/ticgit.git
   pb  git://github.com/paulboone/ticgit.git
   ```
      
   if you want to fetch all the information that Paul has but that you don’t yet have in your repository, you can run git fetch pb:
      * ```git fetch pb```
      
      Paul’s master branch is now accessible locally as pb/master – you can merge it into one of your branches, 
      or you can check out a local branch at that point if you want to inspect it.
      It’s important to note that the ```git fetch``` command only downloads the data to your local repository – 
      it doesn’t automatically merge it with any of your work or modify what you’re currently working on. 
      You have to merge it manually into your work when you’re ready.

   * ```git pull origin master```
      Running ```git pull``` generally fetches data from the server you originally cloned from and automatically tries to merge it into the code you’re currently working on.
     
## Pushing to your remotes (git push)

   When you have your project at a point that you want to share, you have to push it upstream.

   * ```git push [remote-name] [branch-name]```
   
   Example:
      * If you want to push your master branch to your origin server then you can run this to push any commits you’ve done back up to the server:
         * ```git push origin master```
         (This command works only if you cloned from a server to which you have write access and if nobody has pushed in the meantime.)
         
## Inspecting a remote

   * ```git remote show [remote-name]```
   
   Example:
    * ```git remote show origin```

    ```
    * remote origin
      Fetch URL: https://github.com/schacon/ticgit
      Push  URL: https://github.com/schacon/ticgit
      HEAD branch: master
      Remote branches:
        master                               tracked
        dev-branch                           tracked
      Local branch configured for 'git pull':
        master merges with remote master
      Local ref configured for 'git push':
        master pushes to master (up to date)
    ```
   
    The command helpfully tells you that if you’re on the master branch and you run git pull, 
    it will automatically merge in the master branch on the remote after it fetches all the remote references.
   
## Removing or Renaming remotes

1. RENAMING:
   * ```git remote rename pb paul```
   * ```git remote```
   
   ```
   origin
   paul
   ```

2. REMOVING:
   * ```git remote rm paul```
   * ```git remote```
   ```
     origin
   ```
  
## Tagging

1. Listing existing TAGs:
   * ```git tag```          (<== This command lists the tags in alphabetical order)

2. Search all tags with a particular pattern:
   * ```git tag -l "v1.8.5*"```
   
3. Create annotated TAG:
   * ```git tag -a v1.4 -m "my version 1.4"```            (<== You can also not use the -m "message" and add message directly on the editor)
   * ```git tag```
   ```
   v1.4
   ```
   * ```git show v1.4```
   ```
   tag v1.4
   Tagger: Ben Straub <ben@straub.cc>
   Date:   Sat May 3 20:19:12 2014 -0700

    my version 1.4

   commit ca82a6dff817ec66f44342007202690a93763949
   Author: Scott Chacon <schacon@gee-mail.com>
   Date:   Mon Mar 17 21:52:11 2008 -0700

       changed the version number
   ```

4. Create annotated TAG from a specific commit in the past:

   * ```git log --pretty=oneline```
   ```
   15027957951b64cf874c3557a0f3547bd83b3ff6 Merge branch 'experiment'
   a6b4c97498bd301d84096da251c98a07c7723e65 beginning write support
   0d52aaab4479697da7686c15f77a3d64d9165190 one more thing
   6d52a271eda8725415634dd79daabbc4d9b6008e Merge branch 'experiment'
   0b7434d86859cc7b8c3d5e1dddfed66ff742fcbc added a commit function
   4682c3261057305bdd616e23b64b0857d832627b added a todo file
   166ae0c4d3f420721acbb115cc33848dfcc2121a started write support
   9fceb02d0ae598e95dc970b74767f19372d61af8 updated rakefile
   964f16d36dfccde844893cac5b347e7b3d44abbc commit the todo
   8a5cbc430f1a9c3d00faaeffd07798508422908a updated readme
   ```
   
   * ```git tag -a v1.2 9fceb02```

5) Sharing Tags:
   By default, the git push command doesn’t transfer tags to remote servers. You have to do it manually with:
   * ```git push origin v1.4```
   
## Branching - Useful to solve problems without touch the master branch until the resolution

1. To create a branch and switch to it at the same time:
   * ```git checkout -b iss53```
   ```
   Switched to a new branch "iss53"
   ```

2. Write code to solve the issue and commit to the new branch where you are located now
   * ```vim README.md```
   * ```git add README.md```
   * ```git commit```            (<== And add an useful summary message)
   
   a. Pause. A new more critical, issue comes up. Open a new branch from the master branch:
      * ```git checkout master```
      * ```git checkout -b hotfix```
      * ```vim README.md```
      * ```git add README.md```
      * ```git commit```          (<== And add an useful summary message)
   
   b. Merge the hotfix with the master branch code to solve the critical problem:
      * ```git checkout master```       (<== Come back to the master branch)
      * ```git merge hotfix```          (<== Merge the hotfix code)
      * ```git branch -d hotfix```      (<== Delete the useless hotfix branch)
   
3. Now you can return to write code for resolution of the issue 53:
   * ```git checkout iss53```
   * ```vim README.md```
   * ```git add README.md```
   * ```git commit```
   
4. Mow you want to merge the iss53 code on the master branch:
   * ```git checkout master```       (<== Come back to the master branch)
   * ```git merge iss53```           (<== Merge the iss53 code)
   
   ```
   Auto-merging README.md
   CONFLICT (content): Merge conflict in README.md
   Automatic merge failed; fix conflicts and then commit the result.
   ```
   
   * ```git status```   (<== To see which files are unmerged. Anything that has merge conflicts and hasn’t been resolved is listed as unmerged.)
   
   Your file contains a section that looks something like this:
   ```
   DOCUMENTATION

   <<<<<<< HEAD   (<== HEAD point to the "master" branch where we are now, after che "git checkout master" command)
   Documentation for Hotfix
   =======        (<== after this line you will find the things that have created the conflicts.)
   Documentation for issue #53
   >>>>>>> iss53
   ```
      
   To solve the conflicts, you have to open the "README.md" file and edit it manually by removing all "<<<<<<<", "=======" and ">>>>>>>" lines after decided what keep and what discard.
   
   * ```git status```             (<== useful to know if the conflict are solved)
   
   * ```git add README.md```      (<== put the changes into the staging area)
   * ```git commit```             (<== And add an useful summary message)
   * ```git branch -d iss53```    (<== Delete the useless iss53 branch)

5) See how many branches do you have:
   * ```git branch -v```
     ```
       iss53 93b412c fixed code for issue #53
     * master 7a98805 Merge branch 'iss53'                (<== This is the branch that you are in)
     ```
     
## Working with pull requests

Glossary:
   * ```###GIT-REPOSITORY-HTTP-URL###```: Original repository
   * ```###WORKING-DIRECTORY-CREATED###```: Directory created by cloning a repository
   * ```###GIT-REPOSITORY-FORKED-HTTP-URL###```: forked repository from the original repository
   * ```###MASTER-OR-SPECIFIC-BRANCH###```: "master" or any other branch we want to test before merge it with the original repository
   
1. How to try a Pull Request before merge it on the repository
   (https://www.electricmonk.nl/log/2014/03/31/test-a-pull-merge-request-before-accepting-on-bitbucket/)
   * ```git clone ###GIT-REPOSITORY-HTTP-URL###```
   * ```cd ###WORKING-DIRECTORY-CREATED###```
   * ```git fetch ###GIT-REPOSITORY-FORKED-HTTP-URL### ###MASTER-OR-SPECIFIC-BRANCH###```
   * ```git checkout FETCH_HEAD```   (Remember that this refspec is temporary! If you do a new fetch (that includes git pull and various other commands), the FETCH_HEAD could have changed.)
   * ```git log --pretty=oneline```  (You can see all commits added by FETCH_HEAD refspec)
   * If you want to reject the changes of the Pull Request you have to check out the original branch with:
      ```git checkout master```
   * If you're satisfied that the changes made work properly and want to keep the changes from the pull request, you can merge them into master branch with:
      1. ```git checkout master```
      2. ```git merge FETCH_HEAD```
      3. ```git push origin master```   (This will automatically accept the pull request)
      
## How To clean bad data out of your Git repository history

By default the BFG doesn't modify the contents of your latest commit on your master (or 'HEAD') branch, even though it will clean all the commits before it.
Once you've committed your changes- and your latest commit is clean with none of the undesired data in it - you can run the BFG to perform it's simple deletion operations over all your historical commits.

1. Download BFG and rename it into "`bfg.jar`":
   * If you use Java 7: http://repo1.maven.org/maven2/com/madgag/bfg/1.12.16/bfg-1.12.16.jar
   * If you use Java > 7: https://rtyley.github.io/bfg-repo-cleaner/
   
2. Clone a fresh copy of your repo, using the --mirror flag:
   * `git clone --mirror git://example.com/my-repo.git`
   
   (This is a bare repo, which means your normal files won't be visible, but it is a full copy of the Git database of your repository, and at this point you should make a backup of it to ensure you don't lose anything.)

3. Remove all files named "**id_rsa**" or "**id_rsa**" from your repo: 
   * `bfg --delete-files id_{dsa,rsa}  my-repo.git`

4. Apply changes to your local repo:
   * `cd my-repo.git`
   * `git reflog expire --expire=now --all && git gc --prune=now --aggressive`

5. Apply changes to your remote repo:
   * `cd my-repo.git`
   * `git push`

Documentation on BFG 1.13.0: https://repository.sonatype.org/service/local/repositories/central-proxy/content/com/madgag/bfg/1.13.0/bfg-1.13.0.txt
