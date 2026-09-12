# 建立数据库video
数据库是一个兜底方案，避免业务层乱搞，避免后期清洗整理数据
```sql
CREATE DATABASE easylive
CHARACTER SET utf8mb4
COLLATE utf8mb4_general_ci;
```
## user_info表
![[Pasted image 20260426101446.png]]
其中status默认为1，theme默认为1.
索引：
![[Pasted image 20260426101651.png]]
sql语句：
```sql
CREATE TABLE `user_info` (
    `user_id` varchar(10) NOT NULL COMMENT '用户id',
    `nick_name` varchar(20) NOT NULL COMMENT '昵称',
    `email` varchar(150) NOT NULL COMMENT '邮箱',
    `password` varchar(50) NOT NULL COMMENT '密码',
    `sex` tinyint(1) DEFAULT NULL COMMENT '性别：0:女 1:男 2:未知',
    `birthday` varchar(10) DEFAULT NULL COMMENT '出生日期',
    `school` varchar(150) DEFAULT NULL COMMENT '学校',
    `person_introduction` varchar(200) DEFAULT NULL COMMENT '个人简介',
    `join_time` datetime NOT NULL COMMENT '加入时间',
    `last_login_time` datetime DEFAULT NULL COMMENT '最后登录时间',
    `last_login_ip` varchar(15) DEFAULT NULL COMMENT '最后登录IP',
    `status` tinyint(1) NOT NULL DEFAULT '1' COMMENT '0:禁用 1:正常',
    `notice_info` varchar(300) DEFAULT NULL COMMENT '空间公告',
    `total_coin_count` int(11) NOT NULL COMMENT '硬币总数量',
    `current_coin_count` int(11) NOT NULL COMMENT '当前硬币数',
    `theme` tinyint(1) NOT NULL DEFAULT '1' COMMENT '主题',
    PRIMARY KEY (`user_id`),
    UNIQUE KEY `idx_key_email` (`email`),
    UNIQUE KEY `idx_nick_name` (`nick_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户信息';
