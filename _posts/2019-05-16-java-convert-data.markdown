---
title: "Java 형 변환"
date: 2019-05-12
tags: Java
---

int to String
```java
String str = Integer.tostring(i);
```

String to int
```java
int i = Integer.parseInt(str);
int i = Integer.valueOf(str).intValue();
```

doulble to String
```java
String str = Double.toString(d);
```

long to String
```java
String str = Long.toString(l);
```

float to String
```java
String str = Float.toString(f);
```

String to double
```java
double d = Double.valueOf(str).doubleValue();
```

String to long
```java
long l = Long.valueOf(str).longValue();
long l = Long.parseLong(str);
```

String to float
```java
float f = Float.valueOf(str).floatValue();
```

decimal to binary
```java
String binstr = Integer.toBinaryString(i);
```

decimal to hexadecimal
```java
String hexstr = Integer.toString(i, 16);
String hexstr = Integer.toHexString(i);
Integer.toHexString( 0x10000 | i).substring(1).toUpperCase());
```

hexadecimal(String) to int
```java
int i = Integer.valueOf("B8DA3", 16).intValue();
int i = Integer.parseInt("B8DA3", 16);
```

ASCII Code to String
```java
String char = new Character((char)i).toString();
```

Integer to ASCII Code
```java
int i = (int) c;
```

Integer to boolean
```java
boolean b = (i != 0);
```

boolean to Integer
```java
int i = (b)? 1 : 0;
```