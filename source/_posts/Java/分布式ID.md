---
title: 分布式ID

categories: 
- 技术
- 分布式

tags:
- Java
- 分布式

description: 分布式ID的生成与使用

cover: https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/4k%20%E7%BE%8E%E5%A5%B3%20%E7%99%BD%E8%89%B2%E5%8F%A4%E8%A3%85%20%E5%8F%A4%E9%A3%8E%20%D0%A1%CF%AA%20%E7%80%91%E5%B8%83%20%E5%B2%A9%E7%9F%B3%20%E9%AB%98%E6%B8%85%20%E5%A3%81%E7%BA%B8_%E5%BD%BC%E5%B2%B8%E5%9B%BE%E7%BD%91.jpg
---

![4k 美女 白色古装 古风 СϪ 瀑布 岩石 高清 壁纸_彼岸图网](https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/4k%20%E7%BE%8E%E5%A5%B3%20%E7%99%BD%E8%89%B2%E5%8F%A4%E8%A3%85%20%E5%8F%A4%E9%A3%8E%20%D0%A1%CF%AA%20%E7%80%91%E5%B8%83%20%E5%B2%A9%E7%9F%B3%20%E9%AB%98%E6%B8%85%20%E5%A3%81%E7%BA%B8_%E5%BD%BC%E5%B2%B8%E5%9B%BE%E7%BD%91.jpg)

# 分布式ID

## 雪花算法

![image-20220720144035556](https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/image-20220720144035556.png)

共64位，这三个部分是保证唯一性的重要条件，缺一不可，但是可以根据业务需要调整他们占用的位数，添加其他的信息，比如业务代码。

## 工作流程

![img](https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/1734261-20220210191137958-980879319.png)

还需要配合Redis一起使用

## 雪花算法生成ID配合Redis使用实现集群唯一ID(长度为18位)

```java
/**
 * @author sungm
 * @since 2021-11-06 21:16
 */
@Slf4j
@Service
public class SnowflakeManager {

    /** 开始时间戳: 2020-01-01 00:00:00 */
    private static final Long START_TIMESTAMP = 1577808000000L;
    /** 12位最大序号: 2^12 - 1 */
    private static final Long MAX_SEQ = ~(-1L << 12);
    /** 10位最大机器码： 2^10 -1 */
    private static final Long MAX_MACHINE = ~(-1L << 10);

    /** 当前机器码 */
    private Long machine;
    /** 最后生成的序号 */
    private Long lastSeq = 0L;
    /** 最后一个序号生成的时间 */
    private Long lastSqlTimestamp = 0L;

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    /** 定义雪花算法的 Key，把机器码存到Redis中 */
    private static final String RECORD_SNOWFLAKE_MACHINE_SEQ_REDIS_KEY = "RECORD_SNOWFLAKE_MACHINE_SEQ";
    private static final String RECORD_SNOWFLAKE_MACHINE_MAP_REDIS_KEY = "RECORD_SNOWFLAKE_MACHINE_MAP";

    @PostConstruct
    public void init() throws UnknownHostException {
        //获取当前机器的IP地址
        final String hostAddress = InetAddress.getLocalHost().getHostAddress();
        //初始化Redis缓存
        stringRedisTemplate.opsForValue().setIfAbsent(RECORD_SNOWFLAKE_MACHINE_SEQ_REDIS_KEY, "0");
        stringRedisTemplate.opsForHash().putIfAbsent(RECORD_SNOWFLAKE_MACHINE_MAP_REDIS_KEY, "default", "0");
        //不包含当前主机IP地址时，设置递增的值
        if (!stringRedisTemplate.opsForHash().keys(RECORD_SNOWFLAKE_MACHINE_MAP_REDIS_KEY).contains(hostAddress)) {
            stringRedisTemplate.opsForHash().put(RECORD_SNOWFLAKE_MACHINE_MAP_REDIS_KEY, hostAddress
                    , stringRedisTemplate.opsForValue().increment(RECORD_SNOWFLAKE_MACHINE_SEQ_REDIS_KEY, 1L).toString());
        }
        //获取当前主机对应的编码
        machine = Long.parseLong((String) stringRedisTemplate.opsForHash().get(RECORD_SNOWFLAKE_MACHINE_MAP_REDIS_KEY, hostAddress));
        log.info("主机：{}，机器码：{}", hostAddress, machine);
        //做个校验
        if (machine > MAX_MACHINE) {
            throw new RuntimeException("机器码已达到最大值" + MAX_MACHINE + ", 请排查无效数据！");
        }
    }

    public synchronized Long nextId() {
        //获取当前时间
        Long now = System.currentTimeMillis();
        if (lastSqlTimestamp.equals(now) && ++lastSeq > MAX_SEQ) {
            throw new RuntimeException("同一毫秒内生成的序号达到" + MAX_SEQ + ", 请注意并发量！");
        }
        if (!lastSqlTimestamp.equals(now)) {
            lastSeq = 0L;
        }
        lastSqlTimestamp = now;
        /* 0 - 41位时间戳 - 10位机器码 - 12位序列*/
        return ((now - START_TIMESTAMP) << 22) | machine << 12 | lastSeq;
    }

}
```

