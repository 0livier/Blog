Yet Another Linux Nerd

master is for the upstream layout of the blog
wip for work in progress
gh-pages for published version

Deploying :

```
bundler exec jekyll build && git checkout gh-pages && git rebase wip && git checkout - && git push --all
```
or with the t alias in case of tsocks

```
bundler exec jekyll build && git checkout gh-pages && git rebase wip && git checkout - && t git push --all
```

```
bundle install
bundler exec jekyll serve
```


Credits:

    Layout
        n33.co (@n33co dribbble.com/n33)

	Demo Images:
		Unsplash (unsplash.com)

	Icons:
		Font Awesome (fortawesome.github.com/Font-Awesome)

	Other:
		jQuery (jquery.com)
		html5shiv.js (@afarkas @jdalton @jon_neal @rem)
		CSS3 Pie (css3pie.com)
		background-size polyfill (github.com/louisremi)
		skel (getskel.com)
