---
layout: post
title: "Java中Map与TO自动转换"
date: 2013-03-26 11:05
comments: true
categories: J2SE基础
tags: J2SE
keywords: bean J2SE 
description: 
---
>在JAVA开发过程中需要经常使用到TO，TO能方便的对对象成员变量的存取与持久化；但是业务流程开发过程中，我们通常使用Map来进行参数的获取和临时保存，所以在Map与TO之间进行自动转换是非常必要地。   
   
+ 使用Java反射机制来转换
在Java运行时环境中，对于任意一个类，根据反射机制都能知道该类包含的属性和方法；对于该类的任意对象都能调用属于它的方法即__能动态获取该类的信息及动态调用对象的方法__。   
Java反射机制主要提供以下功能：
		在运行时判断任意一个对象所属的类
		在运行时构造任意一个类的对象
		在运行时判断任意一个类所具有的属性和方法
		在运行时调用任意一个对象的方法
在JDK中，主要由以下类来实现Java反射机制，这些类位于java.lang.reflect包中：
		Class类      ：代表一个实体类
		Field类      ：代表实体类的成员变量即类的属性
		Method类	 ：代表实体类的方法
		Constructor类：代表实体类的构造方法
		Array类      ：提供了动态创建数组，以及访问数组的元素的静态方法
通过Class类获取成员变量、成员方法、接口、超类和构造方法等   
<!-- more -->

#### 类的类型
		在java.lang.Object 类中定义了getClass()方法，因此对于任意一个Java对象，都可以通过此方法获得对象的类型。
如果没有对象实例的时候，也有两种办法获取类的类型：
		Class clazz = 类.class;
		Class clazz = Class.forName('包名+类名');
