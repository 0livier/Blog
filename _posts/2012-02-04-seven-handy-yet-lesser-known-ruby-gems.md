If you're into Rails or Ruby development, chances are that you'll already play with popular gems like [Capistrano](https://github.com/capistrano/capistrano) or [Omniauth](https://github.com/intridea/omniauth). Here's seven nice gems that you may not known

## Capistrano Calendar
You probably already use [Capistrano](https://github.com/capistrano/capistrano) for your application deployments. This gem is a addon which posts events to a Google calendar on deployments and rollbacks. [My company](http://creativ-it.net) use a calendar for each project and thanks to our [Continuous Integration](http://en.wikipedia.org/wiki/Continuous_integration) system, [Jenkins](http://jenkins-ci.org/) ([you should use one too](http://martinfowler.com/articles/continuousIntegration.html) too), we also have the information whenever a build is successful or failing, giving us a quick overview of past deployments & build.

[[Capistrano calendar @ github](https://github.com/railsware/capistrano-calendar)]

-----

## Sprite factory
Combined images, also known as [CSS sprites](http://www.alistapart.com/articles/sprites) are a great way to minimize HTTP requests, and thus [speed up web sites](http://developer.yahoo.com/performance/rules.html#opt_images). This technique is effective but its maintenance can quickly turn into a nightmare if you do not have the proper tool to automatize it. The sprite-factory gem help you manage your sprites by automaticly combining them and producing the according CSS. If you're using Rails, add the gem to your Gemfile, create the following file to your lib/tasks directory as assets_resprite, put your sprites in app/assets/images/sprites/ and then call `rake assets:resprite` to obtain your assembled image and resulting CSS. You could also use [Guard](https://github.com/guard/guard) to automaticly update the output image whenever you add or update the sprites.

```ruby
# http://codeincomplete.com/posts/2011/8/6/sprite_factory_1_4_1/
require 'sprite_factory'

namespace :assets do
  desc 'Recreate sprite images and css'
  task :resprite => :environment do
    SpriteFactory.report   = true
    SpriteFactory.selector = '.sprite_'
    SpriteFactory.layout   = :horizontal
    SpriteFactory.library  = :chunkypng                   # use chunkypng as underlying image library
    SpriteFactory.csspath  = "$IMAGE" # embed ERB into css file to be evaluated by asset pipeline
    SpriteFactory.run!('app/assets/images/sprites', :output_style => 'app/assets/stylesheets/sprites.css.erb')
  end

end
```

[[Sprite-factory @ github](https://github.com/jakesgordon/sprite-factory)]

------

## Dropbox-api & Dropbox
Dropbox is a file hosting service, which uses the [cloud](http://en.wikipedia.org/wiki/Cloud_storage) to store them. While I'm not a big fan of cloud based applications ([I am not the only one](http://www.guardian.co.uk/technology/blog/2010/dec/14/chrome-os-richard-stallman-warning)), Dropbox has clients available on several and allows a great proximity with customers: e.g. reports export in their private dropbox directory or automatic file processing whenever they add them in a specific Dropbox directory. The company behind Dropbox also released a gem - called Dropbox -, which syntax differ slightly from Dropbox-api.

[[Dropbox-api @ github](https://github.com/futuresimple/dropbox-api)] - [[Dropbox @ github](https://github.com/RISCfuture/dropbox)]

---------

## Mailcatcher
Mailcatcher is daemon acting as a SMTP server which catches any mails
sent to it, and display them in a web interface like a webmail. It can also
display the HTML or text alternative.

[[Mailcatcher @ github](https://github.com/sj26/mailcatcher)]

------

## Twitter
If you have to develop something on the Internet that is not static HTML, chances are that you'll want to tell about it : new products, new posts or the latest tagged version of your opensource software. With only a few lines to configure it and one more to post a status, the Twitter gem is very easy to use and has access to all the API : direct messages, mentions, saved search, etc.

```ruby
# initialization
Twitter.configure do |config|
  config.consumer_key = YOUR_CONSUMER_KEY
  config.consumer_secret = YOUR_CONSUMER_SECRET
  config.oauth_token = YOUR_OAUTH_TOKEN
  config.oauth_token_secret = YOUR_OAUTH_TOKEN_SECRET
end

Twitter.update "Hi there !"
```

[[Twitter @ github](https://github.com/jnunemaker/twitter)]

------

## Wkhtmltopdf
[Wkhtmltopdf](http://code.google.com/p/wkhtmltopdf/) transforms a HTML page - and all its assets - in a PDF using the webkit renderer. Result are really impressive : see the [rendered pdf](5) for [www.cnn.com](http://www.cnn.com). It is a nice alternative to others systems, notably those - like [prawn](https://github.com/sandal/prawn) - that give an API to manually create PDF (see this [exemple](https://github.com/licensetoil/prawn-invoice-example/blob/master/create_invoice.rb)).

[[Wkhtmltopdf @ github](http://github.com/antialize/wkhtmltopdf)]

-------

## GoogleVisualr
The [Google Chart Tool](http://code.google.com/apis/chart/) service provides javascript that generates client-side [charts](http://code.google.com/apis/chart/interactive/docs/gallery.html). The GoogleVisualr gem allows you to obtain that javascript (with a Rails helper) from simple ruby code. If you prefer to use images rather than a javascript based solution, you may take a look at [Googlecharts gem by Matt Aimonetti](http://googlecharts.rubyforge.org/).

[[google_visualr @ github](https://github.com/winston/google_visualr)] - [Exemples](http://googlevisualr.herokuapp.com/)

---------

Nice icon (colored in red) from [artdesigner.lv](http://artdesigner.lv)
