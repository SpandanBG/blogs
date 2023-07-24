# Customize Bash Shell 

---

---

The bash shell in linux has five environment variables: `PS1`, `PS2`, `PS3`, `PS4` and `PROMPT_COMMAND`. The use of each is listed below:

- **PS1** - The shell string, in noobs term the string before the '`$`' sign. It can show valuable informations, such as date, usename, machine name, path, etc.
- **PS2** - The sub shell string. In a long continuation of a shell command, the default multiline string is '`>`'. PS2 allows to change this string.
- **PS3** - The promt string displayed by the '`select`' command in bash. By default it is '`#?`'. PS3 allows to change this string
- **PS4** - The string at the beginning of each shell command in debug mode. When a shell script is executed with '`set -x`' in the beginning, it runs in a debug mode, which results in a '`++`' sign before each command in the output. PS4 allows you to change this string.
- **PROMPT_COMMAND** - This allows you to execute a command right before the PS1 string.

## Customize the PSx

To change the default interaction prompt type in the following command
```bash
# You can replace PS1 with any of PSx environment variables
export PS1=""
```

And within the double quotes place any of the following keys **(NOTE: may not work for PS3)**:

- **\u** - For username
- **\h** - For hostname
- **\w** - For full path of the current working directory
- **\W** - For the current working directory
- **\t** - For time in hh:mm:ss format
- **\d**- For date in 'Weekday Month Date' format
- **\\#** - For number of commands

This is only a short list. **The complete list of keys can be found (here)[https://ss64.com/bash/syntax-prompt.html]**.

### Execute Scripts in PSx

Apart from simply changing the string it is also possible to execute linux commands or shell scripts to append the outputs in to the string. To do this simply place the command within the `` or within the parantesis of `$()`. For example

```bash
export PS1="$(date) $(uname -r)"
```

### Changing string colour

To change the colour of the string you'll need to appended the colour key codes. The key code starts with `\e[`, the colour key is given by `x;ym` and ends a with `\e[m`. The following are the color keys available:

- **\e[0;30m** = Dark Gray
- **\e[1;30m** = Bold Dark Gray
- **\e[0;31m** = Red
- **\e[1;31m** = Bold Red
- **\e[0;32m** = Green
- **\e[1;32m** = Bold Green
- **\e[0;33m** = Yellow
- **\e[1;33m** = Bold Yellow
- **\e[0;34m** = Blue
- **\e[1;34m** = Bold Blue
- **\e[0;35m** = Purple
- **\e[1;35m** = Bold Purple
- **\e[0;36m** = Turquoise
- **\e[1;36m** = Bold Turquoise
- **\e[0;37m** = Light Gray
- **\e[1;37m** = Bold Light Gray

For example:

```bash
# Bold Red Foreground
export PS1="\e[1;31m\u::[\W] $ \e[m"
```

### Changing background colour
The process of setting a background colour is same as changing the forground colour, but the colour keys are different; they are given by `ym`. Here are the list of keys:

- \e[40m = Dark Gray
- \e[41m = Red
- \e[42m = Green
- \e[43m = Yellow
- \e[44m = Blue
- \e[45m = Purple
- \e[46m = Turquoise
- \e[47m = Light Gray

For example
```bash
# Light Grey Background
export PS1="\e[47m\u::[\W] $ \e[m"
```

## Customize PROMPT_COMMAND

The PROMPT_COMMAND variable is always executed just before displaying the PS1 variable. For example:

```bash
# Display time in hh:mm:ss format
export PROMPT_COMMAND="date +%k:%m:%S"
```

If newline is to be avoided, append `echo -n` at the start of the string, followed by the command inside `$()`. For example:

```bash
# Display time in hh:mm:ss format in the same line
export PROMPT_COMMAND="echo -n [$(date +%k:%m:%S)]"
# result: [14:01:10]superdan::[bash] $
```

## Make Settings Parmanent

To make the settings parmanent it'll require you to modify the profile config file in your home directory. The file might be with any one of the following names: `.bashrc`, `.bash_profile`, `.profile`. After opening the file simply append the `export PSx="..."` or `export PROPMT_COMMAND="..."` commands at the end of the file.

---

Spandan Buragohain,
2018-01-04 09:13:03