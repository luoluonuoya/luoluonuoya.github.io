---
title: Java操作Excel工具
date: : 2016-03-15 21:19:09
categories: [Java]
tags: [java,excel]
---
#### **选择：Apache POI 和 JXL**
---

#### JXL没有使用过，网上查到的两者区别
1. **JVM虚拟机内存消耗的情况：**
1). *数据量3000条数据,每条60列.JVM虚拟机内存大小64M. *
2). *使用POI:运行到2800条左右就报内存溢出. **（实际使用过程操作的数据都是30列几千行以上，并没有这样的现象存在）***
3). *使用JXL:3000条全部出来,并且内存还有21M的空间.*
2. **其它：**
1). *若导出图片的操作较多，建议使用JXL*
2). *额外的Excel特性,例如处理公式,创建单元格样式--颜色,边框,字体,头部,脚部,数据验证,图像,超链接等.，建议使用POI*
3).POI支持XLS和XLSX，且代码看起来比较简洁，而JXL只支持XLS

---
#### **使用：以Apache POI为例**
---
```Java
import java.io.File;
import java.io.FileInputStream;
import java.text.DateFormat;
import java.text.SimpleDateFormat;

import org.apache.poi.hssf.usermodel.HSSFDateUtil;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.ss.util.CellRangeAddress;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;

public class ExcelUtil {

	/**
	* @Title: getWorkBook
	* @Description: 获取Excel文件对象
	* @param @param f
	* @param @return    
	* @return Workbook    
	* @throws
	 */
	public Workbook getWorkBook(File f) {
		Workbook wb = null;
			
		try (FileInputStream fis = new FileInputStream(f)) {
			// 根据后缀获取Excel对象
			if (f.getName().indexOf(".xlsx") > 1) {
				wb = new XSSFWorkbook(fis);
			} else {
				wb = new HSSFWorkbook(fis);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		
		return wb;
	}
	
	/**
	* @Title: readExcel
	* @Description: 读取Excel文件
	* @param @param wb
	* @param @param sheetIndex		sheet页下标，从 0 开始
	* @param @param startReadLine	开始读取的行，从 0 开始
	* @param @param tailLine    	最后读取的行号
	* @return void    
	* @throws
	 */
	public void readExcel(Workbook wb, int sheetIndex, int startReadLine, int tailLine) {
		Sheet sheet = wb.getSheetAt(sheetIndex);
		Row row = null;
		// 行
		for (int i = startReadLine; i < sheet.getLastRowNum() - tailLine + 1; i++) {
			row = sheet.getRow(i);
			// 列
			for (int j = 0; j < row.getLastCellNum(); j++) {
				Cell cell = row.getCell(j);
				// 具体操作业务
			}
		}
	}
	
	/**
	* @Title: getMergedRegionValue
	* @Description: 获取合并单元格的值
	* @param @param sheet
	* @param @param row
	* @param @param column
	* @param @return    
	* @return String    
	* @throws
	 */
	public String getMergedRegionValue(Sheet sheet, int row, int column) {
		// 获取一个 Sheet 表中合并单元格的数量
		int sheetMergeCount = sheet.getNumMergedRegions();
		
		for (int i = 0; i < sheetMergeCount; i++) {
			CellRangeAddress ca = sheet.getMergedRegion(i);
			int firstColumn = ca.getFirstColumn();
			int lastColumn = ca.getLastColumn();
			int firstRow = ca.getFirstRow();
			int lastRow = ca.getLastRow();
			
			if (row >= firstRow && row <= lastRow) {
				if (column >= firstColumn && column <= lastColumn) {
					Row fRow = sheet.getRow(firstRow);
					Cell fCell = fRow.getCell(firstColumn);
					return getCellValue(fCell);
				}
			}
		}
		
		return null;
	}
	
	/**
	* @Title: isMergeRow
	* @Description: 判断合并了行
	* @param @param sheet
	* @param @param row
	* @param @param column
	* @param @return    
	* @return boolean    
	* @throws
	 */
	public boolean isMergeRow(Sheet sheet, int row, int column) {
		// 获取一个 Sheet 表中合并单元格的数量
		int sheetMergeCount = sheet.getNumMergedRegions();
		
		for (int i = 0; i < sheetMergeCount; i++) {
			CellRangeAddress ca = sheet.getMergedRegion(i);
			int firstColumn = ca.getFirstColumn();
			int lastColumn = ca.getLastColumn();
			int firstRow = ca.getFirstRow();
			int lastRow = ca.getLastRow();
			
			if (row == firstRow && row == lastRow) {
				if (column >= firstColumn && column <= lastColumn) {
					return true;
				}
			}
		}
		
		return false;
	}
	
	/**
	* @Title: isMergedRegion
	* @Description: 判断指定的单元格是否是合并单元格
	* @param @param sheet
	* @param @param row
	* @param @param column
	* @param @return    
	* @return boolean    
	* @throws
	 */
	public boolean isMergedRegion(Sheet sheet, int row, int column) {
		int sheetMergeCount = sheet.getNumMergedRegions();

		for (int i = 0; i < sheetMergeCount; i++) {
			CellRangeAddress ca = sheet.getMergedRegion(i);
			int firstColumn = ca.getFirstColumn();
			int lastColumn = ca.getLastColumn();
			int firstRow = ca.getFirstRow();
			int lastRow = ca.getLastRow();
			
			if (row >= firstRow && row <= lastRow) {
				if (column >= firstColumn && column <= lastColumn) {
					return true;
				}
			}
		}
		
		return false;
	}
	
	/**
	* @Title: hasMerged
	* @Description: 判断 Sheet 页中是否含有合并单元格
	* @param @param sheet
	* @param @return    
	* @return boolean    
	* @throws
	 */
	public boolean hasMerged(Sheet sheet) {
		return sheet.getNumMergedRegions() > 0 ? true : false;
	}
	
	/**
	* @Title: mergeRegion
	* @Description: 合并单元格
	* @param @param sheet
	* @param @param firstRow	开始行
	* @param @param lastRow		结束行
	* @param @param firstCol	开始列
	* @param @param lastCol 	 结束列
	* @return void    
	* @throws
	 */
	public void mergeRegion(Sheet sheet, int firstRow, int lastRow, int firstCol, int lastCol) {  
        sheet.addMergedRegion(new CellRangeAddress(firstRow, lastRow, firstCol, lastCol));  
    }
	
	/**
	* @Title: getCellValue
	* @Description: 获取单元格的值
	* @param @param cell
	* @param @return    
	* @return String    
	* @throws
	 */
	public String getCellValue(Cell cell) {
		if (cell == null) {
			return null;
		}
		// 注意： 有时看到单元格是空的，但可能并不是null，只要编辑过的单元格，
		// 删除之后，其实那个位置的对象时存在的，所以可以用这个方法判空
		// ps： 如果是合并列，不管合并前哪个单元格有值，如ABC列，合并后AB列读取到的都会是该类型
		if (cell.getCellType() == Cell.CELL_TYPE_BLANK) {
			return "";
		}
		// 字符型
		if (cell.getCellType() == Cell.CELL_TYPE_STRING) {
			return cell.getStringCellValue();
		}
		// 布尔型
		if (cell.getCellType() == Cell.CELL_TYPE_BOOLEAN) {
			return String.valueOf(cell.getBooleanCellValue());
		}
		// 数字型
		if (cell.getCellType() == Cell.CELL_TYPE_NUMERIC) {
			/* 因为日期和数字都是 Cell.CELL_TYPE_NUMERIC 类型，且 cell.getNumericCellValue() 
			返回的是 double 型，利用 HSSFDateUtil.isCellDateFormatted() 可判断是否是日期类型，但
			日期格式只能处理yyyy-MM-dd, d/m/yyyy h:mm, HH:mm 等不含中文文字的日期格式，如需识别中文，
			只能根据样式来判断，以下是Excel中所有自定义的中文样式，代码if语句中也已添加所有相关中文日期，嫌麻烦可以看文末 */
			/* 
				yyyy年m月d日---31
				yyyy年m月------57
				m月d日---------58
				h"时"mm"分"--32
				h"时"mm"分"ss"秒"--33
				上午/下午h"时"mm"分"--55
				上午/下午h"时"mm"分"ss"秒"--56
			*/
			if (HSSFDateUtil.isCellDateFormatted(cell) || cell.getCellStyle().getDataFormat() == 31 || 
					cell.getCellStyle().getDataFormat() == 32 || cell.getCellStyle().getDataFormat() == 33 || 
					cell.getCellStyle().getDataFormat() == 55 || cell.getCellStyle().getDataFormat() == 56 || 
					cell.getCellStyle().getDataFormat() == 57 || cell.getCellStyle().getDataFormat() == 58) {
				DateFormat df = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");
				return df.format(cell.getDateCellValue());
			} else {
				cell.setCellType(Cell.CELL_TYPE_STRING);
				return cell.getStringCellValue();
			}
		}
		return null;
	}
	
}
```
#### 对于中文日期，也可以直接修改（我用的是POI3.6）org.apache.poi.ss.usermodel.DateUtil.isCellDateFormatted(Cell cell) 的源码，以下是源码
```Java
public static boolean isCellDateFormatted(Cell cell) {
	if (cell == null) return false;
	boolean bDate = false;
	
	double d = cell.getNumericCellValue();
	if ( DateUtil.isValidExcelDate(d) ) {
		CellStyle style = cell.getCellStyle();
		if(style==null) return false;
		int i = style.getDataFormat();
		String f = style.getDataFormatString();
		bDate = isADateFormat(i, f); // 这里调用了日期判断
	}
	return bDate;
}
```
#### 在方法DateUtil.isADateFormat(int formatIndex, String formatString)中找到String fs = formatString;在后面加入中文过滤，重新生成class即可，但是不推荐直接修改源码
```Java
fs = fs.replaceAll("[\"|\']","").replaceAll("[年|月|日|时|分|秒|毫秒|微秒]", ""); 
```