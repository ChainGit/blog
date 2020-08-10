---
title: Java反射和ReflectionUtils
date: 2016/09/12
categories: 学习
tags: 
- Java
- 学习
- 知识点
---

Java反射和ReflectionUtils
===================
复习了佟老师和毕老师的视频，也阅读了一些写的很棒的博客，整理一下Java中关于反射的笔记。包括一个ReflectionUtils。

## 什么是反射
反射，即Reflect，使得程序在运行时能够像照镜子一样获得自身的一些信息。比如一个类中可以有那些方法，那些字段，而不需要知道源代码。

Java是一门静态语言，即在编译是就确定了类型，换句话说程序在运行时是不能修改自身的。这和Python或者JavaScript不一样，后者是动态语言，可以在程序运行时动态的进行修改。

JavaScript动态特性的例子：
```JavaScript
window.onload = function() {
    var person = new Object();

    person.name = "Jack";

    person.getName = function() {
        return this.name;
    }

    console.log(person.getName());

    person.getName = function() {
        return "Mr." + this.name;
    }

    console.log(person.getName());

    person.getName = undefined;

    console.log(person.getName());
}
```

输出结果：
> jack
> Mr.jack
> Uncaught TypeError: person.getName is not a function

## 反射的作用
一个经常被提到的例子就是JBDC中DriverManager使用Class.forName()来加载JDBC的驱动。
```Java
for (String aDriver : driversList) {
    try {
        println("DriverManager.Initialize: loading " + aDriver);
        Class.forName(aDriver, true,
                ClassLoader.getSystemClassLoader());
    } catch (Exception ex) {
        println("DriverManager.Initialize: load failed: " + ex);
    }
}
```

DriverManager只需要根据预先设好的JDBC驱动类的名字就可以加载并生成驱动实例。比如设置“java.mysql.jdbc.Driver”就可以加载好MySQL数据库的驱动，设置“oracle.jdbc.driver.OracleDriver”就可以加载好Oracle数据库的驱动。

在之后的学习中，也了解到比如Struts、Spring、Hibernate等诸多框架中都是使用了反射。比如在配置文件中写上Bean实例的全名，Spring就可以根据这个全名利用反射自动的生成实例放在IOC容器中。

另一方面，反射能够有效降低类之间的耦合，一个大型的框架或者软件中，需要不断的后期去修改升级。如果修改或增加了功能，每次都重新发布整体新包让用户重新安装也不大方便。为此，可以使用很多方法去将程序模块化，降低耦合。这里举反射的例子，还是上面的JDBC的驱动，比如一开始使用版本是1.0，后来升级到了1.1，只需要替换相应的jar包或者再修改下配置文件就行，而不需要修改其他部分的代码。

此外，使用反射可以生成一些模板方法，避免容易犯低级错误的重复劳动，比如下面两对SQL语句。

```SQL
INSERT INTO USER(ID,NAME,AGE) VALUES (123456,'jack',20);
INSERT INTO DEPARTMENT(ID,NAME) VALUES (11111,'技术部');

SELECT * FROM USER WHERE ID = 123456;
SELECT * FROM DEPARTMENT WHERE ID = 123456;
```

观察发现基本语法写法一致，只是内容不一样而已。使用反射原理，传入User和Department两个对象就可以生成这两种SQL语句（Hibernate的原理之一）。

> 可以使用getMethods()方法获得所有可访问的方法，getFields获得所有可访问的字段，然后再获得这个由方法和字段获得对象中存储的字段值(value)，最终就可以拼凑出SQL来。

## 反射工具类
这个是反射工具类，基于佟老师的讲课视频，加以整理。

包括：
1、获得类中某个名为xxx的方法
2、执行类或对象中某个名为xxx的方法
3、获得类中某个名为yyy的字段(属性)
4、设置对象中名为yyy的字段(属性)的值
5、获得类中某个构造方法
6、通过类中的构造方法和参数创建类实例
7、获得类、方法、字段的某个注解
8、获得某个类后者其父类的泛型参数类型

下面贴上源码，完整的包括测试代码在这里可以[下载](/uploads/javase-note-reflection/JavaSE-Reflect.rar)：

