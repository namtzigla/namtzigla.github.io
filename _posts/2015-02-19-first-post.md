---
layout: post
title: "String.split JDK7 vs JDK8"
date: 2015-02-19 15:54:00
categories: java, jdk, string, split
---

I've encounter today a strange behaviour from the most unexpected part of Java, String class.

Consider this code:

~~~ java
import java.util.*;

public class Main {
  public static void main(String args[]) {
    String test = "11111";
    System.out.println(Arrays.toString(test.split("")));
  }
}
~~~

The output that was expected when you compile this with JDK7 is `[, 1, 1, 1, 1, 1]` but when I compiled with JDK8 I've got `[1, 1, 1, 1, 1]`. After I've research a little bit around I've found an article on [stackoverflow] that explains it.

[stackoverflow]: http://stackoverflow.com/questions/22718744/why-does-split-in-java-8-sometimes-remove-empty-strings-at-start-of-result-array
