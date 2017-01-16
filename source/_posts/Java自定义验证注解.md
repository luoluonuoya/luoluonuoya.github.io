---
title: Java自定义验证注解
date: 2016-08-21 16:21:16
categories: [Java]
tags: [Java]
---
**元注解：Java5定义了4个标准的meta-annotation类型**
　**1、@Target：**表示Annotation所修饰的对象范围
	　取值(ElementType)有：
　　　1）CONSTRUCTOR:用于描述构造器
　　　2）FIELD:用于描述域
　　　3）LOCAL_VARIABLE:用于描述局部变量
　　　4）METHOD:用于描述方法
　　　5）PACKAGE:用于描述包
　　　6）PARAMETER:用于描述参数
　　　7）TYPE:用于描述类、接口(包括注解类型) 或enum声明
　**2、@Retention：**表示Annotation被保留的时间长短
　	　取值(RetentionPoicy)有：
　　　1）SOURCE:在源文件中有效（即源文件保留）
　　　2）CLASS:在class文件中有效（即class保留）
　　　3）RUNTIME:在运行时有效（即运行时保留）
　**3、@Documented：**用于描述其它类型的annotation应该被作为被标注的程序成员的公共API，因此可以被例如javadoc此类的工具文档化。Documented是一个标记注解，没有成员
　**4、@Inherited：**阐述了某个被标注的类型是被继承的。如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类