```Java
package com.chain.javase.reflect.utils;

import java.lang.annotation.Annotation;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.ArrayList;
import java.util.List;

/**
 * 反射工具类
 * 
 * @author Chain
 *
 */
public class ReflectionUtils {

	/**
	 * 根据类名（className）获得对应的“类实例”（instance of Class）
	 * 
	 * @param className
	 *            类名
	 * @return 类实例
	 */
	public static Class getClassByName(String className) {
		Class clz = null;
		try {
			clz = Class.forName(className);
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		}
		return clz;
	}

	/**
	 * 根据对象（Object）获得对应的“类实例”（instance of Class）
	 * 
	 * @param obj
	 *            对象
	 * @return 类实例
	 */
	public static Class getClassByObject(Object obj) {
		return obj.getClass();
	}

	public static Method getDeclaredMethod(Object obj, String methodName, Class... parameterTypes) {
		return getDeclaredMethod(getClassByObject(obj), methodName, parameterTypes);
	}

	public static Method getDeclaredMethod(String className, String methodName, Class... parameterTypes) {
		return getDeclaredMethod(getClassByName(className), methodName, parameterTypes);
	}

	/**
	 * 获得类中对应的具有某个参数列表的方法
	 * 
	 * @param clz
	 *            类
	 * @param methodName
	 *            方法名
	 * @param parameterTypes
	 *            方法的参数列表的Class类型
	 * @return
	 */
	public static Method getDeclaredMethod(Class clz, String methodName, Class... parameterTypes) {
		for (Class c = clz; c != Object.class; c = c.getSuperclass()) {
			try {
				Method m = c.getDeclaredMethod(methodName, parameterTypes);
				return m;
			} catch (NoSuchMethodException | SecurityException e) {
				// e.printStackTrace();
				continue;
			}
		}
		return null;
	}

	/**
	 * 忽略或者设置方法或者属性的修饰符（“暴力访问”）
	 * 
	 * @param obj
	 * @param bool
	 */
	private static void setAccessible(Object obj, boolean bool) {
		if (obj instanceof Method)
			((Method) obj).setAccessible(bool);
		else if (obj instanceof Field)
			((Field) obj).setAccessible(bool);
	}

	/**
	 * 根据类名获得类的实例（需要类具有无参构造函数）
	 * 
	 * @param className
	 * @return
	 */
	public static Object getObjectByName(String className) {
		return getObjectByClass(getClassByName(className));
	}

	/**
	 * 根据“类实例”获得类的实例（需要类具有无参构造函数）
	 * 
	 * @param clz
	 * @return
	 */
	public static Object getObjectByClass(Class clz) {
		Object obj = null;
		try {
			obj = clz.newInstance();
		} catch (InstantiationException | IllegalAccessException e) {
			e.printStackTrace();
		}
		return obj;
	}

	public static Object invokeMethod(String classToInvoke, Method m, Object... methodParams) {
		Object obj = getObjectByName(classToInvoke);
		return invokeMethod(obj, m, methodParams);
	}

	public static Object invokeMethod(Class classToInvoke, Method m, Object... methodParams) {
		Object obj = getObjectByClass(classToInvoke);
		return invokeMethod(obj, m, methodParams);
	}

	/**
	 * 执行对象的某个方法
	 * 
	 * @param objToInvoke
	 * @param m
	 * @param methodParams
	 * @return
	 */
	public static Object invokeMethod(Object objToInvoke, Method m, Object... methodParams) {
		Object result = null;
		try {
			setAccessible(m, true);
			result = m.invoke(objToInvoke, methodParams);
		} catch (IllegalAccessException | IllegalArgumentException | InvocationTargetException e) {
			// e.printStackTrace();
		}
		return result;
	}

	public static Field getDeclaredField(Object obj, String fieldName) {
		return getDeclaredField(getClassByObject(obj), fieldName);
	}

	public static Field getDeclaredField(String clzName, String fieldName) {
		return getDeclaredField(getClassByName(clzName), fieldName);
	}

	/**
	 * 获得类中的某个属性（字段）
	 * 
	 * @param clz
	 * @param fieldName
	 * @return
	 */
	public static Field getDeclaredField(Class clz, String fieldName) {
		for (Class c = clz; c != Object.class; c = c.getSuperclass()) {
			try {
				Field f = c.getDeclaredField(fieldName);
				return f;
			} catch (SecurityException | NoSuchFieldException e) {

				// e.printStackTrace();
				continue;
			}
		}
		return null;
	}

	public static Object setField(String clzName, Field f, Object value) {
		return setField(getObjectByName(clzName), f, value);
	}

	public static Object setField(Class clz, Field f, Object value) {
		return setField(getObjectByClass(clz), f, value);
	}

	/**
	 * 设置对象中某个属性的值
	 * 
	 * @param obj
	 * @param f
	 * @param value
	 * @return
	 */
	public static Object setField(Object obj, Field f, Object value) {
		try {
			setAccessible(f, true);
			f.set(obj, value);
		} catch (IllegalArgumentException | IllegalAccessException e) {

			e.printStackTrace();
		}
		return obj;
	}

	public static Constructor getConstructor(String clzName, Class... parameterTypes) {
		return getConstructor(getClassByName(clzName), parameterTypes);
	}

	public static Constructor getConstructor(Object obj, Class... parameterTypes) {
		return getConstructor(getClassByObject(obj), parameterTypes);
	}

	/**
	 * 获得类的某个构造器
	 * 
	 * @param clz
	 * @param parameterTypes
	 * @return
	 */
	public static Constructor getConstructor(Class clz, Class... parameterTypes) {
		for (Class c = clz; c != Object.class; c = c.getSuperclass()) {
			try {
				Constructor cs = c.getConstructor(parameterTypes);
				return cs;
			} catch (SecurityException | NoSuchMethodException e) {
				// e.printStackTrace();
				continue;
			}
		}
		return null;
	}

	/**
	 * 根据构造器创建类实例
	 * 
	 * @param cs
	 * @param initArgs
	 * @return
	 */
	public static Object newInstanceByConstrutor(Constructor cs, Object... initArgs) {
		Object obj = null;
		try {
			obj = cs.newInstance(initArgs);
		} catch (InstantiationException | IllegalAccessException | IllegalArgumentException
				| InvocationTargetException e) {
			e.printStackTrace();
		}
		return obj;
	}

	/**
	 * 获得加在类上的注解
	 * 
	 * @param clz
	 * @param annotationClass
	 * @return
	 */
	public static Annotation getDeclaredAnnotation(Class clz, Class annotationClass) {
		Annotation an = clz.getDeclaredAnnotation(annotationClass);
		return an;
	}

	/**
	 * 获得加在属性上的注解
	 * 
	 * @param f
	 * @param annotationClass
	 * @return
	 */
	public static Annotation getDeclaredAnnotation(Field f, Class annotationClass) {
		Annotation an = f.getDeclaredAnnotation(annotationClass);
		return an;
	}

	/**
	 * 获得加在方法上的注解
	 * 
	 * @param m
	 * @param annotationClass
	 * @return
	 */
	public static Annotation getDeclaredAnnotation(Method m, Class annotationClass) {
		Annotation an = m.getDeclaredAnnotation(annotationClass);
		return an;
	}

	/**
	 * 获得加在构造器上的注解
	 * 
	 * @param c
	 * @param annotationClass
	 * @return
	 */
	public static Annotation getDeclaredAnnotation(Constructor c, Class annotationClass) {
		Annotation an = c.getDeclaredAnnotation(annotationClass);
		return an;
	}

	/**
	 * 获得类（或父类）的泛型参数类型中的某一个
	 * 
	 * @param clz
	 * @param index
	 * @return
	 */
	public static Class getSuperGenericType(Class clz, int index) {
		List<Class> lst = getSuperGenericTypes(clz);
		if (lst.size() - 1 < index)
			return Object.class;
		return lst.get(index);
	}

	/**
	 * 
	 * 获得类（或父类）的泛型参数类型的列表
	 * 
	 * @param clz
	 * @return
	 */
	public static List<Class> getSuperGenericTypes(Class clz) {
		List<Class> lst = new ArrayList<>();

		if (clz == null) {
			lst.add(Object.class);
			return lst;
		}

		Type genericType = clz.getGenericSuperclass();

		if (!(genericType instanceof ParameterizedType)) {
			lst.add(Object.class);
			return lst;
		}

		Type[] params = ((ParameterizedType) genericType).getActualTypeArguments();

		for (int i = 0; i < params.length; i++)
			if (params[i] instanceof Class)
				lst.add((Class) params[i]);
			else
				lst.add(Object.class);

		return lst;
	}

}
```

> 编程就像练功一样，这只是一个简单的工具类。像反射，注解，泛型等Java中哪怕是最基本的知识点，也需要不断的去阅读书籍和源码，才能体会到它们的更牛的用法。
