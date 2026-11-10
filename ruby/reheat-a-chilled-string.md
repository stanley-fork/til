# Reheat A Chilled String

In Ruby 3.4 and later, we no longer have to use the `# frozen_string_literal: true` magic comment at the beginning of all our files. That's because when that
comment is missing, all string literals are treated as _chilled_. They aren't
quite frozen, but now we will see warnings when code tries to mutate those
chilled strings. This is an intermediate step towards strings being frozen by
default.

Occasionally we want a string to be mutable so that the logic of our program can
make changes to it. To avoid a warning or outright error, we need to _reheat_
the string.

This can be done with [the `+` unary operator on the `String`
class](https://docs.ruby-lang.org/en/master/String.html#method-i-2B-40).

> Returns `self` if `self` is not frozen and can be mutated without warning
> issuance. Otherwise returns `self.dup`, which is not frozen.

Let's start an `irb` session (Ruby 4.0.9) with deprecation warnings on.

```ruby
❯ irb -W:deprecated

⢀⡴⠊⢉⡟⢿  IRB v1.18.0 - Ruby 4.0.9
⣎⣀⣴⡋⡟⣻  "show_doc method" to view documentation
⣟⣼⣱⣽⣟⣾  ~/dev/ccsalespro/ccstorage

irb(main):001> str = ""
=> ""
irb(main):002> str.frozen?
=> false
irb(main):003> str << "are you chill?"
(irb):3: warning: literal string will be frozen in the future (run with --debug-frozen-string-literal for more information)
=> "are you chill?"
irb(main):004> str << "??"
=> "are you chill???"
irb(main):005> str2 = +""
=> ""
irb(main):006> str2 << "warmed up!"
=> "warmed up!"
```

In the above example, we see that a given string literal is not _frozen_, but it
is _chilled_ which means that when we first try to mutate it, we get a warning.
Once it has been mutated, it is no longer _chilled_ and we don't get subsequent
warnings for that object.

I can avoid the warning all together (with `str2`) with the `+` unary operator.

Here is an error I kept seeing in my test suite related to this issue:

```
/Users/lastword/.local/share/mise/installs/ruby/4.0.9/lib/ruby/gems/4.0.0/gems/faraday-net_http-3.4.2/lib/faraday/adapter/net_http.rb:194:
warning: literal string will be frozen in the future (run with --debug-frozen-string-literal for more information)
```

This is because a test in the suite was declaring a string literal (`"abc"`)
that some under-the-hood Faraday code was mutating. The most straightforward fix
was to reheat that string when it was declared (`+"abc"`).
