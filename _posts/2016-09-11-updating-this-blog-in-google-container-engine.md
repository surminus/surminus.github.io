---
layout: post
title: Updating this blog in Google Container Engine
---

## Boil the kettle and make a cup of tea

One thing that really fascinates me in new tech is [Kubernetes](http://kubernetes.io/).
Unfortunately it seems unlikely that we'll be using in my current role anytime soon,
so I decided to simultaneously have a play and add some content at the same time; and
when I say have a play, I mean use [Google Container Engine](https://cloud.google.com/container-engine/).

My problem is I always try to do too much and get overwhelmed, but this time
I made myself keep it simple.

I really like the Google `gcloud` toolkit: it's straight forward and documentation
is excellent. This site was originally running on a VM in [Google Compute](https://cloud.google.com/compute/),
and the experience for bringing up a basic VM was as it should be: quick auth, a couple of commands,
and an easy way to get data back for interaction. Super.

The first thing to do was create an image of the site using Docker. Since I'm
using [Jekyll](https://jekyllrb.com/), this was trivial. I [created a `Dockerfile`](https://github.com/surminus/blog/blob/master/Dockerfile),
with `startup.sh` containing [some binding info](https://github.com/surminus/blog/blob/master/startup.sh).

I built the image with the tag [documented by Google Container Registry](https://cloud.google.com/container-registry/docs/pushing). For the purposes
here I'll just refer to it as `$TAG`:

{% highlight bash %}
docker build -t $TAG .
{% endhighlight %}

Run it to make sure it's OK...

{% highlight bash %}
docker run $TAG
{% endhighlight %}

...and push it to the registry:

{% highlight bash %}
gcloud docker push $TAG
{% endhighlight %}

Next we need to make a cluster to run our ["pods"](http://kubernetes.io/docs/user-guide/pods/):
[`gcloud container clusters create blog`](https://cloud.google.com/sdk/gcloud/reference/container/clusters/create)

This can take a little while, probably just enough time to drink the tea you made right at the beginning. When that's done,
and you feel sufficiently warmed by the tea, it's time to push our image into the cluster:

{% highlight bash %}
kubectl run blog --image=$TAG --port=4000
{% endhighlight %}

Before we get to see the service, we need to expose it a load balancer:

{% highlight bash %}
kubectl expose deployment blog --port=80 --target-port=4000 --type="LoadBalancer"
{% endhighlight %}

Note the port forwarding; it's running on tcp/4000 in the container, so we need to tell
the load balancer to forward traffic to it appropriately.

Grab the external IP of the deployment using:
{% highlight bash %}
➜  blog git:(master) ✗ kubectl get service blog
NAME      CLUSTER-IP     EXTERNAL-IP      PORT(S)   AGE
blog      10.3.252.135   23.251.133.125   80/TCP    1m
{% endhighlight %}

It'll probably be in `<pending>` for perhaps a minute or two.

You can see some fuller information of this deployment:
{% highlight bash %}
➜  blog git:(master) ✗ kubectl describe service blog
Name:                   blog
Namespace:              default
Labels:                 run=blog
Selector:               run=blog
Type:                   LoadBalancer
IP:                     10.3.252.135
LoadBalancer Ingress:   23.251.133.125
Port:                   <unset> 80/TCP
NodePort:               <unset> 32368/TCP
Endpoints:              10.0.0.4:4000
Session Affinity:       None
Events:
  FirstSeen     LastSeen        Count   From                    SubobjectPath   Type            Reason                  Message
  ---------     --------        -----   ----                    -------------   --------        ------                  -------
  45m           45m             1       {service-controller }                   Normal          CreatingLoadBalancer    Creating load balancer
  44m           44m             1       {service-controller }                   Normal          CreatedLoadBalancer     Created load balancer
{% endhighlight %}

## Updating my DNS

So now I have something running in Google Container Engine, and it was all pretty
straight forward. Unfortunately I still need to update the DNS, but fortunately
that's also hosted at Google.

Let's find out the current state:

{% highlight bash %}
gcloud dns record-sets list -z $ZONENAME
{% endhighlight %}

To add/edit/remove a record, you must start a transaction:

{% highlight bash %}
gcloud dns record-sets transaction start -z $ZONENAME
{% endhighlight %}

I already had a record set up for this, so I had to remove it, which is exactly the
same command as below, except replacing `remove` for `add`. The following adds my new
record;

{% highlight bash %}
gcloud dns record-sets transaction add 23.251.133.125 --name blog.surminus.co.uk. --ttl 300 --type A -z $ZONENAME
{% endhighlight %}

Then execute the change, and view the changes:

{% highlight bash %}
gcloud dns record-sets transaction execute -z $ZONENAME
gcloud dns record-sets list -z $ZONENAME
{% endhighlight %}

*I know setting the TTL makes me a rebel without a cause. Suck it up.*

## That's it!

That's it!

If you're reading this then it must have worked, and I actually have some content on my blog. Goodness.


