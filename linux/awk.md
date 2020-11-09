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