# KongInstruction
Instruction and details of Kong Implementation

先安kong（官网） 那个包通过git弄上去 然后
sudo yum install kong-0.10.1.aws.rpm --nogpgcheck
kong安好了 检查各种东西吧

因为kong现在的版本要cassandra3.0+ 然而cassandra3.0+要 java1.8 所以 教程有毛病啊 俩坑
java -version 
java version "1.7.0_131" (这个是默认的吧 要升级到1.8 这个看个结果就行)
OpenJDK Runtime Environment (amzn-2.6.9.0.71.amzn1-x86_64 u131-b00)
OpenJDK 64-Bit Server VM (build 24.131-b00, mixed mode)

Install Java 8
sudo yum install java-1.8.0-openjdk-devel
** Select Java 8 Option for: **
sudo /usr/sbin/alternatives --config java
choose 1.8
sudo /usr/sbin/alternatives --config javac
NOTE:  
JAVA_HOME = /usr/lib/jvm/java-1.8.0-openjdk.x86_64

export JAVA_HOME=/opt/jdk1.8.0_121
export JRE_HOME=/opt/jdk1.8.0_121/jre
export PATH=$PATH:/opt/jdk1.8.0_121/bin:/opt/jdk1.8.0_121/jre/bin

mkdir /cassandra
cd /cassandra

choose Option2
curl -OL http://apache.claz.org/cassandra/redhat/30x/cassandra-3.0.13-1.noarch.rpm 

rpm -i cassandra-3.0.13-1.noarch.rpm
sudo service cassandra start 
Starting Cassandra: OK
service cassandra status 
cassandra (pid  3707) is running...  （这个是正确的）
可能的坑：
cassandra dead but pid file exists
这个表示你cassandra 没有成功起来 原因是之前开的时候权限不够 开了个假的 占地方了 后边的开不开了
解决办法：cd /var/run/cassandra  删里边的pid文件

cd /usr/bin
cqlsh
这块试一下cassandra好使不 好使应该近个shell里 用quit退出

回到kong 还没配置数据库  kong start 会出错
cd /etc/kong
sudo chmod 777 kong.conf.default 
vi kong.conf.default
把database 改成cassandra

kong start --conf /etc/kong/kong.conf.default
kong stop --conf /etc/kong/kong.conf.default   

ulimit is currently set to "1024". For better performance set it to at least "4096" using "ulimit -n"
migrating core for keyspace kong
core migrated up to: 2015-01-12-175310_skeleton
core migrated up to: 2015-01-12-175310_init_schema
core migrated up to: 2015-11-23-817313_nodes
core migrated up to: 2016-02-25-160900_remove_null_consumer_id
core migrated up to: 2016-02-29-121813_remove_ttls
core migrated up to: 2016-09-05-212515_retries_step_1
core migrated up to: 2016-09-05-212515_retries_step_2
core migrated up to: 2016-09-16-141423_upstreams
core migrated up to: 2016-12-14-172100_move_ssl_certs_to_core
core migrated up to: 2016-11-11-151900_new_apis_router_1
core migrated up to: 2016-11-11-151900_new_apis_router_2
core migrated up to: 2016-11-11-151900_new_apis_router_3
core migrated up to: 2017-01-24-132600_upstream_timeouts
core migrated up to: 2017-01-24-132600_upstream_timeouts_2
migrating request-transformer for keyspace kong
request-transformer migrated up to: 2016-03-10-160000_req_trans_schema_changes
migrating rate-limiting for keyspace kong
rate-limiting migrated up to: 2015-08-03-132400_init_ratelimiting
rate-limiting migrated up to: 2016-07-25-471385_ratelimiting_policies
migrating hmac-auth for keyspace kong
hmac-auth migrated up to: 2015-09-16-132400_init_hmacauth
migrating jwt for keyspace kong
jwt migrated up to: 2015-06-09-jwt-auth
jwt migrated up to: 2016-03-07-jwt-alg
migrating response-ratelimiting for keyspace kong
response-ratelimiting migrated up to: 2015-08-21_init_response-rate-limiting
response-ratelimiting migrated up to: 2016-08-04-321512_response-rate-limiting_policies
migrating ip-restriction for keyspace kong
ip-restriction migrated up to: 2016-05-24-remove-cache
migrating cors for keyspace kong
cors migrated up to: 2017-03-14_multiple_orgins
migrating galileo for keyspace kong
galileo migrated up to: 2016-04-15_galileo-import-mashape-analytics
migrating acl for keyspace kong
acl migrated up to: 2015-08-25-841841_init_acl
migrating basic-auth for keyspace kong
basic-auth migrated up to: 2015-08-03-132400_init_basicauth
migrating oauth2 for keyspace kong
oauth2 migrated up to: 2015-08-03-132400_init_oauth2
oauth2 migrated up to: 2015-08-24-215800_cascade_delete_index
oauth2 migrated up to: 2016-02-29-435612_remove_ttl
oauth2 migrated up to: 2016-04-14-283949_serialize_redirect_uri
oauth2 migrated up to: 2016-07-15-oauth2_code_credential_id
oauth2 migrated up to: 2016-09-19-oauth2_code_index
oauth2 migrated up to: 2016-09-19-oauth2_api_id
oauth2 migrated up to: 2016-12-15-set_global_credentials
migrating response-transformer for keyspace kong
response-transformer migrated up to: 2016-03-10-160000_resp_trans_schema_changes
migrating key-auth for keyspace kong
key-auth migrated up to: 2015-07-31-172400_init_keyauth
37 migrations ran
Kong started
kong运行成功了！
kong check /etc/kong/kong.conf 
curl 127.0.0.1:8001
测试
测试用例： 建一个nodejs ec2 后台
参考：http://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html
进程序目录后 node app.js   这个语句负责开console跑node程序 给你看结果用 
 测试index主页然后 kong桥接设置好
之后用浏览器带插件设置header 访问kong的8000 要求自动跳转给nodejs ec2 后台

加个我们新的桥接api
curl -i -X POST \
  --url http://localhost:8001/apis/ \
  --data 'name=jintengtest' \
  --data 'hosts=jinteng' \
  --data 'upstream_url=http://ec2-54-219-163-27.us-west-1.compute.amazonaws.com:3000'  
curl -i -X DELETE \
  --url http://localhost:8001/apis/ \
  --data 'name=kongTest3' 




测试方法 去找一个chrome extension东西  叫 modheader  然后如图片里一样设置 把header 设置好  
然后地址输kong的：8000   这样kong给你跳转到你后台的backend的地址的网页了 成功了哈哈哈
html css都会成功跳转 那么看来kong还是很智能的 都给转过去了

kong拓荒完成了 我估计cluster是小事儿了


查看所有的apis
curl -i -X GET\
  --url http://localhost:8001/apis/ 
查看单独api
curl -i -X GET url http://localhost:8001/apis/{name or id}
删除api
curl -i -X DELETE url http://localhost:8001/apis/{name or id}
Update or create api
curl -i -X PUT url http://localhost:8001/apis/
