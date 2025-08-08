# `run`

Zero dependencies task runner script.

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

Then `chmod +x run` and you're ready to go! Just run `./run` to see available tasks, `./run install` to run the `install` task etc.

```bash
$ ./run

install              Install dependencies
dev                  Start development environment
help                 List available tasks

$ ./run install

yarn install v1.22.22
[1/4] 🔍  Resolving packages...
...
```

## Improvements

### Shell Alias

Add the following to `.zshrc`, `.bashrc`:

```bash
alias run="./run"
```

Now you can run `run install`, `run dev`, etc. without needing to specify the script path.

### Tab completions

#### Oh My Zsh

```bash
mkdir ~/.oh-my-zsh/completions/
```

Then create `~/.oh-my-zsh/completions/_run` with the following content:

```zsh
#compdef run
# https://github.com/wowu/run v0.1
_run() {
  case $CURRENT in
    2)
      local -a tasks
      local script="$words[1]"

      if [[ -f "$script" ]]; then
        tasks=(${(f)"$(
          awk '
            BEGIN { FS="(\\(\\))? { ## ?" }

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

Now you can type in `run <TAB>` to auto-complete available tasks.
