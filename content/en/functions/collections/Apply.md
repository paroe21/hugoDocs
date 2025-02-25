---
title: collections.Apply
linkTitle: apply
description: Given an array or slice, `apply` returns a new slice with a function applied over it.
categories: [functions]
keywords: []
menu:
  docs:
    parent: functions
function:
  aliases: [apply]
  returnType: any
  signatures: ['collections.Apply COLLECTION FUNCTION [PARAM...]']
relatedFunctions:
  - collections.Apply
  - collections.Delimit
  - collections.In
  - collections.Reverse
  - collections.Seq
  - collections.Slice
aliases: [/functions/apply]
---

`apply` expects at least three arguments, depending on the function being applied.

1. The first argument is the sequence to operate on.
2. The second argument is the name of the function as a string, which must be the name of a valid [template function].
3. After that, the arguments to the applied function are provided, with the string `"."` standing in for each element of the sequence the function is to be applied against.

Here is an example of a content file with `names:` as a front matter field:

{{< code-toggle file="content/example.md" fm=true copy=false >}}
title: Example
names: [ "Derek Perkins", "Joe Bergevin", "Tanner Linsley" ]
{{< /code-toggle >}}

You can then use `apply` as follows:

```go-html-template
{{ apply .Params.names "urlize" "." }}
```

Which will result in the following:

```
"derek-perkins", "joe-bergevin", "tanner-linsley"
```

This is *roughly* equivalent to using the following with [`range`]:

```go-html-template
{{ range .Params.names }}{{ . | urlize }}{{ end }}
```

However, it is not possible to provide the output of a range to the [`delimit`]function, so you need to `apply` it.

If you have `post-tag-list.html` and `post-tag-link.html` as [partials], you *could* use the following snippets, respectively:

{{< code file="layouts/partials/post-tag-list.html" copy=false >}}
{{ with .Params.tags }}
  <div class="tags-list">
    Tags:
    {{ $len := len . }}
    {{ if eq $len 1 }}
      {{ partial "post-tag-link.html" (index . 0) }}
    {{ else }}
      {{ $last := sub $len 1 }}
      {{ range first $last . }}
        {{ partial "post-tag-link.html" . }},
      {{ end }}
      {{ partial "post-tag-link.html" (index . $last) }}
    {{ end }}
  </div>
{{ end }}
{{< /code >}}

{{< code file="layouts/partials/post-tag-link.html" copy=false >}}
<a class="post-tag post-tag-{{ . | urlize }}" href="/tags/{{ . | urlize }}">{{ . }}</a>
{{< /code >}}

This works, but the complexity of `post-tag-list.html` is fairly high. The Hugo template needs to perform special behavior for the case where there’s only one tag, and it has to treat the last tag as special. Additionally, the tag list will be rendered something like `Tags: tag1 , tag2 , tag3` because of the way that the HTML is generated and then interpreted by a browser.

This first version of `layouts/partials/post-tag-list.html` separates all of the operations for ease of reading. The combined and DRYer version is shown next:

```go-html-template
{{ with .Params.tags }}
  <div class="tags-list">
    Tags:
    {{ $sort := sort . }}
    {{ $links := apply $sort "partial" "post-tag-link.html" "." }}
    {{ $clean := apply $links "chomp" "." }}
    {{ delimit $clean ", " }}
  </div>
{{ end }}
```

Now in the completed version, you can sort the tags, convert the tags to links with `layouts/partials/post-tag-link.html`, [`chomp`] stray newlines, and join the tags together in a delimited list for presentation. Here is an even DRYer version of the preceding example:

{{< code file="layouts/partials/post-tag-list.html" >}}
{{ with .Params.tags }}
  <div class="tags-list">
    Tags:
    {{ delimit (apply (apply (sort .) "partial" "post-tag-link.html" ".") "chomp" ".") ", " }}
  </div>
{{ end }}
{{< /code >}}

{{% note %}}
`apply` does not work when receiving the sequence as an argument through a pipeline.
{{% /note %}}

[`chomp`]: /functions/strings/chomp/
[`delimit`]: /functions/collections/delimit/
[template function]: /functions/
[`range`]: /functions/go-template/range/
