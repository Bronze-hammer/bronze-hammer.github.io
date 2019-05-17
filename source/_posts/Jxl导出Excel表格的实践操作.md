---
title: Jxl导出Excel表格的实践操作
date: 2019-05-17 11:10:28
tags:
- Excel
- Jxl
categories:
- 编程语言
---

**本次测试操作实现的功能主要有：**

- 建立工作薄
- 输出文本到Excel指定单元格
- 设置文本属性（文本大小、字体、粗细、颜色等）
- 设置文本对齐方式
- 合并单元格
- 设置单元格背景颜色

<!-- more -->

```java
package cn.xuzihui.excel;

import java.io.File;
import java.io.IOException;
import jxl.Workbook;
import jxl.format.Alignment;
import jxl.format.Colour;
import jxl.write.Label;
import jxl.write.VerticalAlignment;
import jxl.write.WritableCellFormat;
import jxl.write.WritableFont;
import jxl.write.WritableSheet;
import jxl.write.WritableWorkbook;
import jxl.write.WriteException;

@SuppressWarnings("deprecation")
public class ExportExcel {
	//导出文件的路径
	public String createPath = "C:\\Users\\zihui\\Desktop\\";
	//导出文件的文件名以及后缀
	public String fileName = "text.xls";
	//导出的Excel标题
	public String title = "物品邮寄信息统计表";
	//存储文件的Sheet页签名的数组
	public String[] sheetName = {"First"};
	//Sheet页签中表头字段
	public String[] tableHead = {"序号", "姓名", "性别", "电话号码", "地址", "邮编"}; 
	
	public static void main(String[] args) throws WriteException, IOException {
		ExportExcel excel = new ExportExcel();
		excel.createExcel();
	}
	
	public void createExcel() throws IOException, WriteException {
		File file = new File(createPath + fileName);
		WritableWorkbook workbook = Workbook.createWorkbook(file);
		WritableSheet sheet;
		for(int i = 0 ; i < sheetName.length ; i++) {
			//建立工作薄
			sheet = workbook.createSheet(sheetName[i], i);
			setExcelValue(sheet, tableHead);
		}
		workbook.write();
		workbook.close();
		
	}
	
	/**
	 * <p>向要导出的Excel表中填入数据</p>
	 * <p>创建时间：2018年9月19日 18点00分</p>
	 * @author zihui
	 * @param sheet
	 * @param tableHead
	 * @throws WriteException 
	 */
	private void setExcelValue(WritableSheet sheet, String[] tableHead) throws WriteException {
		int rowNum = 0; //从第一行开始填数数据
		rowNum = setDirectionsLine(sheet, rowNum); //设置说明行
		rowNum = setTitleLine(sheet, rowNum); //设置标题行
		rowNum = setTableHead(sheet, rowNum, tableHead); //设置表头
		
	}

	/**
	 * <p>设置说明行</p>
	 * <p>创建时间：2018年9月19日 18点22分</p>
	 * @author zihui
	 */
	private int setDirectionsLine(WritableSheet sheet, int rowNum) throws WriteException {
		StringBuffer comment = new StringBuffer(); //comment说明的内容
		comment.append("说明：").append("\r\n");
		comment.append("这是一个导出Excel的测试类；").append("\r\n");
		comment.append("和大家一起分享探讨。");
		WritableFont font = new WritableFont(WritableFont.ARIAL, 10, WritableFont.BOLD);
		font.setColour(Colour.RED); //设置说明文字的颜色
		WritableCellFormat cellFormat = new WritableCellFormat(font);
		cellFormat.setVerticalAlignment(VerticalAlignment.CENTRE);
		cellFormat.setAlignment(Alignment.LEFT); //左对齐
		Label commentLabel = new Label(0, rowNum, comment.toString(), cellFormat);
		sheet.addCell(commentLabel);
		sheet.mergeCells(0, rowNum, tableHead.length - 1, rowNum);
		return rowNum + 1;
	}

	/**
	 * <p>设置标题行</p>
	 * <p>创建时间：2018年9月20日 10点09分</p>
	 * @author zihui
	 * @param sheet
	 * @param rowNum
	 * @return
	 * @throws WriteException
	 */
	private int setTitleLine(WritableSheet sheet, int rowNum) throws WriteException {
		WritableFont font = new WritableFont(WritableFont.ARIAL, 14, WritableFont.BOLD);
		WritableCellFormat cellFormat = new WritableCellFormat(font);
		cellFormat.setVerticalAlignment(VerticalAlignment.CENTRE);
		cellFormat.setAlignment(Alignment.CENTRE);
		Label commentLabel = new Label(0, rowNum, title, cellFormat);
		sheet.addCell(commentLabel);
		sheet.mergeCells(0, rowNum, tableHead.length - 1, rowNum);
		return rowNum + 1;
	}
	
	/**
	 * <p>设置表头</p>
	 * <p>创建时间：2018年9月20日 10点35分</p>
	 * @author zihui
	 * @param sheet
	 * @param rowNum
	 * @param tableHead
	 * @return
	 * @throws WriteException
	 */
	private int setTableHead(WritableSheet sheet, int rowNum, String[] tableHead) throws WriteException {
		WritableFont font = new WritableFont(WritableFont.ARIAL, 12);
		WritableCellFormat cellFormat = new WritableCellFormat(font);
		cellFormat.setBackground(Colour.BLUE_GREY);
		cellFormat.setVerticalAlignment(VerticalAlignment.CENTRE);
		cellFormat.setAlignment(Alignment.CENTRE);
		Label num = new Label(0, rowNum, tableHead[0], cellFormat);
		Label username = new Label(1, rowNum, tableHead[1], cellFormat);
		Label sex = new Label(2, rowNum, tableHead[2], cellFormat);
		Label phone = new Label(3, rowNum, tableHead[3], cellFormat);
		Label address = new Label(4, rowNum, tableHead[4], cellFormat);
		Label zipcode = new Label(5, rowNum, tableHead[5], cellFormat);
		sheet.addCell(num);
		sheet.addCell(username);
		sheet.addCell(sex);
		sheet.addCell(phone);
		sheet.addCell(address);
		sheet.addCell(zipcode);
		return rowNum + 1;
	}
}

```

运行结果

![Jxl导出Excel表格的实践操作](20180921161317122.png)