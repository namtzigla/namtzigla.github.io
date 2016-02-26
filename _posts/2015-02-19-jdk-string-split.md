---
title: "String.split JDK7 vs JDK8"
date: 2015-02-19 15:54:00
categories: java jdk string regex split
---

I've encounter today a strange behaviour from the most unexpected part of Java, String class.

Consider this code:

{% highlight java %}
import java.util.*;

public class Main {
  public static void main(String args[]) {
    String test = "11111";
    System.out.println(Arrays.toString(test.split("")));
  }
}
{% endhighlight %}



The output that was expected when you compile this with JDK7 is `[, 1, 1, 1, 1, 1]` but the suprise came when I compile it with JDK8 I've got `[1, 1, 1, 1, 1]`. After I research a little bit around I've found an article on [stackoverflow] that explains it. I don't know, but it seems a pretty big change with a possible huge impact.

_99 little bugs in the code_,
_99 little bugs in the code_,
_Take one down, patch it around 117 little bugs in the code ..._ ([credit])

[stackoverflow]: http://stackoverflow.com/questions/22718744/why-does-split-in-java-8-sometimes-remove-empty-strings-at-start-of-result-array
[credit]:        https://twitter.com/irqed/status/358212928404586498
