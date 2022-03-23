---
title: python编程风格之好的代码逻辑
categories:
 - python
tags:
 - 编程风格
---
<blockquote>
  本文通过一个词频统计的代码，简单描述了python各种编程风格以及其特点，但风格无优劣，只是希望
希望读者能在项目中选择合适的编程风格。
</blockquote>     

### 引言
<blockquote>
  本文的介绍的项目很简单，给定一个文件，在去掉停用词之后，显示频率最高的N个词
</blockquote>

## 基本风格
### 1. 面条风格（瀑布风格）
特点：没有命名函数，没有对象，没有设计模式，拿起键盘直接干。
```python
#!/usr/bin/env python
import string

with open('../stop_words.txt') as f:
    stop_words = f.read().split(',')
stop_words.extend(list(string.ascii_lowercase))

word_count = {}
for line in open('../input.txt'):
    for word in line.split():
        if word not in stop_words:
            word_count.setdefault(word, 0)
            word_count[word] += 1

for tf in sorted(word_count.items(), key=lambda x: x[1], reverse=True):
    print tf[0], tf[1]
```
评价：从计算机诞生初期，该模式就已经兴起。写起来行云流水，一气呵成，外表看起来
波澜壮阔，内部有时却让人无从下手。虽然不被看好，但确延用至今，对于单一简单任务，
新手、老手都对它爱不释手。
<!-- more -->
### 2. 食谱风格
特点：使用过程抽象技术将一个大问题，拆解成若干小问题，并逐个解决，这些过程函数
使用全局变量的形式共享状态。
```python
#!/usr/bin/env python
# coding=utf-8
import string

data = []
word_freqs = []


def read_file(path_to_file):
    """
    读取文件单词
    :param path_to_file: 文件路径
    :return:
    """
    global data
    with open(path_to_file) as f:
        for word in f.readlines():
            data.extend(word.split(" "))


def filter_chars():
    """
    去掉空字符和换行符
    :return:
    """
    global data
    data = [d.strip() for d in data if d.strip()]


def remove_stop_words():
    """
    去掉停用词
    :return:
    """
    global data
    with open('../stop_words.txt') as f:
        stop_words = f.read().split(',')
    # add single-letter words
    stop_words.extend(list(string.ascii_lowercase))
    new_words = []
    for word in data:
        if word not in stop_words:
            new_words.append(word)
    data = new_words


def frequencies():
    """
    生成词频序列
    """
    global data
    global word_freqs
    word_count = {}
    for word in data:
        word_count.setdefault(word, 0)
        word_count[word] += 1
    word_freqs = word_count.items()


def sort():
    """
    词频排序
    """
    global word_freqs
    word_freqs.sort(lambda x, y: cmp(y[1], x[1]))


#
# The main function
#
read_file('../input.txt')
filter_chars()
remove_stop_words()
frequencies()
sort()

for tf in word_freqs[0:25]:
    print tf[0], ' - ', tf[1]
```
评价：每个过程函数独立处理一件事，结构化编程的第一步，通过数据共享为下一个过程函数提供数据，
就像是烹饪我们按照食谱进行每一步操作时，也会随着步骤的推进改变食材的状态。虽然数据共享会有一定的
风险，比如多放几次盐，食材的味道可能变的超出预期，但这就要取决利弊了。
### 3. 流水线风格（函数式编程）
特点：和食谱风格一样，但是这个函数间没有状态共享，函数根据输入形成输出。
```python
#!/usr/bin/env python
# coding=utf-8
import string


def read_file(path_to_file):
    """
    读取文件单词
    :param path_to_file: 文件路径
    :return []:
    """
    words = []
    with open(path_to_file) as f:
        for word in f.readlines():
            words.extend(word.split(" "))
    return words


def filter_chars(words):
    """
    去掉空字符和换行符
    :return:
    """
    return [d.strip() for d in words if d.strip()]


def remove_stop_words(words):
    """
    去掉停用词
    :return:
    """
    with open('../stop_words.txt') as f:
        stop_words = f.read().split(',')
    # add single-letter words
    stop_words.extend(list(string.ascii_lowercase))
    new_words = []
    for word in words:
        if word not in stop_words:
            new_words.append(word)
    return new_words


def frequencies(words):
    """
    生成词频序列
    """
    word_count = {}
    for word in words:
        word_count.setdefault(word, 0)
        word_count[word] += 1
    return word_count.items()


def sort(freq):
    """
    词频排序
    """
    freq.sort(lambda x, y: cmp(y[1], x[1]))
    return freq

print(sort(frequencies(remove_stop_words(filter_chars(read_file('../input.txt'))))))
```
评价：和食谱风格既相似又不同，不同在于1、它不共享数据，2、(保持函数一致性)同函数多次调用输出结果都不变
它高度契合的数学的函数定义使它流传至今，如spark、mapreduce等等都对它进行了支持。
### 4. 极简风格
特点：多使用编程语言的高级特性和库来实现，让代码尽可能简洁
```python
import heapq, re, sys

words = re.findall("[a-z]{2,}", open('../input.txt').read().lower())
for w in heapq.nlargest(25, set(words) - set(open("../stop_words.txt").read().split(",")), words.count):
    print w, "-", words.count(w)
```
评价：运用得当，方可呼风唤雨，运用不当，小心走火入魔。
## 函数风格
### 1.递归风格
特点：使用数学归纳法将问题从0到n再到n+1推导
```python
import re, sys, operator


def count(word_list, stopwords, wordfreqs):
    if not word_list:
        return
    else:
        word = word_list[0]
        if word not in stopwords:
            if word in wordfreqs:
                wordfreqs[word] += 1
            else:
                wordfreqs[word] = 1
        count(word_list[1:], stopwords, wordfreqs)


def wf_print(wordfreq):
    if not wordfreq:
        return
    else:
        (w, c) = wordfreq[0]
        print w, '-', c
        wf_print(wordfreq[1:])


stop_words = set(open('../stop_words.txt').read().split(','))
words = re.findall('[a-z]{2,}', open('../input.txt').read().lower())
word_freqs = {}
CHUNK = 9500
for i in range(0, len(words), CHUNK):
    count(words[i:i+CHUNK], stop_words, word_freqs)

wf_print(sorted(word_freqs.iteritems(), key=operator.itemgetter(1), reverse=True)[:25])
```
评论：递归一时爽，堆栈火葬场，要想用的爽就需要有效减少递归的层数，或者像以上代码一样将数据
分块进行递归处理
### 2.callback风格
特点：每个函数都有一个额外的函数参数，并且在函数结束时调用该参数
```python
import sys, re, operator, string


def read_file(path_to_file, func):
    with open(path_to_file) as f:
        data = f.read()
    func(data, normalize)


def filter_chars(str_data, func):
    pattern = re.compile('[\W_]+')
    func(pattern.sub(' ', str_data), scan)


def normalize(str_data, func):
    func(str_data.lower(), remove_stop_words)


def scan(str_data, func):
    func(str_data.split(), frequencies)


def remove_stop_words(word_list, func):
    with open('../stop_words.txt') as f:
        stop_words = f.read().split(',')
    # add single-letter words
    stop_words.extend(list(string.ascii_lowercase))
    func([w for w in word_list if not w in stop_words], sort)


def frequencies(word_list, func):
    wf = {}
    for w in word_list:
        if w in wf:
            wf[w] += 1
        else:
            wf[w] = 1
    func(wf, print_text)


def sort(wf, func):
    func(sorted(wf.iteritems(), key=operator.itemgetter(1), reverse=True), no_op)


def print_text(word_freqs, func):
    for (w, c) in word_freqs[0:25]:
        print w, "-", c
    func(None)


def no_op(func):
    return


read_file('../input.txt', filter_chars)
```
评价：常用于真对程序的同状态执行不同操作，异步必会写法。但是你凝望着深渊，深渊也在凝望你，小心回调地狱。
### 3.真*流式风格
特点：需要一个可以修改数据的类，并将流水线风格的方法绑定在一起。
```python
import sys, re, operator, string


class TFTheOne:
    def __init__(self, v):
        self._value = v

    def bind(self, func):
        self._value = func(self._value)
        return self

    def printme(self):
        print self._value


def read_file(path_to_file):
    with open(path_to_file) as f:
        data = f.read()
    return data


def filter_chars(str_data):
    pattern = re.compile('[\W_]+')
    return pattern.sub(' ', str_data)


def normalize(str_data):
    return str_data.lower()


def scan(str_data):
    return str_data.split()


def remove_stop_words(word_list):
    with open('../stop_words.txt') as f:
        stop_words = f.read().split(',')
    # add single-letter words
    stop_words.extend(list(string.ascii_lowercase))
    return [w for w in word_list if not w in stop_words]


def frequencies(word_list):
    word_freqs = {}
    for w in word_list:
        if w in word_freqs:
            word_freqs[w] += 1
        else:
            word_freqs[w] = 1
    return word_freqs


def sort(word_freq):
    return sorted(word_freq.iteritems(), key=operator.itemgetter(1), reverse=True)


def top25_freqs(word_freqs):
    top25 = ""
    for tf in word_freqs[0:25]:
        top25 += str(tf[0]) + ' - ' + str(tf[1]) + '\n'
    return top25


TFTheOne(sys.argv[1]) \
    .bind(read_file) \
    .bind(filter_chars) \
    .bind(normalize) \
    .bind(scan) \
    .bind(remove_stop_words) \
    .bind(frequencies) \
    .bind(sort) \
    .bind(top25_freqs) \
    .printme()
```
评价：绑定方法返回自身，并且封装操作隔离了全局变量，绑定操作实现了隐式数据传递，
最后的拆分打开了封装。
## 面向对象风格
### 1.对象风格
### 2.消息风格
### 3.闭域风格
### 4.抽象对象风格
### 5.好莱坞风格
### 6.公告板风格
## 反射与元编程
## 异常处理
### 1.宽容风格
特点：每个参数都检查自身参数合理性，当参数不合理时，返回合理结果或者指定合理值。
```python
import sys, re, operator, string

def extract_words(path_to_file):
    if not isinstance(path_to_file, str) or not path_to_file:
        return []

    try:
        with open(path_to_file) as f:
            str_data = f.read()
    except IOError as e:
        print "I/O error({0}) when opening {1}: {2}".format(e.errno, path_to_file, e.strerror)
        return []

    pattern = re.compile('[\W_]+')
    word_list = pattern.sub(' ', str_data).lower().split()
    return word_list


def remove_stop_words(word_list):
    if not isinstance(word_list, list):
        return []

    try:
        with open('../stop_words.txt') as f:
            stop_words = f.read().split(',')
    except IOError as e:
        print "I/O error({0}) when opening ../stops_words.txt: {1}".format(e.errno, e.strerror)
        return word_list

    stop_words.extend(string.ascii_lowercase)
    return [w for w in word_list if w not in stop_words]


def frequencies(word_list):
    if not isinstance(word_list, list) or not word_list:
        return {}

    word_freqs = {}
    for w in word_list:
        if w in word_freqs:
            word_freqs[w] += 1
        else:
            word_freqs[w] = 1
    return word_freqs


def sort(word_freq):
    if not isinstance(word_freq, dict) or not word_freq:
        return []

    return sorted(word_freq.iteritems(), key=operator.itemgetter(1), reverse=True)


filename = sys.argv[1] if len(sys.argv) > 1 else "../input.txt"
word_freqs = sort(frequencies(remove_stop_words(extract_words(filename))))
for tf in word_freqs[0:25]:
    print tf[0], ' - ', tf[1]

```
评价：发生异常时，跳过异常继续执行，使得函数具有连续性。但是一味的包容可能会导致一些摸不到头脑的错误。
### 2.严厉风格
特点：和包容风格一样，只不过遇到异常就退出，不给你搞事情的机会。
```python
import sys, re, operator, string, traceback

def extract_words(path_to_file):
    assert isinstance(path_to_file, str), "I need a string!"
    assert path_to_file, "I need a non-empty string!"

    try:
        with open(path_to_file) as f:
            str_data = f.read()
    except IOError as e:
        print "I/O error({0}) when opening {1}: {2}! I quit!".format(e.errno, path_to_file, e.strerror)
        raise e

    pattern = re.compile('[\W_]+')
    word_list = pattern.sub(' ', str_data).lower().split()
    return word_list


def remove_stop_words(word_list):
    assert isinstance(word_list, list), "I need a list!"

    try:
        with open('../stop_words.txt') as f:
            stop_words = f.read().split(',')
    except IOError as e:
        print "I/O error({0}) when opening ../stops_words.txt: {1}! I quit!".format(e.errno, e.strerror)
        raise e

    stop_words.extend(string.ascii_lowercase)
    return [w for w in word_list if w not in stop_words]


def frequencies(word_list):
    assert isinstance(word_list,list), "I need a list!"
    assert word_list != [], "I need a non-empty list!"

    word_freqs = {}
    for w in word_list:
        if w in word_freqs:
            word_freqs[w] += 1
        else:
            word_freqs[w] = 1
    return word_freqs


def sort(word_freq):
    assert isinstance(word_freq, dict), "I need a dictionary!"
    assert word_freq != {}, "I need a non-empty dictionary!"

    try:
        return sorted(word_freq.iteritems(), key=operator.itemgetter(1), reverse=True)
    except Exception as e:
        print "Sorted threw {0}: {1}".format(e)
        raise e


try:
    word_freqs = sort(frequencies(remove_stop_words(extract_words("../input.txt"))))

    assert isinstance(word_freqs, list), "OMG! This is not a list!"
    for (w, c) in word_freqs[0:25]:
        print w, ' - ', c
except Exception as e:
    print "Something wrong: {0}".format(e)
    traceback.print_exc()
```
评价：繁琐的代码保证了异常无处可逃，但是不近人情的严厉也会让用户难以接受。
### 3.严厉懒惰风格
特点：严厉是说它和严厉模式一样，遇到异常就退出。懒惰是因为前两种模式都是每个模块处理自己的异常，它都不处理并且丢给别人。
```python
import sys, re, operator, string

def extract_words(path_to_file):
    assert isinstance(path_to_file, str), "I need a string! I quit!"
    assert path_to_file, "I need a non-empty string! I quit!"

    with open(path_to_file) as f:
        data = f.read()
    pattern = re.compile('[\W_]+')
    word_list = pattern.sub(' ', data).lower().split()
    return word_list


def remove_stop_words(word_list):
    assert isinstance(word_list, list), "I need a list! I quit!"

    with open('../stop_words.txt') as f:
        stop_words = f.read().split(',')
    # add single-letter words
    stop_words.extend(list(string.ascii_lowercase))
    return [w for w in word_list if not w in stop_words]


def frequencies(word_list):
    assert isinstance(word_list, list), "I need a list! I quit!"
    assert word_list != [], "I need a non-empty list! I quit!"

    word_freqs = {}
    for w in word_list:
        if w in word_freqs:
            word_freqs[w] += 1
        else:
            word_freqs[w] = 1
    return word_freqs


def sort(word_freqs):
    assert isinstance(word_freqs, dict), "I need a dictionary! I quit!"
    assert word_freqs != {}, "I need a non-empty dictionary! I quit!"

    return sorted(word_freqs.iteritems(), key=operator.itemgetter(1), reverse=True)


try:
    word_freqs = sort(frequencies(remove_stop_words(extract_words("../input.txt"))))

    for tf in word_freqs[0:25]:
        print tf[0], ' - ', tf[1]
except Exception as e:
    print "Something wrong: {0}".format(e)
```
特点：除非局存在有意义的处理方式，更好的做法是将错误上报给调用链的上游。
## 以数据为中心
## 并发
### 1.参与者风格
特点：将大问题分解为问题相关对象，并每个对象都有一个消息队列，对象仅公开接口。
```python

import sys, re, operator, string
from threading import Thread
from Queue import Queue


class ActiveWFObject(Thread):
    def __init__(self):
        Thread.__init__(self)
        self.name = str(type(self))
        self.queue = Queue()
        self._stop = False
        self.start()

    def run(self):
        while not self._stop:
            message = self.queue.get()
            self._dispatch(message)
            if message[0] == 'die':
                self._stop = True

def send(receiver, message):
    receiver.queue.put(message)

class DataStorageManager(ActiveWFObject):
    """ Models the contents of the file """
    _data = ''

    def _dispatch(self, message):
        if message[0] == 'init':
            self._init(message[1:])
        elif message[0] == 'send_word_freqs':
            self._process_words(message[1:])
        else:
            # forward
            send(self._stop_word_manager, message)
 
    def _init(self, message):
        path_to_file = message[0]
        self._stop_word_manager =    [1]
        with open(path_to_file) as f:
            self._data = f.read()
        pattern = re.compile('[\W_]+')
        self._data = pattern.sub(' ', self._data).lower()

    def _process_words(self, message):
        recipient = message[0]
        data_str = ''.join(self._data)
        words = data_str.split()
        for w in words:
            send(self._stop_word_manager, ['filter', w])
        send(self._stop_word_manager, ['top25', recipient])

class StopWordManager(ActiveWFObject):
    """ Models the stop word filter """
    _stop_words = []

    def _dispatch(self, message):
        if message[0] == 'init':
            self._init(message[1:])
        elif message[0] == 'filter':
            return self._filter(message[1:])
        else:
            # forward
            send(self._word_freqs_manager, message)
 
    def _init(self, message):
        with open('../stop_words.txt') as f:
            self._stop_words = f.read().split(',')
        self._stop_words.extend(list(string.ascii_lowercase))
        self._word_freqs_manager = message[0]

    def _filter(self, message):
        word = message[0]
        if word not in self._stop_words:
            send(self._word_freqs_manager, ['word', word])

class WordFrequencyManager(ActiveWFObject):
    """ Keeps the word frequency data """
    _word_freqs = {}

    def _dispatch(self, message):
        if message[0] == 'word':
            self._increment_count(message[1:])
        elif message[0] == 'top25':
            self._top25(message[1:])
 
    def _increment_count(self, message):
        word = message[0]
        if word in self._word_freqs:
            self._word_freqs[word] += 1 
        else: 
            self._word_freqs[word] = 1

    def _top25(self, message):
        recipient = message[0]
        freqs_sorted = sorted(self._word_freqs.iteritems(), key=operator.itemgetter(1), reverse=True)
        send(recipient, ['top25', freqs_sorted])

class WordFrequencyController(ActiveWFObject):

    def _dispatch(self, message):
        if message[0] == 'run':
            self._run(message[1:])
        elif message[0] == 'top25':
            self._display(message[1:])
        else:
            raise Exception("Message not understood " + message[0])
 
    def _run(self, message):
        self._storage_manager = message[0]
        send(self._storage_manager, ['send_word_freqs', self])

    def _display(self, message):
        word_freqs = message[0]
        for (w, f) in word_freqs[0:25]:
            print w, ' - ', f
        send(self._storage_manager, ['die'])
        self._stop = True

word_freq_manager = WordFrequencyManager()

stop_word_manager = StopWordManager()
send(stop_word_manager, ['init', word_freq_manager])

storage_manager = DataStorageManager()
send(storage_manager, ['init', "../input.txt", stop_word_manager])

wfcontroller = WordFrequencyController()
send(wfcontroller, ['run', storage_manager])

[t.join() for t in [word_freq_manager, stop_word_manager, storage_manager, wfcontroller]]
```
评价：模块拆解，通讯独立，各个模块能相互配合，分布式模型之一。但在通信任务较多或者模块配合复杂式通信程序编写复杂不易维护。
### 2.数据空间风格
特点：按照数据流的类型定义数据空间，再根据数据空间定义相应的worker
```python
import re, sys, operator, Queue, threading

word_space = Queue.Queue()
freq_space = Queue.Queue()

stopwords = set(open('../stop_words.txt').read().split(','))


def process_words():
    word_freqs = {}
    while True:
        try:
            word = word_space.get(timeout=1)
        except Queue.Empty:
            break

        if word not in stopwords:
            word_freqs.setdefault(word, 0)
            word_freqs[word] += 1
    freq_space.put(word_freqs)


for word in re.findall('[a-z]{2,}', open("../input.txt", 'r').read().lower()):
    word_space.put(word)

workers = []
for i in range(5):
    workers.append(threading.Thread(target=process_words))
[t.start() for t in workers]
[t.join() for t in workers]

word_freqs = {}
while not freq_space.empty():
    freqs = freq_space.get()
    print(freqs)
    for (k, v) in freqs.iteritems():
        if k in word_freqs:
            count = sum(item[k] for item in [freqs, word_freqs])
        else:
            count = freqs[k]
        word_freqs[k] = count

for (w, c) in sorted(word_freqs.iteritems(), key=operator.itemgetter(1), reverse=True)[:25]:
    print w, '-', c
```
评价：对于数据密集型，该风格得心应手，但对于并发需要相互配合的状态，则无能为力。
### 3.Map Reduce风格
特点：map对每个数据块运用定义函数，reduce对map的结果进行整合
```python
import sys, re, operator, string

def partition(data_str, nlines):
    """ 
    Partitions the input data_str (a big string)
    into chunks of nlines.
    """
    lines = data_str.split('\n')
    for i in xrange(0, len(lines), nlines):
        yield '\n'.join(lines[i:i+nlines])

def split_words(data_str):
    """ 
    Takes a string,  returns a list of pairs (word, 1), 
    one for each word in the input, so
    [(w1, 1), (w2, 1), ..., (wn, 1)]
    """
    def _scan(str_data):
        pattern = re.compile('[\W_]+')
        return pattern.sub(' ', str_data).lower().split()

    def _remove_stop_words(word_list):
        with open('../stop_words.txt') as f:
            stop_words = f.read().split(',')
        stop_words.extend(list(string.ascii_lowercase))
        return [w for w in word_list if not w in stop_words]

    result = []
    words = _remove_stop_words(_scan(data_str))
    for w in words:
        result.append((w, 1))
    return result

def count_words(pairs_list_1, pairs_list_2):
    """ 
    Takes two lists of pairs of the form
    [(w1, 1), ...]
    and returns a list of pairs [(w1, frequency), ...], 
    where frequency is the sum of all the reported occurrences
    """
    mapping = dict((k, v) for k, v in pairs_list_1)
    for p in pairs_list_2:
        mapping.setdefault(p[0], 0)
        mapping[p[0]] += p[1]
    return mapping.items()

def read_file(path_to_file):
    with open(path_to_file) as f:
        data = f.read()
    return data

def sort(word_freq):
    return sorted(word_freq, key=operator.itemgetter(1), reverse=True)

splits = map(split_words, partition(read_file("../input.txt"), 200))
splits.insert(0, []) # Normalize input to reduce
print()
word_freqs = sort(reduce(count_words, splits))

for (w, c) in word_freqs[0:25]:
    print w, ' - ', c
```
特点：分治合并，对于超大数据量威力无穷。
### 4.双重Map Reduce风格
特点：同上
```python
import sys, re, operator, string

def partition(data_str, nlines):
    """ 
    Partitions the input data_str (a big string)
    into chunks of nlines.
    """
    lines = data_str.split('\n')
    for i in xrange(0, len(lines), nlines):
        yield '\n'.join(lines[i:i+nlines])

def split_words(data_str):
    """ 
    Takes a string, returns a list of pairs (word, 1), 
    one for each word in the input, so
    [(w1, 1), (w2, 1), ..., (wn, 1)]
    """
    def _scan(str_data):
        pattern = re.compile('[\W_]+')
        return pattern.sub(' ', str_data).lower().split()

    def _remove_stop_words(word_list):
        with open('../stop_words.txt') as f:
            stop_words = f.read().split(',')
        stop_words.extend(list(string.ascii_lowercase))
        return [w for w in word_list if not w in stop_words]

    result = []
    words = _remove_stop_words(_scan(data_str))
    for w in words:
        result.append((w, 1))
    return result

def regroup(pairs_list):
    """
    Takes a list of lists of pairs of the form 
    [[(w1, 1), (w2, 1), ..., (wn, 1)],
     [(w1, 1), (w2, 1), ..., (wn, 1)],
     ...]
    and returns a dictionary mapping each unique word to the 
    corresponding list of pairs, so
    { w1 : [(w1, 1), (w1, 1)...], 
      w2 : [(w2, 1), (w2, 1)...], 
      ...}
    """
    mapping = {}
    for pairs in pairs_list:
        for p in pairs:
            mapping.setdefault(p[0], [])
            mapping[p[0]].append(p)
    return mapping
    
def count_words(mapping):
    """ 
    Takes a mapping of the form (word, [(word, 1), (word, 1)...)])
    and returns a pair (word, frequency), where frequency is the 
    sum of all the reported occurrences
    """
    def add(x, y):
        return x+y

    return mapping[0], reduce(add, (pair[1] for pair in mapping[1]))

def read_file(path_to_file):
    with open(path_to_file) as f:
        data = f.read()
    return data

def sort(word_freq):
    return sorted(word_freq, key=operator.itemgetter(1), reverse=True)

splits = map(split_words, partition(read_file("../input.txt"), 200))
splits_per_word = regroup(splits)
word_freqs = sort(map(count_words, splits_per_word.items()))

for (w, c) in word_freqs[0:25]:
    print w, ' - ', c
```
特点：解决单一mapreduce时reduce无法并行计算的问题
## 交互
### 1.MVC风格
特点：模型、视图、控制器。模型提供数据，视图展示数据，控制器控制模型和视图。
```python
import sys, re, operator, collections

class WordFrequenciesModel:
    """ Models the data. In this case, we're only interested 
    in words and their frequencies as an end result """
    freqs = {}
    stopwords = set(open('../stop_words.txt').read().split(','))
    def __init__(self, path_to_file):
        self.update(path_to_file)

    def update(self, path_to_file):
        try:
            words = re.findall('[a-z]{2,}', open(path_to_file).read().lower())
            self.freqs = collections.Counter(w for w in words if w not in self.stopwords)
        except IOError:
            print "File not found"
            self.freqs = {}

class WordFrequenciesView:
    def __init__(self, model):
        self._model = model

    def render(self):
        sorted_freqs = sorted(self._model.freqs.iteritems(), key=operator.itemgetter(1), reverse=True)
        for (w, c) in sorted_freqs[0:25]:
            print w, '-', c

class WordFrequencyController:
    def __init__(self, model, view):
        self._model, self._view = model, view
        view.render()

    def run(self):
        while True:
            print "Next file: " 
            sys.stdout.flush() 
            filename = sys.stdin.readline().strip()
            self._model.update(filename)
            self._view.render()


m = WordFrequenciesModel("../input.txt")
v = WordFrequenciesView(m)
c = WordFrequencyController(m, v)
c.run()
```
评价：MVC随处可见，适用于各种规模的设计。
### 2.RESTful风格
特点：交互方和后端直接对应，这使得组件能够独立发展，接口易扩展。
```python
import re, string, sys

with open("../stop_words.txt") as f:
    stops = set(f.read().split(",")+list(string.ascii_lowercase))
data = {}

def error_state():
    return "Something wrong", ["get", "default", None]

def default_get_handler(args):
        rep = "What would you like to do?"
        rep += "\n1 - Quit" + "\n2 - Upload file"
        links = {"1" : ["post", "execution", None], "2" : ["get", "file_form", None]}
        return rep, links

def quit_handler(args):
    sys.exit("Goodbye cruel world...")

def upload_get_handler(args):
    return "Name of file to upload?", ["post", "file"]
        
def upload_post_handler(args):
    def create_data(filename):
        if filename in data:
            return
        word_freqs = {}
        with open(filename) as f:
            for w in [x.lower() for x in re.split("[^a-zA-Z]+", f.read()) if len(x) > 0 and x.lower() not in stops]:
                word_freqs[w] = word_freqs.get(w, 0) + 1
        word_freqsl = word_freqs.items()
        word_freqsl.sort(lambda x, y: cmp(y[1], x[1]))
        data[filename] = word_freqsl

    if args == None:
        return error_state()
    filename = args[0]
    try:
        create_data(filename)
    except:
        return error_state()
    return word_get_handler([filename, 0])

def word_get_handler(args):
    def get_word(filename, word_index):
        if word_index < len(data[filename]):
            return data[filename][word_index]
        else:
            return ("no more words", 0) 

    filename = args[0]; word_index = args[1]
    word_info = get_word(filename, word_index)
    rep = '\n#{0}: {1} - {2}'.format(word_index+1, word_info[0], word_info[1])
    rep += "\n\nWhat would you like to do next?"
    rep += "\n1 - Quit" + "\n2 - Upload file"
    rep += "\n3 - See next most-frequently occurring word"
    links = {"1" : ["post", "execution", None], 
             "2" : ["get", "file_form", None], 
             "3" : ["get", "word", [filename, word_index+1]]}
    return rep, links

handlers = {"post_execution" : quit_handler,
            "get_default" : default_get_handler, 
            "get_file_form" : upload_get_handler, 
            "post_file" : upload_post_handler, 
            "get_word" : word_get_handler }

def handle_request(verb, uri, args):
    def handler_key(verb, uri):
        return verb + "_" + uri

    if handler_key(verb, uri) in handlers:
        return handlers[handler_key(verb, uri)](args)
    else:
        return handlers[handler_key("get", "default")](args)

def render_and_get_input(state_representation, links):
    print state_representation
    sys.stdout.flush()
    if type(links) is dict: # many possible next states
        input = sys.stdin.readline().strip()
        if input in links:
            return links[input]
        else:
            return ["get", "default", None]
    elif type(links) is list: # only one possible next state
        if links[0] == "post": # get "form" data
            input = sys.stdin.readline().strip()
            links.append([input]) # add the data at the end
            return links
        else: # get action, don't get user input
            return links
    else:
        return ["get", "default", None]


request = ["get", "default", None]

while True:
    state_representation, links = handle_request(*request)
    request = render_and_get_input(state_representation, links)
```
评价：客户端和服务器之间接口有唯一映射，而且是无状态通信，状态由客户端session提供，开发简单方便。
