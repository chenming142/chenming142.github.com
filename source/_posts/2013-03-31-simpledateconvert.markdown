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
	package com.oo.repos;

	import java.text.ParseException;
	import java.text.SimpleDateFormat;

	import org.apache.commons.lang.time.DateUtils;
	import org.apache.commons.logging.Log;
	import org.apache.commons.logging.LogFactory;

	import com.oo.gzdec.config.SimpleDateConverter;

	/**
	 * 类 Date 表示特定的瞬间，精确到毫秒。 java.util 类 Date
	 * 
	 * java.lang.Object java.util.Date
	 * 
	 * 所有已实现的接口： Serializable, Cloneable, Comparable<Date>
	 * 
	 * 直接已知子类： java.sql.Date, java.sql.Time, java.sql.Timestamp
	 * 
	 * @author Administrator
	 * 
	 */
	public class SimpleUtilDate {
		public static Log logger = LogFactory.getFactory().getInstance(
				SimpleUtilDate.class);

		private java.util.Date dt;

		public SimpleUtilDate() {
			this.dt = new java.util.Date();
		}

		public java.util.Date getDt() {
			return dt;
		}

		public void setDt(java.util.Date dt) {
			this.dt = dt;
		}

		public String convert2StrDate() {
			return SimpleDateConverter.STANDARD_TIME_FORMAT.format(this.dt);
		}

		public static long convert(Object date) {
			long times = 0;
			String strdate = "";
			if (date instanceof java.util.Date) {
				strdate = SimpleDateConverter.STANDARD_TIME_FORMAT.format(date);
			} else {
				strdate = date.toString();
			}
			try {
				logger.debug("转换后的日期型字符串strdate===" + strdate);
				times = DateUtils.parseDate(strdate,
						SimpleDateConverter.CUSTOM_DATE_FORMATS).getTime();
			} catch (ParseException e) {
				logger.error("convert方法出错，arguments==" + date, e);
			}
			return times;
		}

		public java.sql.Date convert2SqlDate() {
			java.sql.Date jsd = null;
			jsd = new java.sql.Date(convert(this.dt));
			return jsd;
		}

		public java.sql.Timestamp convert2Timestamp() {
			java.sql.Timestamp jst = null;
			jst = new java.sql.Timestamp(convert(this.dt));
			return jst;
		}

		/**
		 * 从 JDK 1.1 开始，应该使用 Calendar 类实现日期和时间字段之间转换，使用 DateFormat 类来格式化和解析日期字符串。
		 * 
		 * @param args
		 */
		public static void main(String[] args) {
			SimpleUtilDate simpleUtilDate = new SimpleUtilDate();
			System.out
					.println("---java.util.Date开始转换--------------------------------------------------------");
			String strdate = simpleUtilDate.convert2StrDate();
			System.out.println("convert2StrDate转换后的结果=" + strdate);
			System.out.println("convert2StrDate转换的类型是否正确："
					+ (strdate instanceof String));
			System.out.println("");

			java.sql.Date dt = simpleUtilDate.convert2SqlDate();
			System.out.println("convert2SqlDate转换后的结果=" + dt);
			System.out.println("convert2SqlDate转换后的类型是否正确："
					+ (dt instanceof java.sql.Date));
			System.out.println("");

			java.sql.Timestamp ts = simpleUtilDate.convert2Timestamp();
			System.out.println("convert2Timestamp转换后的结果=" + ts);
			System.out.println("convert2Timestamp转换后的类型是否正确="
					+ (ts instanceof java.sql.Timestamp));
			System.out
					.println("---end--------------------------------------------------------");
		}
	}
{% endcodeblock %}
