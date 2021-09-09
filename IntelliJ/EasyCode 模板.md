> 这是一套基于MyBatis Plus 的模板，`mapper.xml` 仅生成 `resultMap`、`columnSql`、`whereSql`、`updateSql`
## entity.java
```velocity
##引入宏定义
$!define

##使用宏定义设置回调（保存位置与文件后缀）
#save("/entity", ".java")

##使用宏定义设置包后缀
#setPackageSuffix("entity")

##使用全局变量实现默认包导入
$!autoImport
import java.io.Serializable;
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

##使用宏定义实现类注释信息
#tableComment("实体类")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class $!{tableInfo.name} implements Serializable {
    private static final long serialVersionUID = $!tool.serial();
    
    
#foreach($column in $tableInfo.pkColumn)
    #if(${column.comment})/**
    * ${column.comment}
    */#end
    
    @TableId(value = "id", type = IdType.AUTO)
    private $!{tool.getClsNameByFullName($column.type)} $!{column.name};
    #break
#end
#foreach($column in $tableInfo.otherColumn)

    #if(${column.comment})/**
    * ${column.comment}
    */#end

    private $!{tool.getClsNameByFullName($column.type)} $!{column.name};
#end

}
```

## mapper.xml
```velocity
##引入mybatis支持
$!mybatisSupport

##设置保存名称与保存位置
$!callback.setFileName($tool.append($!{tableInfo.name}, "Mapper.xml"))
$!callback.setSavePath($tool.append($modulePath, "/src/main/resources/mapper"))

##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

##拿到表名缩写
#set($tableNameAbbreviation='')
#foreach ($element in $tableInfo.obj.name.split("_"))
    #set($tableNameAbbreviation=$tableNameAbbreviation+$element.substring(0,1).toLowerCase())
#end

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="$!{tableInfo.savePackageName}.dao.$!{tableInfo.name}Mapper">

    <resultMap id="$!{tableInfo.name}Map" type="$!{tableInfo.savePackageName}.entity.$!{tableInfo.name}">
#foreach($column in $tableInfo.fullColumn)
        <result property="$!column.name" column="$!column.obj.name" jdbcType="$!column.ext.jdbcType"/>
#end
    </resultMap>
    
    <sql id="columnSql">
#foreach($column in $tableInfo.fullColumn)
        $tableNameAbbreviation.$!column.obj.name#if($velocityHasNext), #end
        
#end
    </sql>

    <sql id="whereSql">
        <where>
#foreach($column in $tableInfo.fullColumn)
            <if test="$!column.name != null#if($column.type.equals("java.lang.String")) and $!column.name != ''#end">
                AND $tableNameAbbreviation.$!column.obj.name = #{$!column.name}
            </if>
#end
        </where>
    </sql>
    
    <sql id="updateSql">
        <set>
#foreach($column in $tableInfo.otherColumn)
            <if test="$!column.name != null#if($column.type.equals("java.lang.String")) and $!column.name != ''#end">
                $!column.obj.name = #{$!column.name},
            </if>
#end
        </set>
    </sql>
</mapper>
```

## mapper.java
```velocity
##定义初始变量
#set($tableName = $tool.append($tableInfo.name, "Mapper"))
##设置回调
$!callback.setFileName($tool.append($tableName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/mapper"))

##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}dao;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;

/**
 * $!{tableInfo.comment}($!{tableInfo.name})表数据库访问层
 *
 * @author $!author
 * @since $!time.currTime()
 */
public interface $!{tableName} extends BaseMapper<$!{tableInfo.name}>{


}
```

## service.java
```velocity
##定义初始变量
#set($tableName = $tool.append($tableInfo.name, "Service"))
##设置回调
$!callback.setFileName($tool.append("I", $tableName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/service"))

##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}service;

import com.baomidou.mybatisplus.extension.service.IService;

/**
 * $!{tableInfo.comment}($!{tableInfo.name})表服务接口
 *
 * @author $!author
 * @since $!time.currTime()
 */
public interface I$!{tableName} extends IService<$!{tableInfo.name}>{


}
```

## serviceimpl.java
```velocity
##定义初始变量
#set($tableName = $tool.append($tableInfo.name, "ServiceImpl"))
##设置回调
$!callback.setFileName($tool.append($tableName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/service/impl"))

##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}service.impl;

import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import org.springframework.stereotype.Service;


/**
 * $!{tableInfo.comment}($!{tableInfo.name})表服务实现类
 *
 * @author $!author
 * @since $!time.currTime()
 */
@Service
public class $!{tableName} extends ServiceImpl<$!{tableInfo.name}Mapper, $!{tableInfo.name}>  implements $!{tableInfo.name}Service {

}
```


## controller.java
```velocity
##定义初始变量
#set($tableName = $tool.append($tableInfo.name, "Controller"))
##设置回调
$!callback.setFileName($tool.append($tableName, ".java"))
$!callback.setSavePath($tool.append($tableInfo.savePath, "/controller"))
##拿到主键
#if(!$tableInfo.pkColumn.isEmpty())
    #set($pk = $tableInfo.pkColumn.get(0))
#end

#if($tableInfo.savePackageName)package $!{tableInfo.savePackageName}.#{end}controller;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import io.swagger.annotations.ApiParam;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import io.swagger.annotations.Api;


/**
 * $!{tableInfo.comment}($!{tableInfo.name})表控制层
 *
 * @author $!author
 * @since $!time.currTime()
 */
@RestController
@RequestMapping("$!tool.firstLowerCase($tableInfo.name)")
@Api(value = "$!{tableInfo.comment}", tags = "$!{tableInfo.comment}")
public class $!{tableName} {
    /**
     * 服务对象
     */
    @Autowired
    private I$!{tableInfo.name}Service $!tool.firstLowerCase($tableInfo.name)Service;

}
```