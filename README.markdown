## Thinknear's Engineering Blog

This blog runs on [octopress](http://octopress.org/).

# Getting started

* clone locally, ruby install (if needed), bundle install, and octopress install
```shell
git clone git@github.com:ThinkNear/thinknear.github.com.git
cd thinknear.github.com
rvm install ruby-2.0.0-p481
bundle install
rake install
```
* add post (new posts created in: `_source/_posts`)
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
```shell
rake deploy
```
