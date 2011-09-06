Nice Java HashMap Antipattern
=============================

Don't you just love how simple it is to instantiate maps in Java? Like,
could it be any uglier than

~~~ {.java}
Map<String, String> map = new HashMap<String, String>();
map.put("fruit", "banana");
map.put("vegetable", "carrot");
~~~

I saw someone using a pattern that made it look a bit nicer, like

~~~ {.java}
Map<String, String> map = new HashMap<String, String>() {
  put("fruit", "banana");
  put("vegetable", "carrot");
}
~~~

It kinda puts a bit more structure to this mess. However, you'll get
some unwanted runtime side-effects.

When you use this "syntax", you'll actually create a new anonymous inner
class. Each anonymous inner class instace will have a runtime reference
to the `this` object for the context where you define the anonymous
inner class in. Thus, if your this happens to have references to 100
megabytes of porn, that stuff will not be GDd as long as this
Map or yours is alive. Also, should you try to serialize your map for
sending or storing it somewhere, you'll also be sending your 100
megabytes of porn.

Enjoy your cup of Java!
