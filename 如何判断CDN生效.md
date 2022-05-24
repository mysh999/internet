在Linux系统中执行以下命令，获取对应加速域名资源的response，查看是否有CDN加速的对应节点信息，从而判断CDN是否生效





```bash
# curl -I https://assets.robot.com/upgrade-pro/upgrade/2022/04/1649992035901/full_X100-ota-1649923355.zip
HTTP/1.1 200 OK
Server: Tengine
Content-Type: application/zip
Content-Length: 1450918362
Connection: keep-alive
Date: Mon, 23 May 2022 04:17:51 GMT
x-oss-request-id: 628B0AEFE6819C36364E0D97
x-oss-cdn-auth: success
Accept-Ranges: bytes
ETag: "E27C7DC9AFA8BD0B219561E4C373306A-1384"
Last-Modified: Fri, 15 Apr 2022 03:14:21 GMT
x-oss-object-type: Multipart
x-oss-hash-crc64ecma: 9259759875652541608
x-oss-storage-class: Standard
x-oss-server-time: 109
Ali-Swift-Global-Savetime: 1653279471
Via: cache6.l2nu20-3[0,0,200-0,H], cache42.l2nu20-3[2,0], cache7.cn1247[0,1,200-0,H], cache14.cn1247[12,0]#有此部分字样说明CDN生效
Age: 115031
X-Cache: HIT TCP_HIT dirn:12:811068157         #有此部分字样说明CDN生效
X-Swift-SaveTime: Tue, 24 May 2022 06:16:10 GMT #有此部分字样说明CDN生效
X-Swift-CacheTime: 165701  #有此部分字样说明CDN生效
Access-Control-Allow-Origin: *
Timing-Allow-Origin: *
EagleId: 276076a216533945029501634e
```

