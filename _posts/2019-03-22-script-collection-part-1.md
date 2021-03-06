---
layout: post
title:  "Script collection - Part 1"
author: Jeremy
---

# Clone script
Clone git repositories and automatically set custom configs, such as commit author or gpg settings.

`.zshrc`
```bash
clone() {
    username=$1
    repository=$2

    if [ -d "~github/$username/$repository" ]; then
        echo "Repository already exists"
        return 1;
    fi

    git clone git@github.com:$username/$repository.git ~github/$username/$repository

    if [ "$username" = "myelin-ai" ]; then
        cat ~/.myelin_gitconfig >> ~github/$username/$repository/.git/config
    fi
}
```

`.myelin_gitconfig`
```properties
[user]
    name = Jeremy Stucki
    email = jeremy@myelin.ch
    signingkey = 6FBAAA51EA742855
[commit]
    gpgsign = true
```


# LaTeX spell check
Spellcheck your documents using a  default language and a custom dictionary.

To spellcheck a whole directory you can use `find` and `xargs` like this:  
`find . -type f -name '*.tex' | xargs -L1 spellcheck.sh`

The script has some issues. `dictionary.txt` must not contain empty lines.

`spellcheck.sh`
```bash
SPELLING_MISTAKES="$(cat "$1" | aspell list -t -d en_US | grep -i -v -f dictionary.txt)";

if [[ "${SPELLING_MISTAKES}" != "" ]]
then
    echo "Spelling mistakes found in $1"
    echo "${SPELLING_MISTAKES}" | sed "s/^/    /g"
    exit 1
fi
```


# Extract a directory from a git repository
1. `git filter-branch --prune-empty --subdirectory-filter <directory> master`
2. (Optional) Set yourself as the commiter and re-commit everything using the script below. (If you require signed commits). Then run:  
`git filter-branch -f --commit-filter 'git commit-tree -S "$@";' -- --all`
3. `git remote set-url origin <new repository>`
4. `git push`

```bash
git filter-branch -f --env-filter '
NEW_COMMITER_NAME="Jeremy Stucki"
NEW_COMMITER_EMAIL="jeremy@myelin.ch"

if [[ "$GIT_AUTHOR_NAME" != "$NEW_COMMITER_NAME" ]]
then
    export GIT_AUTHOR_NAME="$GIT_COMMITTER_NAME"
    export GIT_AUTHOR_EMAIL="$GIT_COMMITTER_EMAIL"
    export GIT_COMMITTER_NAME="$NEW_COMMITER_NAME"
    export GIT_COMMITTER_EMAIL="$NEW_COMMITER_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```


# Other random stuff

## macOS - Cleanup brew cache
If you just installed or updated a lot of stuff, you should consider running `brew cleanup`.  
It also runs automatically every 30 days.


## Rename a lot of files quickly
With `zmv` you can quickly rename all the files in a directory.

Example script: `zmv '(*)S(??)E(??)(*).(???)' 'Season $2 - Episode $3.$4'`

Input | Output
---   | ---
Silicon.Valley.S04E05.HDTV.x264-SVA[ettv].mkv | Season 04 - Episode 05.mkv
