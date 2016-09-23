---
layout: post
title: go get on your domain
categories: [blog,hacking]
share: true
comments: false
excerpt: Serve `go get` content from your own domain
tags: [hacking,golang]
author: maze
image:
  feature: /posts/hacking.jpg
date: 2016-09-22T12:22:00+01:00
---
I'm currently migrating my [Github](https://github.com/tehmaze) projects to the
excellent self-hosted [Gogs][Gogs] service at [https://git.maze.io/][Git].

Playing with the excellent [gopkg.in][gopkg.in] service, I figured it
would be awesome to use **maze.io** as the primary name space for the packages
I'm building. As a start I setup a `go` organiation, [https://git.maze.io/go][Git go].

At first I played with [go-import-redirector][go-import-redirector] which
serves as a simple metadata provider for the ``go get`` command. Its usage is
fairly straight forward:

{% highlight bash %}
go-import-redirector [-addr address] [-vcs vcs] [-parts parts] <import> <repo>
{% endhighlight %}

So, in order to serve up my [go organisation's repository][Git go] on the
landing page of the **maze.io** domain, I can simply run:

{% highlight bash %}
go-import-redirector -addr 127.0.0.1:3000 / https://git.maze.io/go
{% endhighlight %}

To make this work in conjunction with the other content hosted on my site, we
use the following piece of nginx configuration:

{% highlight nginx %}
upstream go-import-redirector {
  server 127.0.0.1:3000;
}

server {
  server_name maze.io

  # ... other definitions ...

  location / {
    proxy_set_header Host $host;
    if ($args ~ "go-get") {
      proxy_pass http://go-import-redirector;
    }

    # ... default location configuration ...
  }
}
{% endhighlight %}

Let's test ``go get``:

{% highlight bash %}
% go get -v maze.io/phi
Fetching https://maze.io/phi?go-get=1
Parsing meta tags from https://maze.io/phi?go-get=1 (status code 200)
get "maze.io/phi": found meta tag main.metaImport{Prefix:"maze.io/phi", VCS:"git", RepoRoot:"https://git.maze.io/go/phi"} at https://maze.io/phi?go-get=1
maze.io/phi (download)
maze.io/phi/header
maze.io/phi
{% endhighlight %}

It worked! And as a bonus, the [supervisord][supervisord] program definition:

{% highlight ini %}
[program:go-import-redirector]
command         = go-import-redirector -addr 127.0.0.1:3000 / https://git.maze.io/go
directory       = /tmp
priority        = 100
startretries    = 99999999
stdout_logfile  = /var/log/go-import-redirector.log
redirect_stderr = true
{% endhighlight %}

[Gogs]: https://github.com/gogs
[gopkg.in]: http://gopkg.in/
[Git]: https://git.maze.io/
[Git go]: https://git.maze.io/go
[go-import-redirector]: https://github.com/noonien/go-import-redirector
[supervisord]: http://supervisord.org/
