---
title: POI导出excel表
date: 2017-01-03 17:06:39
categories: [Java]
tags: [Java]
---
##### 列标题属性注解类
```Java
package excel;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.apache.poi.hssf.util.HSSFColor;
import org.apache.poi.ss.usermodel.CellStyle;

/**  
 * @Title: ExcelAnnotation.java
 * @Package excel
 * @Description: 列标题属性注解
 * @author Zoro
 * @date 2016年12月30日 下午4:13:54
 * @version V1.0  
 */
@Retention(RetentionPolicy.RUNTIME)  
@Target({ElementType.FIELD})
public @interface ExcelAnnotation {

	// 列标题名
	public String name() default "";
	// 行号
	public int col() default 0;
	// 日期格式
	public String dateFormat() default "";
	
	/** 列标题样式 start */
	// 水平对齐（默认居中）
	public short alignment() default CellStyle.ALIGN_CENTER;
	// 垂直对齐（默认）
	public short verticalAlignment() default CellStyle.VERTICAL_CENTER;
	// 填充信息模式和纯色填充单元（默认不填充）
	public short fillPattern() default CellStyle.SOLID_FOREGROUND;
	// 背景色填充（默认白色）
	public short fillBackgroundColor() default HSSFColor.GREY_40_PERCENT.index;
	// 字号（默认16）
	public short fontHeighInPoints() default 16;
	// 字体颜色（默认黑色）
	public short fontColor() default HSSFColor.BLACK.index;
	/** 列标题样式 end */
	
}
```
##### Excel表格属性类
```Java
package excel;

import java.io.File;
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

/**  
 * @Title: ExcelArgs.java
 * @Package excel
 * @Description: Excel表格属性
 * @author Zoro
 * @date 2016年12月30日 下午2:38:48
 * @version V1.0  
 */
public class ExcelArgs {

	// 工作表
    private List<SheetArgs> sheets = new ArrayList<SheetArgs>();
    // 文件名
    private String fileName;
    // 文件保存路径
    private String filePath;
    // 完整路径+文件名
    private String excelName;
    
    /**
    * @Title: init
    * @Description: 初始化excel
    * @param     
    * @return void    
    * @throws
     */
    public void init() {
    	if (null == fileName || "".equals(fileName.trim())) {
    		throw new RuntimeException("Excel 文件名不能为空！");
    	}
    	fileName = fileName.trim();
    	if (!(fileName.endsWith(".xls") || fileName.endsWith(".xlsx"))) {
    		// 设置默认文件后缀
    		fileName = fileName + ".xls";
    	}
    	DateFormat df = new SimpleDateFormat("yyyyMMddHHmmss");
    	fileName = df.format(new Date()) + "_" + fileName;
    	if (null == filePath || "".equals(filePath.trim())) {
    		// 设置默认文件生成路径
    		filePath = "C:\\excel";
    	}
    	if (!(filePath.endsWith(File.separator))) {  
    		filePath += File.separator;
        }
    }
      
    public List<SheetArgs> getSheets() {  
        return sheets;  
    }  
  
    public void setSheets(SheetArgs sheet){  
        if(sheets==null){  
            sheets = new ArrayList<>();  
        }  
        sheets.add(sheet);  
    }
	public String getFileName() {
		return fileName;
	}

	public void setFileName(String fileName) {
		this.fileName = fileName;
	}

	public String getFilePath() {
		return filePath;
	}

	public void setFilePath(String filePath) {
		this.filePath = filePath;
	}

	public String getExcelName() {
		return filePath + fileName;
	}

	public void setExcelName(String excelName) {
		this.excelName = excelName;
	}  
	
}
```
##### Sheet表格属性类
```Java
package excel;

import java.util.List;

import org.apache.poi.hssf.util.HSSFColor;
import org.apache.poi.ss.formula.functions.T;
import org.apache.poi.ss.usermodel.CellStyle;
import org.apache.poi.ss.usermodel.IndexedColors;

/**  
 * @Title: SheetArgs.java
 * @Package excel
 * @Description: Sheet表格属性
 * @author Zoro
 * @date 2016年12月30日 下午2:36:25
 * @version V1.0  
 */
@SuppressWarnings("hiding")
public class SheetArgs<T> {

	// 工作表名
    public String sheetName;
    // 标题名
    public String titleName;
    // 注入类
    public Class<?> clazz;
    // 数据源
    public List<T> source;
    
    /** 标题样式 start */
    // 标题从第几行开始（默认第一行）
    public int defaultRow = 0;
    // 标题从第几行结束（默认第一行）
    public int endRow = 0;
    // 标题从第几列开始（默认第一列）
    public int defaultCol = 0;
    // 标题从第几列结束（默认第一列到最大列数 ）
    public int endCol = 0;
    // 行高
    public int heightInPoints = 40;
    // 默认列宽（默认25）
    public int defaultColumnWidth = 20;
	// 水平对齐（默认居中）
    public short alignment = CellStyle.ALIGN_CENTER;
	// 垂直对齐（默认）
	public short verticalAlignment = CellStyle.VERTICAL_CENTER;
	// 填充信息模式和纯色填充单元（默认全填充）
	public short fillPattern = CellStyle.SOLID_FOREGROUND;
	// 背景色填充（默认浅黄色）
	public short fillBackgroundColor = IndexedColors.LEMON_CHIFFON.index;
	// 字号（默认24）
	public short fontHeighInPoints = 24;
	// 字体颜色（默认黑色）
	public short fontColor = HSSFColor.BLACK.index;
	/** 标题样式  end*/
	
}

```
##### 报表导出数据属性类
```Java
package excel;
/**  
 * @Title: ExcelExport.java
 * @Package excel
 * @Description: 报表导出数据属性
 * @author Zoro
 * @date 2016年12月30日 下午2:24:51
 * @version V1.0  
 */
public class ExcelExport implements Comparable<ExcelExport> {

	public String name;
	
	public int order;
	
	public String dateFormat;

	public ExcelExport(String name, int order, String dateFormat) {
		super();
		this.name = name;
		this.order = order;
		this.dateFormat = dateFormat;
	}

	@Override
	public int compareTo(ExcelExport o) {
		return this.order - o.getOrder();
	}
	
	public int getOrder() {
		return order;
	}
}
```
##### 报表操作工具类
```Java
package excel;

import java.io.FileOutputStream;
import java.lang.reflect.Field;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import org.apache.poi.hssf.usermodel.HSSFFont;
import org.apache.poi.hssf.usermodel.HSSFWorkbook;
import org.apache.poi.hssf.util.HSSFColor;
import org.apache.poi.ss.formula.functions.T;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.CellStyle;
import org.apache.poi.ss.usermodel.Font;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.util.CellRangeAddress;

/**  
 * @Title: ExcelUtil.java
 * @Package excel
 * @Description: 报表操作工具类
 * @author Zoro
 * @date 2016年12月30日 下午3:05:19
 * @version V1.0  
 */
public class ExcelUtil {
	
	/**
	 * @throws SecurityException 
	 * @throws NoSuchFieldException 
	* @Title: exportExcel
	* @Description: 导出报表
	* @param     
	* @return void    
	* @throws
	 */
	public static void exportExcel(ExcelArgs excelArgs) throws Exception {
		// 初始化存储路径
		excelArgs.init();
		// 声明一个工作薄  
        HSSFWorkbook workbook = new HSSFWorkbook();
        // 获取所有工作表
        List<SheetArgs> sheets = excelArgs.getSheets();
        // 遍历所有工作表
        for (SheetArgs sheet : sheets) {
        	createSheet(workbook, sheet);
        }
        
        // 创建一个文件流 
		try(FileOutputStream fos = new FileOutputStream(excelArgs.getExcelName());) {
			// 把内容写入流  
			workbook.write(fos);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	/**
	 * @throws SecurityException 
	 * @throws NoSuchFieldException 
	* @Title: createSheet
	* @Description: 生成工作表
	* @param @param workbook
	* @param @param sheet
	* @return void    
	* @throws
	 */
	private static void createSheet(HSSFWorkbook workbook, SheetArgs mySheet) throws Exception, SecurityException {
		// 创建工作表
		Sheet sheet = workbook.createSheet(mySheet.sheetName);
		sheet.setDefaultColumnWidth(mySheet.defaultColumnWidth);
		Row row = null;
		Cell cell = null;
		int columnlength = 0;
		int rowNumber = 0;
		List<ExcelExport> sourceList = new ArrayList<ExcelExport>();
		
		// 获取所有的属性
		Field[] fields = mySheet.clazz.getDeclaredFields();
		// 设置列标题行
		if (null != fields && fields.length > 0) {
			row = sheet.createRow(mySheet.endRow + 1);
			for (Field field : fields) {
				ExcelAnnotation annotation = field.getAnnotation(ExcelAnnotation.class);
				if (annotation instanceof ExcelAnnotation) {
					// 创建列
					cell = row.createCell(mySheet.defaultCol + annotation.col());
					// 设置列名样式 
					CellStyle titleStyle = workbook.createCellStyle();
			        titleStyle.setAlignment(annotation.alignment());  
			        titleStyle.setVerticalAlignment(annotation.verticalAlignment());
			        titleStyle.setFillForegroundColor(annotation.fillBackgroundColor());
			        titleStyle.setFillPattern(annotation.fillPattern());
			        Font font = workbook.createFont();
			        font.setFontHeightInPoints(annotation.fontHeighInPoints());
			        font.setColor(annotation.fontColor());
			        font.setBoldweight(HSSFFont.BOLDWEIGHT_BOLD);
			        titleStyle.setFont(font);
			        cell.setCellStyle(titleStyle);
			        cell.setCellValue(annotation.name());
			        
			        ExcelExport excelExport = new ExcelExport(field.getName(), annotation.col(), annotation.dateFormat());
			        sourceList.add(excelExport);
			        columnlength++;
				}
			}
		}
		// 排序列
		Collections.sort(sourceList);
		
		// 设置总标题
		if (null != mySheet.titleName && !"".equals(mySheet.titleName.trim())){
			row = sheet.createRow(mySheet.defaultRow);
			cell = row.createCell(mySheet.defaultCol);
            row.setHeightInPoints(mySheet.heightInPoints);  
            cell.setCellValue(mySheet.titleName);
            
            CellStyle titleStyle = workbook.createCellStyle();
	        titleStyle.setAlignment(mySheet.alignment);  
	        titleStyle.setVerticalAlignment(mySheet.verticalAlignment);  
	        titleStyle.setFillForegroundColor(mySheet.fillBackgroundColor);  
	        titleStyle.setFillPattern(mySheet.fillPattern);
	        Font font = workbook.createFont();
	        font.setFontHeightInPoints(mySheet.fontHeighInPoints);
	        font.setColor(mySheet.fontColor);
	        titleStyle.setFont(font);
            cell.setCellStyle(titleStyle);
            
            // 合并单元格
            if (0 == mySheet.endCol && columnlength > 0) {
            	mySheet.endCol = mySheet.defaultCol + columnlength - 1;
            }
            sheet.addMergedRegion(new CellRangeAddress(
            		mySheet.defaultRow, 
            		mySheet.endRow, 
            		mySheet.defaultCol, 
            		mySheet.endCol));
		}
		
		// 设置数据
		List<T> datas = mySheet.source;
		rowNumber = mySheet.endRow + 2;
		if (null != datas && !datas.isEmpty() && 0 < datas.size()) {
			for (Object data : datas) {
				row = sheet.createRow(rowNumber++);
				for (int i = 0; i < columnlength; i++) {
					ExcelExport excelExport = sourceList.get(i);
					cell = row.createCell(mySheet.defaultCol + i);
					Object source = null;
					String value = null;
					
					// 反射获取数据
					Field field = mySheet.clazz.getDeclaredField(excelExport.name);
					field.setAccessible(true);
					source = field.get(data);
					// 单元格样式
					CellStyle cellStyle = workbook.createCellStyle();
					cellStyle.setAlignment(CellStyle.ALIGN_CENTER);
					cellStyle.setVerticalAlignment(CellStyle.VERTICAL_CENTER);
			        Font font = workbook.createFont();
			        font.setFontHeightInPoints((short) 12);
			        font.setColor(HSSFColor.BLACK.index);
			        cellStyle.setFont(font);
			        cell.setCellStyle(cellStyle);
			        
			        value = source.toString();
		            if (null != excelExport.dateFormat && !"".equals(excelExport.dateFormat)) {
		            	value = new SimpleDateFormat(excelExport.dateFormat).format(source);
		            	sheet.setColumnWidth(mySheet.defaultCol + i, 36 * 256);
		            }
					cell.setCellValue(value);
				}
			}
		}
		
	}

}
```
##### 测试要导出的数据源类
```Java
package excel;

import java.math.BigDecimal;
import java.util.Date;

/**  
 * @Title: ExcelSource.java
 * @Package excel
 * @Description: 报表数据源
 * @author Zoro
 * @date 2016年12月30日 下午2:33:36
 * @version V1.0  
 */
public class ExcelSource {

	@ExcelAnnotation(name = "账单id", col = 0)
	private Long billingId;
	
	@ExcelAnnotation(name = "账单号", col = 1)
    private String billingNum;

	@ExcelAnnotation(name = "店铺id", col = 4)
    private Long storeId;
    
	@ExcelAnnotation(name = "店铺名", col = 3)
    private String storeName;
    
	@ExcelAnnotation(name = "开户账号", col = 5)
    private String bankAccount;
    
	@ExcelAnnotation(name = "开户用户账号", col = 6)
    private String bankUsername;
    
	@ExcelAnnotation(name = "开户行", col = 7)
    private String bankName;
    
	@ExcelAnnotation(name = "账单总金额", col = 8)
    private BigDecimal orderAmount;

	@ExcelAnnotation(name = "应付金额", col = 9)
    private BigDecimal handleMoney;

	@ExcelAnnotation(name = "实付金额", col = 10)
    private BigDecimal paidMoney;

	@ExcelAnnotation(name = "账单月", col = 11)
    private String billingMonth;

	@ExcelAnnotation(name = "账单状态", col = 12)
    private Integer status;

	@ExcelAnnotation(name = "出账日期", col = 13, dateFormat = "yyyy年MM月dd日 HH时mm分ss秒")
    private Date issueDateTime;

	@ExcelAnnotation(name = "完成时间", col = 15, dateFormat = "yyyy年MM月dd日 HH时mm分ss秒")
    private Date completeDateTime;

	@ExcelAnnotation(name = "开始时间", col = 14, dateFormat = "yyyy年MM月dd日 HH时mm分ss秒")
    private Date startDateTime;

	@ExcelAnnotation(name = "结束时间", col = 2, dateFormat = "yyyy年MM月dd日 HH时mm分ss秒")
    private Date endDateTime;

	public ExcelSource(Long billingId, String billingNum, Long storeId,
			String storeName, String bankAccount, String bankUsername,
			String bankName, BigDecimal orderAmount, BigDecimal handleMoney,
			BigDecimal paidMoney, String billingMonth, Integer status,
			Date issueDateTime, Date completeDateTime, Date startDateTime,
			Date endDateTime) {
		super();
		this.billingId = billingId;
		this.billingNum = billingNum;
		this.storeId = storeId;
		this.storeName = storeName;
		this.bankAccount = bankAccount;
		this.bankUsername = bankUsername;
		this.bankName = bankName;
		this.orderAmount = orderAmount;
		this.handleMoney = handleMoney;
		this.paidMoney = paidMoney;
		this.billingMonth = billingMonth;
		this.status = status;
		this.issueDateTime = issueDateTime;
		this.completeDateTime = completeDateTime;
		this.startDateTime = startDateTime;
		this.endDateTime = endDateTime;
	}

	public Long getBillingId() {
		return billingId;
	}

	public void setBillingId(Long billingId) {
		this.billingId = billingId;
	}

	public String getBillingNum() {
		return billingNum;
	}

	public void setBillingNum(String billingNum) {
		this.billingNum = billingNum;
	}

	public Long getStoreId() {
		return storeId;
	}

	public void setStoreId(Long storeId) {
		this.storeId = storeId;
	}

	public String getStoreName() {
		return storeName;
	}

	public void setStoreName(String storeName) {
		this.storeName = storeName;
	}

	public String getBankAccount() {
		return bankAccount;
	}

	public void setBankAccount(String bankAccount) {
		this.bankAccount = bankAccount;
	}

	public String getBankUsername() {
		return bankUsername;
	}

	public void setBankUsername(String bankUsername) {
		this.bankUsername = bankUsername;
	}

	public String getBankName() {
		return bankName;
	}

	public void setBankName(String bankName) {
		this.bankName = bankName;
	}

	public BigDecimal getOrderAmount() {
		return orderAmount;
	}

	public void setOrderAmount(BigDecimal orderAmount) {
		this.orderAmount = orderAmount;
	}

	public BigDecimal getHandleMoney() {
		return handleMoney;
	}

	public void setHandleMoney(BigDecimal handleMoney) {
		this.handleMoney = handleMoney;
	}

	public BigDecimal getPaidMoney() {
		return paidMoney;
	}

	public void setPaidMoney(BigDecimal paidMoney) {
		this.paidMoney = paidMoney;
	}

	public String getBillingMonth() {
		return billingMonth;
	}

	public void setBillingMonth(String billingMonth) {
		this.billingMonth = billingMonth;
	}

	public Integer getStatus() {
		return status;
	}

	public void setStatus(Integer status) {
		this.status = status;
	}

	public Date getIssueDateTime() {
		return issueDateTime;
	}

	public void setIssueDateTime(Date issueDateTime) {
		this.issueDateTime = issueDateTime;
	}

	public Date getCompleteDateTime() {
		return completeDateTime;
	}

	public void setCompleteDateTime(Date completeDateTime) {
		this.completeDateTime = completeDateTime;
	}

	public Date getStartDateTime() {
		return startDateTime;
	}

	public void setStartDateTime(Date startDateTime) {
		this.startDateTime = startDateTime;
	}

	public Date getEndDateTime() {
		return endDateTime;
	}

	public void setEndDateTime(Date endDateTime) {
		this.endDateTime = endDateTime;
	}
	
}
```
##### 测试类
```Java
package excel;

import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

/**  
 * @Title: ExcelTest.java
 * @Package excel
 * @Description: TODO
 * @author Zoro
 * @date 2016年12月30日 下午5:12:52
 * @version V1.0  
 */
public class ExcelTest {

	public static void main(String[] args) {

		// 数据源
		ExcelSource source1 = new ExcelSource(10L, "1", 11L, "01", "31", "21", "41", new BigDecimal(51), new BigDecimal(61), new BigDecimal(1), "201601", 71, new Date(), new Date(), new Date(), new Date());
		ExcelSource source2 = new ExcelSource(20L, "2", 22L, "02", "32", "22", "42", new BigDecimal(52), new BigDecimal(62), new BigDecimal(2), "202602", 72, new Date(), new Date(), new Date(), new Date());
		ExcelSource source3 = new ExcelSource(30L, "3", 33L, "03", "33", "23", "43", new BigDecimal(53), new BigDecimal(63), new BigDecimal(3), "203603", 73, new Date(), new Date(), new Date(), new Date());
		ExcelSource source4 = new ExcelSource(40L, "4", 44L, "04", "34", "24", "44", new BigDecimal(54), new BigDecimal(64), new BigDecimal(4), "204604", 74, new Date(), new Date(), new Date(), new Date());
		ExcelSource source5 = new ExcelSource(50L, "5", 55L, "05", "35", "25", "45", new BigDecimal(55), new BigDecimal(65), new BigDecimal(5), "205605", 75, new Date(), new Date(), new Date(), new Date());
		ExcelSource source6 = new ExcelSource(60L, "6", 66L, "06", "36", "26", "46", new BigDecimal(56), new BigDecimal(66), new BigDecimal(6), "206606", 76, new Date(), new Date(), new Date(), new Date());
		List<ExcelSource> list = new ArrayList<ExcelSource>();
		list.add(source1);
		list.add(source2);
		list.add(source3);
		list.add(source4);
		list.add(source5);
		list.add(source6);
		
		// sheet表设置
		SheetArgs<ExcelSource> sheetArgs = new SheetArgs<ExcelSource>();
		sheetArgs.sheetName = "sheet表名";
		sheetArgs.titleName = "总标题名";
		sheetArgs.clazz = ExcelSource.class;
		sheetArgs.source = list;
		
		// excel文件设置
		ExcelArgs excelArgs = new ExcelArgs();
		excelArgs.setSheets(sheetArgs);
		excelArgs.setFileName("excel表名");
		
		try {
			// 导出
			ExcelUtil.exportExcel(excelArgs);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```
##### 效果图（为了截到完整的数据图，调过列宽）
![](image/POI导出excel表/excel文件信息.jpg)

![](image/POI导出excel表/excel导出效果图.jpg)
