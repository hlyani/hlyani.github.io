# mapreduce 相关

## 一、例1

[](https://www.cnblogs.com/kaituorensheng/p/3826114.html)

> python利用streaming编写mapreducer程序

##### 1、先编写一个map.py

```
# coding: utf8

import sys

for line in sys.stdin:
   line = line.strip()
   film_d = line.split(";")
   print(film_d[0])
```

##### 2、再编写一个red.py：

```
# coding: utf8

import sys

cur_film = "惊天魔盗团2"
cur_count = 0
for line in sys.stdin:
    if cur_film  in line:
        cur_count  += 1
print('%s总共出现了，%s次。' % ( cur_film,cur_count))
```

##### 3、运行程序：

```
1）数据上传
• Hdfs dfs –put ~/dat0203_1.log input/

2）修改文件属性
• chmod 777 map.py
• chmod 777 red.py

3）开始运行
Hadoop jar /usr/local/hadoop/share/hadoop/tools/lib/hadoopstreaming-2.7.1.jar
–file ~/map.py –mapper ~/map.py
–file ~/red.py –reducer ~/red.py –input input –output
output
```

## 二、例2

##### 1、mapper.py

```
#!/usr/bin/python
# -*- coding: utf-8 -*-
 
import sys

# input comes from STDIN (standard input)
for line in sys.stdin:
   # remove leading and trailing whitespace
   line = line.strip()
   # split the line into words
   words = line.split()
   # increase counters
   for word in words:
       # write the results to STDOUT (standard output);
       # what we output here will be the input for the
       # Reduce step, i.e. the input for reducer.py
       #
       # tab-delimited; the trivial word count is 1
       print '%s\t%s' % (word, 1)

```

##### 2、recuder.py

```
#!/usr/bin/python
# -*- coding: utf-8 -*-

from operator import itemgetter
import sys

current_word = None
current_count = 0
word = None

# input comes from STDIN
for line in sys.stdin:
   # remove leading and trailing whitespace
   line = line.strip()

   # parse the input we got from mapper.py
   word, count = line.split('\t', 1)

   # convert count (currently a string) to int
   try:
       count = int(count)
   except ValueError:
       # count was not a number, so silently
       # ignore/discard this line
       continue

   # this IF-switch only works because Hadoop sorts map output
   # by key (here: word) before it is passed to the reducer
   if current_word == word:
       current_count += count
   else:
       if current_word:
           # write result to STDOUT
           print '%s\t%s' % (current_word, current_count)
       current_count = count
       current_word = word

# do not forget to output the last word if needed!
if current_word == word:
   print '%s\t%s' % (current_word, current_count)
```

##### 3、test.txt

```
hello
hello world
world hi
hi world
```

##### 4、运行程序

```
chmod 777 mapper.py
chmod 777 reducer.py

ls /

root@hadoop1:~# ls
mapper.py  reducer.py  test.txt

hdfs dfs -put test.txt /

hadoop jar /opt/hadoop-2.7.3/share/hadoop/tools/lib/hadoop-streaming-2.7.3.jar ©¦
 -file mapper.py -mapper mapper.py  -file reducer.py -reducer reducer.py -input /test.txt -outp©¦
ut /output

hdfs dfs -cat /output/*
```