```

| 缩写        | 全称                         | 含义                                | 使用场景      |
| --------- | -------------------------- | --------------------------------- | --------- |
| **VO**    | View Object / Value Object | 视图对象 / 值对象                        | 前端展示、接口返回 |
| **PO**    | Persistent Object          | 持久化对象                             | 数据库表映射    |
| **DTO**   | Data Transfer Object       | 层间传输数据（如 Service 和 Controller 之间） |           |
| **BO**    | Business Object            | 业务层处理的对象                          |           |
| **DO**    | Domain Object              | 领域对象（DDD 中使用）                     |           |
| **Query** | Query Object               | 封装查询条件                            |           |
po，就是服务端和数据库对应的。
vo，返回给前端额的。
dto，业务与业务之间传参定义的bean对象。

```java
// 这样可以用header中拿到token
@RequestHeader("token") String token
```

|对象|类比|生命周期|主要用途|
|---|---|---|---|
|`HttpServletRequest`|点菜单|一次请求|获取前端传来的数据|
|`HttpServletResponse`|上菜盘|一次请求|返回数据给前端|
|`HttpSession`|会员档案|整个会话（多次请求）|保存用户登录状态|
```sql
CREATE TABLE `category_info` (
    ->   `category_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '自增分类ID',
    ->   `category_code` varchar(30) NOT NULL COMMENT '分类编码',
    ->   `category_name` varchar(30) NOT NULL COMMENT '分类名称',
    ->   `p_category_id` int(11) NOT NULL COMMENT '父级分类ID',
    ->   `icon` varchar(50) DEFAULT NULL COMMENT '图标',
    ->   `background` varchar(50) DEFAULT NULL COMMENT '背景图',
    ->   `sort` tinyint(4) NOT NULL COMMENT '排序号',
    ->   PRIMARY KEY (`category_id`) USING BTREE,
    ->   UNIQUE KEY `idx_key_category_code` (`category_code`) USING BTREE
    -> ) ENGINE=InnoDB AUTO_INCREMENT=34 DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='分类信息';
```
## 视频部分
### 视频信息 - 投稿表
```sql
CREATE TABLE `video_info_post` (
    `video_id` varchar(10) NOT NULL DEFAULT '0' COMMENT '视频ID',
    `video_cover` varchar(50) NOT NULL COMMENT '视频封面',
    `video_name` varchar(100) NOT NULL COMMENT '视频名称',
    `user_id` varchar(10) NOT NULL COMMENT '用户ID',
    `create_time` datetime NOT NULL COMMENT '创建时间',
    `last_update_time` datetime NOT NULL COMMENT '最后更新时间',
    `p_category_id` int(11) NOT NULL COMMENT '父级分类ID',
    `category_id` int(11) DEFAULT NULL COMMENT '分类ID',
    `status` tinyint(1) NOT NULL COMMENT '0:转码中 1:转码失败 2:待审核 3:审核成功 4:审核失败',
    `post_type` tinyint(4) NOT NULL COMMENT '0:自制作品 1:转载',
    `origin_info` varchar(200) DEFAULT NULL COMMENT '原资源说明',
    `tags` varchar(300) DEFAULT NULL COMMENT '标签',
    `duration` int(11) DEFAULT NULL COMMENT '播放时长',
    `interaction` varchar(5) DEFAULT NULL COMMENT '互动设置',
    `keywords` varchar(500) DEFAULT NULL COMMENT '关键词'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='视频信息';
```
### 视频文件信息 - 投稿表
```sql
CREATE TABLE `video_info_file_post` (
    `file_id` varchar(20) NOT NULL COMMENT '唯一ID',
    `upload_id` varchar(15) NOT NULL COMMENT '上传ID',
    `user_id` varchar(10) NOT NULL COMMENT '用户ID',
    `video_id` varchar(10) NOT NULL COMMENT '视频ID',
    `file_index` int(11) NOT NULL COMMENT '文件索引',
    `file_name` varchar(200) DEFAULT NULL COMMENT '文件名',
    `file_size` bigint(20) DEFAULT NULL COMMENT '文件大小',
    `file_path` varchar(100) DEFAULT NULL COMMENT '文件路径',
    `update_type` tinyint(4) DEFAULT NULL COMMENT '0:无更新 1:有更新',
    `transfer_result` tinyint(4) DEFAULT NULL COMMENT '0:转码中 1:转码成功 2:转码失败',
    `duration` int(11) DEFAULT NULL COMMENT '持续时间 (秒)',
    PRIMARY KEY (`file_id`) USING BTREE,
    UNIQUE KEY `idx_key_upload_id` (`upload_id`, `user_id`) USING BTREE,
    KEY `idx_video_id` (`video_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='视频文件信息';
```
### 视频信息 - 主表
```sql
CREATE TABLE `video_info` (
    `video_id` varchar(10) NOT NULL COMMENT '视频ID',
    `video_cover` varchar(50) NOT NULL COMMENT '视频封面',
    `video_name` varchar(100) NOT NULL COMMENT '视频名称',
    `user_id` varchar(10) NOT NULL COMMENT '用户ID',
    `create_time` datetime NOT NULL COMMENT '创建时间',
    `last_update_time` datetime NOT NULL COMMENT '最后更新时间',
    `p_category_id` int(11) NOT NULL COMMENT '父级分类ID',
    `category_id` int(11) DEFAULT NULL COMMENT '分类ID',
    `post_type` tinyint(4) NOT NULL COMMENT '0: 自制作品  1: 转载',
    `origin_info` varchar(200) DEFAULT NULL COMMENT '原资源说明',
    `tags` varchar(300) DEFAULT NULL COMMENT '标签',
    `duration` int(11) DEFAULT NULL COMMENT '播放时长 (秒)',
    `play_count` int(11) DEFAULT '0' COMMENT '播放次数',
    `like_count` int(11) DEFAULT '0' COMMENT '点赞数量',
    `danmu_count` int(11) DEFAULT '0' COMMENT '弹幕数量',
    `comment_count` int(11) DEFAULT '0' COMMENT '评论数量',
    `coin_count` int(11) DEFAULT '0' COMMENT '投币数量',
    `collect_count` int(11) DEFAULT '0' COMMENT '收藏数量',
    `recommend_type` tinyint(1) DEFAULT '0' COMMENT '是否推荐：0:未推荐 1:已推荐',
    `last_play_time` datetime DEFAULT NULL COMMENT '最后播放时间',
    PRIMARY KEY (`video_id`) USING BTREE,
    KEY `idx_create_time` (`create_time`) USING BTREE,
    KEY `idx_user_id` (`user_id`) USING BTREE,
    KEY `idx_category_id` (`category_id`) USING BTREE,
    KEY `idx_pcategory_id` (`p_category_id`) USING BTREE,
    KEY `idx_recommend_type` (`recommend_type`) USING BTREE,
    KEY `idx_last_update_time` (`last_play_time`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='视频信息';
```
### 视频文件信息 - 主表
```sql
CREATE TABLE `video_info_file` (
    `file_id` varchar(20) NOT NULL COMMENT '唯一ID',
    `user_id` varchar(10) NOT NULL COMMENT '用户ID',
    `video_id` varchar(10) NOT NULL COMMENT '视频ID',
    `file_name` varchar(200) DEFAULT NULL COMMENT '文件名',
    `file_index` int(11) NOT NULL COMMENT '文件索引',
    `file_size` bigint(20) DEFAULT NULL COMMENT '文件大小',
    `file_path` varchar(100) DEFAULT NULL COMMENT '文件路径',
    `duration` int(11) DEFAULT NULL COMMENT '持续时间 (秒)',
    PRIMARY KEY (`file_id`) USING BTREE,
    KEY `idx_video_id` (`video_id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 ROW_FORMAT=DYNAMIC COMMENT='视频文件信息';
```
多线程上传视频切片，需要保证得知最后一个切片的id。

| 场景        | 推荐注解        | 原因                       |
| --------- | ----------- | ------------------------ |
| ID、编号     | `@NotNull`  | ID 可能是空串吗？一般不会，但理论上可以不为空 |
| 用户名、标题    | `@NotBlank` | 不能是空或纯空格                 |
| 列表、集合     | `@NotEmpty` | 传空列表没意义                  |
| 数值（0是合法值） | `@NotNull`  | 0 是有效值，不能用 @NotEmpty     |
| 描述（可选字段）  | 不加注解        | 可以为 null                 |
