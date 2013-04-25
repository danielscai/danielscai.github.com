---
layout: post
title: "python 框架flask介绍 
(python flask web framework intro)"
date: 2013-04-25 12:45
comments: true
categories: coding
---

{% img right /images/main/python-logo.gif 250 150 'openstack' 'openstack' %}

flask action是python轻量级框架flask的一个管理器，

可以帮助我们自动生成代码框架，实现一些管理功能，

理论上来说给予wsgi的python程序都可以管理，不仅限于flask 

<!-- more -->

1.安装flask action

     pip install Flask-Actions

2. 生成一个项目

     flask_admin.py startproject helloproject

3.运行该项目

     ./manager runserver 

4.部署到nginx 
---
4.1 启用 `default_server_actions `
---

    manager = Manager(app,default_server_actions=True)

4.4 运行daemon
---

    python manage.py runfcgi --protocol=fcgi --daemonize --pidfile=/var/run/flaskapp.pid --host=0.0.0.0 --port=8056

4.5 配置nginx 
---

    upstream flaskapp {
       server 0.0.0.0:8056;
       }
    
    server {
    listen 8055;
    server_name  0.0.0.0;
    
    
    location / {
      fastcgi_pass  flaskapp;
      fastcgi_param REQUEST_METHOD    $request_method;
      fastcgi_param QUERY_STRING      $query_string;
      fastcgi_param CONTENT_TYPE      $content_type;
      fastcgi_param CONTENT_LENGTH    $content_length;
      fastcgi_param SERVER_ADDR       $server_addr;
      fastcgi_param SERVER_PORT       $server_port;
      fastcgi_param SERVER_NAME       $server_name;
      fastcgi_param SERVER_PROTOCOL   $server_protocol;
      fastcgi_param PATH_INFO         $fastcgi_script_name;
      fastcgi_param REMOTE_ADDR       $remote_addr;
      fastcgi_param REMOTE_PORT       $remote_port;
      fastcgi_pass_header Authorization;
      fastcgi_intercept_errors off;
    }
    }


5. 编写init.d脚本
---

参考django fastcgi init.d脚本，编写一个启动脚本,将脚本copy到/etc/init.d/flask 
运行update-rc.d flask defaults 添加到开机启动服务

    #! /bin/sh
    ### BEGIN INIT INFO
    # Provides:          FastCGI servers for Django
    # Required-Start:    networking
    # Required-Stop:     networking
    # Default-Start:     2 3 4 5
    # Default-Stop:      S 0 1 6
    # Short-Description: Start FastCGI servers with Django.
    # Description:       Django, in order to operate with FastCGI, must be started
    #                    in a very specific way with manage.py. This must be done
    #                    for each DJango web server that has to run.
    ### END INIT INFO
    #
    # Author:  Guillermo Fernandez Castellanos
    #          <guillermo.fernandez.castellanos AT gmail.com>.
    #
    # Version: @(#)fastcgi 0.1 11-Jan-2007 guillermo.fernandez.castellanos AT gmail.com
    #
    
    #### SERVER SPECIFIC CONFIGURATION
    DJANGO_SITES="zeusapi"
    SITES_PATH=/opt
    RUNFILES_PATH="/var/run"
    HOST=127.0.0.1
    PORT_START=8056
    RUN_AS=www-data
    FCGI_METHOD=threaded
    #### DO NOT CHANGE ANYTHING AFTER THIS LINE!
    
    set -e
    
    #PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
    DESC="Python Flask FastCGI servers"
    NAME=$0
    SCRIPTNAME=/etc/init.d/$NAME
    
    #
    #       Function that starts the daemon/service.
    #
    d_start()
    {
        # Starting all Django FastCGI processes
        PORT=$PORT_START
        for SITE in $DJANGO_SITES
        do
            echo -n " $SITE"
            if [ -f $RUNFILES_PATH/$SITE.pid ]; then
                echo -n " already running"
            else
                start-stop-daemon --start --quiet \
                           --pidfile $RUNFILES_PATH/$SITE.pid \
                           --exec /usr/bin/env -- python \
                           $SITES_PATH/$SITE/manage.py runfcgi \
                           --daemonize \
                           --host=$HOST --port=$PORT --pidfile=$RUNFILES_PATH/$SITE.pid
                chmod 400 $RUNFILES_PATH/$SITE.pid
            fi
            #let "PORT = $PORT + 1"
        done
    }
    
    #
    #       Function that stops the daemon/service.
    #
    d_stop() {
        # Killing all Django FastCGI processes running
        for SITE in $DJANGO_SITES
        do
            echo -n " $SITE"
            start-stop-daemon --stop --quiet --pidfile $RUNFILES_PATH/$SITE.pid \
                              || echo -n " not running"
            if [ -f $RUNFILES_PATH/$SITE.pid ]; then
               rm -f $RUNFILES_PATH/$SITE.pid
            fi
        done
    }
    
    d_status() {
        # Killing all Django FastCGI processes running
        for SITE in $DJANGO_SITES
        do
            echo -n " $SITE"
            start-stop-daemon --status --quiet --pidfile $RUNFILES_PATH/$SITE.pid \
                              && echo -n " is running" \
                              || echo -n " not running"
        done
    }
    
    ACTION="$1"
    case "$ACTION" in
        start)
            echo -n "Starting $DESC: "
            echo
            d_start
            echo "."
            ;;
    
        stop)
            echo -n "Stopping $DESC: "
            echo
            d_stop
            echo "."
            ;;
            
        status)
            echo -n "status $DESC: "
            echo
            d_status
            echo "."
            ;;
    
    
        restart|force-reload)
            echo -n "Restarting $DESC: "
            echo
            d_stop
            sleep 1
            d_start
            echo "."
            ;;
    
        *)
            echo "Usage: $NAME {start|stop|restart|force-reload}" >&2
            exit 3
            ;;
    esac
    
    exit 0

6.启用flask action调试模式
---

flask action中并没有直接提供调试模式，使用起来非常不方便，
开启debug模式需要在`settings.py` 中设置 `DEBUG=True `


参考
[https://code.djangoproject.com/wiki/InitdScriptForLinux](https://code.djangoproject.com/wiki/InitdScriptForLinux)


