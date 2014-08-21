## Thinknear's Engineering Blog

This blog runs on [octopress](http://octopress.org/).

# Getting started

* clone locally
```shell
git clone git@github.com:ThinkNear/thinknear.github.com.git
```
* bundle 
  * you may need to install the right ruby. currently:
```shell
bundle install
```
```shell
rvm install ruby-2.0.0-p481
```
* VERY IMPORTANT: development is done on branch `source`, not `master`
```shell
git co source
```
* add post
```shell
rake new_post["Creating a Github Blog Using Octopress"]
```
  * this goes to `_source/_posts`
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
