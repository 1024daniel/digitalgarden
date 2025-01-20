---
{"dg-publish":true,"permalink":"/CodeSnippets/找到符号在那个so中定义/","noteIcon":"3"}
---

深度搜索，定义的，而非undefined

```bash info:12,16
# Check if the correct number of arguments are provided
if [ "$#" -ne 2 ]; then
    echo "Usage: $0 <executable> <symbol>"
    exit 1
fi

# Get the arguments
executable="$1"
symbol="$2"

# Find the shared libraries that the executable depends on
libraries=$(ldd "$executable" | awk '{print $3}')

# Search for the symbol in each shared library
for library in $libraries; do
    if nm -D "$library" 2>/dev/null | grep -q "$symbol"; then
        echo "Symbol '$symbol' found in shared library: $library"
        exit 0
    fi
done

echo "Symbol '$symbol' not found in any shared library"

```
