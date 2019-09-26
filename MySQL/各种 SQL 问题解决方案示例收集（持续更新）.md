# 各种 SQL 问题解决方案示例收集
## MySQL 插入数据时判断是否存在，若不存在则进行插入操作
```sql
<insert id="insertIfNotExists">
    INSERT INTO t_user (user_code, user_name) SELECT
        #{userCode},
        #{userName}
    FROM
        DUAL
    WHERE
        NOT EXISTS (
            SELECT
                1
            FROM
                t_user
            WHERE
                user_code = #{userCode}
        )
</insert>
```
> - DUAL 为临时表，可理解为 MySQL 的固定写法
> - 此示例是结合 Mybatis 的实例

## 查询列名存在于哪些表中
```sql
SELECT
    TABLE_NAME,
    COLUMN_NAME
FROM
    information_schema.`COLUMNS`
WHERE
    TABLE_SCHEMA = 'TABLE_SCHEMA'
AND COLUMN_NAME = 'COLUMN_NAME'
```

## 根据条件进行批量更新操作
- 语法一
```sql
<update id="batchUpdateUserInfo">
    UPDATE t_user
    SET user_name = CASE user_id
            <foreach collection="users" item="u">
                WHEN #{u.userId} THEN #{u.userName}
            </foreach>
        END
    WHERE user_id IN (
        <foreach collection="users" item="u" separator=",">
            #{u.userId}
        </foreach>
    )
</update>
```
- 语法二
```sql
<update id="batchUpdateUserInfo">
    UPDATE t_user
    SET user_name = CASE 
            <foreach collection="users" item="u">
                WHEN user_id = #{u.userId} THEN #{u.userName}
            </foreach>
        END
    WHERE user_id IN (
        <foreach collection="users" item="u" separator=",">
            #{u.userId}
        </foreach>
    )
</update>
```
> - 以上两种语法均可同时对多个值修改
> - 需要注意的是 MySQL 通常会设置 SQL 的长度（可自定义），所以实际生产中需要根据实际情况预估此操作是否有可能会超出 MySQL 的长度限制，如果超出的话需要考虑分多批进行修改

## 分组排序显示序号
> 场景设定：根据月份分组，查看每个人在每个月的排名情况
- 现有数据

| user_name | month | score |
|:---------:|:-----:|:-----:|
| 张三 | 201901 | 90  |
| 李四 | 201901 | 100 |
| 王五 | 201901 | 80  |
| 张三 | 201902 | 100 |
| 李四 | 201902 | 80  |
| 王五 | 201902 | 90  |

- SQL
```sql
SELECT
    us1.`month`,
    us1.user_name,
    us1.score,
    COUNT(*) 'top'
FROM
    user_score us1
LEFT JOIN user_score us2 ON us2.`month` = us1.`month`
AND us2.score >= us1.score
GROUP BY
    us1.`month`,
    us1.score DESC
```
- 查询结果

| month | user_name | score | top |
|:-----:|:---------:|:-----:|:---:|
| 201901 | 李四 | 100 | 1 |
| 201901 | 张三 | 90  | 2 |
| 201901 | 王五 | 80  | 3 |
| 201902 | 张三 | 100 | 1 |
| 201902 | 王五 | 90  | 2 |
| 201902 | 李四 | 80  | 3 |

> 注意：此语法要求 `month` + `score` 不重复