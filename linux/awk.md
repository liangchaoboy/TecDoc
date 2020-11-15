### awk 命令使用参考


### 正在匹配使用示例

- zcat part-0000-cn.gz  | awk -F '|' '{if($3~/21:12:4[0-4]/) print $3}' | sort -u // 如果字段 3 包含字符串则打印字段 3, 不精确匹配
- zcat part-0000-cn.gz  | awk -F '|' '{if($3=="2020-10-28 21:12:47")print $3}' // 精确匹配
  - zcat part-0000-cn.gz  | awk -F '|' '$3=="2020-10-28 21:12:47"{print $3}' // 这样也是可以的
- 不匹配
  - zcat part-0000-cn.gz  | awk -F '|' '{if($3!~/21:1[0-9]/)print $3}'
- 小于、大于
  - zcat part-0000-cn.gz  | awk -F '|' '{if($6<500)print $0}'
  - zcat part-0000-cn.gz  | awk -F '|' '{if($6>200)print $6}'
- 小于等于
  - zcat part-0000-cn.gz  | awk -F '|' '{if($6<=200)print $6}'
- 或关系匹配
  - zcat part-0000-cn.gz  | awk -F '|' '{if($3~/(21:12:1[0-5]|21:22:1[0-5])/)print $3}' //匹配|两边模式之一
  - 或的关系
 ```
    cat 1 | awk -F '\t' '{if($8~/200/||$8 ~/304/) print $8}'     
      200
      200
      304
      200
```
  - 与的关系
    ```
~$ cat 1 | awk -F '\t' '{if($6~/baidu/&&$8~/200/){print $8,$6}}' | head
200 {"Accept-Encoding":"gzip, deflate, br","Host":"qiniu.com","IP":"10.20.92.26","RawQuery":"imageView2/2/w/200/q/100","User-Agent":"%E5%BE%AE%E5%95%86%E6%B0%B4%E5%8D%B0%E7%9B%B8%E6%9C%BA/1 CFNetwork/1128.0.1 Darwin/19.6.0","X-Forwarded-For":"112.96.166.243,221.5.75.88,116.129.251.88, 58.216.2.141","X-From-Cdn":"baidu","X-Hostname":"jjh1540","X-Idc":"jjh","X-Isp":"tel","X-Real-Ip":"58.216.2.141","X-Scheme":"https","X-Server-Addr":"117.80.171.97","bs":0}
200 {"Accept-Encoding":"gzip, deflate, br","Host":"qiniu.com","IP":"10.20.92.93","RawQuery":"imageView2/2/w/200/q/100","User-Agent":"%E5%BE%AE%E5%95%86%E6%B0%B4%E5%8D%B0%E7%9B%B8%E6%9C%BA/1 CFNetwork/1128.0.1 Darwin/19.6.0","X-Forwarded-For":"112.96.166.243,221.5.75.84,112.84.34.84, 58.216.2.84","X-From-Cdn":"baidu","X-Hostname":"jjh2191","X-Idc":"jjh","X-Isp":"tel","X-Real-Ip":"58.216.2.84","X-Scheme":"https","X-Server-Addr":"117.80.171.100","bs":0}
    ```

- 参考: https://my.oschina.net/u/2391658/blog/703922

### awk 数组计算

- cat test | awk -F ' ' '{a[$1]+=$3}END{for (i in a) print i,a[i]}'  // 统计单个同学的总分数
    ```
    root@testw:~# cat test // 每个同学分数
    王五 语文 35
    张三 数学 50
    李四 英语 40
    王五 数学 35
    张三 英语 30
    李四 数学 40
    王五 英语 35
    root@testw:~# cat test | awk -F ' ' '{a[$1]+=$3}END{for (i in a) print i,a[i]}'
    王五 105
    李四 80
    张三 80

    ```

### awk 格式化打印

- printf
  ```
root@testw:~# zcat part-0000-cn.gz  | head -20 | awk -F '|' '{printf("%s\tqiniu\t%s\n","sanmu",$2)}' | head -5
sanmu	qiniu	111.123.53.243
sanmu	qiniu	111.123.53.243
sanmu	qiniu	111.123.53.243
sanmu	qiniu	139.227.230.33
sanmu	qiniu	113.6.227.216
  ```
- sprintf // 使用方式和 printf 相同，但是是返回字符串


### awk 内置函数

- sub 字符串替换 sub(regexp, repla) 如果没有指定目标字符串就默认使用整个记录。替换只发生在第一次匹配的时候
  - 有两种格式： 
    - sub (regular expression, substitution string):
    - sub (regular expression, substitution string, target string)
      ```
      cat 1 | head -2 | awk -F '\t' '{sub(/Host.+com/,"qiniu.com",$6);print $0;}' | head -5
      ```
- gsub 字符串替换 使用方式和 sub 相同，不同点在于 sub 只替换第一次匹配的时候，gsub 替换替换全部
  
