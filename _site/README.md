## blog

This is my blog using [Jekyll](https://jekyllrb.com/).

[I used the Cayman Blog theme](http://adueck.github.io/cayman-blog/)

## Deployment

Deployed to Kubernetes:

```
export TAG=<tag>
gcloud docker push $TAG
kubectl run blog --image=$TAG --port=4000
```

Check service IP:

```
kubectl get service blog
```
