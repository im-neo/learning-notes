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
> DUAL 为临时表，可理解为 MySQL 的固定写法
> 此示例是结合 Mybatis 的实例

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
> - 此语法也可以同时对多个值修改
> - 需要注意的是 MySQL 通常会设置 SQL 的长度（可自定义），所以实际生产中需要根据实际情况预估此操作是否有可能会超出 MySQL 的长度限制，如果超出的话需要考虑分多批进行修改