---
title: httpie人性化curl工具使用详解
date: 2016-08-15 13:36:19
categories:
- Python模块学习
tags:
- Python
- httpie
---

## httpie简介
httpie是一个python写的类curl的命令行工具，跨平台，支持python2和3，友好的高亮显示以及其他的特性，基于`Requests`和`Pygments`库编写

![](https://raw.githubusercontent.com/jkbrzt/httpie/master/httpie.png)

<!-- more -->

### 主要特性

- Expressive and intuitive syntax
- Formatted and colorized terminal output
- Built-in JSON support
- Forms and file uploads
- HTTPS, proxies, and authentication
- Arbitrary request data
- Custom headers
- Persistent sessions
- Wget-like downloads
- Python 2.6, 2.7 and 3.x support
- Linux, Mac OS X and Windows support
- Plugins
- Documentation
- Test coverage

## 基本使用

### 语法

```powershell
http [--json] [--form] [--pretty {all,colors,format,none}]
     [--style STYLE] [--print WHAT] [--headers] [--body] [--verbose]
     [--all] [--history-print WHAT] [--stream] [--output FILE]
     [--download] [--continue]
     [--session SESSION_NAME_OR_PATH | --session-read-only SESSION_NAME_OR_PATH]
     [--auth USER[:PASS]] [--auth-type {basic,digest}]
     [--proxy PROTOCOL:PROXY_URL] [--follow]
     [--max-redirects MAX_REDIRECTS] [--timeout SECONDS]
     [--check-status] [--verify VERIFY]
     [--ssl {ssl2.3,ssl3,tls1,tls1.1,tls1.2}] [--cert CERT]
     [--cert-key CERT_KEY] [--ignore-stdin] [--help] [--version]
     [--traceback] [--default-scheme DEFAULT_SCHEME] [--debug]
     [METHOD] URL [REQUEST_ITEM [REQUEST_ITEM ...]]
```

简写就是：

```powershell
$ http [flags] [METHOD] URL [ITEM [ITEM]]
```

## METHOD

如果不带METHOD参数，这默认为GET(没有附带请求参数)或POST(附带请求参数,默认以json格式传输)

```powershell
$ http example.org               # => GET
$ http example.org hello=world   # => POST
```

## URL
默认协议为`http://`,如果主机是`localhost`,还可以如下简写：

```powershell
$ http :3000                    # => http://localhost:3000
$ http :/foo                    # => http://localhost/foo
```

另外可以使用`param==value`语法像url添加参数，所产生的效果就是浏览器中通过`&`连接的参数，注意区分POST方法所使用的`param=value`语法

```powershell
$ http www.google.com search=='HTTPie logo' tbm==isch

# 
```

linux系统中可以通过

```powershell
$ alias https='http --default-scheme=https'
```

来创建更方便https的命令

## Request items

|Item Type|Description|
|:---:|:---:|
|HTTP Headers `Name:Value`|Arbitrary HTTP header, e.g. `X-API-Token:123`|
|URL parameters `name==value`|Appends the given name/value pair as a query string parameter to the URL. The `==` separator is used.|
|Data Fields `field=value`, `field=@file.txt`|Request data fields to be serialized as a JSON object (default), or to be form-encoded (`--form, -f`).|
|Raw JSON fields `field:=json`, `field:=@file.json`|Useful when sending JSON and one or more fields need to be a `Boolean`, `Number`, nested `Object`, or an `Array`, e.g., `meals:='["ham","spam"]'` or `pies:=[1,2,3]` (note the quotes).|
|Form File Fields `field@/dir/file`|Only available with `--form, -f`. For example `screenshot@~/Pictures/img.png`. The presence of a file field results in a `multipart/form-data` request.|

## JSON

`param=value`格式的参数全部会转换成json格式传输，并且value全是字符串

```powershell
$ http PUT example.org name=John email=john@example.org
```

```powershell
PUT / HTTP/1.1
Accept: application/json, */*
Accept-Encoding: gzip, deflate
Content-Type: application/json
Host: example.org

{
    "name": "John",
    "email": "john@example.org"
}
```

Non-string fields use the `:=` separator, which allows you to embed raw JSON into the resulting object. Text and raw JSON files can also be embedded into fields using `=@` and `:=@`:

```powershell
$ http PUT api.example.com/person/1 \
    name=John \
    age:=29 married:=false hobbies:='["http", "pies"]' \  # Raw JSON
    description=@about-john.txt \   # Embed text file
    bookmarks:=@bookmarks.json      # Embed JSON file
```

```powershell
PUT /person/1 HTTP/1.1
Accept: application/json, */*
Content-Type: application/json
Host: api.example.com

{
    "age": 29,
    "hobbies": [
        "http",
        "pies"
    ],
    "description": "John is a nice guy who likes pies.",
    "married": false,
    "name": "John",
    "bookmarks": {
        "HTTPie": "http://httpie.org",
    }
}
```

## Forms
Submitting forms is very similar to sending JSON requests. Often the only difference is in adding the `--form, -f` option, which ensures that data fields are serialized as, and `Content-Type` is set to, `application/x-www-form-urlencoded; charset=utf-8`.

```powershell
$ http --form POST api.example.org/person/1 name='John Smith' \
    email=john@example.org cv=@~/Documents/cv.txt
```

```powershell
POST /person/1 HTTP/1.1
Content-Type: application/x-www-form-urlencoded; charset=utf-8

name=John+Smith&email=john%40example.org&cv=John's+CV+...
```

### File upload forms

```powershell
$ http -f POST example.com/jobs name='John Smith' cv@~/Documents/cv.pdf
```

上面的效果和下面一样：

```powershell
<form enctype="multipart/form-data" method="post" action="http://example.com/jobs">
    <input type="text" name="name" />
    <input type="file" name="cv" />
</form>
```

## HTTP headers

```powershell
$ http example.org  User-Agent:Bacon/1.0  'Cookie:valued-visitor=yes;foo=bar'  \
    X-Foo:Bar  Referer:http://httpie.org/
```

```powershell
GET / HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Cookie: valued-visitor=yes;foo=bar
Host: example.org
Referer: http://httpie.org/
User-Agent: Bacon/1.0
X-Foo: Bar
```

There are a couple of default headers that HTTPie sets:

```powershell
GET / HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
User-Agent: HTTPie/<version>
Host: <taken-from-URL>
```

## Authentication
Basic auth:

```powershell
$ http -a username:password example.org
```

Digest auth:

```powershell
$ http -A digest -a username:password example.org
```

## HTTP redirects
By default, HTTP redirects are not followed and only the first response is shown. To instruct HTTPie to follow the `Location` header of `30x` responses and show the final response instead, use the `--follow, -F` option.

If you additionally wish to see the intermediary requests/responses, then use the `--all` option as well.

To change the default limit of maximum 30 redirects, use the `--max-redirects=<limit>` option.

```powershell
$ http --follow --all --max-redirects=5 httpbin.org/redirect/3
```

## Proxies

```powershell
$ http --proxy=http:http://user:pass@10.10.1.10:3128 --proxy=https:https://10.10.1.10:1080 example.org
```

## SOCKS

```powershell
$ http --proxy=http:socks5://user:pass@host:port --proxy=https:socks5://user:pass@host:port example.org
```

## HTTPS
可以通过`--verify=no`忽略证书检查

```powershell
$ http --verify=no https://example.org
```

可以使用`--ssl=<PROTOCOL>`制定ssl版本

```powershell
$ http --ssl=ssl3 https://vulnerable.example.org
```

## Output options

|Param|Description|
|:---:|:---:|
|`--headers, -h`|Only the response headers are printed.|
|`--body, -b`|Only the response body is printed.|
|`--verbose, -v`|Print the whole HTTP exchange (request and response). This option also enables `--all` (see bellow).|
|`--print, -p`|Selects parts of the HTTP exchange.|

|Character|Stands for|
|:---:|:---:|
|`H`|request headers|
|`B`|request body|
|`h`|response headers|
|`b`|response body|

```powershell
$ http --print=Hh PUT httpbin.org/put hello=world
```

## Redirected output
Download a file:

```powershell
$ http example.org/Movie.mov > Movie.mov
```

## Download mode
HTTPie features a download mode in which it acts similarly to `wget`.

When enabled using the `--download, -d` flag, response headers are printed to the terminal (`stderr`), and a progress bar is shown while the response body is being saved to a file.

```powershell
$ http --download https://github.com/jkbrzt/httpie/archive/master.tar.gz
```

```powershell
HTTP/1.1 200 OK
Content-Disposition: attachment; filename=httpie-master.tar.gz
Content-Length: 257336
Content-Type: application/x-gzip

Downloading 251.30 kB to "httpie-master.tar.gz"
Done. 251.30 kB in 2.73862s (91.76 kB/s)
```

## Sessions
By default, every request is completely independent of any previous ones. HTTPie also supports persistent sessions, where custom headers (except for the ones starting with `Content-` or `If-`), authorization, and cookies (manually specified or sent by the server) persist between requests to the same host.

### Named sessions
Create a new session named `user1` for `example.org`:

```powershell
$ http --session=user1 -a user1:password example.org X-Foo:Bar
```

Now you can refer to the session by its name, and the previously used authorization and HTTP headers will automatically be set:

```powershell
$ http --session=user1 example.org
```

## 参考文档

- [httpie官方文档](https://github.com/jkbrzt/httpie)