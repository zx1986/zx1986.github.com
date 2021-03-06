---
layout: post
title: "Nginx"
date: 2012-10-11 11:04
comments: true
categories: ["Web Server"]
---

    # Passenger
    server {
      listen 8080;
      server_name localhost;
      root /Users/laas/proged/rack_test/public;
      passenger_enabled on;
      rack_env production;
      passenger_min_instances 4;
    }
 
    # Unicorn
    upstream unicorn_server {
      server unix:/Users/laas/proged/rack_test/tmp/unicorn.sock fail_timeout=0;
    }
 
    server {
      listen 8081;
      server_name localhost;
      root /Users/laas/proged/rack_test/public;
 
      location / {
        proxy_pass http://unicorn_server;
      }
    }
 
    # Thin
    upstream thin_server{
      server unix:/Users/laas/proged/rack_test/tmp/thin.0.sock fail_timeout=0;
      server unix:/Users/laas/proged/rack_test/tmp/thin.1.sock fail_timeout=0;
      server unix:/Users/laas/proged/rack_test/tmp/thin.2.sock fail_timeout=0;
      server unix:/Users/laas/proged/rack_test/tmp/thin.3.sock fail_timeout=0;
    }
 
    server {
      listen 8082;
      server_name localhost;
      root /Users/laas/proged/rack_test/public;
 
      location / {
        proxy_pass http://thin_server;
      }
    }
