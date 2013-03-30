---
layout: post
title: "在Java中日期时间类型的相互转换"
date: 2013-03-31 02:17
comments: true
categories: J2SE基础
tags: J2SE
keywords: J2SE基础 simpleDateConvert
description: 
---
>在上篇[《Java中Map与TO自动转换》](http://chenming142.github.com/blog/2013/03/26/beanutils/ 'Java中Map与TO自动转换')中的存在一个难点，就是如何把Map（一般是String类型，但不一定保证都是）中的数据，转换为To带类型的属性，接下来我们将解决这个问题。

#### 使用java.beanutil包中ConvertUtil类，进行基本类型的转换
ConvertUtil工具类职责是在字符串和指定类型的实例之间进行转换。目前支持的类型有：
	java.lang.BigDecimal；java.lang.BigInteger   
	java.lang.Class
	   
	byte；java.lang.Byte
	char；java.lang.Character
	short；java.lang.Short
	int；java.lang.Integer
	long；java.lang.Long
	boolean；java.lang.Boolean
	float；java.lang.Float
	double；java.lang.Double

	java.lang.String
	java.io.File
	java.net.URL

	java.sql.Time
	java.sql.Timestamp
总体提供两大类功能：     
1. 将字符串转换为指定类型     
2. 注册、注销和查询类型转换器   
<!-- more -->
{% codeblock Java Syntax lang:java http://j.mp/pPUUmW MDN Documentation %}
	convert(java.lang.Object value) //将任意的实例转变为String
	convert(java.lang.String value, java.lang.Class clazz) //将字符串value转换为clazz的一个实例；如果失败的话，就以String的形式返回value
	convert(java.lang.String[] values, java.lang.Class clazz)//将数组中的每个value都进行转换，最后以Object返回
{% endcodeblock %}

#### 日期时间类型的相互转换
1. 类java.util.Date
该类是java.sql.Date与java.sql.Timestamp的直接父类。该类表示特定的瞬间，精确到毫秒。
{% codeblock Java Syntax lang:java http://j.mp/pPUUmW MDN Documentation %}
	//分配java.util.Date 对象并初始化此对象，以表示分配它的时间（精确到毫秒）
	java.util.Date dt = new java.util.Date();
	//转换为：周 月 日 hh:mm:ss 时区 年份
	//        Wed Mar 27 21:01:44 CST 2013
	String strdate = dt.toString();
{% endcodeblock %}
附：从 JDK 1.1 开始，应该使用 Calendar 类实现日期和时间字段之间转换，使用 DateFormat 类来格式化和解析日期字符串。

2. 类java.sql.Date
该类是类java.util.Date的直接子类。一个包装了毫秒值的瘦包装器 (thin wrapper)，它允许 JDBC 将毫秒值标识为 SQL DATE 值。毫秒值表示自 1970 年 1 月 1 日 00:00:00 GMT 以来经过的毫秒数。
{% codeblock Java Syntax lang:java http://j.mp/pPUUmW MDN Documentation %}
	//使用给定毫秒时间值构造一个java.sql.Date 对象
	java.sql.Date dt = new java.sql.Date(long ms)
	//转换为：yyyy-MM-dd
	//		  2013-03-27
	dt = new java.sql.Date(1364389304272l);//注：最后一个是小写字符L
{% endcodeblock %}

3. 类java.sql.Timestamp
该类是类java.util.Date的直接子类。一个与 java.util.Date 类有关的瘦包装器 (thin wrapper)，它允许 JDBC API 将该类标识为 SQL TIMESTAMP 值。它通过允许小数秒到纳秒级精度的规范来添加保存 SQL TIMESTAMP 小数秒值的能力。
{% codeblock Java Syntax lang:java http://j.mp/pPUUmW MDN Documentation %}
	//使用毫秒时间值构造Timestamp 对象
	java.sql.Timestamp tts = new java.sql.Timestamp(long ms)
	//转换为：yyyy-MM-dd hh:mm:ss.SSS
	//		  2013-03-27 21:01:44.0
	dt = new java.sql.Date(1364389304272l);//注：最后一个是小写字符L
{% endcodeblock %}

#### 类SimpleDateConvert
{% codeblock Java Syntax lang:java http://j.mp/pPUUmW MDN Documentation %}
	package com.oo.gzdec.config;

	import java.sql.Timestamp;
	import java.text.ParseException;
	import java.text.SimpleDateFormat;

	import org.apache.commons.lang.time.DateUtils;
	import org.apache.commons.logging.Log;
	import org.apache.commons.logging.LogFactory;

	public class SimpleDateConverter {
		
		public static Log logger = LogFactory.getLog(SimpleDateConverter.class);
		
		/**
		 * 自定义日期格式数组
		 * @author Administrator
		 * @param
		 */
		public static final String[] CUSTOM_DATE_FORMATS = new String[] {
			 "yyyy-MM",
			 "yyyyMM",
			 "yyyy/MM",
			
			 "yyyyMMdd",
			 "yyyy-MM-dd",
			 "yyyy/MM/dd",
			
			 "yyyyMMddHHmmss",
			 "yyyy-MM-dd HH:mm:ss",
			 "yyyy/MM/dd HH:mm:ss",
			
			 "yyyyMMdd HH",
			 "yyyy/MM/dd HH" ,
			 "yyyy-MM-dd HH" ,
			
			 "yyyyMMdd HH:mm",
			 "yyyy/MM/dd HH:mm" ,
			 "yyyy-MM-dd HH:mm",
			
			 "yyyyMMdd HH:mm:ss.SSS",
			 "yyyy/MM/dd HH:mm:ss.SSS",
			 "yyyy-MM-dd HH:mm:ss.SSS",
		};
		
		/**
		 * 自定义标准的日期格式
		 */
		public static SimpleDateFormat STANDARD_TIME_FORMAT = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		
		/**
		 * 将传入的日期型字符串或日期时间类型转换为时间毫秒数
		 * @param date
		 * @return
		 */
		public static long convert(Object date){
			long timeMillis = 0;
			String strdate = convert2String(date);
			try {
				timeMillis = DateUtils.parseDate(strdate, SimpleDateConverter.CUSTOM_DATE_FORMATS).getTime();
			} catch (ParseException e) {
				logger.error("日期时间转换失败，日期型字符串为："+strdate, e);
			}
			return timeMillis;
		}
		
		/**
		 * 将传入的日期型字符串或日期时间类型转换为日期型字符串
		 * @param date
		 * @return
		 */
		public static String convert2String(Object date){
			if( date instanceof java.util.Date){
				return  STANDARD_TIME_FORMAT.format(date);
			}
			return date.toString();
		}
		
		/**
		 * 将传入的日期型字符串或日期时间类型转换为 java.sql.Date 类型变量
		 * @param date
		 * @return
		 */
		public static java.sql.Date convert2SqlDate(Object date){
			java.sql.Date dt = null;
			dt = new java.sql.Date(convert(date));
			return dt;
		}
		
		/**
		 * 将传入的日期型字符串或日期时间类型转换为 java.util.Date 类型变量
		 * @param date
		 * @return
		 */
		public static java.util.Date convert2UtilDate(Object date){
			java.util.Date dt = null;
			dt = new java.util.Date(convert(date));
			return dt;
		}
		
		/**
		 * 将传入的日期型字符串或日期时间类型转换为 java.sql.Timestamp 类型变量
		 * @param date
		 * @return
		 */
		public static java.sql.Timestamp convert2Timestamp(Object date){
			java.sql.Timestamp tts = null;
			tts = new java.sql.Timestamp(convert(date));
			return tts;
		}
	}
{% endcodeblock %}
