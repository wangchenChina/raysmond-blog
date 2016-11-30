---
title: 在Spring MVC中網站配置存儲在Redis和數據庫中的解決方法
date: 2015-09-28 00:05:48
tags:
  - Spring
  - Redis
  - 數據庫
---

有些網站配置比如網站名稱，簡介，郵箱等等變量需要存儲在數據庫中，但是又希望不要每次都從數據庫中查詢，最好可以緩存起來。簡單整理一下需求：

網站配置存儲到數據庫中，配置的對象類型可能不同，比如有可能是String, Long, Date等；
配置可以在Redis中緩存起來，而不是每次從數據庫中查詢。
首先需要配置Spring Data Redis和Hibernate等，請參考文章: 在Spring MVC中使用Spring Data Redis。

先建立Setting的Entity。由于存儲的變量可能類型不同，存儲到數據庫中要考慮序列化存儲，這裏使用了Serializable接口，並用Lob標注表示這個字段在數據庫存儲爲blob的類型。

<!-- more -->

```java
// Setting.java
package com.raysmond.blog.models;

import javax.persistence.*;
import java.io.Serializable;
import java.util.Date;

/**
 * A generic setting model
 *
 * @author Raysmond
 */
@Entity
@Table(name = "settings")
public class Setting{
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(name = "_key", unique = true, nullable = false)
    private String key;

    @Lob
    @Column(name = "_value")
    private Serializable value;

    @Column(nullable = false)
    private Date createdAt;

    @Column(nullable = false)
    private Date updatedAt;

    @PrePersist
    public void prePersist(){
        if (createdAt == null){
            createdAt = new Date();
        }

        updatedAt = new Date();
    }

    public Long getId() {
        return id;
    }

    public void setId(Long _id) {
        id = _id;
    }

    public String getKey() {
        return key;
    }

    public void setKey(String key) {
        this.key = key;
    }

    public Serializable getValue() {
        return value;
    }

    public void setValue(Serializable value) {
        this.value = value;
    }

    public Date getCreatedAt() {
        return createdAt;
    }

    public void setCreatedAt(Date createdAt) {
        this.createdAt = createdAt;
    }

    public Date getUpdatedAt() {
        return updatedAt;
    }

    public void setUpdatedAt(Date updatedAt) {
        this.updatedAt = updatedAt;
    }
}
```

然後新建一個Setting的Repository。

```java
// SettingRepository.java
package com.raysmond.blog.repositories;

import com.raysmond.blog.models.Setting;
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;

@Repository
@Transactional
public interface SettingRepository extends CrudRepository<Setting, Long> {
    Setting findByKey(String key);
}
```

新建一個用于訪問和存儲變量的Service接口。

```java
// SettingService.java
package com.raysmond.blog.services;

import java.io.Serializable;


public interface SettingService {
    Serializable get(String key);
    Serializable get(String key, Serializable defaultValue);
    void put(String key, Serializable value);
}
```

實現具有Cache功能的Service接口。

```java
// CacheSettingService.java
package com.raysmond.blog.services;

import com.raysmond.blog.models.Setting;
import com.raysmond.blog.repositories.SettingRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

import java.io.Serializable;

@Service
public class CacheSettingService implements SettingService {

    private SettingRepository settingRepository;

    private static final String CACHE_NAME = "cache.settings";

    @Autowired
    public CacheSettingService(SettingRepository settingRepository) {
        this.settingRepository = settingRepository;
    }

    private static final Logger logger = LoggerFactory.getLogger(SettingService.class);

    @Override
    @Cacheable(value = CACHE_NAME, key = "#key")
    public Serializable get(String key) {
        Setting setting = settingRepository.findByKey(key);
        Serializable value = null;
        try {
            value = setting == null ? null : setting.getValue();
        } catch (Exception ex) {
            logger.info("Cannot deserialize setting value with key = " + key);
        }

        logger.info("Get setting " + key + " from database. Value = " + value);

        return value;
    }

    @Override
    @Cacheable(value = CACHE_NAME, key = "#key")
    public Serializable get(String key, Serializable defaultValue) {
        Serializable value = get(key);
        return value == null ? defaultValue : value;
    }

    @Override
    @CacheEvict(value = CACHE_NAME, key = "#key")
    public void put(String key, Serializable value) {
        logger.info("Update setting " + key + " to database. Value = " + value);
        Setting setting = settingRepository.findByKey(key);
        if (setting == null) {
            setting = new Setting();
            setting.setKey(key);
        }
        try {
            setting.setValue(value);
            settingRepository.save(setting);
        } catch (Exception ex) {
            ex.printStackTrace();
            logger.info("Cannot save setting value with type: " + value.getClass() + ". key = " + key);
        }
    }
}
```

**如何使用？**

```java
@Autowired SettingService settingService;

@RequestMapping("")
public String test(){
    settingService.put("SITE_NAME", "Raysmond");
    settingService.put("PAGE_SIZE", Integer.valueOf(10));

    settingService.get("SITE_NAME");
    settingService.get("PAGE_SIZE");

    // other code
}
```