- substr 字符串截取函数使用, substr(string, start[, length])
  ```
qboxserver@xs337:~$ cat 2 | head -2
16051680011105845
16051680019153981
qboxserver@xs337:~$ cat 2 | awk -F ' ' '{print substr($1,1,10)}' | head -2
1605168001
1605168001
  ```
- strftime 时间格式化函数, strftime(format, 时间戳字段)
  ```
  cat 1
  partition71offset52742960755 REQ	QNM3	16051680020006737	GET	/per-plugin/dl/addons/list/win-i386/10.8.2.7072/et/index.ini	{"Host":"qiniu.com","IP":"10.20.92.47","X-Forwarded-For":"222.178.154.50, 220.170.48.195","X-Hostname":"uu","X-Idc":"jjh","X-Isp":"tel","X-Real-Ip":"220.170.48.195","X-Scheme":"http","X-Server-Addr":"117.80.171.97","bs":0}		404	{"Content-Length":"162","Content-Type":"text/html; charset=utf-8","WildcardDomain":"qiniu.com","X-Hit-Detail":"0ma"}		162	297828
  
  cat 1 | awk -F '\t' '{print strftime("%m/%d/%Y-%H:%M:%S",substr($3,1,10)),$0}' | head -2
  11/12/2020-16:00:01 partition71offset52742960625 REQ	QNM3	16051680019153981	GET	/team/pic/qvg4k8/seq/BF9CED64-1877-4A9A-B9C6-7A7B5811FC28.jpg	{"Accept-Encoding":"gzip, deflate, br","Host":"qiniu.com","IP":"10.20.92.26","RawQuery":"imageView2/2/w/200/q/100","User-Agent":"%E5%BE%AE%E5%95%86%E6%B0%B4%E5%8D%B0%E7%9B%B8%E6%9C%BA/1 CFNetwork/1128.0.1 Darwin/19.6.0","X-Forwarded-For":"112.96.166.243,221.5.75.88,116.129.251.88, 58.216.2.141","X-From-Cdn":"baidu","X-Hostname":"jjh1540","X-Idc":"jjh","X-Isp":"tel","X-Real-Ip":"58.216.2.141","X-Scheme":"https","X-Server-Addr":"117.80.171.97","bs":0}		200	{"Content-Length":"10412","Content-Type":"image/jpeg","WildcardDomain":"qiniu.com","X-Hit-Detail":"0ma"}		10412	1836334

  ```

- split(string, a, r)  字符串分割， 将 String 参数指定的参数分割为数组元素 a[1], a[2], . . ., a[n]，并返回 n 变量的值， 其实可以直接使用 awk -F '指定分隔符'
  ```
sanmu@xs337:~$ cat 1 | awk -F '\t' '{print $5}' | head -2
/cowtransfer-file-45b38aad-77e2-4af5-ba27-0e4ea4095eff%2F%E8%BE%BE%E8%8A%AC%E5%A5%87%E8%BD%AF%E4%BB%B6%2B%E6%95%99%E7%A8%8B%2F%E8%BD%AF%E4%BB%B6%2FWIN%E7%B3%BB%E7%BB%9F%2FWin%2015(%E9%80%82%E5%90%88WIN10)%2FDaVinci_Resolve_Studio_15.3.1.exe
/team/pic/qvg4k8/seq/BF9CED64-1877-4A9A-B9C6-7A7B5811FC28.jpg


sanmu:~$ cat 1 | awk -F '\t' '{print $5}' | head -2  | awk 'BEGIN{}{len=split($1,a,"/"); for (i in a) printf("%s\t",a[i]);print "\n"}END{}'
	cowtransfer-file-45b38aad-77e2-4af5-ba27-0e4ea4095eff%2F%E8%BE%BE%E8%8A%AC%E5%A5%87%E8%BD%AF%E4%BB%B6%2B%E6%95%99%E7%A8%8B%2F%E8%BD%AF%E4%BB%B6%2FWIN%E7%B3%BB%E7%BB%9F%2FWin%2015(%E9%80%82%E5%90%88WIN10)%2FDaVinci_Resolve_Studio_15.3.1.exe

qvg4k8	seq	BF9CED64-1877-4A9A-B9C6-7A7B5811FC28.jpg		team	pic


  ```
  
- mktime(datespec) // 获取指定时间的时间戳 , datespec 时间字符串，格式: "YYYY MM DD HH MM SS"
  ```
    ➜  ~ echo "2020 11 12 14 15 00s" | awk -F 's' '{print mktime($1)}'
    ➜  ~ 1605161700

    ➜  ~ date -r 1605161700
    Thu Nov 12 14:15:00 CST 2020

  ```
- match(string, regexp) // 字段正则匹配，如果匹配成功返回 1 匹配失败返回 0
  
  ````
