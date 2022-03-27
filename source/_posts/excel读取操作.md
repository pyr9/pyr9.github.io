---
title: excel读取操作
date: 2022-03-26 22:25:52
tags: poi
categories: excel
---

# excel 读取

## HSSF(03)

```java
  @Test
  public void testRead03() throws IOException {
    InputStream inputStream = new FileInputStream(PATH+"/excel写入.xls");
    final HSSFWorkbook workbook = new HSSFWorkbook(inputStream);
    final HSSFSheet sheet = workbook.getSheetAt(0);
    final HSSFRow row = sheet.getRow(0);
    final HSSFCell cell = row.getCell(0);
    // 读取值的时候，一定要注意类型！！
    final String cellValue = cell.getStringCellValue();
    System.out.println(cellValue);
    inputStream.close();
  }
```

## XSSF(07)

```java
  @Test
  public void testRead07() throws IOException {
    InputStream inputStream = new FileInputStream(PATH+"/excel写入.xlsx");
    final XSSFWorkbook workbook = new XSSFWorkbook(inputStream);
    final XSSFSheet sheet = workbook.getSheetAt(0);
    final XSSFRow row = sheet.getRow(0);
    final XSSFCell cell = row.getCell(0);
    final String cellValue = cell.getStringCellValue();
    System.out.println(cellValue);
    inputStream.close();
  }
```

## 读取不同类型的数据

```java
public void testCellType() throws IOException {
    InputStream inputStream = new FileInputStream("/Users/panyurou/IdeaProjects/exceldemo/src/test/resources/明细表.xlsx");
    final XSSFWorkbook workbook = new XSSFWorkbook(inputStream);
    // 读取标题行
    final XSSFSheet sheet = workbook.getSheetAt(0);
    final XSSFRow headerRow = sheet.getRow(0);
    final int colCount = headerRow.getPhysicalNumberOfCells();
    for (int i = 0; i < colCount; i++) {
      final XSSFCell cell = headerRow.getCell(i);
      if(cell != null){
        final String cellValue = cell.getStringCellValue();
        System.out.print(cellValue + "|");
      }
    }
    System.out.println();
    // 读取内容
    final int rowsCount = sheet.getPhysicalNumberOfRows();
    for (int i = 1; i < rowsCount; i++) {
      final XSSFRow row = sheet.getRow(i);
      for (int j = 0; j < colCount; j++) {
        final XSSFCell cell = row.getCell(j);
        if(cell != null){
          final CellType cellType = cell.getCellType();
          String cellValue = getCellValue(cell, cellType);
          System.out.print(cellValue + "|");
        }
      }
      System.out.println();
    }
    inputStream.close();
  }

  private String getCellValue(XSSFCell cell, CellType cellType) {
    String cellValue = "";
     switch (cellType){
      case STRING:
        cellValue = cell.getStringCellValue();
         break;
       case NUMERIC:
         if(DateUtil.isCellDateFormatted(cell)){
           final Date date = cell.getDateCellValue();
           cellValue = new DateTime(date).toString("yyyy-MM-dd");
         }else{
           cellValue = String.valueOf(cell.getNumericCellValue());
         }
         break;
       case FORMULA:
        cellValue = cell.getCellFormula();
         break;
       case BOOLEAN:
        cellValue = String.valueOf(cell.getBooleanCellValue());
         break;
       case ERROR:
         System.out.println("类型错误");
         break;
    }
    return cellValue;
  }
```

