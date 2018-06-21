---
title: Spring Data JPA 初次使用心得
date: 2018-6-21 10:53:20
categories: [开发,总结]
tags: [Spring Data JPA,Java]
---
## 前言
- 最近在给公司的老项目换个架构，之前的架构太繁琐古老了些，用的是SSH，还是老版的Hibernate3，配置文件看花眼还有Struts2确实看着不习惯，感觉有点费眼，不过在改的过程我觉得它能共享参数有时候还是挺便捷的。
-  因为之前用的是Hibernate所以我决定用Spring Data JPA,对了，今天我要说下有关于JPA的一些注意点，也是我亲自踩完后发现的，因为之前没有接触过它。
-  Entity实体类与Hibernate没有多大区别@Entity定义实体 @Table定义表明 @Id定义主键 @Column定义数据库中字段映射
```
@Entity
@Table(name = "alarmtable")
public class AlarmEntity {
    private String channelNumber;
    private double alarmValue;

    @Id
    @Column(name = "channelNumber", length = 20, nullable = false)
    public String getChannelNumber() {
        return this.channelNumber;
    }

    public void setChannelNumber(String channelNumber) {
        this.channelNumber = channelNumber;
    }

    @Column(name = "alarmValue", nullable = false)
    public double getAlarmValue() {
        return this.alarmValue;
    }

    public void setAlarmValue(double alarmValue) {
        this.alarmValue = alarmValue;
    }
}
```

**这里注意的是当你用驼峰命名字段时会自动用下划线分割，如`alarmValue`会变成`alarm_value`**

## JPA优点
- 很多人喜欢用JPA的原因是什么，不就是它能省事么，一些基本的CRUD都能替你完成只需要调其方法即可。不过JPA它有特定的命名规则，不能瞎写，写之前还是要去网上看看，这里就不贴示例了。
- 首先需要先添加依赖，我用的是maven：
```
    <!-- JPA依赖 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
```
- 如果你要自定义方法SQL查询，直接使用@Query注解即可，示例：
```
    /**
     * 查询所有的端口号
     * 
     * @return
     */
    @Query(value = "select distinct port from relationtable order by port", nativeQuery = true)
    List<Integer> getAllPort();

    /**
     * 根据设备号删除数据
     * 
     * @param instru_serial
     */
    @Transactional
    @Modifying
    @Query(value = "delete from relationTable where instru_serial = :instru_serial", nativeQuery = true)
    void deleteByInstru_serial(@Param("instru_serial") String instru_serial);
```
`nativeQuery = true`说明该sql为原生sql，本地sql查询。
> 所谓本地查询，就是使用原生的sql语句（根据数据库的不同，在sql的语法或结构方面可能有所区别）进行查询数据库的操作。

`做删除操作时记得加上这两个注解 @Transactional @Modifying`

**Query的写法有很多种，我这只是其中一种个人认为较为不错的写法，其余可以自行搜索。**

## Dao层
- 这里我还是习惯称之为Dao，也就是Repository。如果要使用JPA来操作实体映射类的方法时，你需要继承JpaRepository。JpaRepository有两个参数，第一个是相对于的Entity类即映射实体类，第二个为主键类。

`JpaRepository<RelationEntity, Integer>` 第一个参数好理解，第二个参数就是对应Entity中用@Id注解表示的字段的类型，若是int则为Integer，String则为String等等。

- 不过这里我要说的是复合主键，就是当你一个表中存在多个主键的时候。通过查找许多资料后得出方法。`@IdClass`可以解决我们的问题。把Entity中用`@Id`定义的字段重新建一个class来制定。请看示例：

**Entity Class :**
```
package com.tonglei.aiot.entity;

import java.io.Serializable;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.IdClass;
import javax.persistence.Table;

import com.tonglei.aiot.entity.idclass.DVWIdClass;
/**
 * 应变映射实体类
 * 
 * @author ffj
 *
 */
@Entity
@IdClass(DVWIdClass.class)
@Table(name = "dvw16_instrument_Info")
public class DVWInstrumentInfoEntity implements Serializable {
    private static final long serialVersionUID = 1L;
    private String instruSerial;
    /**
     * 通道
     */
    private int channel;
    /**
     * 采样率
     */
    private String ratio;
    private String channelNumber;

    @Id
    @Column(name = "Instru_serial", length = 8, nullable = false)
    public String getInstruSerial() {
        return this.instruSerial;
    }

    public void setInstruSerial(String instru_serial) {
        this.instruSerial = instru_serial;
    }

    @Id
    @Column(name = "channel", length = 11, nullable = false)
    public int getChannel() {
        return this.channel;
    }

    public void setChannel(int channel) {
        this.channel = channel;
    }

    @Column(name = "ratio", length = 30, nullable = false)
    public String getRatio() {
        return this.ratio;
    }

    public void setRatio(String ratio) {
        this.ratio = ratio;
    }

    @Column(name = "number", length = 20, nullable = false)
    public String getChannelNumber() {
        return this.channelNumber;
    }

    public void setChannelNumber(String channelNumber) {
        this.channelNumber = channelNumber;
    }
}
```

**Id Class :**
```
package com.tonglei.aiot.entity.idclass;

import java.io.Serializable;
/**
 * 应变复合主键类
 * 
 * @author ffj
 *
 */
public class DVWIdClass implements Serializable {
    /**
     * 
     */
    private static final long serialVersionUID = 1L;
    String instruSerial;
    int channel;

    public String getInstruSerial() {
        return instruSerial;
    }

    public void setInstruSerial(String instruSerial) {
        this.instruSerial = instruSerial;
    }

    public int getChannel() {
        return channel;
    }

    public void setChannel(int channel) {
        this.channel = channel;
    }

}
```

**Dao ：**
```
package com.tonglei.aiot.dao;

import org.springframework.data.jpa.repository.JpaRepository;

import com.tonglei.aiot.entity.DVWInstrumentInfoEntity;
import com.tonglei.aiot.entity.idclass.DVWIdClass;
/**
 * 应变Dao层接口
 * 
 * @author ffj
 *
 */
public interface DVWInstrumentInfoDao extends JpaRepository<DVWInstrumentInfoEntity, DVWIdClass> {

    DVWInstrumentInfoEntity findByInstruSerialAndChannel(String instruSerial, int channel);
}
```

**需要注意的是，复合主键类必须实现Serializable接口不然启动会报错。**

## 总结
- 由于也是第一次使用Spring Data JPA，之前都是用Mybatis的比较多。通过这次的使用发现如果业务上都是一些不太复杂的增删改的话确实能提高不少的效率的。
- 相对于Mybatis节省了许多配置，懒人首选。
- 现在还没有用到它的一些复杂的操作，比如多表联查，多表关联什么的，后面用到再详细说明。
- 通过这次框架的重构也使得我对于其内部方法的调用更加熟悉，虽然还有好多方法我觉得可以更加简单，这个慢慢再优化吧。
- 终于不用看那一大堆配置文件了。

## 参考资料
- [Spring Data JPA - Reference Documentation](https://docs.spring.io/spring-data/jpa/docs/2.1.0.M3/reference/html/)
- [Spring Data Jpa 复合主键](https://blog.csdn.net/qq_35056292/article/details/77892012)
- [Spring Data系列四 @Query注解及@Modifying注解](https://www.cnblogs.com/zhaobingqing/p/6864223.html)
