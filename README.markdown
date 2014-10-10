## Thinknear's Engineering Blog

This blog runs on [octopress](http://octopress.org/).

# Getting started

Thanks to [this great post](http://www.tomordonez.com/blog/2012/06/04/creating-a-github-blog-using-octopress/).

* clone locally, ruby install (if needed), bundle install, and octopress install
```shell
git clone git@github.com:ThinkNear/thinknear.github.com.git
cd thinknear.github.com
rvm install ruby-2.0.0-p481
bundle install
rake install (-> note, if it asks if a theme is present and to overwrite, say no as rake has already been installed.)
```
* add post (new posts created in: `source/_posts`. [More on directory layout](http://stackoverflow.com/questions/12328828/directory-structure-of-octopress))
```shell
rake new_post["Creating a Github Blog Using Octopress"] 
```
* generate, preview, iterate, commit
```shell
rake generate
rake preview
git add .
git commit -m "Blog post." 
git push origin source
```
* deploy

One time set up:

```shell
rake setup_github_pages
```

Then each time you want to deploy:

```shell
rake deploy
```