#### 类的成员变量
{% codeblock Java Syntax lang:java http://j.mp/pPUUmW MDN Documentation %}
		public Field getDeclaredField(String name)     //获取任意指定名字的成员
		public Field[] getDeclaredFields()             //获取所有的成员变量
		public Field getField(String name)             //获取任意public成员变量
		public Field[] getFields()                     //获取所有的public成员变量
{% endcodeblock %}

#### 类的方法
{% codeblock Java Syntax lang:java http://j.mp/pPUUmW MDN Documentation %}
		public Method[] getMethods()   // 获取所有的共有方法的集合
		public Method getMethod(String name,Class<?>... parameterTypes) //获取指定公有方法 参数1：方法名 参数2：参数类型集合  
		public Method[] getDeclaredMethods()  //获取所有的方法
		public Method getDeclaredMethod(String name,Class<?>... parameterTypes) //获取任意指定方法
{% endcodeblock %}

#### 类的构造器
{% codeblock Java Syntax lang:java http://j.mp/pPUUmW MDN Documentation %}
		public Constructor<?>[] getConstructors()     // 返回类中所有的public构造器集合，默认构造器的下标为0
		public Constructor<T> getConstructor(Class<?>... parameterTypes)  // 返回指定public构造器，参数为构造器参数类型集合
		public Constructor<?>[] getDeclaredConstructors() // 返回类中所有的构造器，包括私有
		public Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes) //返回任意指定的构造器
{% endcodeblock %}

{% codeblock Java Syntax lang:java http://j.mp/pPUUmW MDN Documentation %}
	package com.oo.repos;

	import java.lang.reflect.Constructor;
	import java.lang.reflect.Field;
	import java.lang.reflect.Method;
	import java.lang.reflect.Type;

	public class RefConstructor {
		 
		public static void main(String args[]) throws Exception {
			RefConstructor ref = new RefConstructor();
			ref.getConstructor();
		}
	 
		public void getConstructor() throws Exception {
			Class c = null;
			c = Class.forName("java.lang.Long");
			Class cs[] = {java.lang.String.class};
	 
			System.out.println("\n-------------------------------\n");
	 
			Constructor cst1 = c.getConstructor(cs);
			System.out.println("1、通过参数获取指定Class对象的构造方法：");
			System.out.println(cst1.toString());
	 
			Constructor cst2 = c.getDeclaredConstructor(cs);
			System.out.println("2、通过参数获取指定Class对象所表示的类或接口的构造方法：");
			System.out.println(cst2.toString());
	 
			Constructor cst3 = c.getEnclosingConstructor();
			System.out.println("3、获取本地或匿名类Constructor 对象，它表示基础类的立即封闭构造方法。");
			if (cst3 != null) System.out.println(cst3.toString());
			else System.out.println("-- 没有获取到任何构造方法！");
	 
			Constructor[] csts = c.getConstructors();
			System.out.println("4、获取指定Class对象的所有构造方法：");
			for (int i = 0; i < csts.length; i++) {
				System.out.println(csts[i].toString());
			}
	 
			System.out.println("\n-------------------------------\n");
	 
			Type types1[] = c.getGenericInterfaces();
			System.out.println("1、返回直接实现的接口：");
			for (int i = 0; i < types1.length; i++) {
				System.out.println(types1[i].toString());
			}
	 
			Type type1 = c.getGenericSuperclass();
			System.out.println("2、返回直接超类：");
			System.out.println(type1.toString());
	 
			Class[] cis = c.getClasses();
			System.out.println("3、返回超类和所有实现的接口：");
			for (int i = 0; i < cis.length; i++) {
				System.out.println(cis[i].toString());
			}
	 
			Class cs1[] = c.getInterfaces();
			System.out.println("4、实现的接口");
			for (int i = 0; i < cs1.length; i++) {
				System.out.println(cs1[i].toString());
			}
	 
			System.out.println("\n-------------------------------\n");
	 
			Field fs1[] = c.getFields();
			System.out.println("1、类或接口的所有可访问公共字段：");
			for (int i = 0; i < fs1.length; i++) {
				System.out.println(fs1[i].toString());
			}
	 
			Field f1 = c.getField("MIN_VALUE");
			System.out.println("2、类或接口的指定已声明指定公共成员字段：");
			System.out.println(f1.toString());
	 
			Field fs2[] = c.getDeclaredFields();
			System.out.println("3、类或接口所声明的所有字段：");
			for (int i = 0; i < fs2.length; i++) {
				System.out.println(fs2[i].toString());
			}
	 
			Field f2 = c.getDeclaredField("serialVersionUID");
			System.out.println("4、类或接口的指定已声明指定字段：");
			System.out.println(f2.toString());
	 
			System.out.println("\n-------------------------------\n");
	 
			Method m1[] = c.getMethods();
			System.out.println("1、返回类所有的公共成员方法：");
			for (int i = 0; i < m1.length; i++) {
				System.out.println(m1[i].toString());
			}
	 
			Method m2 = c.getMethod("longValue", new Class[]{});
			System.out.println("2、返回指定公共成员方法：");
			System.out.println(m2.toString());
		}
	}
{% endcodeblock %}

##### 运行时复制对象
{% codeblock Java Syntax lang:java http://j.mp/pPUUmW MDN Documentation %}
	package com.oo.repos;

	import java.lang.reflect.Field;
	import java.lang.reflect.Method;

	class Customer {
		private Long id;
	 
		private String name;
	 
		private int age;
	 
		public Customer() {
		}
	 
		public Customer(String name, int age) {
			this.name = name;
			this.age = age;
		}
	 
		public Long getId() {
			return id;
		}
	 
		public void setId(Long id) {
			this.id = id;
		}
	 
		public String getName() {
			return name;
		}
	 
		public void setName(String name) {
			this.name = name;
		}
	 
		public int getAge() {
			return age;
		}
	 
		public void setAge(int age) {
			this.age = age;
		}
	}
	public class ReflectTester {
		public Object copy(Object object) throws Exception {
			// 获得对象的类型
			Class<?> classType = object.getClass();
			System.out.println("Class:" + classType.getName());
	 
			// 通过默认构造方法创建一个新的对象
			Object objectCopy = classType.getConstructor(new Class[]{}).newInstance(new Object[]{});
	 
			// 获得对象的所有属性
			Field fields[] = classType.getDeclaredFields();
	 
			for (int i = 0; i < fields.length; i++) {
				Field field = fields[i];
	 
				String fieldName = field.getName();
				String firstLetter = fieldName.substring(0, 1).toUpperCase();
				// 获得和属性对应的getXXX()方法的名字
				String getMethodName = "get" + firstLetter + fieldName.substring(1);
				// 获得和属性对应的setXXX()方法的名字
				String setMethodName = "set" + firstLetter + fieldName.substring(1);
	 
				// 获得和属性对应的getXXX()方法
				Method getMethod = classType.getMethod(getMethodName, new Class[]{});
				// 获得和属性对应的setXXX()方法
				Method setMethod = classType.getMethod(setMethodName, new Class[]{field.getType()});
	 
				// 调用原对象的getXXX()方法
				Object value = getMethod.invoke(object, new Object[]{});
				System.out.println(fieldName + ":" + value);
				// 调用拷贝对象的setXXX()方法
				setMethod.invoke(objectCopy, new Object[]{value});
			}
			return objectCopy;
		}
	 
		public static void main(String[] args) throws Exception {
			Customer customer = new Customer("Tom", 21);
			customer.setId(new Long(1));
	 
			Customer customerCopy = (Customer) new ReflectTester().copy(customer);
			System.out.println("Copy information:" + customerCopy.getId() + " " + customerCopy.getName() + " "
					+ customerCopy.getAge());
		}
	}
{% endcodeblock %}

##### 用反射机制调用对象的方法
{% codeblock Java Syntax lang:java http://j.mp/pPUUmW MDN Documentation %}
	package com.oo.repos;

	import java.lang.reflect.Method;

	public class InvokeTester {
		public int add(int param1, int param2) {
			return param1 + param2;
		}
	 
		public String echo(String msg) {
			return "echo: " + msg;
		}
	 
		public static void main(String[] args) throws Exception {
			Class<?> classType = InvokeTester.class;
			Object invokeTester = classType.newInstance();
	 
			//获取InvokeTester类的add()方法
			Method addMethod = classType.getMethod("add", new Class[]{int.class, int.class});
			//调用invokeTester对象上的add()方法
			Object result = addMethod.invoke(invokeTester, new Object[]{new Integer(100), new Integer(200)});
			System.out.println((Integer) result);
	 
			//获取InvokeTester类的echo()方法
			Method echoMethod = classType.getMethod("echo", new Class[]{String.class});
			//调用invokeTester对象的echo()方法
			result = echoMethod.invoke(invokeTester, new Object[]{"Hello"});
			System.out.println((String) result);
		}
	}
{% endcodeblock %}
##### 动态创建和访问数组
{% codeblock Java Syntax lang:java http://j.mp/pPUUmW MDN Documentation %}
	package com.oo.repos.ref;

	import java.lang.reflect.Array;

	public class ArrayTester {
		public static void main(String args[]) throws Exception {
			Class<?> classType = Class.forName("java.lang.String");
			// 创建一个长度为10的字符串数组
			Object array = Array.newInstance(classType, 10);
			// 把索引位置为5的元素设为"hello"
			Array.set(array, 5, "hello");
			// 获得索引位置为5的元素的值
			String s = (String) Array.get(array, 5);
			System.out.println(s);
		}
	}
{% endcodeblock %}

##### 运行时变更field内容
{% codeblock Java Syntax lang:java http://j.mp/pPUUmW MDN Documentation %}
	package com.oo.repos.ref;

	import java.lang.reflect.Field;

	public class RefFiled {
		public double x;
		public Double y;
	 
		public static void main(String args[]) throws NoSuchFieldException, IllegalAccessException {
			Class c = RefFiled.class;
			Field xf = c.getField("x");
			Field yf = c.getField("y");
	 
			RefFiled obj = new RefFiled();
	 
			System.out.println("变更前x=" + xf.get(obj));
			//变更成员x值
			xf.set(obj, 1.1);
			System.out.println("变更后x=" + xf.get(obj));
	 
			System.out.println("变更前y=" + yf.get(obj));
			//变更成员y值
			yf.set(obj, 2.1);
			System.out.println("变更后y=" + yf.get(obj));
		}
	}
{% endcodeblock %}

#### JAVA反射机制Map与TO的转换
{% codeblock Java Syntax lang:java http://j.mp/pPUUmW MDN Documentation %}
package com.test;

import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.HashMap;
import java.util.Locale;
import java.util.Map;

/**
 * java bean 反射的方法
 * @author Michael
 */
public class BeanRefUtil {

    /**
     * 取Bean的属性和值对应关系的MAP
     * @param bean
     * @return Map
     */
    public static Map<String, String> getFieldValueMap(Object bean) {
        Class<?> cls = bean.getClass();
        Map<String, String> valueMap = new HashMap<String, String>();
        // 取出bean里的所有方法
        Method[] methods = cls.getDeclaredMethods();
        Field[] fields = cls.getDeclaredFields();

        for (Field field : fields) {
            try {
                String fieldType = field.getType().getSimpleName();
                String fieldGetName = parGetName(field.getName());
                if (!checkGetMet(methods, fieldGetName)) {
                    continue;
                }
                Method fieldGetMet = cls
                        .getMethod(fieldGetName, new Class[] {});
                Object fieldVal = fieldGetMet.invoke(bean, new Object[] {});
                String result = null;
                if ("Date".equals(fieldType)) {
                    result = fmtDate((Date) fieldVal);
                } else {
                    if (null != fieldVal) {
                        result = String.valueOf(fieldVal);
                    }
                }
                valueMap.put(field.getName(), result);
            } catch (Exception e) {
                continue;
            }
        }
        return valueMap;

    }

    /**
     * set属性的值到Bean
     * @param bean
     * @param valMap
     */
    public static void setFieldValue(Object bean, Map<String, String> valMap) {
        Class<?> cls = bean.getClass();
        // 取出bean里的所有方法
        Method[] methods = cls.getDeclaredMethods();
        Field[] fields = cls.getDeclaredFields();

        for (Field field : fields) {
            try {

                String fieldSetName = parSetName(field.getName());
                if (!checkSetMet(methods, fieldSetName)) {
                    continue;
                }
                Method fieldSetMet = cls.getMethod(fieldSetName, field
                        .getType());
                String value = valMap.get(field.getName());
                if (null != value && !"".equals(value)) {
                    String fieldType = field.getType().getSimpleName();
                    if ("String".equals(fieldType)) {
                        fieldSetMet.invoke(bean, value);
                    } else if ("Date".equals(fieldType)) {
                        Date temp = parseDate(value);
                        fieldSetMet.invoke(bean, temp);
                    } else if ("Integer".equals(fieldType)
                            || "int".equals(fieldType)) {
                        Integer intval = Integer.parseInt(value);
                        fieldSetMet.invoke(bean, intval);
                    } else if ("Long".equalsIgnoreCase(fieldType)) {
                        Long temp = Long.parseLong(value);
                        fieldSetMet.invoke(bean, temp);
                    } else if ("Double".equalsIgnoreCase(fieldType)) {
                        Double temp = Double.parseDouble(value);
                        fieldSetMet.invoke(bean, temp);
                    } else if ("Boolean".equalsIgnoreCase(fieldType)) {
                        Boolean temp = Boolean.parseBoolean(value);
                        fieldSetMet.invoke(bean, temp);
                    } else {
                        System.out.println("not supper type" + fieldType);
                    }
                }
            } catch (Exception e) {
                continue;
            }
        }

    }

    /**
     * 格式化string为Date
     * @param datestr
     * @return date
     */
    public static Date parseDate(String datestr) {
        if (null == datestr || "".equals(datestr)) {
            return null;
        }
        try {
            String fmtstr = null;
            if (datestr.indexOf(':') > 0) {
                fmtstr = "yyyy-MM-dd HH:mm:ss";
            } else {

                fmtstr = "yyyy-MM-dd";
            }
            SimpleDateFormat sdf = new SimpleDateFormat(fmtstr, Locale.UK);
            return sdf.parse(datestr);
        } catch (Exception e) {
            return null;
        }
    }

    /**
     * 日期转化为String
     * @param date
     * @return date string
     */
    public static String fmtDate(Date date) {
        if (null == date) {
            return null;
        }
        try {
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss",
                    Locale.US);
            return sdf.format(date);
        } catch (Exception e) {
            return null;
        }
    }

    /**
     * 判断是否存在某属性的 set方法
     * @param methods
     * @param fieldSetMet
     * @return boolean
     */
    public static boolean checkSetMet(Method[] methods, String fieldSetMet) {
        for (Method met : methods) {
            if (fieldSetMet.equals(met.getName())) {
                return true;
            }
        }
        return false;
    }

    /**
     * 判断是否存在某属性的 get方法
     * @param methods
     * @param fieldGetMet
     * @return boolean
     */
    public static boolean checkGetMet(Method[] methods, String fieldGetMet) {
        for (Method met : methods) {
            if (fieldGetMet.equals(met.getName())) {
                return true;
            }
        }
        return false;
    }

    /**
     * 拼接某属性的 get方法
     * @param fieldName
     * @return String
     */
    public static String parGetName(String fieldName) {
        if (null == fieldName || "".equals(fieldName)) {
            return null;
        }
        return "get" + fieldName.substring(0, 1).toUpperCase()
                + fieldName.substring(1);
    }

    /**
     * 拼接在某属性的 set方法
     * @param fieldName
     * @return String
     */
    public static String parSetName(String fieldName) {
        if (null == fieldName || "".equals(fieldName)) {
            return null;
        }
        return "set" + fieldName.substring(0, 1).toUpperCase()
                + fieldName.substring(1);
    }
}
{% endcodeblock %}

+ 使用内省Introspector类|PropertyDescriptor类|BeanUtils工具包   
###Introspector类
将JavaBean中的属性封装起来进行操作
	在程序把一个类当做JavaBean来看，就是调用Introspector.getBeanInfo()方法，得到的BeanInfo对象封装了把这个类当做JavaBean看的结果信息，即属性的信息。需要导包java.beans.*
	getPropertyDescriptors()，获得属性的描述，可以采用遍历BeanInfo的方法，来查找、设置类的属性


###PropertyDescriptor类
表示JavaBean类通过存储器导出一个属性。主要方法：
	getPropertyType()                 获得属性的Class对象。
	getReadMethod()                   获得用于读取属性值的方法；
	getWriteMethod()                  获得用于写入属性值的方法。
	hashCode()                        获取对象的哈希值。
	setReadMethod(Method readMethod)  设置用于读取属性值的方法；
	setWriteMethod(MethodwriteMethod) 设置用于写入属性值的方法；
附：导包java.bean.*;

{% codeblock Java Syntax lang:java http://j.mp/pPUUmW MDN Documentation %}
	package com.oo.gzdec.config;

	import java.beans.BeanInfo;
	import java.beans.IntrospectionException;
	import java.beans.Introspector;
	import java.beans.PropertyDescriptor;
	import java.lang.reflect.Field;
	import java.lang.reflect.InvocationTargetException;
	import java.lang.reflect.Method;
	import java.sql.Blob;
	import java.sql.Clob;
	import java.text.ParseException;
	import java.util.HashMap;
	import java.util.Map;

	import org.apache.commons.beanutils.ConvertUtils;

	public class BeanUtils {

		/**
		 * 将一个 TO对象 转化为一个Map
		 * 
		 * @param <T>
		 * @param TO
		 * @return
		 */
		public static <T> Map<String, Object> setTo2Map(T TO) {
			Class<? extends Object> toType = TO.getClass();
			Map<String, Object> returnMap = new HashMap<String, Object>();
			try {
				BeanInfo beanInfo = Introspector.getBeanInfo(toType);
				PropertyDescriptor[] propertyDescriptors = beanInfo.getPropertyDescriptors();

				for (PropertyDescriptor descriptor : propertyDescriptors) {
					String propName = descriptor.getName();
					if (!propName.equals("class")) {
						Method readMethod = descriptor.getReadMethod();
						Object result = readMethod.invoke(TO, new Object[0]);
						returnMap.put(propName, result != null ? result : "");
					}
				}
			} catch (IllegalArgumentException e) {
				throw new RuntimeException("不合法或不正确的参数", e);
			} catch (IntrospectionException e) {
				throw new RuntimeException("分析类属性失败", e);
			} catch (IllegalAccessException e) {
				throw new RuntimeException("实例化JavaBean失败", e);
			} catch (InvocationTargetException e) {
				throw new RuntimeException("调用目标失败", e);
			}
			return returnMap;
		}

		/**
		 * 将一个 Map对象 转换为对应的 TO
		 * 
		 * @param <T>
		 * @param valueMap -
		 *            包含属性值的map
		 * @param clazz -
		 *            要转化的类型
		 * @return 转化出来的结果TO
		 */
		public static <T> T setMap2To(Map<String, Object> paraMap, Class<T> clazz) {
			T TO = null;
			try {
				TO = clazz.newInstance();
				String propName = null;
				Object propValue = null;
				for (Map.Entry<String, Object> entity : paraMap.entrySet()) {
					propName = entity.getKey();
					propValue = entity.getValue();
					setProperties(TO, propName, propValue);
				}
			} catch (IllegalArgumentException e) {
				throw new RuntimeException("不合法或不正确的参数", e);
			} catch (IllegalAccessException e) {
				throw new RuntimeException("实例化JavaBean失败", e);
			} catch (Exception e) {
				throw new RuntimeException("Map转换为Java Bean对象失败", e);
			}
			return TO;
		}

		/**
		 * 根据 TO 和 属性名/值对,设置TO值
		 * 
		 * @param <T>
		 * @param entity
		 * @param propName
		 * @param propValue
		 */
		public static <T> void setProperties(T entity, String propName,Object propValue) throws IntrospectionException,
				IllegalArgumentException, IllegalAccessException,InvocationTargetException, ParseException {
			BeanInfo beanInfo = Introspector.getBeanInfo(entity.getClass());
			PropertyDescriptor[] propertyDescriptors = beanInfo.getPropertyDescriptors();
			Object[] args = new Object[1];

			for (int i = 0; i < propertyDescriptors.length; i++) {
				PropertyDescriptor propertyDescriptor = propertyDescriptors[i];
				String properyName = propertyDescriptor.getName();
				Class propertyTypeClass = propertyDescriptor.getPropertyType();

				if (propName.equals(properyName)) {
					// TODO : 因为 java.sql.Date与java.sql.Timestamp 是 java.util.Date 的子类，所以先判断是否是这两种类型
					if (!propValue.getClass().getName().equals(propertyTypeClass.getName())) {
						if (propValue instanceof java.sql.Date) {
							args[0] = SimpleDateConverter.convert2UtilDate(propValue);
						} else if(propValue instanceof java.sql.Timestamp){
							args[0] = SimpleDateConverter.convert2Timestamp(propValue);
						} else if (propValue instanceof java.util.Date) {
							args[0] = SimpleDateConverter.convert2SqlDate(propValue);
						} else {
							args[0] = ConvertUtils.convert(propValue.toString(),propertyTypeClass);
						}
					} else {
						args[0] = propValue;
					}
					propertyDescriptor.getWriteMethod().invoke(entity, args[0]);
					break;
				}
			}
		}
	}
{% endcodeblock %}

#### 最后的最后
最后的数据转换调用的是java.beanutil包的ConvertUtils类，能进行一些基本字段的转换；日期型转换可以说是该功能的难点，这个以后说。
