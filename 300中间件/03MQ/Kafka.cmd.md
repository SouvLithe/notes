## 启动服务
```bash
cd /tools/kafka #进入目录
sh start-kafka.sh #启动kafka
```

### 创建主题
```bash
cd /tools/kafka

# 基本创建
bin/kafka-topics.sh --create --topic my-topic --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1

# 带更多配置的创建
bin/kafka-topics.sh --create \
  --topic my-topic \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 1 \
  --config retention.ms=604800000
```

### 查看所有主题
```bash
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

### 查看主题详情
```bash
# 查看特定主题详情
bin/kafka-topics.sh --describe --topic my-topic --bootstrap-server localhost:9092

# 查看所有主题详情
bin/kafka-topics.sh --describe --bootstrap-server localhost:9092
```

### 删除主题
```bash
bin/kafka-topics.sh --delete --topic my-topic --bootstrap-server localhost:9092
```

## 常用测试命令
### 生产消息
```bash
cd /tools/kafka

# 启动生产者（交互式）
bin/kafka-console-producer.sh --topic my-topic --bootstrap-server localhost:9092

# 一次性发送消息
echo "Hello Kafka" | bin/kafka-console-producer.sh --topic my-topic --bootstrap-server localhost:9092
```

### 消费消息
```bash
cd /tools/kafka

# 从头开始消费
bin/kafka-console-consumer.sh --topic my-topic --from-beginning --bootstrap-server localhost:9092

# 实时消费新消息
bin/kafka-console-consumer.sh --topic my-topic --bootstrap-server localhost:9092

# 消费特定分区的消息
bin/kafka-console-consumer.sh --topic my-topic --partition 0 --bootstrap-server localhost:9092
```

## 不同用途的主题配置
```bash
# 日志主题（保留7天）
bin/kafka-topics.sh --create \
  --topic app-logs \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 1 \
  --config retention.ms=604800000 \
  --config cleanup.policy=delete

# 事件主题（保留1天，紧凑型）
bin/kafka-topics.sh --create \
  --topic user-events \
  --bootstrap-server localhost:9092 \
  --partitions 6 \
  --replication-factor 1 \
  --config retention.ms=86400000 \
  --config cleanup.policy=compact

# 监控指标主题（保留1小时）
bin/kafka-topics.sh --create \
  --topic metrics \
  --bootstrap-server localhost:9092 \
  --partitions 1 \
  --replication-factor 1 \
  --config retention.ms=3600000
```
