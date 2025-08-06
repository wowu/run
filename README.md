# `run`

Zero dependencies task runner.

### Sample `run` script

```bash
#!/bin/bash -e

function task:install { ## Install dependencies
  yarn install
}

function task:dev { ## Start development environment
  yarn dev
}

function task:help { ## List available tasks
  awk 'BEGIN {FS=" { [#][#] ?"} /^function task:.*\{ [#][#] ?/ {sub("function task:", "", $1); printf "\033[36m%-20s\033[0m %s\n", $1, $2}' "$0"
}

task:"${@:-help}"
```

Then `chmod +x run` and you're ready to go!

### Alias

`.zshrc`, `.bashrc`:

```bash
alias run="./run"
```

## Tab completions

### Oh My Zsh

```bash
mkdir ~/.oh-my-zsh/completions/
```

Then create `~/.oh-my-zsh/completions/_run` with the following content:

```zsh
#compdef run
_run() {
  case $CURRENT in
    2)
      local -a tasks
      local script="$words[1]"

      if [[ -f "$script" ]]; then
        tasks=(${(f)"$(
          awk '
            BEGIN { FS=" { ## ?" }
            /^function task:.*\{ ## ?/ {
              sub("function task:", "");
              gsub(":", "\\:");
              printf "%s:%s\n", $1, $2;
            }
          ' "$script"
        )"})
      fi

      _describe 'task' tasks
      ;;
    *)
      _files
      ;;
  esac
}
```
