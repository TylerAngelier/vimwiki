# Bash and Shell Scripting

This article is everything about writing bash and shell scripts.

# Shell script things

## Parse long and short args

https://stackoverflow.com/a/30026641/8156025

```bash
# Transform long options to short ones  
for arg in "$@"; do  
  shift  
  case "$arg" in  
  "--help") set -- "$@" "-h" ;;  
  "--verbose") set -- "$@" "-v" ;;  
  "--region") set -- "$@" "-r" ;;  
  "--zone") set -- "$@" "-z" ;;  
  "--environmentClass") set -- "$@" "-e" ;;  
  "--instance") set -- "$@" "-i" ;;  
  "--subscriber") set -- "$@" "-s" ;;  
  "--system") set -- "$@" "-y" ;;  
  "--alias") set -- "$@" "-a" ;;  
  "--port") set -- "$@" "-p" ;;  
  *) set -- "$@" "$arg" ;;  
  esacdone  
  
local OPTIND=1  
# Resetting OPTIND is necessary if getopts was used previously in the script.  
# It is a good idea to make OPTIND local if you process options in a function.  
  
while getopts h?vf:z:e:i:s:y:a:r:p: opt; do  
  case "$opt" in  
  h | \?)  
    show_help  
    exit 0  
    ;;  
  f)  
    output_format=${OPTARG}  
    ;;  
  v)  
    verbose=$((verbose + 1))  
    ;;  
  p)  
     port=${OPTARG}  
     ;;  
  a)  
    alias=${OPTARG}  
    ;;  
  r)  
    region=${OPTARG}  
    ;;  
  z)  
    zone=${OPTARG}  
    ;;  
  e)  
    environmentClass=${OPTARG}  
    ;;  
  i)  
    instance=${OPTARG}  
    ;;  
  s)  
    subscriber=${OPTARG}  
    ;;  
  y)  
    system=${OPTARG}  
    ;;  
  esacdone  
shift "$((OPTIND - 1))" # Discard the options and sentinel --  
[ "${1:-}" = "--" ] && shift
```

## Get exit code of last command

```bash
echo $?
```

## Colors and emoji commands

```bash
# Color and style stuff  
emoji_check='\U2705'  
emoji_cross='\U274C'  
emoji_eyes='\U1F440'  
  
success(){  
  echo -e "$(tput setaf 2)$1$(tput sgr0)"  
}  
  
failure(){  
  echo -e "$(tput setaf 1)$1$(tput sgr0)"  
}  
  
success-check() {  
  success "$emoji_check $1"  
}  
  
failure-cross() {  
  failure "$emoji_cross $1"  
}  
  
cyan() {  
  echo -e "$(tput setaf 6)$1$(tput sgr0)"  
}  
  
cyan-eyes() {  
  cyan "$emoji_eyes $1"  
}
```

#sh

# FZF

## git checkout branches

https://polothy.github.io/post/2019-08-19-fzf-git-checkout/

```bash
# Interactive git checkout using fzf
# alias gch="git checkout $(git branch | fzf)"
fzf-git-branch() {
    git rev-parse HEAD > /dev/null 2>&1 || return

    git branch --color=always --all --sort=-committerdate |
        grep -v HEAD |
        fzf --height 50% --ansi --no-multi --preview-window right:65% \
            --preview 'git log -n 50 --color=always --date=short --pretty="format:%C(auto)%cd %h%d %s" $(sed "s/.* //" <<< {})' |
        sed "s/.* //"
}
fzf-git-checkout() {
    git rev-parse HEAD > /dev/null 2>&1 || return

    local branch

    branch=$(fzf-git-branch)
    if [[ "$branch" = "" ]]; then
        echo "No branch selected."
        return
    fi

    # If branch name starts with 'remotes/' then it is a remote branch. By
    # using --track and a remote branch name, it is the same as:
    # git checkout -b branchName --track origin/branchName
    if [[ "$branch" = 'remotes/'* ]]; then
        git checkout --track $branch
    else
        git checkout $branch;
    fi
}
alias gb='fzf-git-branch'
alias gco='fzf-git-checkout'
```

#git #fzf

## Cool prompt with dev tooling

```bash
# Prompt the user for an alias using fzf and a pretty preview window. The selected alias name will  
# be returned to stdout.  
selectAlias() {  
  get_keys "cp.aliases" | fzf --height 30% --ansi --no-multi --preview-window right:55% \  
    --preview 'cat '"${user_config}"' | jq -r ".cp.aliases.$(sed "s/.* //" <<< {}) "'  
}
```

# jq

JQ manual: https://stedolan.github.io/jq/manual/

## Create a JSON string

```bash
jq -n \  
  --arg environmentClass "$environmentClass" \  
  --arg instance "$instance" \  
  --arg subscriber "$subscriber" \  
  --arg system "$system" \  
  --arg processChildren "$processChildrenFlag" \  
  '{  
     environmentClass: $environmentClass,
     instance: $instance,
     subscriber: $subscriber,
     resourceNameType: "SYSTEM",
     resourceNames: [ $system ],
     processChildren: true
   }'
```

#jq

## Filter array of objects 

```bash
echo '[{"name": "foo"},{"name": "bar"}]' | jq '.[] | select(.name=="foo")'
# outputs {"name":"foo"}
```

## Get raw data

The `-r` flag will trim off quotes.


# ZSH


> Here is a non-exhaustive list, in execution-order, of what each file tends to contain:
> 1.  `.zshenv` is _always_ sourced. It often contains exported variables that should be available to other programs. For example, `$PATH`, `$EDITOR`, and `$PAGER` are often set in `.zshenv`. Also, you can set `$ZDOTDIR` in `.zshenv` to specify an alternative location for the rest of your zsh configuration.
>2.  `.zprofile` is for login shells. It is basically the same as `.zlogin` except that it's sourced before `.zshrc` whereas `.zlogin` is sourced after `.zshrc`. According to the zsh documentation, _"`.zprofile` is meant as an alternative to `.zlogin` for ksh fans; the two are not intended to be used together, although this could certainly be done if desired."_
>3.  `.zshrc` is for interactive shells. You set options for the interactive shell there with the `setopt` and `unsetopt` commands. You can also load shell modules, set your history options, change your prompt, set up zle and completion, et cetera. You also set any variables that are only used in the interactive shell (e.g. `$LS_COLORS`).
>4.  `.zlogin` is for login shells. It is sourced on the start of a login shell but after `.zshrc`, if the shell is also interactive. This file is often used to start X using `startx`. Some systems start X on boot, so this file is not always very useful.
>5.  `.zlogout` is sometimes used to clear and reset the terminal. It is called when exiting, not when opening.
>
You should go through [the configuration files of random Github users](https://github.com/search?q=zsh+dotfiles&ref=commandbar) to get a better idea of what each file should contain.

Stolen from: https://unix.stackexchange.com/a/71258
