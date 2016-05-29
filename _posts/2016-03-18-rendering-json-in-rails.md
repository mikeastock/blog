---
layout: post
title: Rendering JSON in Rails
---

While rendering trying to render JSON in Rails today I discovered that Rails
does not play nicely with strings/text that is stored in UTF-8. The basis of
this problem lies in Rails modifying the behavior of `String#to_json`. In my
specific case Rails was escaping my '&' to '\u0026'.

After some Googling I decided to dive into Rails source code and look deeper
into the `#to_json` monkey patch. I was rewarded for my searching.

```ruby
# activesupport/lib/active_support/json/encoding.rb
class EscapedString < String #:nodoc:
  def to_json(*)
    if Encoding.escape_html_entities_in_json
      super.gsub ESCAPE_REGEX_WITH_HTML_ENTITIES, ESCAPED_CHARS
    else
      super.gsub ESCAPE_REGEX_WITHOUT_HTML_ENTITIES, ESCAPED_CHARS
    end
  end
end
```

The next step was figuring out what `ESCAPE_REGEX_WITH_HTML_ENTITIES` and
`ESCAPE_REGEX_WITHOUT_HTML_ENTITIES` represent

```ruby
ESCAPE\_REGEX\_WITH\_HTML\_ENTITIES = `/[\u2028\u2029><&]/u`
ESCAPE\_REGEX\_WITHOUT\_HTML\_ENTITIES = `/[\u2028\u2029]/u` (removes the regex matchers for `'<', '>', and '&'`)
```

The solution is to set `Encoding.escape_html_entities_in_json` to false.
This can simply be done as an option in the render method.

```ruby
render json: post, options: { escape_html_entities_in_json: false }
```

I couldn't find any documentation of this option anywhere and so I hope that
if you are trying to render UTF-8 strings in JSON and don't want them escaped
you stumble upon this post!
