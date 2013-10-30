# Frequently Asked Questions

## Haml

### Why is my markup indented properly in development mode, but not in production?
{#q-indentation-in-production}

To improve performance, Haml defaults to {file:HAML_REFERENCE.md#ugly-option "ugly" mode} in Rails
apps running in production.


### How do I put a punctuation mark after an element, like "`I like <strong>cake</strong>!`"?
{#q-punctuation}

Expressing the structure of a document
and expressing inline formatting are two very different problems.
Haml is mostly designed for structure,
so the best way to deal with formatting is to leave it to other languages
that are designed for it.
You could use Textile:

    %p
      :textile
        I like *cake*!

or Markdown:

    %p
      :markdown
        I like **cake**!

or plain old XHTML:

    %p I like <strong>cake</strong>!

If you're inserting something that's generated by a helper, like a link,
then it's even easier:

    %p== I like #{link_to 'chocolate', 'http://franschocolates.com'}!

### How do I stop Haml from indenting the contents of my `pre` and `textarea` tags?
{#q-preserve}

Because Haml automatically indents the HTML source code,
the contents of whitespace-sensitive tags like `pre` and `textarea`
can get screwed up.
The solution is to replace the newlines inside these tags
with HTML newline entities (`&#x000A;`),
which Haml does using the {Haml::Helpers#preserve} and {Haml::Helpers#find_and_preserve} helpers.

Normally, Haml will do this for you automatically
when you're using a tag that needs it
(this can be customized using the {file:HAML_REFERENCE.md#preserve-option `:preserve`} option.
For example,

    %p
      %textarea= "Foo\nBar"

will be compiled to

    <p>
      <textarea>Foo&#x000A;Bar</textarea>
    </p>

However, if a helper is generating the tag,
Haml can't detect that and so you'll have to call {Haml::Helpers#find_and_preserve} yourself.
You can also use `~`, which is the same as `=`
except that it automatically runs `find_and_preserve` on its input.
For example:

    %p= find_and_preserve "<textarea>Foo\nBar</textarea>"

is the same as

    %p~ "<textarea>Foo\nBar</textarea>"

and renders

    <p><textarea>Foo&#x000A;Bar</textarea></p>

### How do I make my long lines of Ruby code look nicer in my Haml document?
{#q-multiline}

Put them in a helper or your model.

Haml purposefully makes it annoying to put lots of Ruby code into your templates,
because lots of code doesn't belong in the view.
If you take that huge `link_to_remote` call
and move it to a `update_sidebar_link` helper,
it'll make your view both easier to read and more semantic.

If you absolutely must put lots of code in your template,
Haml offers a somewhat awkward multiline-continuation tool.
Put a `|` (pipe character) at the end of each line you want to be merged into one
(including the last line!).
For example:

    %p= @this.is(way.too.much). |
        code("and I should").   |
        really_move.it.into(    |
          :a => @helper)        |

Note that sometimes it is valid to include lots of Ruby in a template
when that Ruby is a helper call that passes in a lot of template information.
Thus when a function has lots of arguments,
it's possible to wrap it across multiple lines
as long as each line ends in a comma.
For example:

    = link_to_remote "Add to cart",
        :url => { :action => "add", :id => product.id },
        :update => { :success => "cart", :failure => "error" }

### `form_for` is printing the form tag twice!

Make sure you're calling it with `-`, not `=`.
Just like in ERB, you have to do

    <% form_for stuff do %>
      ...
    <% end %>

in Haml, you have to do

    - form_for stuff do
      ...

### I have Haml installed. Why is Rails (only looking for `.html.erb` files | rendering Haml files as plain text | rendering Haml files as blank pages)?
{#q-blank-page}

There are several reasons these things might be happening.
First of all, make sure that Haml really is installed;
either you've loaded the gem (via `config.gem` in Rails 2.3 or in the Gemfile in Rails 3),
or `vendor/plugins/haml` exists and contains files.
Then try restarting Mongrel or WEBrick or whatever you might be using.

Finally, if none of these work,
chances are you've got some localization plugin like Globalize installed.
Such plugins often don't play nicely with Haml.
Luckily, there's usually an easy fix.
For Globalize, just edit `globalize/lib/globalize/rails/action_view.rb`
and change

    @@re_extension = /\.(rjs|rhtml|rxml)$/

to

    @@re_extension = /\.(rjs|rhtml|rxml|erb|builder|haml)$/

For other plugins, a little searching will probably turn up a way to fix them as well.

## You still haven't answered my question!

Sorry! Try looking at the [Haml](http://haml.info/docs/yardoc/file.REFERENCE.html) reference,
If you can't find an answer there,
feel free to ask in `#haml` on irc.freenode.net
or send an email to the [mailing list](http://groups.google.com/group/haml).