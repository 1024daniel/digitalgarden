---
{"dg-publish":true,"permalink":"/websites/dataview/","noteIcon":"3"}
---

#dataview

https://zhuanlan.zhihu.com/p/393550306
https://obsidian.rocks/dataview-in-obsidian-a-beginners-guide/
https://blacksmithgu.github.io/obsidian-dataview/queries/structure/

```js title:"get the note number in a line"
438

```

```sql title:"select from specific tag and display folder, tags, date"
table file.folder as folder, file.etags as tags, file.ctime as date
from #storage
sort date desc

```

```sql title:"list modify days and files"
list file.day
from ""
sort file.mtime desc
```


```sql title:"one tag a row, sort by tag"
TABLE WITHOUT ID (tag + "(" + length(rows.file.link) + ")") AS Tags, link(sort(rows.file.name)) AS Files
FROM ""
WHERE file.tags 
FLATTEN file.tags AS tag 
GROUP BY tag
SORT length(rows.file.link) DESC
```