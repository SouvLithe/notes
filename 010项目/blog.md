# 数据库搭建
es和redis都在docker中
```
es 密码：147258 用户：elastic
redis 密码：123321
```
1. 建库
```sql
CREATE DATABASE blog DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
2. 建表
1.用户表：
```sql
CREATE TABLE `user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `username` varchar(32) NOT NULL COMMENT '用户名（唯一）',
  `password` varchar(128) NOT NULL COMMENT '密码（加密后）',
  `nickname` varchar(32) DEFAULT NULL COMMENT '昵称',
  `email` varchar(64) NOT NULL COMMENT '邮箱',
  `avatar` varchar(255) DEFAULT NULL COMMENT '头像URL',
  `role` tinyint(4) NOT NULL DEFAULT '1' COMMENT '角色：0-管理员，1-普通用户',
  `status` tinyint(4) NOT NULL DEFAULT '1' COMMENT '状态：0-禁用，1-正常',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '注册时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_username` (`username`),
  UNIQUE KEY `uk_email` (`email`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```
2.分类栏目表：
```sql
CREATE TABLE `category` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(32) NOT NULL COMMENT '分类名称',
  `slug` varchar(64) NOT NULL COMMENT 'URL标识（唯一）',
  `description` varchar(128) DEFAULT NULL COMMENT '描述',
  `sort` int(11) NOT NULL DEFAULT '0' COMMENT '排序值（越小越靠前）',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_slug` (`slug`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='分类表';
```
3.标签表：
```sql
CREATE TABLE `tag` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(32) NOT NULL COMMENT '标签名',
  `slug` varchar(64) NOT NULL COMMENT 'URL标识',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_slug` (`slug`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='标签表';
```
4.文章表：
```sql
CREATE TABLE `article` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `title` varchar(200) NOT NULL COMMENT '文章标题',
  `summary` varchar(500) DEFAULT NULL COMMENT '摘要（可自动截取正文）',
  `content` longtext NOT NULL COMMENT '正文（Markdown或HTML）',
  `cover_image` varchar(255) DEFAULT NULL COMMENT '封面图URL',
  `category_id` bigint(20) NOT NULL COMMENT '所属分类ID',
  `author_id` bigint(20) NOT NULL COMMENT '作者ID（关联user.id）',
  `status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '状态：0-草稿，1-已发布，2-私密',
  `view_count` bigint(20) NOT NULL DEFAULT '0' COMMENT '阅读量（实际可从Redis累加，定时同步）',
  `like_count` bigint(20) NOT NULL DEFAULT '0' COMMENT '点赞数',
  `comment_count` bigint(20) NOT NULL DEFAULT '0' COMMENT '评论数',
  `is_top` tinyint(1) NOT NULL DEFAULT '0' COMMENT '是否置顶：0-否，1-是',
  `publish_time` datetime DEFAULT NULL COMMENT '发布时间（只有status=1时才有值）',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `idx_category_id` (`category_id`),
  KEY `idx_author_id` (`author_id`),
  KEY `idx_status_publish` (`status`, `publish_time`),
  KEY `idx_view_count` (`view_count`),      -- 用于热门文章排序
  FULLTEXT KEY `ft_title_content` (`title`, `summary`, `content`)  -- 简单全文搜索
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='文章表';
```
5.评论表：
```sql
CREATE TABLE `comment` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `article_id` bigint(20) NOT NULL COMMENT '所属文章ID',
  `user_id` bigint(20) NOT NULL COMMENT '评论人ID',
  `parent_id` bigint(20) NOT NULL DEFAULT '0' COMMENT '父评论ID（0表示顶级评论）',
  `reply_to_user_id` bigint(20) DEFAULT NULL COMMENT '回复的目标用户ID（冗余，便于展示）',
  `content` varchar(1000) NOT NULL COMMENT '评论内容',
  `like_count` int(11) NOT NULL DEFAULT '0',
  `status` tinyint(4) NOT NULL DEFAULT '0' COMMENT '0-待审核，1-已通过，2-已删除',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `idx_article_id` (`article_id`),
  KEY `idx_parent_id` (`parent_id`),
  KEY `idx_user_id` (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='评论表';
```
6.关联表：
```sql
CREATE TABLE `article_tag` (
  `article_id` bigint(20) NOT NULL,
  `tag_id` bigint(20) NOT NULL,
  PRIMARY KEY (`article_id`, `tag_id`),
  KEY `idx_tag_id` (`tag_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='文章标签关联表';
```
7.操作日志表
记录后台敏感操作，用于安全审计。
```sql
CREATE TABLE `operation_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `user_id` bigint(20) NOT NULL,
  `operation` varchar(50) NOT NULL COMMENT '操作类型（如DELETE_ARTICLE）',
  `target_id` bigint(20) DEFAULT NULL COMMENT '操作对象ID',
  `detail` varchar(500) DEFAULT NULL COMMENT '详情',
  `ip` varchar(45) DEFAULT NULL,
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `idx_user_id` (`user_id`),
  KEY `idx_create_time` (`create_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='操作日志表';
```
# 后端
测试数据：
```sql
-- 关闭外键检查，便于插入
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- 1. 用户表 (user)
-- ----------------------------
INSERT INTO `user` (`id`, `username`, `password`, `nickname`, `email`, `avatar`, `role`, `status`, `create_time`, `update_time`) VALUES
(1, 'admin', '$2a$10$N.zmdr9k7uOCQb376NoUnuTJ8iAt6Z5EHsM8lE9lBOsl7iKTVKIUi', '管理员', 'admin@blog.com', NULL, 0, 1, NOW(), NOW()),
(2, 'zhangsan', '$2a$10$N.zmdr9k7uOCQb376NoUnuTJ8iAt6Z5EHsM8lE9lBOsl7iKTVKIUi', '张三', 'zhangsan@test.com', NULL, 1, 1, NOW(), NOW()),
(3, 'lisi', '$2a$10$N.zmdr9k7uOCQb376NoUnuTJ8iAt6Z5EHsM8lE9lBOsl7iKTVKIUi', '李四', 'lisi@test.com', NULL, 1, 1, NOW(), NOW()),
(4, 'wangwu', '$2a$10$N.zmdr9k7uOCQb376NoUnuTJ8iAt6Z5EHsM8lE9lBOsl7iKTVKIUi', '王五', 'wangwu@test.com', NULL, 1, 0, NOW(), NOW());

-- ----------------------------
-- 2. 分类表 (category)
-- ----------------------------
INSERT INTO `category` (`id`, `name`, `slug`, `description`, `sort`, `create_time`) VALUES
(1, '技术', 'tech', '技术相关文章，涵盖编程、架构等', 1, NOW()),
(2, '生活', 'life', '生活随笔、感悟', 2, NOW()),
(3, '旅行', 'travel', '旅行见闻与攻略', 3, NOW());

-- ----------------------------
-- 3. 标签表 (tag)
-- ----------------------------
INSERT INTO `tag` (`id`, `name`, `slug`, `create_time`) VALUES
(1, 'Java', 'java', NOW()),
(2, 'Spring Boot', 'spring-boot', NOW()),
(3, '微服务', 'microservice', NOW()),
(4, '旅行游记', 'travel-note', NOW()),
(5, '美食', 'food', NOW());

-- ----------------------------
-- 4. 文章表 (article)
-- ----------------------------
INSERT INTO `article` (`id`, `title`, `summary`, `content`, `cover_image`, `category_id`, `author_id`, `status`, `view_count`, `like_count`, `comment_count`, `is_top`, `publish_time`, `create_time`, `update_time`) VALUES
(1, 'Spring Boot 入门指南', '快速上手 Spring Boot 框架', '本文详细介绍了 Spring Boot 的基本用法，包括自动配置、Starter 依赖、REST API 开发等。', NULL, 1, 1, 1, 150, 25, 5, 1, '2025-01-10 10:00:00', NOW(), NOW()),
(2, '如何高效学习 Java', '分享 Java 学习路线与实用技巧', '从基础语法到并发编程，再到 JVM 调优，本文为你规划了一条高效的学习路径。', NULL, 1, 2, 1, 80, 8, 2, 0, '2025-02-15 14:30:00', NOW(), NOW()),
(3, '微服务架构设计思考', '探讨微服务拆分、通信与数据一致性', '本文分析了微服务架构的优缺点，并结合实际案例讨论如何应对分布式事务挑战。', NULL, 1, 1, 0, 0, 0, 0, 0, NULL, NOW(), NOW()),
(4, '我的私人日记', '仅自己可见的个人记录', '今天天气很好，心情也不错。', NULL, 2, 2, 2, 0, 0, 0, 0, NULL, NOW(), NOW()),
(5, '西藏旅行记', '布达拉宫、纳木错、珠峰大本营的难忘经历', '一路向西，感受高原的壮美与神秘。', NULL, 3, 3, 1, 220, 42, 8, 0, '2025-03-01 08:00:00', NOW(), NOW());

-- ----------------------------
-- 5. 评论表 (comment)
-- ----------------------------
INSERT INTO `comment` (`id`, `article_id`, `user_id`, `parent_id`, `reply_to_user_id`, `content`, `like_count`, `status`, `create_time`) VALUES
(1, 1, 2, 0, NULL, '写得非常清晰，感谢分享！', 2, 1, NOW()),
(2, 1, 3, 1, 2, '回复 @张三：是的，确实入门好文。', 1, 1, NOW()),
(3, 1, 4, 0, NULL, '请问 Spring Boot 3 与 2 的区别大吗？', 0, 0, NOW()),  -- 待审核
(4, 2, 1, 0, NULL, 'Java 学习路线非常实用，收藏了。', 1, 1, NOW()),
(5, 2, 3, 4, 1, '回复 @管理员：我也觉得很有帮助。', 0, 1, NOW()),
(6, 5, 2, 0, NULL, '西藏一直想去，看完更心动了！', 3, 1, NOW());

-- ----------------------------
-- 6. 文章-标签关联表 (article_tag)
-- ----------------------------
INSERT INTO `article_tag` (`article_id`, `tag_id`) VALUES
(1, 1),  -- 文章1 → Java
(1, 2),  -- 文章1 → Spring Boot
(2, 1),  -- 文章2 → Java
(5, 4);  -- 文章5 → 旅行游记

-- ----------------------------
-- 7. 操作日志表 (operation_log)
-- ----------------------------
INSERT INTO `operation_log` (`id`, `user_id`, `operation`, `target_id`, `detail`, `ip`, `create_time`) VALUES
(1, 1, 'DELETE_COMMENT', 3, '删除待审核评论（内容：请问 Spring Boot 3 与 2 的区别大吗？）', '192.168.1.100', NOW()),
(2, 1, 'UPDATE_ARTICLE', 3, '将文章「微服务架构设计思考」保存为草稿', '192.168.1.101', NOW());

-- 重新开启外键检查
SET FOREIGN_KEY_CHECKS = 1;
```
