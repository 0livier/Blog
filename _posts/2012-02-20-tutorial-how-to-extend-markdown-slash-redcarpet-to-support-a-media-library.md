---
layout: post
excerpt: first half of this two-part tutorial to modify Redcarpet library in order to support images and files of a media library
title: Extend redcarpet to support a media library 1/2
---

<div class="alert alert-info">
  <i class="icon-info-sign"></i>
  You're reading the first half of this two-part tutorial that explains how to modify Redcarpet library in order to support images and files of a media library. If you're familiar with RoR or Ruby, you can go straight to <a href="http://yet.another.linux-nerd.com/blog/how-to-extend-redcarpet-to-support-a-media-library-part-2">part two</a>.
</div>

If you use the [Markdown](http://en.wikipedia.org/wiki/Markdown) syntax for a
Rails based blog or website, you probably have a media library with
images or files to use in your posts. If Markdown supports external
images but there's no out-of-the box support to tie it to
your media using an id or a logical name. [Redcarpet](https://github.com/tanoku/redcarpet) - a Ruby
library that converts Markdown in HTML - allows overriding the HTML output
functions and I'll present in this tutorial a way to reference your images which will be stored using the [Paperclip](https://github.com/thoughtbot/paperclip) gem.


# Project Initialization 
Let's start by generating a Rails (this tutorial is based on Rails 3.2.1 and bundler 0.9) project, reference needed gems
in `Gemfile` and install them. We're also using the Paperclip-meta gem
to store image width and height.

{% highlight bash %}
rails new blog
cd blog
rm public/index.html
echo 'gem "redcarpet"' >> Gemfile
echo 'gem "paperclip"' >> Gemfile
echo 'gem "paperclip-meta"' >> Gemfile
echo 'gem "therubyracer"' >> Gemfile
bundle install
{% endhighlight %}

## Simple media library ever
Create the `Post` and `Image` thanks to Rails's scaffolding then generate the needed database migrations for Paperclip's images and their meta-data.

{% highlight bash %}
rails g scaffold post title:string body:text published:boolean
rails g scaffold image name:string
rails g paperclip image file
rails g migration AddPaperclipMetaToImage file_meta:text
{% endhighlight %}

Add the line `has_attached_file` to your `app/models/image.rb` as shown below so each uploaded file will get a thumbnail generated automatically :

{% highlight ruby %}
class Image < ActiveRecord::Base
  has_attached_file :file, :styles => { :thumb => "212x141>" }
end
{% endhighlight %}

## Add Markdown support into the views
Since we're talking about generating HTML from views, the most appropriate place to plug Redcarpet is a view helper. As the Markdown converter is reused over and over again, we do not want to instantiate it on every request, so we'll use a class variable to store it. Create the following helper, named `app/helpers/redcarpet_helper.rb`

{% highlight ruby %}
module RedcarpetHelper
  def redcarpet_render(content)
    @@markdown ||= Redcarpet::Markdown.new(Redcarpet::Render::HTML, :autolink => true)
    @@markdown.render(content).html_safe
  end
end
{% endhighlight %}

Adjust the `app/view/posts/show.html.erb` to use the Redcarpet renderer from the helper we created :

{% highlight html %}
<p id="notice"><%= notice %></p>

<p>
  <b>Title:</b>
  <%= @post.title %>
</p>
<p>
  <b>Body:</b>
  <%= redcarpet_render @post.body %>
</p>
<p>
  <b>Published:</b>
  <%= @post.published %>
</p>

<%= link_to 'Edit', edit_post_path(@post) %> |
<%= link_to 'Back', posts_path %>
{% endhighlight %}

Modify `app/views/images/_form.html.erb` to allow images upload :

{% highlight html %}
<%= form_for(@image) do |f| %>
  <% if @image.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@image.errors.count, "error") %> prohibited this image from being saved:</h2>

      <ul>
      <% @image.errors.full_messages.each do |msg| %>
        <li><%= msg %></li>
      <% end %>
      </ul>
    </div>
  <% end %>

  <div class="field">
    <%= f.label :name %><br />
    <%= f.text_field :name %>
  </div>
  <div class="field">
    <%= f.label :file %><br />
    <%= f.file_field :file %>
  </div>
  <div class="actions">
    <%= f.submit %>
  </div>
<% end %>
{% endhighlight %}

Edit `app/views/images/show.html.erb` to add the `image_tag` to display uploaded images :


{% highlight html %}
<p id="notice"><%= notice %></p>

<p>
  <b>Name:</b>
  <%= @image.name %>
</p>

<%= image_tag @image.file.url(:original) %>


<%= link_to 'Edit', edit_image_path(@image) %> |
<%= link_to 'Back', images_path %>
{% endhighlight %}


Now let's run the database migration and run the rails server

{% highlight bash %}
bundle exec rake db:migrate
rails s
{% endhighlight %}

You can now connect to http://localhost:3000/images and
http://localhost:3000/posts to upload images and create content.


In the next part of this tutorial, we'll see how to use uploaded images withing Markdown content.
