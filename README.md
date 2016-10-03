## blog

This is my blog using [Jekyll](https://jekyllrb.com/).

## Deployment
1. `bundle install`
2. `bundle exec jekyll build`
3. `export TAG=<tag name>`
4. `docker build -t $TAG .`
5. `gcloud docker push $TAG`

Then use kubectl to edit the deployment or roll the upgrade.

[I used the Cayman Blog theme](http://adueck.github.io/cayman-blog/)
