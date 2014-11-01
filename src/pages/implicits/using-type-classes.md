---
layout: page
title: Using Type Classes
---

We have seen how to define type classes. In this section we'll see some conveniences for using them: **context bounds** and the **implicitly** method.

## Context Bounds

When we use type classes we often end up requiring implicit parameters that we pass onward to a type class handler. For example, using our `HtmlWriter` example we might want to define some kind of page template that accepts content rendered by a writer.

~~~ scala
def pageTemplate[A](body: A)(implicit writer: HtmlWriter[A]): String = {
  val renderedBody = body.toHtml

  s"<html><head>...</head><body>${renderedBody}</body></html>"
}
~~~

We don't actually use the implicit `writer` in our code, but we need it available to we can call the `toHtml` enrichment.

Context bounds allow us to write this more compactly, with a notation that is reminiscent of a type bound.

~~~ scala
def pageTemplate[A : HtmlWriter](body: A): String = {
  val renderedBody = body.toHtml

  s"<html><head>...</head><body>${renderedBody}</body></html>"
}
~~~

The context bound is the notation `[A : HtmlWriter]` and it expands into the equivalent implicit parameter list in the prior example.

<div class="callout callout-info">
#### Context Bound Syntax

A context bound is an annotation on a generic type variable like so:

~~~ scala
[A : Context]
~~~

It expands into a generic type parameter `[A]` along with an implicit parameter for a `Context[A]`.
</div>

## Implicitly

Context bounds give us a short-hand syntax for declaring implicit parameters, but since we don't have an explicit name for the parameter we cannot use it in our methods. Normally we use context bounds when we don't need access to the implicit parameter, but rather just pass it on to some other method expecting an implicit. However if we do need access for some reason, we can use the `implicitly` method.

~~~ scala
scala> case class Example(name: String)
defined class Example

scala> implicit val implicitExample = Example("implicit")
implicitExample: Example = Example(implicit)

scala> implicitly[Example]
res4: Example = Example(implicit)

scala> implicitly[Example] == implicitExample
res5: Boolean = true
~~~

The `implicitly` method takes no parameters but has a generic type parameters. It returns the implicit matching the given type, assuming there is no ambiguity.