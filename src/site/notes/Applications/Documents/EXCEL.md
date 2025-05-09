---
{"dg-publish":true,"permalink":"/Applications/Documents/EXCEL/","noteIcon":"3"}
---

#excel
### 1. 查看某一列中是否有重复项(为了使公式生效，需要确认单元格格式为general)
```excel
# check which value in col S3 is repeated with col A in sheet
=IF(COUNTIF(Sheet2!A:A,S3)>1,"repeated","not repeated")

=IF(OR(COUNTIF(Sheet1!A:A,S3)>0,COUNTIF(Sheet2!A:A,S3)>0),"repeated,"not repeated")
```
### 2. 查看range内是否存在指定值，并返回对应行内的某一列信息
[vlookup函数](https://zh-cn.extendoffice.com/excel/functions/excel-vlookup-function.html)
[named range](https://exceljet.net/glossary/named-range)
[xlookup search range](https://exceljet.net/formulas/xlookup-match-any-column)
```excel
# VLOOKUP可以搜索多列，但是有一个限制就是需要返回的列必须在lookup_value的右边，因为vlookup默认是从左往右搜索的
=VLOOKUP (lookup_value, table_array, col_index, [range_lookup])
#如果返回的列在需要查找列的左边，可以通过将返回列复制到右边或者使用以下公式
=VLOOKUP(V90,IF({1,0},D:D,A:A),2,0)
# xlookup的lookup array只能是一列，返回array的行数应与lookup array一致
XLOOKUP(V90,'0529全量盘点机器'!K:K,'0529全量盘点机器'!:C:C)
# 如果想要查看制定range(多列)，而非某一列，可以结合MMULT函数通过矩阵计算来达到
XLOOKUP(1,MMULT(--(RANGE=S22),SEQUENCE(COLUMNS(RANGE),1,1,0)),RETURNARRAY)
```


### 3.多个搜索条件
```bash
=concate(xlookup())
```
### 4.快速设置一列格式一致
首先右键设置一列为文本格式，使用智能快速分列功能，选中文本，点击finish

### 5.计算筛选列数字之和

```
=SUBTOTAL(9, A1:A100)

```
SUBTOTAL函数会忽略被筛选掉的行，只计算当前显示的行的和。
`9`代表“求和”功能。

### 6.计算文本长度
```
=LEN(A1)
```