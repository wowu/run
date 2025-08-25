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

function task:deploy { ## Deploy the app ## <env>
  yarn deploy -- "$@"
}

function task:help { ## List available tasks
  awk -v pad=25 'BEGIN {FS="(\\(\\))? ({ )?[#][#] ?"} /^function task:.*\{ [#][#] ?/ {sub("function task:", "", $1); printf "\033[36m%-"pad"s\033[0m %s\n", $1 " \033[32m" $3, $2}' "$0"
}

task:"${@:-help}"
```

Save the script above as `run`. Then execute `chmod +x run` and you're ready to go! Just run `./run` to see available tasks, `./run install` to run the `install` task etc.

```bash
$ ./run

install              Install dependencies
dev                  Start development environment
deploy <env>         Deploy the app
help                 List available tasks

$ ./run install

yarn install v1.22.22
[1/4] 🔍  Resolving packages...
...
```

## Optional Improvements

### Shell Alias

Add the following to `.zshrc` or `.bashrc` depending on your shell:

```bash
alias run="./run"
```

Now you can run `run install`, `run dev`, etc., instead of `./run install`.

### Tab completions

#### Oh My Zsh

Create the completions directory if it doesn't exist:

```bash
mkdir -p ~/.oh-my-zsh/completions/
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
            BEGIN { FS=" ?(\\(\\))? { ## ?" }

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

## Credits

Inspired by [Enrise/Taskfile](https://github.com/Enrise/Taskfile), [adriancooney/Taskfile](https://github.com/adriancooney/Taskfile), and [Self-documenting Makefile](https://marmelab.com/blog/2016/02/29/auto-documented-makefile.html).