~$ cat 1 | awk -F '\t' '{print strftime("%Y-%m-%d-%H:%M:%S",substr($3,1,10)),$4,$8}' | head -2
2020-11-12-16:00:01 GET 206
2020-11-12-16:00:01 GET 200
~$ cat 1 | awk -F '\t' '{print strftime("%Y-%m-%d-%H:%M:%S",substr($3,1,10)),$4,$8}' | awk -F ' ' '{print match($3,/200/),$0}'
0 2020-11-12-16:00:01 GET 206
1 2020-11-12-16:00:01 GET 200
1 2020-11-12-16:00:01 GET 200
0 2020-11-12-16:00:02 GET 304
1 2020-11-12-16:00:02 GET 200
1 2020-11-12-16:00:02 OPTIONS 200
0 2020-11-12-16:00:02 GET 404
0 2020-11-12-16:00:01 GET 206
  ````
- rand() // 随机函数
  ```
  ~$ cat tmp_count | awk '{print rand(),$1}' | head -5
    0.237788 1
    0.291066 2
    0.845814 3
    0.152208 4
    0.585537 5
  ```  
- zcat part-0000-cn.gz  | awk -F '|' '{print toupper($1)}' // toupper() 将小写更改为大写
- echo "tlslsllslslsslssslsll" | awk '{print length($1)}' // 计算字符串的长度


### 程序处理框架
- 一个完整的 awk 处理过程分为三部
  - BEGIN{} 的程序预处理部分，只在开始文件扫描前执行一次
  - {} 程序的主体部分，每一行被扫描到都会执行一次
  - END {} 所有行扫描完成后执行一次，通常作为收尾的统计工作
  ```
  BEGIN{

  }
  {

  }
  END{

  }

  # 示例：
cat 1 | awk -F '\t' '{print $5}' | head -2  | awk 'BEGIN{}{len=split($1,a,"/"); for (i in a) printf("%s\t",a[i]);print "\n"}END{}'
cowtransfer-file-45b38aad-77e2-4af5-ba27-0e4ea4095eff%2F%E8%BE%BE%E8%8A%AC%E5%A5%87%E8%BD%AF%E4%BB%B6%2B%E6%95%99%E7%A8%8B%2F%E8%BD%AF%E4%BB%B6%2FWIN%E7%B3%BB%E7%BB%9F%2FWin%2015(%E9%80%82%E5%90%88WIN10)%2FDaVinci_Resolve_Studio_15.3.1.exe
qvg4k8	seq	BF9CED64-1877-4A9A-B9C6-7A7B5811FC28.jpg		team	pic

  ```

### 执行 shell 命令

- system() // 能够执行 shell 命令 , 返回 0 表示调用成功
  ```
  ~$ cat tmp_count | head -2  | awk -F ' ' '{cmd=sprintf("curl -voa www.baidu.com?v=%s",$1);system(cmd)}'
* Rebuilt URL to: www.baidu.com/?v=1
* Hostname was NOT found in DNS cache
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0*   Trying 180.101.49.12...
* Connected to www.baidu.com (180.101.49.12) port 80 (#0)
> GET /?v=1 HTTP/1.1
> User-Agent: curl/7.35.0
> Host: www.baidu.com
> Accept: */*
>
< HTTP/1.1 200 OK
< Accept-Ranges: bytes
< Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
< Connection: keep-alive
< Content-Length: 2381
< Content-Type: text/html
< Date: Sun, 15 Nov 2020 10:10:08 GMT
< Etag: "588604c8-94d"
< Last-Modified: Mon, 23 Jan 2017 13:27:36 GMT
< Pragma: no-cache
* Server bfe/1.0.8.18 is not blacklisted
< Server: bfe/1.0.8.18
< Set-Cookie: BDORZ=27315; max-age=86400; domain=.baidu.com; path=/
<
{ [data not shown]
100  2381  100  2381    0     0  89226      0 --:--:-- --:--:-- --:--:-- 91576
* Connection #0 to host www.baidu.com left intact
* Rebuilt URL to: www.baidu.com/?v=2
* Hostname was NOT found in DNS cache
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0*   Trying 180.101.49.12...
* Connected to www.baidu.com (180.101.49.12) port 80 (#0)
> GET /?v=2 HTTP/1.1
> User-Agent: curl/7.35.0
> Host: www.baidu.com
> Accept: */*
>
< HTTP/1.1 200 OK
< Accept-Ranges: bytes
< Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
< Connection: keep-alive
< Content-Length: 2381
< Content-Type: text/html
< Date: Sun, 15 Nov 2020 10:10:08 GMT
< Etag: "588604c8-94d"
< Last-Modified: Mon, 23 Jan 2017 13:27:36 GMT
< Pragma: no-cache
* Server bfe/1.0.8.18 is not blacklisted
< Server: bfe/1.0.8.18
< Set-Cookie: BDORZ=27315; max-age=86400; domain=.baidu.com; path=/
<
{ [data not shown]
100  2381  100  2381    0     0  77285      0 --:--:-- --:--:-- --:--:-- 79366
* Connection #0 to host www.baidu.com left intact

  ```


- 参考: https://www.ibm.com/developerworks/cn/linux/shell/awk/awk-1/index.html    
- 参考: https://book.saubcy.com/AwkInAction/section_2/case_2.html // awk 编程指南
- 参考: http://blog.51yip.com/shell/1151.html