---
layout: post
title: Java 8 函数式编程
categories: Blog
description: Java 8 函数式编程
keywords: java 8, lambda, completableFuture, Function, Consumer
---

# Function Program
+ 抽象方法参数（(), event）,返回值（有值，则return）
+ 函数式接口只包含一个方法，有默认方法：为了解决之前实现多接口问题
+ 函数式接口，可以有Object中的方法，因为可以从Objejct中继承
+ 函数式接口中有静态方法

## 集合

+ 示例：

```
package io.vertx.example.util;

import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class ContainList {

	public static void main(String[] args) {
		Person p1 = new Person();
		p1.setName("䓲撒");
		
		Person p2 = new Person();
		p2.setName("㊙️");
		List<String> nameList = Arrays.asList(p1, p2)
				.stream()
				//这里是Function，传入String，返回String
				.map(Person::getName)   
				.collect(Collectors.toList());
		System.out.println(nameList);
	}

}
class Person {
	private String name;
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
}

```

## 并行处理

## 函数式接口

+ Function,接收一个T类型参数，返回一个R类型的结果（T -> Function -> R）

```
@FunctionalInterface
public interface Function<T, R> {

    R apply(T t);
}
/*接收一个long类型参数，返回一个R类型结果*/
@FunctionalInterface
public interface LongFunction<R> {

    R apply(long value);
}
/*接收两个参数：T、U， 返回一个R类型的结果*/
@FunctionalInterface
public interface BiFunction<T, U, R> {
    R apply(T t, U u);
}
/*接收一个T类型参数，返回long*/
@FunctionalInterface
public interface ToLongFunction<T> {

    long applyAsLong(T value);
}
```

+ Consumer(T -> Consumer)

```
@FunctionalInterface
public interface Consumer<T> {

    void accept(T t);
}
```

+ Predicate(T -> Predicate -> boolean)

```
@FunctionalInterface
public interface Predicate<T> {

    boolean test(T t);
}
```

+ Supplier(Supplier -> T)

```
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```


## jdk四个函数式接口总结

```
package io.vertx.example.util;

import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.function.Supplier;

public class FunctionClient {

	public static void main(String[] args) {
		FunctionClient.sayFunction(s -> {
			return s;
		});
		
		FunctionClient.sayConsumer(s -> {
			System.out.println(s);
		});
		
		FunctionClient.saySupplier(() -> {
			return "测试";
		}); 
		
		FunctionClient.sayPredicate(s -> {
			if(s.equals("测试")) {
				return true;
			} else {
				return false;
			}
		});
	}
	
	public static void sayFunction(Function<String, String> strFunction) {
		String message = strFunction.apply("测试");
		System.out.println(message);
	}
	
	public static void sayConsumer(Consumer<String> strConsumer) {
		strConsumer.accept("测试");
	}
	
	public static void saySupplier(Supplier<String> strSupplier) {
		String str = strSupplier.get();
		System.out.println(str);
	}
	
	public static void sayPredicate(Predicate<String> strPredicate) {
		boolean flag = strPredicate.test("测试");
		System.out.println(flag);
	}

}

```

