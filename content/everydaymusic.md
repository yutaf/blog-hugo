+++
date = "2016-07-14T11:32:37+09:00"
draft = false
title = "everydaymusic"
tags = ["portfolio"]

+++

<!--more-->

<p style="text-align: center;">Login page</p>
<p style="text-align: center;"><img src="/images/everydaymusic/everydaymusic_01.png" class="image" style="border: 1px solid rgba(0, 0, 0, 0.3);"></p>
　  
<p style="text-align: center;">List page</p>
<p style="text-align: center;"><img src="/images/everydaymusic/everydaymusic_02.png" class="image" style="border: 1px solid rgba(0, 0, 0, 0.3);"></p>
　  
<p style="text-align: center;">Video page</p>
<p style="text-align: center;"><img src="/images/everydaymusic/everydaymusic_03.png" class="image" style="border: 1px solid rgba(0, 0, 0, 0.3);"></p>

## Info

<http://everydaymusic.net/>

#### Released
February 2016

#### Overview  
This web app searches music you might prefer from youtube once a day.  
Then it notifies you of music it found by email.  

#### github

<https://github.com/yutaf/everydaymusic>  
<https://github.com/yutaf/everydaymusic-facebook-service>  

#### Technology stacks

- DigitalOcean (IaaS)
- SendGrid (Mail service)
- Docker Compose
- nginx
- Ruby On Rails 4.2
- php 5.6
- CoffeeScript
- mysql 5.6
- Redis

#### Technical insights

- *How it searches music*  
The app knows users' prefernces from facebook likes or users' manual inputs.  
Then it requests to Youtube search API with the data and find music.  

- *Find music outside user's prefernce list*  
I think finding new music is a great joy.  
This app has a trick to find new music/artists with [Spotify's Related Artists API](https://developer.spotify.com/web-api/get-related-artists/).  
The API helps the app find other artists related to users' prefernce list.  

- *Microsevices*  
I tried microsevices for the first time with this app development.  
It is built with *Ruby On Rails* and *php*.  
*php* is only used for facebook login part.  
The other parts were developed with *Ruby On Rails*.  
It was just because I wanted to develop with official facebook's php sdk.  
Of course we can build facebook login with Rails adequately.
So it's nonsense, I know.  
But I just wanted to try.  
　  
However, Looking back the development process, I didn't get much profits from it.  
I wasted too much time for the jobs made from microsevices architecture.  
　  
A lesson I got is that *microsevices* makes a lot of duplicated works.  
I mean, If I implement model jobs like fetching records from MySql in Rails service, I should implement another same model jobs in php.  
This made me feel like the code I should implement getting twice.  
Besides, there was a context switch when I moved from php to rails and vice versa.  
*Microsevices* might be a good architecture for scaling, though this app apparently doesn't need it.  
Because this app is very small and just started.  

- *[Sidekiq](http://sidekiq.org/)*  
This is a great concurrency job tool offered by ruby gems.  
It really makes concurrency implemention easier.  
I used it for batch jobs sending emails to users.  
It also offers cool management dashbord.  
Awesome!  
