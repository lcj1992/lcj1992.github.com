---
layout: post
title: 负载均衡算法(notes)
date: 2015-12-27
categories: algorithm
tags: load_balance
---

常用的负载均衡算法包括`轮询法(round robin)`，`随机法`，`源地址哈希法`，`加权轮询法`，`加权随机法`，`最小连接数算法`,`sticky`

这里初始化一个serverWeightMap的Map变量来表示服务器地址和权重的映射，以此来模拟轮询算法的实现，其中设置的权重值在后面加权算法会使用到

    public static Map<String, Integer> serverWeightMap = Maps.newHashMap();
    static Integer pos = 0;
    static {
        serverWeightMap.put("192.168.1.100", 1);
        serverWeightMap.put("192.168.1.101", 1);
        serverWeightMap.put("192.168.1.102", 4);
        serverWeightMap.put("192.168.1.103", 1);
        serverWeightMap.put("192.168.1.104", 1);
        serverWeightMap.put("192.168.1.105", 3);
        serverWeightMap.put("192.168.1.106", 1);
        serverWeightMap.put("192.168.1.107", 2);
        serverWeightMap.put("192.168.1.108", 1);
        serverWeightMap.put("192.168.1.109", 1);
        serverWeightMap.put("192.168.1.110", 1);
    }

#### 轮询法

将请求按顺序轮流地分配的后端服务器上，它均衡的对待后端每一台服务器，而不关心服务器实际的连接数和当前的系统负载

    public static String getByRoundRobin() {
        Map<String, Integer> serverMap = Maps.newHashMap();
        serverMap.putAll(serverWeightMap);
        Set<String> keySet = serverMap.keySet();
        List<String> keyList = Lists.newArrayList();
        keyList.addAll(keySet);

        String server = null;
        synchronized (pos) {
            if (pos >= keySet.size()) {
                pos = 0;
            }

            server = keyList.get(pos);
            pos++;
        }
        return server;
    }

由于serverWeightMap中的地址列表是动态的，随时可能有机器上线，下线或者宕机，因此，为了避免可能出现的并发问题，如数组越界，通过新建方法内的局部变量serverMap，先将域变量复制到线程本地，以避免被多个线程修改，这样可能会引入新的问题，复制以后serverWeightMap的修改将无法反映给serverMap，也就是说，在这一轮选择服务器的过程中，新增服务器或者下线服务器，负载均衡算法中将无法获知，新增比较好处理，而当服务器下线或者宕机时，服务消费者将有可能访问到不存在的地址。因此，在服务消费者的实现端需要考虑该问题，并且进行相应的容错处理，比如重新发起一次调用。

对于当前轮询的位置变量pos，为了保证服务器选择的顺序性，需要在操作时对其加上synchronized锁，使得在同一时刻只有一个线程能够修改pos的值，否则当pos变量被并发修改，则无法保证服务器选择的顺序性，甚至有可能导致keyList数组越界。

使用轮询策略的目的在于，希望做到请求转移的绝对均衡，但付出的代价也是相当大的。为了pos保证变量修改的互斥性，需要引入重量级的悲观锁synchronized，将会导致该段轮询代码的并发吞吐量发生明显的下降。

#### 随机法

通过系统随机函数，根据后端服务器列表的大小值来随机的选择其中一台进行访问。由概率统计理论可以得知，随着调用量的增大，其实际效果越来越接近于平均分配流量到没一台后端服务器，也就是轮询的效果。

    public static String getByRondom() {
        Map<String, Integer> serverMap = Maps.newHashMap();
        serverMap.putAll(serverWeightMap);
        Set<String> keySet = serverMap.keySet();
        List<String> keyList = Lists.newArrayList();
        keyList.addAll(keySet);

        String server = null;

        Random random = new Random();
        int randomPos = random.nextInt(keySet.size());
        server = keyList.get(randomPos);
        return server;
    }

基于概率统计的理论，吞吐量越大，随机算法的效果越接近轮询算法的效果。因此，你还会考虑一定要使用需要付出一定代价的轮询算法么？

#### 源地址哈希法

源地址哈希的思想是获取客户端访问的ip地址值，通过哈希函数计算得到一个数值，用该数值对服务器列表的大小进行取模运算，得到的结果便是要访问的服务器的序号。采用哈希法进行负载均衡，同一个ip地址的客户端，当后端服务器列表不变时，它每次都会被映射到同一台后端服务器进行访问。

    public static String getServerByConsumerHash(String remoteIp) {
        Map<String, Integer> serverMap = Maps.newHashMap();
        serverMap.putAll(serverWeightMap);
        Set<String> keySet = serverMap.keySet();
        List<String> keyList = Lists.newArrayList();
        keyList.addAll(keySet);

        String server = null;

        int hashCode = remoteIp.hashCode();
        int serverListSize = keyList.size();
        int serverPos = hashCode % serverListSize;
        server = keyList.get(serverPos);

        return server;
    }

