---
{"dg-publish":true,"permalink":"/OpenSource/根据开源社区学bash- rustup.sh/","noteIcon":"3"}
---


---
dg-publish: true
---
`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs -o rustup.sh`
<font color="#d99694">${TERM+SET}可以判断TERM这个变量是否有被赋值，如果有被赋值，则结果为set，否者为空</font>

```bash
local _ansi_escapes_are_valid=false
if [ -t 2 ]; then
 if [ "${TERM+set}" = 'set' ]; then
	 case "$TERM" in
		 xterm*|rxvt*|urxvt*|linux*|vt*)
			 _ansi_escapes_are_valid=true
		 ;;
	 esac
 fi
fi
```

对于一个命令是否存在一般使用command -v来判断
```bash
say() {
    printf 'rustup: %s\n' "$1"
} 

err() {
    say "$1" >&2 
    exit 1
}

need_cmd() {
    if ! check_cmd "$1"; then
        err "need '$1' (command not found)"
    fi  

}

check_cmd() {
    command -v "$1" > /dev/null 2>&1
} 

assert_nz() {
    if [ -z "$1" ]; then err "assert_nz $2"; fi
}

# Run a command that should never fail. If the command fails execution
# will immediately terminate with an error showing the failing
# command.
ensure() {
    if ! "$@"; then err "command failed: $*"; fi
}

# This is just for indicating that commands' results are being
# intentionally ignored. Usually, because it's being executed
# as part of error handling.
ignore() {
    "$@"
}
```