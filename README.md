jduan.github.io
===============

# basic info
* blog url: jduan.github.io
* github repo: https://github.com/jduan/jduan.github.io

# how the source code is organized?
* The root of the repo contains the "source code" of the blog
* The \_deploy folder contains the "data" of the blog
* Both of them point to the same github repo. However,
* The root uses the "source" branch
* The \_deploy uses the "master" branch, which is mandated by github

# how to write a post and preview it locally?
* To start a preview, run 'bundle exec rake preview'
* To create a post, run 'bundle exec rake new_post["title of your post"]'
* The web server automatically reloads whenever you save your post

# How to publish to github.io?
* Run 'bundle exec rake deploy'
