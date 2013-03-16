---
layout: post
title: "Spring Test整合JUnit 4"
date: 2013-03-13 23:03
comments: true
categories: J2EE应用
tags: J2EE Spring JUnit
---
> JUnit 框架原本就能用来进行单元测试,但是在使用了Spring之后测试就变得复杂了,但是幸运的是Spring提供了[Spring-test](http://chenming142.github.com 'spring-test.jar'),可以用来整合JUnit,使测试变得简单.

###加入依赖包
使用Spring整合JUnit来测试需要加入以下依赖包:  
1. [JUnit 4](http://www.junit.org/ "JUnit 4 必不可少")  
2. Spring Test(Spring 框架中的test包)  
3. Spring 相关的其他依赖包及其配置  

###创建测试源目录和包结构
推荐创建一个和src平级的源test目录,因为src中的内都是以后需要打包的应用类,而test中的类仅仅只用来测试.test目录下的包名称和结构也建议与src中的保持一致,这样既不会产生冲突也容易识别,方便检索.   
![目录及包结构](/images/common/2013-03-13-spring-test/20130314000238.jpg '目录及包结构')

###创建测试框架抽象类
Spring的测试机制是基于JUnit的扩展,在org.springframework.test包下,可以看到6个从 TestCase基础上扩展出来的抽象类,分别是:  
1. __ConditionalTestCase__ - 可以有选择地关闭掉一些测试方法,不让它们在测试用例中执行,而无需将这些方法注释掉  
2. __AbstractSpringContextTests__ - 运行多个测试用例和测试方法时,Spirng上下文只需创建一次  
3. __AbstractSingleSpringContextTests__ - 方便手工执行Spring配置文件,手工设定Spring容器是否需要重新加载  
4. __AbstractDependencyInjectSpringContextTests__ - 自动装配\依赖检查\自动注入  
5. __AbstractTransactionalSpringContextTests__ - 自动恢复数据库现场即自动回滚  
6. __AbstractTransactionalDataSourceSpringContextTests__ - 通过JDBC访问数据库,检查数据库操作正确性  

上述抽象类按照先后顺序逐步加强了每个抽象类的功能，并且按照逐步继承的关系，使得子抽象类具有父抽象类的所有特性，因此最终的AbstractTransactionalDataSourceSpringContextTests抽象类具有其所有祖先抽象类的特性以及其自身的特性，实际应用中可以根据需要选择需要使用的抽象基类进行扩展.

<!-- more -->

__基于AbstractDependencyInjectionSpringContextTests的抽象测试类__
	package com.test.common;

	import org.springframework.test.AbstractDependencyInjectionSpringContextTests;

	/**
	 * 
	 * 基于AbstractDependencyInjectionSpringContextTests的抽象测试类
	 * 具体测试时，继承此抽象类，完成对Service的测试。此时可以不用关注事务相关的问题。
	 * 
	 * 注意：在实际测试时，发现如果在Spring配置文件中指定了如下所示的Spring使用的资源文件列表
	 *		<bean id="propertyConfigure"
	 *			class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
	 *			<property name="locations">
	 *				<list>
	 *					<value>file:WebContent/WEB-INF/db.properties</value>
	 *				</list>
	 *			</property>
	 *		</bean>
	 *则上文中的[file:] 必须加上，否则系统会抛一个FileNotFoundException，而实际运行时，则必须将此处去掉。
	 * @author Administrator
	 */
	public abstract class AbstractServiceInjectionTest 
		extends AbstractDependencyInjectionSpringContextTests {
		// 配置文件地址的前缀
		String filePathSufix = "file:WebContent/WEB-INF/springConfig/"; 
	    // applicationContext.xml文件地址
		String appContextFile = "file:WebContent/WEB-INF/springConfig/applicationContext.xml";
		
		public AbstractServiceInjectionTest() {
			super();
		}

		public AbstractServiceInjectionTest(String name) {
			super(name);
		}
		/**
		 * 抽象方法 - 子类必须实现的获取需要配置的文件地址
		 * @return
		 */
		abstract String[] getOtherConfigs();
		
		/**
		 * 覆盖的获取配置文件地址的方法
		 * @return
		 * @author Administrator
		 */
		@Override
		protected String[] getConfigLocations() {
			String[] otherConfigs = getOtherConfigs();
			//所有配置文件列表
			String[] configFiles = new String[otherConfigs.length + 1]; 
			configFiles[0] = appContextFile;
			System.arraycopy(otherConfigs, 0, configFiles, 1, otherConfigs.length);
			return configFiles;
		}
		
		/**
		 * 抽象方法 - 子类必须实现的需忽略的方法列表
		 * @return
		 */
		abstract String[] getIgnoredMethods();

		@Override
		protected boolean isDisabledInThisEnvironment(String testMethodName) {
			for(String methodName : getIgnoredMethods()){
				if(methodName.equals(testMethodName)){
					return true;
				}
			}
			return false;
		}
		
	}

__基于AbstractTransactionalSpringContextTests扩展的抽象类 - 可以自动回滚事务的测试抽象基类__
	package com.test.common;

	import org.springframework.test.AbstractTransactionalSpringContextTests;

	/**
	 * AbstractTransactionalSpringContextTests提供了一套数据库自动回滚的功能，方便恢复数据库现场，从而不必每次测试完成后都重新恢复数据库的操作。
	 * 基于AbstractTransactionalSpringContextTests扩展的抽象类 - 可以自动回滚事务的测试抽象基类
	 * 
	 * 如果在测试时，需要将测试结果持久化，可以直接在测试的方法末尾加上“setComplete()”方法。
	 * 由此，可以看出基于AbstractTransactionalSpringContextTests抽象基类的扩展，同样可以将数据持久化到数据库中，
	 * 因此，实际测试中，可以使用此方式实现不启用自动事务回滚功能。
	 * 
	 * 说明：如果使用Spring的事务管理功能，此处在事务处理上可能会有一些冲突，例如将以“query”开头的方法的propagation设置为“NEVER”，
	 * 测试时可能会造成事务自动回滚或者不回滚的异常，
	 * 此时，需要做的操作是将以“query”开头的方法的propagation设置为默认的“REQUIRED”，同时加上read-only="true"，可以解决此问题。
	 * 
	 * 另外，AbstractTransactionalSpringContextTests提供了一些可以在事务处理过程中进行操作的各种入口。
	 * 	onSetUpBeforeTransaction()与onTearDownAfterTransaction()：子类可以覆盖这两个方法，可以在事务测试方法运行的前后执行一些数据库初始化的操作并在事务完成后将其清除；
	 * 	onSetUpInTransaction()与onTearDownInTransaction()：这对方法和上面介绍的方法完成的功能相同，只不过它们在测试方法的相同事务中执行的。
	 * 	AbstractTransactionalSpringContextTests还提供了一组用于测试延迟数据加载的方法：
	 * 		endTransaction()与startNewTransaction()。
	 * 	当用户在测试Hibernate、JPA等允许延迟数据加载的应用时，模拟数据在Service层事务中被部分加载，当传递到Web层时重新打开事务完成延迟部分数据加载的测试场景。
	 * 	可以在测试方法中显式调用endTransaction()方法以模拟从Service层中获取部分数据后返回，
	 * 	然后，通过startNewTransaction()开启一个和原事务无关新事务——模拟在Web层中重新打开事务，接下来就可以访问延迟加载的数据，进行测试了。
	 */
	public abstract class AbstractServiceTransactionalTest 
		extends AbstractTransactionalSpringContextTests {
		// 配置文件地址的前缀,如果直接配置在WEB-INF目录下的话,可不用前缀
		String filePathSufix = "file:WebContent/WEB-INF/springConfig/"; 
		// applicationContext.xml文件地址,如果直接配置在WEB-INF目录下的话,直接写比如spring/jdbc-config.xml
		String appContextFile = "file:WebContent/WEB-INF/springConfig/applicationContext.xml";
		
		public AbstractServiceTransactionalTest() {
			super();
		}
		public AbstractServiceTransactionalTest(String name) {
			super(name);
		}
		
		/**
		 * 抽象方法 - 子类必须实现的获取需要配置的文件地址
		 * @return
		 */
		abstract String[] getOtherConfigs();
		
		/**
		 * 覆盖的获取配置文件地址的方法
		 * @return
		 * @author Administrator
		 */
		@Override
		protected String[] getConfigLocations() {
			String[] otherConfigs = getOtherConfigs();
			//所有配置文件列表
			String[] configFiles = new String[otherConfigs.length + 1]; 
			configFiles[0] = appContextFile;
			System.arraycopy(otherConfigs, 0, configFiles, 1, otherConfigs.length);
			return configFiles;
		}
		
		/**
		 * 抽象方法 - 子类必须实现的需忽略的方法列表
		 * @return
		 */
		abstract String[] getIgnoredMethods();

		@Override
		protected boolean isDisabledInThisEnvironment(String testMethodName) {
			for(String methodName : getIgnoredMethods()){
				if(methodName.equals(testMethodName)){
					return true;
				}
			}
			return false;
		}
	}

__基于AbstractTransactionalDataSourceSpringContextTests抽象类具有其所有祖先抽象类的特性以及其自身的特性__
	package com.test.common;

	import org.springframework.test.AbstractTransactionalDataSourceSpringContextTests;
	/**
	 * 使用JdbcTemplate获取数据库数据的方式，多用于更新以及复杂的业务逻辑中的数据验证。
	 * 有了Spring的测试机制，对于使用Spring的应用，可以方便的进行集成测试，从而简化开发的工作量，减少繁琐的重复工作，提供工作效率。
	 * 
	 * 注意：如果采用byName的自动装配机制，数据源Bean的名称必须取名为“dataSource”。
	 * 
	 */
	public abstract class AbstractJdbcServiceTest extends AbstractTransactionalDataSourceSpringContextTests {
		String filePathSufix = "file:WebContent/WEB-INF/springConfig/"; // 配置文件地址的前缀
		String appContextFile = "file:WebContent/WEB-INF/springConfig/applicationContext.xml";// applicationContext.xml文件地址

		public AbstractJdbcServiceTest() {
			super();
		}

		public AbstractJdbcServiceTest(String name) {
			super(name);
		}
		/**
		 * 抽象方法 - 子类必须实现的获取需要配置的文件地址
		 * @return
		 */
		abstract String[] getOtherConfigs();
		
		/**
		 * 覆盖的获取配置文件地址的方法
		 * @return
		 * @author Administrator
		 */
		@Override
		protected String[] getConfigLocations() {
			String[] otherConfigs = getOtherConfigs();
			//所有配置文件列表
			String[] configFiles = new String[otherConfigs.length + 1]; 
			configFiles[0] = appContextFile;
			System.arraycopy(otherConfigs, 0, configFiles, 1, otherConfigs.length);
			return configFiles;
		}
		
		/**
		 * 抽象方法 - 子类必须实现的需忽略的方法列表
		 * @return
		 */
		abstract String[] getIgnoredMethods();

		@Override
		protected boolean isDisabledInThisEnvironment(String testMethodName) {
			for(String methodName : getIgnoredMethods()){
				if(methodName.equals(testMethodName)){
					return true;
				}
			}
			return false;
		}
	}

__直接使用JUnit进行测试的抽象基类__
	package com.test.common;

	import org.springframework.context.ApplicationContext;
	import org.springframework.context.support.FileSystemXmlApplicationContext;

	import junit.framework.TestCase;

	/**
	 * 直接使用JUnit进行测试的抽象基类
	 * @author Administrator
	 *
	 */
	public abstract class AbstractServiceTestJUnit extends TestCase {
		/**
		 * applicationContext.xml文件的路径,
		 * 如果配置文件没有放在classpath下，必须使用此写法；
		 * 如果配置文件放在classpath下，则可以直接文件名
		 */
		String applicationContextFile = "WebContent\\WEB-INF\\springConfig\\applicationContext.xml";
		
		static ApplicationContext ctx;

		public AbstractServiceTestJUnit() {
			super();
		}

		public AbstractServiceTestJUnit(String name) {
			super(name);
		}
		
		/**
		 * 抽象方法 - applicationContext.xml文件以外的配置文件的路径列表
		 * @return
		 */
		abstract String[] getOtherConfigFiles();
		
		/**
		 * 获取 Spring 上下文实例
		 * 上下文实例简单使用 singleton,保证上下文只创建一次
		 * @return
		 */
		ApplicationContext getApplicationContext() {
			if( null == ctx){
				String[] otherConfigs = getOtherConfigFiles();
				String[] configFiles = new String[otherConfigs.length + 1]; // 所有配置文件列表
				
				configFiles[0] = applicationContextFile;
				System.arraycopy(otherConfigs, 0, configFiles, 1,otherConfigs.length);// 将所有的配置文件列表放入目标配置文件列表
				
				ctx = new FileSystemXmlApplicationContext(configFiles); // 获取Spring上下文
			}
			return ctx;
		}
		
		/**
		 * 覆盖 setUp 方法，获取Spring上下文实例
		 */
		@Override
		protected void setUp() throws Exception {
			getApplicationContext();
		}
	}
###创建、配置测试类及创建测试方法
上述所说的均为测试基类(抽象类),如创建测试类时根据实际情况继承上述某一抽象类即可.  
在抽象类中已配置了数据库连接配置文件appContextFile.xml,在实际测试类中,仅配置所需用到的Bean配置Spring配置文件.   
测试方法自行添加,并可在方法getIgnoredMethods()中设置是否忽略,而无需注释.

###通过JUnit 4 执行测试
右键方法名，选择则“Run As” or “Debug As” → “JUnit Test”即可
