---
title: 建立Bilibili排行榜数据库
abbrlink: 2428385357
date: 2021-11-29 09:26:56
tags:
  - JAVA
categories: JAVA
description: Github上易姐的BilibiliAPI 调用的应用 简单的爬取B站排行榜数据
---
建立Bilibili排行榜信息

- 建立数据库

  - ![image-20210930210133264](http://typa.qingyun.run/img/image-20210930210133264.png)	

  - 导出的语句

    ```sql
    SET NAMES utf8mb4;
    SET FOREIGN_KEY_CHECKS = 0;
    
    -- ----------------------------
    -- Table structure for bilibilirank
    -- ----------------------------
    DROP TABLE IF EXISTS `bilibilirank`;
    CREATE TABLE `bilibilirank`  (
      `id` int(0) NOT NULL AUTO_INCREMENT COMMENT '主键',
      `rid` varchar(10) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NULL DEFAULT NULL COMMENT '分区类型',
      `aid` varchar(15) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NULL DEFAULT NULL COMMENT 'aid',
      `bvid` varchar(15) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NULL DEFAULT NULL COMMENT 'bid',
      `typename` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NULL DEFAULT NULL COMMENT '分区名字',
      `title` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NULL DEFAULT NULL COMMENT '视频标题',
      `play` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NULL DEFAULT NULL COMMENT '播放量',
      `review` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NULL DEFAULT NULL COMMENT '评论量',
      `favoritea` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NULL DEFAULT NULL COMMENT '收藏量',
      `author` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NULL DEFAULT NULL COMMENT '作者',
      `description` varchar(200) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NULL DEFAULT NULL COMMENT '简介',
      `create` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NULL DEFAULT NULL COMMENT '上传时间',
      `pic` varchar(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NULL DEFAULT NULL COMMENT '封面链接',
      PRIMARY KEY (`id`) USING BTREE
    ) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_bin ROW_FORMAT = Dynamic;
    
    SET FOREIGN_KEY_CHECKS = 1;
    ```

    

- 通过mybatis-plus逆向生成实体类

  - 默认 均会使用代码构造器和mysql

  - 生成效果如下

  - ![image-20210930212043923](http://typa.qingyun.run/img/image-20210930212043923.png)

    

- 调整代码

  - Controller层设置为Springboot的测试类  执行主要方法（下面是测试代码）

  - ```java
    package edu.sdut.bilibili.controller;
    
    
    import edu.sdut.bilibili.service.impl.BilibilirankServiceImpl;
    import org.junit.jupiter.api.Test;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    
    /**
     * <p>
     *  bilibiliRank排行榜 主控制类
     * </p>
     *
     * @author qingyun
     * @since 2021-09-30
     */
    @SpringBootTest
    public class BilibilirankController {
    
        @Autowired
        BilibilirankServiceImpl bilibilirankService;
    
        @Test
        public void testInset(){
            bilibilirankService.Insert();
        }
    }
    ```

  - Service

  - ```java
    package edu.sdut.bilibili.service.impl;
    
    import edu.sdut.bilibili.entity.Bilibilirank;
    import edu.sdut.bilibili.mapper.BilibilirankMapper;
    import edu.sdut.bilibili.service.IBilibilirankService;
    import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;
    
    import javax.sql.DataSource;
    
    /**
     * <p>
     *  服务实现类
     * </p>
     *
     * @author qingyun
     * @since 2021-09-30
     */
    @Service
    public class BilibilirankServiceImpl extends ServiceImpl<BilibilirankMapper, Bilibilirank> implements IBilibilirankService {
    
        @Autowired
        BilibilirankMapper bilibilirankMapper;
    
        public void Insert() {
            Bilibilirank bilibilirank = new Bilibilirank();
            bilibilirank.setAid("1");
            bilibilirankMapper.insert(bilibilirank);
        }
    }
    
    ```

    修改主要逻辑代码

    ```java
    package edu.sdut.bilibili.controller;
    
    
    import edu.sdut.bilibili.entity.Bilibilirank;
    import edu.sdut.bilibili.service.impl.BilibilirankServiceImpl;
    import org.junit.jupiter.api.Test;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    
    import java.util.List;
    
    /**
     * <p>
     *  bilibiliRank排行榜 主控制类
     * </p>
     *
     * @author qingyun
     * @since 2021-09-30
     */
    @SpringBootTest
    public class BilibilirankController {
    
        @Autowired
        BilibilirankServiceImpl bilibilirankService;
    
        @Test
        public void GetListBiliBiliRanks() {
            //获取所有的排行榜信息
            List<Bilibilirank> biliBiliRank = bilibilirankService.getBiliBiliRank();
            System.out.println(biliBiliRank);
        }
    }
    
    ```

    ```java
    package edu.sdut.bilibili.service.impl;
    
    import cn.hutool.json.JSONArray;
    import cn.hutool.json.JSONObject;
    import cn.hutool.json.JSONUtil;
    import com.github.kevinsawicki.http.HttpRequest;
    import edu.sdut.bilibili.entity.Bilibilirank;
    import edu.sdut.bilibili.mapper.BilibilirankMapper;
    import edu.sdut.bilibili.service.IBilibilirankService;
    import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
    import jdk.nashorn.internal.ir.IfNode;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;
    
    import javax.sql.DataSource;
    import java.util.ArrayList;
    import java.util.List;
    
    /**
     * <p>
     *  服务实现类
     * </p>
     *
     * @author qingyun
     * @since 2021-09-30
     */
    @Service
    public class BilibilirankServiceImpl extends ServiceImpl<BilibilirankMapper, Bilibilirank> implements IBilibilirankService {
        @Autowired
        BilibilirankMapper bilibilirankMapper;
    
        @Override
        public List<Bilibilirank> getBiliBiliRank() {
            int low = 300; int row = 10;
            List<Bilibilirank> BiliBiliRankS = new ArrayList<>();
            for (int i = 1; i < low; i++) {
    //            数据主体
                String InfoBody = HttpRequest.get("http://api.bilibili.com/x/web-interface/ranking/region?rid=" + i).body();
                JSONObject BiliBiliInfoBody = JSONUtil.parseObj(InfoBody);
                //有请求信息
                if (BiliBiliInfoBody.getInt("code")==0){
                    //获取data
                    JSONArray data = BiliBiliInfoBody.getJSONArray("data");
                    for (int j = 0; j < row; j++) {
                        //建立 bilibilirank
                        Bilibilirank bilibilirank = new Bilibilirank();
                        JSONObject rows = data.getJSONObject(j);
                        if (rows == null){
                           continue;
                        }
                        bilibilirank.setRid(Integer.toString(j));
                        bilibilirank.setAid(rows.getStr("aid"));
                        bilibilirank.setBvid(rows.getStr("bvid"));
                        bilibilirank.setTypename(rows.getStr("typename"));
                        bilibilirank.setTitle(rows.getStr("title"));
                        bilibilirank.setPlay(rows.getStr("play"));
                        bilibilirank.setReview(rows.getStr("review"));
                        bilibilirank.setFavoritea(rows.getStr("favorite"));
                        bilibilirank.setAuthor(rows.getStr("author"));
                        bilibilirank.setDescription(rows.getStr("description"));
                        bilibilirank.setCreate(rows.getStr("create"));
                        bilibilirank.setPic(rows.getStr("pic"));
                        BiliBiliRankS.add(bilibilirank);
                    }
                }
    
            }
            return BiliBiliRankS;
        }
    }
    
    ```

    效果如下

  - ![image-20211001132050684](http://typa.qingyun.run/img/image-20211001132050684.png)

    下面进行插入数据库操作，使用mybatis-plus的批量插入操作

    注：直接提供的批量插入并不是真的批量插入。读者请自行查找真正的批量插入

    ```java
    package edu.sdut.bilibili.controller;
    
    import edu.sdut.bilibili.entity.Bilibilirank;
    import edu.sdut.bilibili.service.impl.BilibilirankServiceImpl;
    import org.junit.jupiter.api.Test;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    
    import java.util.List;
    
    /**
     * <p>
     * bilibiliRank排行榜 主控制类
     * </p>
     * @author qingyun
     * @since 2021-09-30
     */
    @SpringBootTest
    public class BilibilirankController {
    
        @Autowired
        BilibilirankServiceImpl bilibilirankService;
    
        @Test
        public void GetListBiliBiliRanks() {
            List<Bilibilirank> biliBiliRanks = bilibilirankService.getBiliBiliRank();
            bilibilirankService.instertBiliBili(biliBiliRanks);
            System.out.println("插入完成");
        }
    }
    
    ```

    ```java
    package edu.sdut.bilibili.service.impl;
    
    import cn.hutool.json.JSONArray;
    import cn.hutool.json.JSONObject;
    import cn.hutool.json.JSONUtil;
    import com.github.kevinsawicki.http.HttpRequest;
    import edu.sdut.bilibili.entity.Bilibilirank;
    import edu.sdut.bilibili.mapper.BilibilirankMapper;
    import edu.sdut.bilibili.service.IBilibilirankService;
    import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
    import jdk.nashorn.internal.ir.IfNode;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;
    
    import javax.sql.DataSource;
    import java.util.ArrayList;
    import java.util.List;
    
    /**
     * <p>
     *  服务实现类
     * </p>
     *
     * @author qingyun
     * @since 2021-09-30
     */
    @Service
    public class BilibilirankServiceImpl extends ServiceImpl<BilibilirankMapper, Bilibilirank> implements IBilibilirankService {
        @Autowired
        BilibilirankMapper bilibilirankMapper;
    
        @Override
        public List<Bilibilirank> getBiliBiliRank() {
            int low = 300; int row = 10;
            List<Bilibilirank> BiliBiliRankS = new ArrayList<>();
            for (int i = 1; i < low; i++) {
    //            数据主体
                String InfoBody = HttpRequest.get("http://api.bilibili.com/x/web-interface/ranking/region?rid=" + i).body();
                JSONObject BiliBiliInfoBody = JSONUtil.parseObj(InfoBody);
                //有请求信息
                if (BiliBiliInfoBody.getInt("code")==0){
                    //获取data
                    JSONArray data = BiliBiliInfoBody.getJSONArray("data");
                    for (int j = 0; j < row; j++) {
                        //建立 bilibilirank
                        Bilibilirank bilibilirank = new Bilibilirank();
                        JSONObject rows = data.getJSONObject(j);
                        if (rows == null){
                           continue;
                        }
                        bilibilirank.setRid(Integer.toString(j));
                        bilibilirank.setAid(rows.getStr("aid"));
                        bilibilirank.setBvid(rows.getStr("bvid"));
                        bilibilirank.setTypename(rows.getStr("typename"));
                        bilibilirank.setTitle(rows.getStr("title"));
                        bilibilirank.setPlay(rows.getStr("play"));
                        bilibilirank.setReview(rows.getStr("review"));
                        bilibilirank.setFavoritea(rows.getStr("favorites"));
                        bilibilirank.setAuthor(rows.getStr("author"));
                        bilibilirank.setDescription(rows.getStr("description"));
    //                bilibilirank.setCreate(rows.getStr("create"));
                        bilibilirank.setPic(rows.getStr("pic"));
                        BiliBiliRankS.add(bilibilirank);
                    }
                }
    
            }
            return BiliBiliRankS;
        }
    
    
        /**
         * 添加实体类信息置入数据库
         */
        public void instertBiliBili(List<Bilibilirank> biliBiliRanks){
            for (Bilibilirank biliBiliRank : biliBiliRanks) {
                try{
                    bilibilirankMapper.insert(biliBiliRank);
                }catch (Exception e){
                    e.printStackTrace();
                }
    
            }
    //        bilibilirankMapper.insertBatchSomeColumn(biliBiliRanks);
        }
    }
    
    ```

    

![image-20211001152759709](http://typa.qingyun.run/img/image-20211001152759709.png)

