---
layout:     post
title:      "java poi操作Excel"
subtitle:   "读取、生成Excel，以及修改Excel的样式"
date:       2017-4-3 23:50:00
author:     "Tang"
header-img: "img/java-poi-excel.jpg"
catalog:    true
tags:
    - java
    - Excel
---

poi 需要用到的jar包：

 - poi-3.9.jar
 - poi-ooxml-3.9.jar
 - poi-ooxml-schemas-3.9.jar
 - poi-scratchpad-3.9.jar
 - xmlbeans.jar
 - dom4j.jar
 - commons-lang3-3.5.jar

### 读取Excel

#### 通过文件路径直接读取Excel

```java
// 因为2007年后的Excel格式都是 .xlsx，因此相关的类名前都是 XSSF，类似 XSSFWorkbook
// 而2007年前的是 .xls，相关类名前缀都是 HSSF，类似 HSSFWorkbook
File file = new File("C:/Users/18072/Workspaces/数据.xlsx");

// 创建流  
InputStream input = new FileInputStream(file);

// 创建Workbook,一个Excel只有一个Workbook
Workbook wb = null;  

// 创建sheet，一个Excel有多个sheet（工作表）
Sheet sheet = null;

// 用读取的文件流来生成Workbook对象，以便我们访问里面的内容
wb = new XSSFWorkbook(input);

List<String> list = new ArrayList<String>();
// 循环sheet工作表来获取里面的内容
for (int i = 0; i < sheetNumbers; i++) {
    sheet = wb.getSheetAt(i);

    // 循环sheet里每行的内容
    for (int rowNum = 0; rowNum < sheet.getLastRowNum(); rowNum++) {
        Row row = sheet.getRow(rowNum);

        // 循环一行里面每个单元格的内容
        for (int cellNum = 0; cellNum < row.getLastCellNum(); cellNum++) {
            Cell cell = row.getCell(cellNum);

            // 将每个单元格的内容存储到ArrayList中，方便以后读取
            list.add(cell.getCell(cellNum).toString());
        }
    }
}
```

读取单元格内容的方式有多种：

 - cell.getStringCellValue() 获取单元格内的字符串，
 - cell.getNumericCellValue() 获取单元格内的数字
 - cell.toString() 将单元格内的变成字符串后并返回

> 第一次尝试着用了第一种，结果在单元格内是纯数字的情况下，报错了，无法获取，
> 第二种则是在单元格内是字符串的情况下报错了，
> 只有第三种是都适用的，我也默认使用这一种。

#### 通过 `input[type='file']` 的形式上传Excel文件

