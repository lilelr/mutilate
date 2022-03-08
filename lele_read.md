
docker run -p 11211:11211 --name lele_memcache memcached

### telnet testing
https://blog.csdn.net/think2me/article/details/52091740

telnet 127.0.0.1 11211
set name 16 10 5 #设置name 为 5个字符，过期时间为10秒
12345
STORED #成功

get name #读取name的值
VALUE name 16 5
12345
END

set name 16 10 5
1234

CLIENT_ERROR bad data chunk
ERROR  #字符长度不符合，报错。

get name
END #过期了，没取到值

set name 16 0 5
123456
CLIENT_ERROR bad data chunk
ERROR #字符长度不符合，报错。

set name 16 0 5 #设置为不过期
12345
STORED

get name
VALUE name 16 5
12345
END #正确返回值

set name2 16 2592000 3 #设置为刚好30天过期
123
STORED #成功

get name2
VALUE name2 16 3
123
END # 成功取出数据


# mutilate

## install  
sudo apt-get install -y libzmq3-dev libzmq5

sudo apt-get install -y scons libevent-dev gengetopt

###  10 threads, and 8 connections, expoentential distribution, peak QPS,
./mutilate -s localhost:11211 -T 10 -c 8 \
-i exponential \
-q 50000 \
-t 20

./mutilate -s slave4:11211 -T 10 -c 8 \
-i exponential \
-q 50000 \
-t 10

./mutilate -s localhost:11211 -T 10 -c 8 -t 2

#### on slave4
./mutilate -s slave4:11211 -T 10 -c 8 -t 2

#### cluster mode
##### cd open-source/mutilate/cmake-build-debug/
#####  on p09
cd open-source/mutilate/build/ && ./mutilate -T 16 -A

./mutilate -T 1 -A
##### on slave4
cd open-source/mutilate/cmake-build-debug/
./mutilate -s slave4 --loadonly

./mutilate -s slave4 --noload -i exponential \
-B -T 16 -Q 1000 -D 4 -C 4 \
-a p09 -a p10 \
-c 4 -q 20000

gdb --args ./mutilate -s slave4 --noload -i exponential \
-B -T 4 -Q 1000 -D 4 -C 4 \
-a p09  \
-c 4 -q 2000

./mutilate -s slave4 --noload -i exponential \
-B -T 4 -Q 1000 -D 4 -C 4 \
-a slave4 -a p10 \
-c 4 -q 2000

#### test 
./mutilate -s slave4 --noload -i exponential \
-B -T 4 -Q 1000 -D 4 -C 4 \
-a p09  \
-c 4 -q 20000

#### cluster-mode
##### on master 
./mutilate -s slave4 --noload -i pareto:0.0,16.0292,0.154971  -B -T 4 -Q 1000 -D 4 -C 4 -a p09 -a p10 -c 4 -q 20000

./mutilate -s slave4 --noload -i pareto:0.0,16.0292,0.154971  -B -T 4 -Q 10000 -D 4 -C 16 -a p09 -a p10 -c 16 -q 200000 -t 10


./mutilate -s slave4:11211 --noload -i pareto:0.0,16.0292,0.154971  -B -T 4 -Q 1000 -D 4 -C 4 -a p09 -a p10 -c 4 -t 10
./mutilate -s slave4:11211 -T 10 -c 8 -i exponential  -t 10

#### example 
./mutilate -s slave4 --noload -i pareto:0.0,16.0292,0.154971  -B -T 4 -Q 100 -D 4 -C 4 -a p09 -a p10 -c 8 -d 1000 -q 2000 -t 10

##### on agent
on p09
cd /home/lemaker/open-source/mutilate/build
./mutilate -T 4 -A
./mutilate -T 16 -d 16 -A



### 5 seconds, 10 threads, and 8 connections
Facebook "gev:30.7984,8.20449,0.078688", key-size distribution
./mutilate -s localhost:11211 -T 10 -c 8 -t 5 -i gev:30.7984,8.20449,0.078688

### must set the number of connections <=8 
### example2 cluster
### on slave4
lemaker@slave4:~/open-source/mutilate/build$ 
./mutilate -s slave4:11211 --noload -i pareto:0.0,16.0292,0.154971  -B -T 4 -Q 10000 -D 4 -C 4 -a p09 -a p10 -c 10 -d 10 -t 10
-q 10000000

./mutilate -s slave4:11211 --noload -i exponential:1  -B -T 4 -Q 10000 -D 4 -C 4 -a p09 -a p10 -c 10 -d 10 -t 10 -q 10000000



p09
./mutilate -T 48 -A
p10
./mutilate -T 26 -A

/home/lemaker/open-source/mutilate/mutilate.cc(748): Local QPS = 9967.7 (99677 / 10.0s)
#type       avg     std     min     5th    10th    90th    95th    99th
read      149.6   581.5    34.0    60.7    63.9   263.7   367.7   672.7
update      0.0     0.0     0.0     0.0     0.0     0.0     0.0     0.0
op_q        1.1     0.3     1.0     1.0     1.0     1.1     2.0     2.1

Total QPS = 154882.6 (1548866 / 10.0s)

Misses = 0 (0.0%)
Skipped TXs = 0 (0.0%)

RX  382569902 bytes :   36.5 MB/s
TX   55964196 bytes :    5.3 MB/s


top -Hp 120333

# libevent2
### http://www.wangafu.net/~nickm/libevent-book/Ref3_eventloop.html
Bufferevents: concepts and basics
A "bufferevent" consists of an underlying transport(like a socket), a read buffer, and a write buffer. Instead of regular events, which give callbacks when the underlying transport is ready to be read or written, a bufferevent invokes its user-supplied callbacks when it has read or written enough data. 

- Write data to a bufferevent buffer. 
  - int bufferevent_write(struct bufferevent *bufev,
  const void *data, size_t size);
  
- Read a single line from an evbuffer.
  - char *evbuffer_readln(struct evbuffer *buffer, size_t *n_read_out,
    enum evbuffer_eol_style eol_style);

- Remove a specified number of bytes data from the beginning of an evbuffer.
  - int evbuffer_drain(struct evbuffer *buf, size_t len);
  
###  ZeroMQ Socket lifetime
https://zeromq.org/socket-api/
Socket lifetime
ZeroMQ sockets have a life in four parts, just like BSD sockets:

- Creating and destroying sockets, which go together to form a karmic circle of socket life

- Configuring sockets by setting options on them and checking them if necessary

- Plugging sockets into the network topology by creating ZeroMQ connections to and from them.

- Using the sockets to carry data by writing and receiving messages on them.

### exponential distribution
https://en.wikipedia.org/wiki/Exponential_distribution
### pareto distribution
https://en.wikipedia.org/wiki/Pareto_distribution

### uniform distribution
https://en.wikipedia.org/wiki/Continuous_uniform_distribution
https://en.wikipedia.org/wiki/Discrete_uniform_distribution


### Generating an exponential distribution random variable can be done by:

-ln(U)/lambda (where U~Uniform(0,1)).
More information can be found in this wikipedia article
https://en.wikipedia.org/wiki/Exponential_distribution#Generating_exponential_variates

https://www.ttested.com/generating-normal-random-variables-part-1/

In exponential distribution: lamda = 1/mean, so it gets you:

myVar = -ln(U) * mean (where U~Uniform(0,1)).
### Converting a Uniform Distribution to a Normal Distribution
https://www.baeldung.com/cs/uniform-to-normal-distribution

### gengetopt
https://www.gnu.org/software/gengetopt/gengetopt.html#Basic-Usage