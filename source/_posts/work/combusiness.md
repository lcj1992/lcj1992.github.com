---
layout: post
title: 一些常用业务逻辑
date: 2016-04-06
categories: work
tag: 短信验证码 订单同步 分布式锁 分布式事务 防抓取
---

###  1.防抓取&&防占座

#### 发送短信(图片)验证码 

1.  生成验证码,及对应的code (图片验证码类似,可以使用谷歌的kaptcha类库)

        public class ConfirmCodeUtil {

            private final static String SESSIONKEY_PREFIX = "aa_bb_xx";

            public static String generate() {
                String chars = "0123456789";
                char[] rands = new char[6];
                for (int i = 0; i < 6; i++) {
                    int rand = (int) (Math.random() * 9);
                    rands[i] = chars.charAt(rand);
                }
                return new String(rands);
            }

            //生成手机验证码session
            //生成图片验证码session
            public static String createSessionKey() {
                return System.currentTimeMillis() + DigestUtils.md5Hex(SESSIONKEY_PREFIX + new Random().nextInt());
            }

        }

2. 相关信息放入缓存(验证码,验证次数以及过期时间等),并把缓存的key作为cookie的值,存入客户端,如下

        co_qsid(cookie的key) :  memkey(cookie的值,也是验证码缓存的key)
        memkey:    verifyCode(验证码)

3.  通知用户

4.  验证:从cookie中获取对应的缓存的key,从缓存中获取对应key的值,比较request中的验证码和缓存中的.

#### 黑名单策略

#### url加密

### 2.订单同步

1.  业务方订单状态进行更改,发送消息给订单中心,附加有版本号version v1,标识订单当前的状态
2.  订单中心接收到消息,主动拉取业务方订单详情,此时订单详情也带一个version v2.
3.  应该是v2 >= v1,订单中心才会进行同步.否则校验出错.

### 3.分布式锁

setnx + expire 这个方案有个问题,一定几率出现数据被永久锁定,一个故障就是这样引起的.

     private boolean tryLockTask(TaskInfoType type) {
            String key = makeTaskKey(type);
            long ret = sedis.setnx(key, key);
            // 这里如果机器down掉,网络断了,数据就被永久锁定了.
            if (ret == 1L) {
                sedis.expire(key, 60);
                locked.set(true);
                return true;
            } else {
                locked.set(false);
                return false;
            }
        }

改进方案:
使用redis的

    SET key value [EX seconds] [PX milliseconds] [NX|XX]

    127.0.0.1:6379> set lockKey lockValue PX 5000 NX
    OK
    127.0.0.1:6379> get lockKey
    "lockValue"

对应于jedis的public String set(final String key, final String value, final String nxxx,final String expx, final long time);
