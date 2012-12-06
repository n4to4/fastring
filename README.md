# Fastring

**Fastring** is a string formatting library for Scala.
`Fastring` can also be used as a template engine,
and it is an excellent replacement of JSP or [FreeMarker](http://freemarker.sourceforge.net/).

## It's simple to use

`Fastring` use [string interpolation](http://docs.scala-lang.org/sips/pending/string-interpolation.html) syntax.
For example, if you are writing a CGI page:

    import com.dongxiguo.fastring.Fastring.Implicits._
    def printHtml(link: java.net.URL) {
      val fastHtml = fast"<html><body><a href='$link'>Click Me!</a></body></html>"
      print(fastHtml)
    }

## It's extremely fast

I made a [benchmark](https://github.com/Atry/fastring/blob/master/benchmark/src/main/scala/com/dongxiguo/fastring/benchmark/FastringBenchmark.scala).
I use 4 different ways to create a 545-length string.

1. Fastring (`fast"Concat with $something"` syntax);
2. Simple string concatenation (`s"Concat with $something"` syntax);
3. Handwritten `StringBuilder` (`stringBuilder ++= "Build from " ++= something` syntax);
4. `java.util.Formatter` (`f"Format with $something"` syntax).

This is the result from my Intel i5-3450 computer:

<table>
<tr>
<th>
Fastring
</th>
<td>
<pre><code>def fast(a: Int) =
  fast"head ${
    (for (j &lt;- 0 until 10 view) yield {
      fast"baz$j $a foo ${
        (for (i &lt;- 0 until 4 view) yield {
          fast"$a i=$i"
        }).mkFastring(",")
      } bar\n"
    }).mkFastring("&lt;hr/&gt;")
  } tail"

fast(0).toString</code></pre>
</td>
<td>
Took 669 nanoseconds to generate a 545-length string.<br/>(Simple and fast)
</td>
</tr>
<tr>
<th>
Simple string concatenation
</th>
<td>
<pre><code>def s(a: Int) =
  s"head ${
    (for (j &lt;- 0 until 10 view) yield {
      s"baz$j $a foo ${
        (for (i &lt;- 0 until 4 view) yield {
          s"$a i=$i"
        }).mkString(",")
      } bar\n"
    }).mkString("&lt;hr/&gt;")
  } tail"

s(0)</code></pre>
</td>
<td>
Took 1738 nanoseconds to generate a 545-length string.<br/>(Simple but slow)
</td>
</tr>
<tr>
<th>
Handwritten <code>StringBuilder</code>
</th>
<td>
<pre><code>def sb(sb: StringBuilder, a: Int) = {
  sb ++= "head "
  var first = true
  for (j &lt;- 0 until 10 view) {
    if (first) {
      first = false
    } else {
      sb ++= ""&lt;hr/&gt;""
    }
    sb ++= "baz" ++= j.toString ++= " " ++= a.toString ++= " foo ";
    {
      var first = true
      for (i &lt;- 0 until 4 view) {
        if (first) {
          first = false
        } else {
          sb ++= ","
        }
        sb ++= a.toString
        sb ++= " i="
        sb ++= i.toString
      }
    }
    sb ++= " bar\n"
  }
  sb ++= " tail"
  sb
}

val s = new StringBuilder
sb(s, 0)
s.toString</code></pre>
</td>
<td>
Took 537 nanoseconds to generate a 545-length string.<br/>(Fast but too trivial)
</td>
</tr>
<tr>
<th>
<code>java.util.Formatter</code>
</th>
<td>
<pre><code>def f(a: Int) =
    f"head ${
      (for (j &lt;- 0 until 10 view) yield {
        f"baz$j $a foo ${
          (for (i &lt;- 0 until 4 view) yield {
            f"$a i=$i"
          }).mkString(",")
        } bar\n"
      }).mkString("&lt;hr/&gt;")
    } tail"

f(0)</code></pre>
</td>
<td>
Took 7436 nanoseconds to generate a 545-length string.<br/>(Simple but extremely slow)
</td>
</tr>
</table>

## You lazy bone

`Fastring` is a `Traversable[String]`, and it is lazily evaluated.

For example, in the previous benchmark for `Fastring`, the most of time is spend on invoking `toString`.
You can avoid these overhead if you do not need a whole string. For example:

    // Faster than: print(fast"My lazy string from $something")
    fast"My lazy string from $something".foreach(print)

## Utilities

There is a `mkFastring` method for all Scala collections:

    // Enable mkFastring method
    import com.dongxiguo.fastring.Fastring.Implicits._
    
    // Got Fastring("Hello, world")
    Seq("Hello", "world").mkFastring(", ")

And a `filled` method for `Byte`, `Short`, `Int` and `Long`:

    // Enable filled method
    import com.dongxiguo.fastring.Fastring.Implicits._
    
    // Got Fastring("  123")
    123.filled(5)
    
    // Got Fastring("00123")
    123.filled(5, '0')

## Installation

Put these lines in your `build.sbt`:

    libraryDependencies += "com.dongxiguo" %% "fastring" % "0.2.0"
    
    scalaVersion := "2.10.0-RC3"

Note that `Fastring` requires [Scala](http://www.scala-lang.org/) `2.10.x` and [Sbt](http://www.scala-sbt.org/) `0.12.x`.