通过参数传入的客户端remoteip参数，取得它的哈希值，对服务器列表的大小取模，结果便是选用的服务器在服务器列表中的索引值。该算法保证了相同的客户端ip地址将会被“哈希”到同一台后端服务器，直到后端服务器列表变更。根据此特性可以在服务消费者和服务提供者之间建立有状态的session会话。

#### 加权轮询法

不同的后端服务器可能机器的配置和当前系统的负载并不相同，因此他们的抗压能力也不尽相同，给配置高，负载低的机器配置更高的权重，让其处理更过的请求，而地配置，负载高的机器，则给其分配较低的权重，降低其系统负载，加权轮询能很好地处理这一问题，并将请求顺序且按照权重分配到后端。

    public static String getByWeightRoundRobin() {
        Map<String, Integer> serverMap = Maps.newHashMap();
        serverMap.putAll(serverWeightMap);
        Set<String> keySet = serverMap.keySet();
        Iterator<String> it = keySet.iterator();
        List<String> serverList = Lists.newArrayList();

        while (it.hasNext()) {
            String server = it.next();
            Integer weight = serverMap.get(server);
            for (int i = 0; i < weight; i++) {
                serverList.add(server);
            }
        }
        String server = null;
        synchronized (pos) {
            if (pos >= serverList.size()) {
                pos = 0;
            }

            server = serverList.get(pos);
        pos ++;    }

        return server;
    }

与轮询算法类似，只是在获取服务器地址之前增加了一段权重计算的代码，根据权重的大小，将地址重复增加到服务器地址列表中，权重越大，该服务器每轮所获得的请求数量越多。

#### 加权随机法

    public static String getByWeightRandom() {
        Map<String, Integer> serverMap = Maps.newHashMap();
        serverMap.putAll(serverWeightMap);
        Set<String> keySet = serverMap.keySet();
        Iterator<String> it = keySet.iterator();
        List<String> serverList = Lists.newArrayList();

        while (it.hasNext()) {
            String server = it.next();
            Integer weight = serverMap.get(server);
            for (int i = 0; i < weight; i++) {
                serverList.add(server);
            }
        }
        String server = null;
        Random random = new Random();
        int randomPos = random.nextInt(serverList.size());
        server = serverList.get(randomPos);
        return server;
    }

#### 最小连接数算法

最小连接数算法比较灵活和智能，由于后端服务器的配置不尽相同，对于请求的处理有快有慢，它正是根据后端服务器当前的链接情况，动态的选取其中积压连接数最少的一台服务器来处理当前请求，尽可能的提高后端服务器的利用效率，将负载合理的分流到每一台机器。

#### sticky

保证始终只在一台处理，如果这台服务器挂了，会自动切换到下一台上。

#### 一致性hash

主要用于分布式缓存集群,避免扩容或宕机带来的命中率急剧下降.


先构造一个长度为2^32的整数环(这个环被称作一致性hash环)根据节点名称的hash值(其分布范围为\[0,0^32 -1\]),将缓存服务器节点放置在这个Hash环上,然后根据需要缓存的数据的Key值计算得到其Hash值(其
分布范围也同样为\[0,2^32-1\]),然后在Hash环上顺时针查找距离这个Key的Hash值最近的缓存服务器节点,完成Key到服务器的Hash映射查找


这个长度为2^32的一致性Hash环通常使用二叉查找树实现,Hash查找过程实际上是在二叉查找树中查找小于查找树的最小数值,当然这个二叉树的最右边叶子节点和最左边的叶子节点相连接,构成环.

还有问题,新加入节点后,节点之间的数据负载是不一样的,所以又引入虚拟节点的概念,就是将物理缓存服务器虚拟为一组虚拟缓存服务器,分散在整个环上.

       public class ConsistentHash<T> {

           private final HashFunction hashFunction;
           private final int numberOfReplicas;
           private final SortedMap<Integer, T> circle = new TreeMap<Integer, T>();

           public ConsistentHash(HashFunction hashFunction, int numberOfReplicas,
                                 Collection<T> nodes) {
               this.hashFunction = hashFunction;
               this.numberOfReplicas = numberOfReplicas;

               for (T node : nodes) {
                   add(node);
               }
           }

           public void add(T node) {
               for (int i = 0; i < numberOfReplicas; i++) {
                   circle.put(hashFunction.hash(node.toString() + i), node);
               }
           }

           public void remove(T node) {
               for (int i = 0; i < numberOfReplicas; i++) {
                   circle.remove(hashFunction.hash(node.toString() + i));
               }
           }

           public T get(Object key) {
               if (circle.isEmpty()) {
                   return null;
               }
               int hash = hashFunction.hash(key);
               if (!circle.containsKey(hash)) {
                   SortedMap<Integer, T> tailMap = circle.tailMap(hash);
                   hash = tailMap.isEmpty() ? circle.firstKey() : tailMap.firstKey();
               }
               return circle.get(hash);
           }

       }

#### 参考

[Consistent Hashing Blog]<https://community.oracle.com/blogs/tomwhite/2007/11/27/consistent-hashing>    
