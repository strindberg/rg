# Examples

Search for string 'port' in all files (case insensitive):

```zsh
$ rg port
```

Search for string 'port' in all files (case sensitive):
```zsh
$ rG port
```

Search for string 'no valid' in all files:
```zsh
$ rg no\ valid
```

Search for string 'port' AND string 'http':

```zsh
$ rg port http
```

Search for string 'port' OR string 'http':

```zsh
$ rg 'port|http'
```

Search for string 'port' AND (string 'ftp' OR string 'http'):

```zsh
$ rg port 'ftp|http'
```

Search for string 'port' in all files but display only files, not file contents. Pipe result to "ls -l" to display file stats:

```zsh
$ rg port -l | xargs ls -l
```

Search for string 'port' in all files but display only matching lines, not file names:

```zsh
$ rg port -h
```

Search for string 'port' and exclude lines containing 'import' OR 'support':

```zsh
$ rg port -v import -v support
```

Same as above. Search for string 'port' and exclude lines containing 'import' OR 'support':

```zsh
$ rg port -v 'import|support'
```

Search for the exact word 'port' (with word boundaries before and after):

```zsh
$ rg '\bport\b'
```

Search for lines beginning with 'port':

```zsh
$ rg ^port
```

Search for lines beginning with 'port', possibly after initial whitespace:

```zsh
$ rg '^\s*port'
```

Search for string 'port' in specified root directory:

```zsh
$ rg port -D /opt/intyg
```

Search for lines with string 'port' AND 'http'. Display matches with two context lines before and after match:

```zsh
$ rg port http -C
```

Search for lines with string 'port' AND 'http'. Display matches with three context lines before and after match:

```zsh
$ rg port http -C3
```

Search for lines with string 'port' AND 'http'. Display matches with two context lines after match:

```zsh
$ rg port http -A
```

Search for lines with string 'port' AND 'http'. Display matches with two context lines before match:

```zsh
$ rg port http -B
```

Search for lines with string 'port' in java and javascript files:

```zsh
$ rg port -e java -e js
```

Search for lines with string 'port' in all files EXCEPT html files:

```zsh
$ rg port -E html
```

Search for lines with string 'port' in all files except files with names beginning with 'intyg':

```zsh
$ rg port -f intyg*
```

Search for lines with string 'port' in all files, including in 'build' directories (i.e. override limit set in alias):

```zsh
$ rg port -F build
```

Search for lines with string 'port' in all files with names containing 'pom.xml':

```zsh
$ rg port -p pom.xml
```

Search for lines with string 'port' in all files with names exactly pom.xml (anywhere in file tree):

```zsh
$ rg port -p /pom.xml$
```

Search for lines with string 'port' in all files where a segment of the file's path is 'src/main':

```zsh
$ rg port -d src/main
```

Search for lines with string '-i'. Use '--' to stop parsing of options and search for option-like strings:

```zsh
$ rg -- -i
```

Search for lines with string '$html'. Note that literal '$' (as well as '^', '(' and ')') must be both quoted and protected with backslash:

```zsh
$ rg '\$html'
```

For installation and general motivation, see [README](README.md).
