---
layout: post
title:  "Azure Timeout Issue"
date:   2015-05-16 21:42:52
categories: azure
author: Martin Brusevold Helgesen
---

We recently experienced a strange issue on one of our producs. A program deployed in Azure which communicates with a WCF service hosted in Norway started to get timeouts when requests lasted longer than 4 minutes. After searching the internet we found out that the Azure Load Balancer SILENTLY drops connections after 4 minutes(!). So our program just waited for the callback which never came, and in the end, it got a timeout exception.

I know that 4 minutes is a long time for a WCF requests, but we currently have some long running requests, and there is no alternative to rewrite the architecture for the moment. But we got pointed to a solution by [this blog post by Microsoft](http://azure.microsoft.com/blog/2014/08/14/new-configurable-idle-timeout-for-azure-load-balancer/), which claims that you can manually configure the timeout values on the Load Balancer. We tried just about every alternative in this blog posts, but none of them worked. Microsoft support told us to give our VM a Public Instance IP (PIP), because these IPs was supposed to be routed outside of MS internal load balancer. This did not work either.

So we came over [another older blog post](http://azure.microsoft.com/blog/2014/08/14/new-configurable-idle-timeout-for-azure-load-balancer/), and the solution to our problem was to configure the channel to send KeepAlive packets every thirty seconds.

{% highlight vb.net %}
	ServicePointManager.SetTcpKeepAlive(true, 30000, 30000)
{% endhighlight %}

