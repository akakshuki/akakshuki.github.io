---
title: "git_tricks"
date: 2022-11-07T00:14:08+07:00
type: "post"
tags: ["blog"]
---


# Git tricks
##  1. Git amend

- When you want to commit code without changing commit message.

  ```Bash
  
      git add . && git commit --amend --no-edit -a && git push -f 
  
  ```

## 2. Git rebase 
    
when you want to merge your "stupid" commits to 1 commits you can try this way:
   
 - Step 1: check commits
      ```Bash
       git logs --oneline
      ```
      It will be return:
      ```
       7446b5d add image
       d3f4011 add image
       6e55b32 update code
       e2b30b3 Create hugo.yml
       aa924f8 add git tips
       a2eb2dc init code
      ```
 - Step 2:
      ```Bash
        git rebase -i HEAD~N
      ```
      with N is number commit you want to merge
      
      example:
      ```Bash
        git rebase -i HEAD~3
      ```
 - Step 3:
     -  change only the first row as "pick" and s(slash) to others
   ![Text](https://www.git-tower.com/learn/media/pages/git/faq/git-squash/b34873f2ec-1667823657/cli-interactive-rebase@2x.gif "Title")
     - Save
     - Edit the commit messages
- Step 4: 
    ```Bash
     git push -f
    ```