如果不知道怎么在spring MVC的Controller层获取文件的请戳 [spring图片上传](https://tangliangdong.github.io/2017/03/10/photo-upload/)

```java
// 使用的是spring MVC框架
@RequestMapping(value="post")
public void post(HttpServletRequest req,HttpServletResponse res,
    @RequestParam("data") MultipartFile exceldata) throws IOException{
    if(!exceldata.isEmpty()){
        // 创建流，MultipartFile也有获取字节数组byte[]和inputStream对象的方法
        InputStream input = exceldata.getInputStream();

        // 用读取的文件流来生成Workbook对象，以便我们访问里面的内容
        Workbook wb = new XSSFWorkbook(input);

        // 后面都一样
    }
}
```

### 生成新的Excel文件

```java
// 循环生成三个Excel文件
for(int excelIndex = 0 ; excelIndex < 3 ; excelIndex++ ){
    Workbook newWb = new XSSFWorkbook();

    Sheet sheet1 = newWb.createSheet("封面");
    Sheet sheet2 = newWb.createSheet("问题分析页");
    Sheet sheet3 = newWb.createSheet("问题清单页");

    // 循环行
    for(int i = 0 ; i < 5 ; i++ ){
        Row row = sheet1.createRow(i);

        // 循环行内单元格
        for(int j = 0 ; j < 6 ; j++ ){
            Cell cell = row.createCell(j);

            // 往单元格内 写入内容
            cell.setCellValue("Betahouse, better again");
        }
    }

    // ...

    String filePath = "C:"+java.io.File.separator;
    // 输出Excel，设置输出Excel的文件名和文件路径
    FileOutputStream out = new FileOutputStream(filePath+excelIndex+".xlsx");  
    newWb.write(out);
    out.close();
}

```

> **关于 行、单元格的创建、读取、修改有非常需要关注的几点：**

 - 在我们需要获取某Row对象时，如果之前没有`createRow(行号)`创建过改行，则需要先`sheet.createRow(行号)`，`不能先用getRow(行号)`。不然会报 `NullPointExpection` 。
 - 相反，若之前已经创建过Row对象，则我们必须用`getRow()`来获取Row对象，不然后一个`createRow(行号)`会覆盖所有之前在该行设置的属性以及内容
 - `Cell`【单元格】和`Row`【行】类似。同一个单元格只能`createCell(列号)` 一次，而且必须在所有`getCell(列号)`之前，
 - 同样获取`Cell`时，必须首先保证获取`Row`没错，不然也会出现报错或者覆盖的情况。
 - ***这个坑需要格外注意！！！***

### 修改Excel的样式（字体、背景色、合并单元格、单元格宽高）

#### 修改字体

```java
// 创建单元格的属性对象CellStyle
// 注意需要用将要设置的单元格所在workbook来创建CellStyle对象
CellStyle cellH1 = newWb.createCellStyle();

// 创建Font对象
Font fontH1 = newWb.createFont();

// 设置字体的大小，对应的就是Excel中的字体数值
fontH1.setFontHeightInPoints((short)22);
// fontH1.setFontHeight((short)(256*10));
// 使用这个不太方便，数值在乘上256之后才是代表几个字符的大小。

// 设置字体类型
fontH1.setFontName("微软雅黑");

// 设置粗体
fontH1.setBoldweight(XSSFFont.BOLDWEIGHT_BOLD);

// 将字体的样式添加到单元格样式上，
cellH1.setFont(fontH1);

// 将单元格的样式应用到指定单元格。
sheet1.getRow(0).getCell(0).setCellStyle(cellH1);
```

#### 设置背景色

> 通过`IndexedColors`来获取颜色，再通过`.getIndex()`来获取short类型的数据

```java
// 创建单元格的属性对象CellStyle
CellStyle cBackgound = newWb.createCellStyle();

//　设置背景色
cBackgound.setFillForegroundColor(IndexedColors.SKY_BLUE.getIndex());

// 设置填充方式
cBackgound.setFillPattern(XSSFCellStyle.SOLID_FOREGROUND);

// 设置边框
cBackgound.setBorderLeft((short)XSSFCellStyle.BORDER_THIN);
cBackgound.setBorderRight((short)XSSFCellStyle.BORDER_THIN);

// 设置边框颜色
cBackgound.setBottomBorderColor(IndexedColors.BLACK.getIndex());

// 将单元格的样式应用到指定单元格。
sheet1.getRow(0).getCell(0).setCellStyle(cBackgound);
```

#### 设置合并单元格

```java
// 设置 合并单元格 
// 合并第一行的第一个到第五个单元格
CellRangeAddress region = new CellRangeAddress(
    0, // first row
    0, // last row
    0, // first column
    4  // last column
);
// 设置sheet1 合并单元格
sheet1.addMergedRegion(region);
```

#### 设置单元格宽、行高

```java
// 设置默认的 单元格宽 和 行高
sheet1.setDefaultColumnWidth(20);
sheet1.setDefaultRowHeight((short)20);

Row r = sheet1.createRow(1);
Cell c = r.createCell(1);
c.setCellValue("Hello Betahouse!");

// 设置成6个字符的高度
r.setHeight((short)(256*6));

// 设置成10个字符的宽度
sheet3.setColumnWidth(0, (short)(256*10)));
```

#### 设置居中

```java
XSSFCellStyle cCenter=(XSSFCellStyle) newWb.createCellStyle();

// 设置自动换行
cCenter.setWrapText(true);

// 设置竖向居中
cCenter.setVerticalAlignment(CellStyle.VERTICAL_CENTER);

// 设置水平居中
cCenter.setAlignment(CellStyle.ALIGN_CENTER);

// 将单元格的样式应用到指定单元格。
sheet1.getRow(0).getCell(0).setCellStyle(cCenter);
```

> 查阅java文档可以有很全面的认识

> 下次分享如何将Excel的图片插入到另一个Excel中。






