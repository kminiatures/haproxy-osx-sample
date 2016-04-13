Mac で HAProxy のテスト

以下で動くとおもう。

```
git clone https://github.com/kminiatures/haproxy-osx-sample
cd haproxy-osx-sample
brew install haproxy
sh ./start_web1.sh
sh ./start_web2.sh
sh ./start_haproxy.sh
```

# HAProxy

## Install

```bash
brew install haproxy
```

## `vi conf/haproxy.cfg`


```bash
# vim: ft=ruby
global
  log 127.0.0.1 local0
  log 127.0.0.1 local1 notice
  pidfile ~/tmp/haproxy-queue.pid
  stats socket /tmp/stats.socket uid 105 gid 105

defaults
  log global
  mode http
  option httplog
  option forwardfor # Backend へのリクエストに "X-Forwarded-For" を追加する
  option httpchk HEAD / HTTP/1.0 # Health Check
  option http-server-close
  timeout connect 5000
  timeout client 10000
  timeout server 10000

frontend http-farm
  bind *:80
  use_backend app1

backend app1
  server app1_a localhost:9001
  server app1_b localhost:9002

# ステータス画面
listen hastats
  bind *:8088
  mode http
  stats enable
  stats show-legends
  stats uri /
  # IDとパスワード
  stats auth admin:adminadmin
```

##Start Proxy

```
sudo haproxy -f haproxy.cfg
```


# WEBRick

以下のコマンドで、9001, 9002 ポートでWEBサーバが立ちあがる。

##サーバの開始

### Terminal 1
```
ruby -rwebrick -e 'WEBrick::HTTPServer.new(DocumentRoot: "./", Port: 9001).start'
```

### Terminal 2
```
ruby -rwebrick -e 'WEBrick::HTTPServer.new(DocumentRoot: "./", Port: 9002).start'
```

WEBRick の終了

```bash
ps aux | grep webrick | grep -v grep | awk '{print $2}' | xargs kill
```

# URL
ステータス http://localhost:8088

backend1 http://localhost/

backend2 http://localhost/


#参考

HAProxy

http://nepalonrails.tumblr.com/post/9674428224/setup-haproxy-for-development-environment-on-mac

WEBRick

http://qiita.com/sudahiroshi/items/e74d61d939f18779970d


