---
title: Excel基本写操作
date: 2022-03-20 21:40:36
tags: poi
categories: excel
---

# Excel基本写操作

- 引入依赖

```java
implementation 'org.apache.poi:poi:5.2.0'
```

#### 实战

```java
public class ExcelWrite {
  // 项目目录
  private static String PATH = "/Users/panyurou/IdeaProjects/exceldemo";

  @Test
  public void excelWrite() throws IOException {
    //1.创建一个工作簿
    Workbook workbook = new HSSFWorkbook();
    //2.创建一个工作表
    Sheet sheet = workbook.createSheet("测试excel写入");
    //3.创建一行,第0行
    Row row1 = sheet.createRow(0);
    //4.创建一个单元格
    Cell cell11 = row1.createCell(0);
    //5.给单元格写入值
    cell11.setCellValue("我是单元格（1，1）");

    //6.创建新的一行,第1行
    Row row2 = sheet.createRow(1);
    //7.创建一个单元格
    Cell cell21 = row2.createCell(0);
    //8.给单元格写入值
    String time = new DateTime().toString("yyyy-MM-dd HH:mm:ss");
    cell21.setCellValue(time);

    //9.将创建好的工作簿写出
    FileOutputStream outputStream = new FileOutputStream(PATH + "/excel写入.xlsx");
    workbook.write(outputStream);
    outputStream.close();
    System.out.println("excel 写入完毕！");
  }
}
```

<img src="https://tva1.sinaimg.cn/large/e6c9d24ely1h0gpje9k7lj20os0r6jt6.jpg" style="zoom:50%;" />

