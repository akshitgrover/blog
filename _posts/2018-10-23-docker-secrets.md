---
layout: post
title: "Docker Secrets in/out"
categories: [docker]
link: https://medium.com/acmvit/docker-secret-in-out-94c66eb4376b
share: true
---

Docker in version 1.13 introduced the concept of secrets. For any application managing configuration variables, ssh keys, passwords, API keys etc. is a crucial part, This data is not be disclosed to anyone outside the authority.

Initially this data was set manually by setting environment variables on the node (which is a part of the swarm), But setting this data on all the nodes turned out to be quite a tedious task, I mean one can also copy a config file but again how many times will you copy the same file on different nodes, But why would anyone copy when you could put all these variables either in your YML stack files or Dockerfiles (NOTE: Anyone can have access to your data when you push your image to docker hub), YML stack files is not a good option as pushing sensitive data on websites like Github (A must to work in a team) is not what is preferred and YML stack files are to be shared for the application to work.
<!--more-->

Continue reading at [ACM VIT medium](https://medium.com/acmvit/docker-secret-in-out-94c66eb4376b)
