---
{"dg-publish":true,"permalink":"/OperatingSystem/Linux/sed/","noteIcon":"3"}
---

#sed

```bash
# 文件中第6行插入一行
sed -i '6i this is the new line' test.txt
sed -i 's/原始内容/替换内容/\$' 文件名
#替换文件中第6行
sed -i '6s/.*/替换内容/'
sed -n '\$p' 文件名
# 删除文件中的第4行到文件尾
sed -i '4,\$d' 文件名
# 删除空行
sed  '/^&/d' 文件名
```

`sed` (stream editor) is a powerful command-line utility used for parsing and transforming text in a stream or file. Here is a summary of common `sed` usage and commands:

### Basic Usage
```sh
sed [options] 'script' [file...]
```
- `script`: The set of instructions to be applied to the input text.
- `file...`: One or more files to be processed. If no file is specified, `sed` reads from standard input.

### Common Options
- `-n`: Suppresses automatic printing of pattern space. Use with the `p` command to print specific lines.
- `-e script`: Adds the `script` to the commands to be executed.
- `-f script-file`: Adds the contents of `script-file` to the commands to be executed.
- `-i[SUFFIX]`: Edits files in place (makes backup if extension is supplied).

### Addressing
`sed` commands can be restricted to specific lines or ranges of lines:
- `number`: A specific line number.
- `$`: The last line.
- `/regex/`: Lines matching the regular expression `regex`.
- `addr1,addr2`: A range of lines from `addr1` to `addr2`.

### Commands
- `p`: Print the current pattern space.
- `d`: Delete the pattern space (do not print it).
- `s/regexp/replacement/flags`: Substitute `regexp` with `replacement`. Common flags include:
  - `g`: Global replacement (all occurrences).
  - `i`: Case-insensitive replacement.
  - `p`: Print the result of the substitution.
  - `w file`: Write the result to `file`.
- `a\ text`: Append `text` after each line matched.
- `i\ text`: Insert `text` before each line matched.
- `c\ text`: Replace lines with `text`.
- `y/src/dest/`: Transliterate characters (like `tr` command).

### Examples

1. **Print specific lines**:
   ```sh
   sed -n '5p' file.txt  # Print the 5th line
   sed -n '5,10p' file.txt  # Print lines 5 to 10
   ```

2. **Substitute text**:
   ```sh
   sed 's/foo/bar/' file.txt  # Replace first occurrence of 'foo' with 'bar' in each line
   sed 's/foo/bar/g' file.txt  # Replace all occurrences of 'foo' with 'bar' in each line
   ```

3. **Delete lines**:
   ```sh
   sed '3d' file.txt  # Delete the 3rd line
   sed '/^$/d' file.txt  # Delete all empty lines
   ```

4. **Insert and append text**:
   ```sh
   sed '3i\Inserted text' file.txt  # Insert text before the 3rd line
   sed '3a\Appended text' file.txt  # Append text after the 3rd line
   ```

5. **Extract a block of text**:
   ```sh
   sed -n '/start/,/end/p' file.txt  # Print lines from 'start' to 'end'
   ```

6. **In-place editing**:
   ```sh
   sed -i 's/foo/bar/g' file.txt  # Replace 'foo' with 'bar' directly in the file
   sed -i.bak 's/foo/bar/g' file.txt  # Same as above but create a backup with .bak extension
   ```

7. **Using multiple commands**:
   ```sh
   sed -e 's/foo/bar/' -e '/baz/d' file.txt  # Substitute and delete in one command
   ```

### Special Characters
- `&`: Represents the matched text in the replacement part of the `s` command.
  ```sh
  sed 's/foo/&bar/' file.txt  # Replaces 'foo' with 'foobar'
  ```
- `\1`, `\2`, ... `\9`: Refer to matched groups in the pattern.
  ```sh
  sed 's/\(foo\) \(bar\)/\2 \1/' file.txt  # Swaps 'foo bar' to 'bar foo'
  ```

### Escaping Characters
Characters like `/`, `&`, and `\` have special meanings in `sed` and must be escaped with a backslash (`\`) if they are to be used literally.

By mastering these `sed` commands and techniques, you can efficiently manipulate and transform text in a variety of powerful ways directly from the command line.