**hibernate的@Valid注解为我们提供了很多的验证方式，但是并不能完全满足我们的需求**
如下的一个bean，需要联合起来判断，上下午不能同时为空，当开始/结束时间不为空时，结束/开始时间也不能为空
```Java
package com.sq580.mall.goods.web.vo;
/**  
 * @Title: DayTime.java
 * @Package com.sq580.mall.goods.web.vo
 * @company：     社区580
 * @Description: TODO
 * @author zfs
 * @date 2016年11月5日 下午2:05:52
 * @version V1.0  
 */
@DateTimeNotNull
public class DayTime {

    private String morningStartDateTime;
    
    private String morningEndDateTime;
    
    private String afternoonStartDateTime;
    
    private String afternoonEndDateTime;

	public String getMorningStartDateTime() {
		return morningStartDateTime;
	}

	public void setMorningStartDateTime(String morningStartDateTime) {
		this.morningStartDateTime = morningStartDateTime;
	}

	public String getMorningEndDateTime() {
		return morningEndDateTime;
	}

	public void setMorningEndDateTime(String morningEndDateTime) {
		this.morningEndDateTime = morningEndDateTime;
	}

	public Integer getMorningNum() {
		return morningNum;
	}

	public void setMorningNum(Integer morningNum) {
		this.morningNum = morningNum;
	}

	public String getAfternoonStartDateTime() {
		return afternoonStartDateTime;
	}

	public void setAfternoonStartDateTime(String afternoonStartDateTime) {
		this.afternoonStartDateTime = afternoonStartDateTime;
	}

	public String getAfternoonEndDateTime() {
		return afternoonEndDateTime;
	}

	public void setAfternoonEndDateTime(String afternoonEndDateTime) {
		this.afternoonEndDateTime = afternoonEndDateTime;
	}

	public Integer getAfternoonNum() {
		return afternoonNum;
	}

	public void setAfternoonNum(Integer afternoonNum) {
		this.afternoonNum = afternoonNum;
	}

	@Override
	public String toString() {
		return "DayTime [morningStartDateTime=" + morningStartDateTime
				+ ", morningEndDateTime=" + morningEndDateTime
				+ ", morningNum=" + morningNum + ", afternoonStartDateTime="
				+ afternoonStartDateTime + ", afternoonEndDateTime="
				+ afternoonEndDateTime + ", afternoonNum=" + afternoonNum + "]";
	}

}

```
定义一个验证接口（标准格式）
```Java
package test;

import java.lang.annotation.Documented;
import java.lang.annotation.Retention;
import java.lang.annotation.Target;

import javax.validation.Constraint;
import javax.validation.Payload;

import test.DayTimeValid;

@Constraint(validatedBy = DayTimeValid.class)
@Target( {
	java.lang.annotation.ElementType.METHOD,    
    java.lang.annotation.ElementType.FIELD,
    java.lang.annotation.ElementType.TYPE
} )
@Retention(java.lang.annotation.RetentionPolicy.RUNTIME)    
@Documented
public @interface DateTimeNotNull {

	String message() default "{时间不能为空！}";
	
	Class<?>[] groups() default {};
	Class<? extends Payload>[] payload() default {};
	
}
```
验证类
```Java
package test;

import java.text.ParseException;
import java.text.SimpleDateFormat;

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

import test.DayTime;
import test.DateTimeNotNull;

public class DayTimeValid implements ConstraintValidator<DateTimeNotNull, DayTime> {

	private final static SimpleDateFormat sdf = new SimpleDateFormat("HH:mm");

	@Override
	public void initialize(DayTimeVo obj) {

	}

	@Override
	public boolean isValid(DayTime daytime,
			ConstraintValidatorContext constraintValidatorContext) {
		constraintValidatorContext.disableDefaultConstraintViolation();
		if (daytime.getMorningStartDateTime() == null
				&& daytime.getMorningEndDateTime() == null
				&& daytime.getAfternoonStartDateTime() == null
				&& daytime.getAfternoonEndDateTime() == null) {
			constraintValidatorContext.buildConstraintViolationWithTemplate(
					"服务时间不能为空！").addConstraintViolation();
			return false;
		} else if (daytime.getMorningStartDateTime() == null
				&& daytime.getMorningEndDateTime() != null) {
			constraintValidatorContext.buildConstraintViolationWithTemplate(
					"上午开始时间不能为空！").addConstraintViolation();
			return false;
		} else if (daytime.getMorningStartDateTime() != null
				&& daytime.getMorningEndDateTime() == null) {
			constraintValidatorContext.buildConstraintViolationWithTemplate(
					"上午结束时间不能为空！").addConstraintViolation();
			return false;
		} else if (daytime.getAfternoonStartDateTime() == null
				&& daytime.getAfternoonEndDateTime() != null) {
			constraintValidatorContext.buildConstraintViolationWithTemplate(
					"下午开始时间不能为空！").addConstraintViolation();
			return false;
		} else if (daytime.getAfternoonStartDateTime() != null
				&& daytime.getAfternoonEndDateTime() == null) {
			constraintValidatorContext.buildConstraintViolationWithTemplate(
					"下午结束时间不能为空！").addConstraintViolation();
			return false;
		} else if (daytime.getMorningStartDateTime() != null
				&& daytime.getMorningEndDateTime() != null) {
			try {
				sdf.parse(daytime.getMorningStartDateTime());
			} catch (ParseException e) {
				constraintValidatorContext
						.buildConstraintViolationWithTemplate("上午开始时间格式不正确！")
						.addConstraintViolation();
				return false;
			}
			try {
				sdf.parse(daytime.getMorningEndDateTime());
			} catch (ParseException e) {
				constraintValidatorContext
						.buildConstraintViolationWithTemplate("上午结束时间格式不正确！")
						.addConstraintViolation();
				return false;
			}
		} else if (daytime.getAfternoonStartDateTime() != null
				&& daytime.getAfternoonEndDateTime() != null) {
			try {
				sdf.parse(daytime.getAfternoonStartDateTime());
			} catch (ParseException e) {
				constraintValidatorContext
						.buildConstraintViolationWithTemplate("下午开始时间格式不正确！")
						.addConstraintViolation();
				return false;
			}
			try {
				sdf.parse(daytime.getAfternoonEndDateTime());
			} catch (ParseException e) {
				constraintValidatorContext
						.buildConstraintViolationWithTemplate("下午结束时间格式不正确！")
						.addConstraintViolation();
				return false;
			}
		}

		return true;
	}

}
```