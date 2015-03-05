---
layout: default
title: Python代码片段
---


## 例子

* 输出a-z字符

```python

for i in range(97,123):
    print  chr(i)
```


* 随机手机号码

```python

import random

random.choice(['139','188','185','136','158','151']) + "".join(random.choice("0123456789") for i in range(8))
```

## Python实现Hadoop MapReduce程序

* mapper.py

```python

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
* reducer.py

```python
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
* 测试

```shell
hadoop@derekUbun:/usr/local/hadoop$ echo "foo foo quux labs foo bar quux" | ./mapper.py
foo 	 1
foo 	 1
quux 	 1
labs 	 1
foo 	 1
bar 	 1
quux 	 1
hadoop@derekUbun:/usr/local/hadoop$ echo "foo foo quux labs foo bar quux" |./mapper.py | sort |./reducer.py
bar 	1
foo 	3
labs 	1
quux 	2
```

