---
layout: post
title: excel解析
categories: java
tags: excel java
---

#### jar包
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>3.10-FINAL</version>
        </dependency>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>3.10-FINAL</version>
        </dependency>

#### 读　
[例1](http://examples.javacodegeeks.com/core-java/writeread-excel-files-in-java-example)
例2

    public List<QTgqInfo> parse(String filePath) throws Exception {

        FileInputStream fis = null;
        List<QTgqInfo> qTgqInfoList = Lists.newArrayList();

        try {

            fis = new FileInputStream(filePath);

            // Using XSSF for xlsx format, for xls use HSSF
            Workbook workbook = new HSSFWorkbook(fis);

            int numberOfSheets = workbook.getNumberOfSheets();

            //looping over each workbook sheet
            for (int i = 0; i < numberOfSheets; i++) {
                Sheet sheet = workbook.getSheetAt(i);
                Iterator rowIterator = sheet.iterator();
                int rowNum = 0;
                //iterating over each row
                String siteTmp = "";
                String airLineTmp = "";
                String dateTmp = "";
                String passengerTmp = "";
                while (rowIterator.hasNext()) {
                    rowNum++;
                    Row row = (Row) rowIterator.next();
                    Iterator cellIterator = row.cellIterator();
                    if (rowNum > 1) {
                        //Iterating over each cell (column wise)  in a particular row.
                        QTgqInfo qTgqInfo = new QTgqInfo();
                        Cell cell = (Cell) cellIterator.next();
                        siteTmp = !Strings.isNullOrEmpty(cell.getStringCellValue()) ? cell.getStringCellValue() : siteTmp;
                        qTgqInfo.setSite(siteTmp);
                        cell = (Cell) cellIterator.next();
                        airLineTmp = !Strings.isNullOrEmpty(cell.getStringCellValue()) ? cell.getStringCellValue() : airLineTmp;
                        qTgqInfo.setAirline(airLineTmp);
                        cell = (Cell) cellIterator.next();
                        dateTmp = !Strings.isNullOrEmpty(cell.getStringCellValue()) ? cell.getStringCellValue() : dateTmp;
                        qTgqInfo.setDate(dateTmp);
                        cell = (Cell) cellIterator.next();
                        passengerTmp = !Strings.isNullOrEmpty(cell.getStringCellValue()) ? cell.getStringCellValue() : passengerTmp;
                        if (passengerTmp.contains("成人")) {
                            passengerTmp = "adult";
                        }
                        if (passengerTmp.contains("儿童")) {
                            passengerTmp = "child";
                        }
                        qTgqInfo.setPassenger(passengerTmp);
                        cell = (Cell) cellIterator.next();
                        qTgqInfo.setCabin(cell.getStringCellValue());
                        cell = (Cell) cellIterator.next();
                        qTgqInfo.setRefundFormula(cell.getStringCellValue());
                        cell = (Cell) cellIterator.next();
                        qTgqInfo.setChangeFormula(cell.getStringCellValue());
                        cell = (Cell) cellIterator.next();
                        int type = cell.getCellType();
                        if (type == Cell.CELL_TYPE_STRING) {
                            qTgqInfo.setIsEndorse(String.valueOf(cell.getStringCellValue()));
                        }
                        if (type == Cell.CELL_TYPE_NUMERIC) {
                            qTgqInfo.setIsEndorse(String.valueOf(cell.getNumericCellValue()));
                        }
                        qTgqInfoList.add(qTgqInfo);
                    }
                }
                System.out.println(JsonUtil.writeObjectToString(qTgqInfoList));
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            fis.close();
        }
        return qTgqInfoList;
    }
