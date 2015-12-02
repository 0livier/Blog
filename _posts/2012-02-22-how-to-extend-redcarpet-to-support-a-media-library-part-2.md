---
layout: post
excerpt: second half of this two-part tutorial thatexplains how to modify Redcarpet library in order to support images and files of a media library
---

<div class="alert alert-info">
  <i class="icon-info-sign"></i>
  You're reading the second half of this two-part tutorial that
  explains how to modify Redcarpet library in order to support images
  and files of a media library. You may look at <a href="/blog/tutorial-how-to-extend-markdown-slash-redcarpet-to-support-a-media-library">part one</a> if you want
  to setup a simple environment to implement the changes I'll present
  here.
</div>

In the previous part, we've setup a sandbox rails application that
will allow you to upload images or write content.

## Markdown syntax for images and links 
The Markdown link and image syntax are similar : 

```
[Here is a link.](http://linux-nerd.com)
```

will produce

```
<a href="http://linux-nerd.com">Here is a link.</a>
```

and

```html
![my avatar](http://lol.cat/1p)
```

will output

```html
<img src="http://lol.cat/1p" alt="my avatar"/>
```

That syntax works for external resources and we need to find a way to specify our images while keeping that syntax working as  still

## Defining the URL pattern to reference Paperclip file, size and class attribute 
Editors will use most of the time an absolute URL or a relative one,
but the use of a filename without leading slash or protocol is
unlikely (especially if we're using a classic Rails route but it's
still possible), so we'll base our notation on that assumption : if
the filename is only constituted by letters or numbers and
punctuation, we'll look in database for a match. If none is found,
we'll use the string for the URL, unmodified :

```markdown
![A wonderful lolcat identified by the id 3](3)
[Click here to download report referenced by its filename](export.pdf)
```

We'll enhance the integration with Paperclip by adding the size. To differentiate the file id from the size parameter, we'll separate them by a pipe :

```markdown
![A wonderful lolcat identified by the id 3](3|thumb)
[Click here to download report referenced by its filename, the original size is implied](export.pdf)
```

It's often nice to use HTML class attribute for images within content and we'll continue to use a pipe separator.

```markdown
![A wonderful lolcat identified by the id 3](3|thumb|right illustration)
```

We could write the following regexp : the first captured text will be the id, the optional second capture the size and the last optional capture will be HTML classes.

```ruby
/^([-\w\d\.]+)(?:\|(\w+))?(?:\|([-\w\s\d]+))?$/
```

## Adding support into Redcarpet

Our helper will use a new class, that extends the default Redcarpet renderer and override the methods for image and links.

```ruby
module RedcarpetHelper
  def redcarpet_render(content)
    @@markdown ||= Redcarpet::Markdown.new(HTMLBlockCode, :fenced_code_blocks => true)
    @@markdown.render(s)
  end
end

class HTMLBlockCode < Redcarpet::Render::HTML
  include Sprockets::Helpers::RailsHelper
  include Sprockets::Helpers::IsolatedHelper
  include ActionView::Helpers::UrlHelper

  def parse_media_link(link)
    puts link
    matches = link.match(/^([\w\d\.]+)(?:\|(\w+))?(?:\|([\w\s\d]+))?$/)
    {
        :id => matches[1],
        :size => (matches[2] || 'original').to_sym,
        :class => matches[3]

    } if matches
  end

  def image(link, title, alt_text)
    size = nil
    klass = nil

    if nil != (parse = parse_media_link(link))
      media = Media.find_by_id(parse[:id]) || Media.find_by_name(parse[:id])
      if media
        size = media.file.image_size(parse[:size])
        link = media.file.url(parse[:size])
        klass = parse[:class]
      end
    end

    image_tag(link, :size => size, :title => title, :alt => alt_text, :class => klass)
  end

  def link(link, title, content)
    klass = nil

    if nil != (parse = parse_media_link(link))
      media = Media.find_by_id(parse[:id]) || Media.find_by_name(parse[:id])
      if media
        link = media.file.url(parse[:size])
        klass = parse[:class]
      end
    end

    link_to(content, link, :title => title, :class => klass)
  end
end
```

# Possible enhancements
## Add support for other resources
In our Blog example and its PostController, you may want to link other posts or another model you have, but with the default Redcarpet renderer the only way to do it is to use a absolute URL : something like /controler/action/id. It's not a perennial way as URLs / routes may change. One way to address that problem is to extend or replace the pattern we defined earlier. A possible notation could be :

```markdown
[See my model](model_name:id)
```

## Add support for your own markup
Wordpress has a handy feature, called shortcode, that allow editor to add content, external or internal - e.g. include a tweet using its id, embed a video from Youtube or a Google Documents Spreadsheet - with simple syntax like 

```
[tweet 168468146263560192] 
```

Such functionality could easily be added to Redcarpet, using the `preprocess` function.

```ruby
  def preprocess(full_document)
    full_document.gsub!(/\[tweet (\d+)\]/) {|m| Tweet.to_html(m[1]) }
    full_document
  end
```
