# rg
A fast and versatile code searching tool.

## Motivation

Quickly finding your way in a possibly large code base is important to work efficiently as a programmer. If you're using an IDE (or an
advanced editor), you have a lot of help from your tool, but searching within an IDE still have several limitations:

- The IDE works best when it searches within the realm of its semantic knowledge of the code base. It can quickly give you the callers of a
  method, but it doesn't work as well for example when looking for a certain word in a restricted set of property files.
- The IDE is usually bad at finding files outside its definition of what a project is.
- A IDE typically indexes its project files, to provide quick searching. This can be a source of nuisance, however, since it usually doesn't
  allow searching while indexing. If you often switch between git branches or do other big changes to your code base, this re-indexing will
  happen irritatingly often.
- Although most IDEs have a lot of search options to narrow your search, progressively narrowing your search is often a difficult task, and
  it's very difficult to go back to a previous search and repeat it.

For all of these reasons, a good command-line search tool is a very useful addition to your toolbox. `rg` has been designed to make it fast
and easy to find exactly what you are looking for in a large directory of text files. It allows you to quickly add search parameters to
limit your searches and pinpoint the matches.

Moreover, it offers an extremely quick way of getting back into your IDE or editor exactly at the place you found.

`rg` relies on `ag` (["the silver searcher"](https://github.com/ggreer/the_silver_searcher)) to quickly find matches, and [`zsh`](http://www.zsh.org/)
and [`perl`](https://www.perl.org/) to narrow down the hits. You need `zsh`, `perl` and `ag` installed to use `rg`, but you don't need to use `zsh` as your default shell. For an example how you can use the tool under `bash`, see [below](#bash).

## Usage

The main feature of `rg`, and the main reason why you'd want to use it over pure `ag` or `grep`, is the way it automatically combines your
search terms to only display lines that match all search criteria. It searches recursively in the current directory by default (this can be
changed with the -D parameter to change root directory, and the -m parameter to change the search depth).

As an example, to search for lines containing all three words 'no', 'valid' and 'miu', in any order, in Java files found anywhere containing
a path segment `'src/main'` under your current directory, use:

```zsh
$ rg no valid miu -e java -d src/main
```

For more examples on searching, see [Examples](EXAMPLES.md).

For opening the search results in your IDE, see below in the section [Completion on results](#completion-on-results).

## Installation

Installing `rg` is very easy.

First, you need to install recent enough versions of `zsh`, `perl` and `ag` (also known as "the silver searcher"). The oldest versions
that have been tested are: `zsh 5.0.8`, `perl 5.14.0` and `ag 0.32.0`.

Next, you need to clone this repo, and then add the following lines to your `'~/.zshrc'` (remember to change `'<PATH TO>'` to the path where
you cloned the repo):

```zsh
source <PATH TO>/rg/rgf

RG_EXCLUDES=(build target node node_modules bower_components \
                   '.idea' '.settings' '.git' '.svn' '.gradle' '*min.js' '*min.css' '*js.map' '*css.map')

alias rG='noglob rgf -f ${=${(j: -f :)RG_EXCLUDES}}'
alias rg='rG -i'

declare -a lastoutput
```

You now have two new commands: `rg` to search case-insensitively, and `rG` to search case-sensitively.

You can of course customize the `RG_EXCLUDES` array to exclude any files and directories that you never want to search (but see the
documentation on the `-F` parameter if you want to temporarily override the filtering).

## Completion on results

A killer feature of `rg` is that matching results can instantly be opened in whatever editor you use. Once you have made an `rg`
search, you can press `<alt-e>` to cycle between the search results. This works twice over, both for file names and line numbers. The effect
is that your editor will open exactly on your search result.

Here's how you use it. Search for something: `'rg certifikat'`. `rg` is displaying (possibly many) search results. Now, type `'ii'` (or
`'ee'` or `'vv'`), `'<space>'` and then press `'<alt-e>'` (this is called `'opt-e'` on Mac). `rg` will now cycle through all the files matching your
previous query. Hitting `'<alt-e>'` several times cycles through the results. Hitting `'<alt-E>'` will cycle backwards. Once you've found
the file you want, hit `'<space>'`, and then `'<alt-e>'` again. `rg` will start cycling through all the line numbers where matches were
found in the file you selected in the first step. Hit `'<enter>'` and you will find yourself on the right line in your favorite editor.

To use the completion functionality, put the following in your `~/.zshrc`. Note that you also need to include at least one of the editor
functions specified in the [next section](#editor-helpers).

```zsh
# Offer alternatives from the result of the previous command. Depending on what the command currently on the
# command line is, there's three alternatives:
# 1) If the command is one of the editing commands ("ee", "ii" or "vv"), and the current position is the
#    third word, then offer line numbers from the output of the latest invocation of rg/rG (stored in the
#    global array "lastoutput"), matching the file name currently in the second position. I.e., if rg found
#    a match in "file1" on lines 23 and 30, and the command line is currently "ii file1", then hitting alt-e
#    suggests 23 or 30.
# 2) If the last command was rg/rG (and we're at position 2), then alternatives are offered from the list of
#    matching files.
# 3) If the last command was NOT rg/rG, the last command is re-run, and the output from this command is
#    offered as completion.
#
# The alternatives can be further filtered by writing some file-name fragment before pressing alt-e. In this case, 
# only the alternatives matching what's given as a filter is offered. The match can be anywhere within the
# file name, and matches case-insensitively.
_previous-result() {
    setopt LOCAL_OPTIONS EXTENDED_GLOB
    local -a hlist
    local lastcommand="$(fc -l -n -1)" # Find out what the last command was.

    if [[ $words[1] =~ "^(ee|ii|vv)$" ]] && (( CURRENT >= 3 )); then
        hlist=(${${(M)lastoutput:##$words[2]*}//(#b)*:([0-9]##):*/${match[1]}}) # Matching line numbers.
    elif [[ $lastcommand =~ "^(rg|rG|rgf) .*$" ]]; then
        hlist=(${(u)${lastoutput%%:*}}) # Keep file names, make unique.
    else
        hlist=("${(@f)$(eval "$lastcommand")//$'\e'\[[0-9;]#[mK]/}") # Run last command again. Remove color.
    fi

    compadd -n -V previous_result -M 'l:|=* m:{[:lower:]}={[:upper:]}' -- $hlist
}

# Cycle forward in list of matches.
zle -C previous-comp menu-complete _previous-result
bindkey '\ee' previous-comp

# Cycle backward in list of matches.
zle -C rev-previous-comp reverse-menu-complete _previous-result
bindkey '\eE' rev-previous-comp

zstyle ':completion:*previous-comp:*:*' menu no
```

## Editor helpers

Here are some examples on how to wrap your editor in a function definition to make it easy to launch with the search results from `rg`. Put
the functions in your `'~/.zshrc'`.

To open a file in IDEA from the command line, you need to generate the `'idea'` script from within IDEA. Select 'Tools' -> 'Create Command-line
Launcher', and save the created script somewhere in your path.

If you are using Mac and IDEA, you probably want to uncomment the line within the `ii` function so that IDEA is activated as soon as you
hit '`<enter>`.'

```zsh
# Edit given file with IDEA (The 'idea' script must be created first from within IDEA).
ii() {
    local numline
    [[ -n $2 ]] && numline=:$2

    (idea "$1$numline" &)

    # If on Mac, uncomment this to immediately switch to IDEA
    # osascript -e 'activate application "IntelliJ IDEA.app"'
}

# Edit given file with emacsclient.
ee() {
    local numline
    [[ -n $2 ]] && numline=+$2

    emacsclient --no-wait $numline "$1"
}

# Edit given file with vi.
vv() {
    local numline
    [[ -n $2 ]] && numline=+$2

    vi $numline "$1"
}
```

## Bash

If you're using `bash` as your interactive shell (and are unwilling to upgrade to `zsh`), you can easily integrate `rg` for use on your
command line. You need to do two things.

First, add these lines in your `'~/.bashrc'` (remember to change `'<PATH TO>'` to the path where you cloned the repo):

```bash
RG_EXCLUDES=(-f build -f target -f node -f node_modules -f bower_components -f '.idea' -f '.settings' -f '.git' \
    -f '.svn' -f '.gradle' -f '*min.js' -f '*min.css' -f '*js.map' -f '*css.map')

rG() {
    <PATH TO>/rg/rgf ${RG_EXCLUDES[@]} "$@"
}

alias rg="rG -i"
```

Second, find the following line at the end of `<PATH TO>/rg/rgf`, and uncomment it:

```bash
# noglob rgf "$@"
```

You now have two new commands: `rg` to search case-insensitively, and `rG` to search case-sensitively.

You can of course customize the `RG_EXCLUDES` array to exclude any files and directories that you never want to search (but see the
documentation on the `-F` parameter if you want to temporarily override the filtering).

Unfortunately, the completion integration to automatically use the search results in your editor does not work under `bash`.
