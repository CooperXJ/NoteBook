使用步骤

 1. 下载Redis并启动服务

 2. 导入相应maevn

    ```xml
    <!--redis依赖配置-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    ```

	3. 修改SpringBoot配置文件配置Redis

    在spring节点下进行配置

    ```yaml
    redis:
      host: localhost # Redis服务器地址
      database: 0 # Redis数据库索引（默认为0）
      port: 6379 # Redis服务器连接端口
      password: # Redis服务器连接密码（默认为空）
      jedis:
        pool:
          max-active: 8 # 连接池最大连接数（使用负值表示没有限制）
          max-wait: -1ms # 连接池最大阻塞等待时间（使用负值表示没有限制）
          max-idle: 8 # 连接池中的最大空闲连接
          min-idle: 0 # 连接池中的最小空闲连接
      timeout: 3000ms # 连接超时时间（毫秒）
    ```

    也可以自定义一些key的配置

    ```yaml
    # 自定义redis key
    redis:
      key:
        prefix:
          authCode: "portal:authCode:"
        expire:
          authCode: 120 # 验证码超期时间
    ```

	4. 定义操作Redis的接口以及对应的实现类

    ```java
    public interface RedisService {
        /**
         * 存储数据
         */
        void set(String key, String value);
    
        /**
         * 获取数据
         */
        String get(String key);
    
        /**
         * 设置超期时间
         */
        boolean expire(String key, long expire);
    
        /**
         * 删除数据
         */
        void remove(String key);
    
        /**
         * 自增操作
         * @param delta 自增步长
         */
        Long increment(String key, long delta);
    
    }
    ```

    ```java
    @Service
    public class RedisServiceImpl implements RedisService {
        @Autowired
        private StringRedisTemplate stringRedisTemplate;
    
        @Override
        public void set(String key, String value) {
            stringRedisTemplate.opsForValue().set(key, value);
        }
    
        @Override
        public String get(String key) {
            return stringRedisTemplate.opsForValue().get(key);
        }
    
        @Override
        public boolean expire(String key, long expire) {
            return stringRedisTemplate.expire(key, expire, TimeUnit.SECONDS);
        }
    
        @Override
        public void remove(String key) {
            stringRedisTemplate.delete(key);
        }
    
        @Override
        public Long increment(String key, long delta) {
            return stringRedisTemplate.opsForValue().increment(key,delta);
        }
    }
    ```