---
title: JAVA实现图片合并
abbrlink: 756143886
date: 2021-11-29 09:31:05
tags:
  - JAVA
categories: JAVA
description: JAVA应用 实现两张图片合并（可多张未实现）
---
JAVA  实现两张图片合并（可多张未实现）

```java
package edu.sdut.Picture.Main;

import javax.imageio.ImageIO;
import java.awt.*;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.net.URL;

/**
 * @author qingyun
 * @version 1.0
 * @date 2021/9/28 20:13
 */
//实现图片合并
public class PictureMerge {


    private  Graphics2D g        = null;

    //读取网络图片
//    public static BufferedImage readUrl(String ImageUrl){
//        BufferedImage bf = null;
//        try{
//            URL urlfile = new URL(ImageUrl);
//            InputStream inStream = urlfile.openStream();
//            bf = ImageIO.read(inStream);
//        }catch (IOException e) {
//            e.printStackTrace();
//        }
//        return bf;
//    }


    /**
     * 导入本地图片到缓冲区
     */
    public BufferedImage loadImageLocal(String imgName) {
        try {
            return ImageIO.read(new File(imgName));
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
        return null;
    }

    /**
     * 将两张图片合并在一起
     * @param b
     * @param d
     * @return
     */
    public BufferedImage modifyImagetogeter(BufferedImage b, BufferedImage d) {
        try {
            int w = b.getWidth();
            int h = b.getHeight();
            //进行图片绘制
            g = d.createGraphics();
            g.drawImage(b, 300, -800, w, h, null);
            g.dispose();
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
        return d;
    }


    /**
     * 生成新图片到本地
     */
    public void writeImageLocal(String newImage, BufferedImage img) {
        if (newImage != null && img != null) {
            try {
                File outputfile = new File(newImage);
                ImageIO.write(img, "jpg", outputfile);
            } catch (IOException e) {
                System.out.println(e.getMessage());
            }
        }
    }

    /**
     * 图片测试
     * @param args
     */
    public static void main(String[] args) {
        PictureMerge pictureMerge = new PictureMerge();
        //图片A
        BufferedImage d = pictureMerge.loadImageLocal("D:\\GIF\\jpg\\1.jpg");
        //t
        BufferedImage b = pictureMerge.loadImageLocal("D:\\GIF\\jpg\\2.jpg");
        pictureMerge.writeImageLocal("D:\\GIF\\gif\\10.jpg", pictureMerge.modifyImagetogeter(b, d));
        //将多张图片合在一起
        System.out.println("success");
    }
}

```

测试图片![1](http://typa.qingyun.run/img/1.jpg)

![2](http://typa.qingyun.run/img/2.jpg)

生成图片如下

![10](http://typa.qingyun.run/img/10.jpg)

不符合期望，故进行代码修改

修改后代码如下

```java
package edu.sdut.Picture.Main;

import javax.imageio.ImageIO;
import javax.swing.*;
import java.awt.*;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.net.URL;

/**
 * @author qingyun
 * @version 1.0
 * @date 2021/9/28 20:13
 */
//实现图片合并
public class PictureMerge {


    private  Graphics2D g        = null;

    //读取网络图片
//    public static BufferedImage readUrl(String ImageUrl){
//        BufferedImage bf = null;
//        try{
//            URL urlfile = new URL(ImageUrl);
//            InputStream inStream = urlfile.openStream();
//            bf = ImageIO.read(inStream);
//        }catch (IOException e) {
//            e.printStackTrace();
//        }
//        return bf;
//    }


    /**
     * 导入本地图片到缓冲区
     */
    public BufferedImage loadImageLocal(String imgName) {
        try {
            return ImageIO.read(new File(imgName));
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
        return null;
    }

    /**
     * 将两张图片合并在一起
     * @param b
     * @param d
     * @return
     */
    public BufferedImage modifyImagetogeter(BufferedImage b, BufferedImage d) {
        try {
            int w = d.getWidth();
            int h = d.getHeight();
            //进行图片绘制
            g = d.createGraphics();
            //更改生成的位置  图片进行覆盖
            g.drawImage(b, 0, 0, w, h, null);
            g.dispose();
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
        return d;
    }


    /**
     * 生成新图片到本地
     */
    public void writeImageLocal(String newImage, BufferedImage img) {
        if (newImage != null && img != null) {
            try {
                File outputfile = new File(newImage);
                ImageIO.write(img, "jpg", outputfile);
            } catch (IOException e) {
                System.out.println(e.getMessage());
            }
        }
    }



    /**
     * 将背景替换为透明
     *
     * @return
     * @throws IOException the io exception
     * @author Jack Que
     * @created 2021 -07-08 10:25:10 Change img color.
     */
    public static BufferedImage changeImgColor(BufferedImage bi) throws IOException {
        Image image = (Image) bi;
        //将原图片的二进制转化为ImageIcon
        ImageIcon imageIcon = new ImageIcon(image);
        int width = imageIcon.getIconWidth();
        int height = imageIcon.getIconHeight();
        //图片缓冲流
        BufferedImage bufferedImage = new BufferedImage(width, height, BufferedImage.TYPE_4BYTE_ABGR);
        Graphics2D graphics2D = (Graphics2D) bufferedImage.getGraphics();
        graphics2D.drawImage(imageIcon.getImage(), 0, 0, imageIcon.getImageObserver());
        int alpha = 255;
        //这个背景底色的选择，我这里选择的是比较偏的位置，可以修改位置。背景色选择不知道有没有别的更优的方式（比如先过滤一遍获取颜色次数最多的，但是因为感觉做起来会比较复杂没去实现），如果有可以评论。
        int RGB=bufferedImage.getRGB(width-1, height-1);
        for(int i = bufferedImage.getMinX(); i < width; i++) {
            for(int j = bufferedImage.getMinY(); j < height; j++) {
                int rgb = bufferedImage.getRGB(i, j);
                int r = (rgb & 0xff0000) >>16;
                int g = (rgb & 0xff00) >> 8;
                int b = (rgb & 0xff);
                int R = (RGB & 0xff0000) >>16;
                int G = (RGB & 0xff00) >> 8;
                int B = (RGB & 0xff);
                //a为色差范围值，渐变色边缘处理，数值需要具体测试，50左右的效果比较可以
                int a = 45;
                if(Math.abs(R-r) < a && Math.abs(G-g) < a && Math.abs(B-b) < a ) {
                    alpha = 0;
                } else {
                    alpha = 255;
                }
                rgb = (alpha << 24)|(rgb & 0x00ffffff);
                bufferedImage.setRGB(i,j,rgb);
            }
        }
            return bufferedImage;
    }
    public static String convertRgbStr(int color) {
        // 获取color(RGB)中R位
        int red = (color & 0xff0000) >> 16;
        // 获取color(RGB)中G位
        int green = (color & 0x00ff00) >> 8;
        // 获取color(RGB)中B位
        int blue = (color & 0x0000ff);
        return red + "," + green + "," + blue;
    }


    /**
     * 图片测试
     * @param args
     */
    public static void main(String[] args) throws IOException {
        PictureMerge pictureMerge = new PictureMerge();
        //图片A
        BufferedImage d = pictureMerge.loadImageLocal("D:\\GIF\\gif\\1.jpg");
        //t
        BufferedImage b = pictureMerge.loadImageLocal("D:\\GIF\\jpg\\2.jpg");
        //实现图片透明
//        d = changeImgColor(d);
        b = changeImgColor(b);
        pictureMerge.writeImageLocal("D:\\GIF\\gif\\10.jpg", pictureMerge.modifyImagetogeter(b, d));
        //将多张图片合在一起
        System.out.println("success");
    }
}

```

产生效果如下  （修改了背景图片素描画并且进行了透明化但不符合期望，继续修改）

![11](http://typa.qingyun.run/img/11.jpg)

替换了另一套代码 使用透明度进行图片合并 代码如下

```java
package edu.sdut.Picture.Main;
import java.awt.AlphaComposite;
import java.awt.Graphics2D;
import java.awt.Image;
import java.awt.RenderingHints;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.FileOutputStream;
import java.io.OutputStream;

import javax.imageio.ImageIO;
import javax.swing.ImageIcon;

/**
 * @author qingyun
 * @version 1.0
 * @date 2021/9/28 22:23
 */
public class ImageMarkLogoByIcon {

    /**
     * @param args
     */
    public static void main(String[] args) {
        String srcImgPath = "D:\\GIF\\jpg\\1.jpg";
        String iconPath = "D:\\GIF\\jpg\\2.jpg";
        String targerPath = "D:\\GIF\\gif\\3.jpg" ;
        // 给图片添加水印
        ImageMarkLogoByIcon.markImageByIcon(iconPath, srcImgPath, targerPath , 0);
    }
    /**
     * 给图片添加水印
     * @param iconPath 水印图片路径
     * @param srcImgPath 源图片路径
     * @param targerPath 目标图片路径
     */
    public static void markImageByIcon(String iconPath, String srcImgPath,
                                       String targerPath) {
        markImageByIcon(iconPath, srcImgPath, targerPath, null) ;
    }
    /**
     * 给图片添加水印、可设置水印图片旋转角度
     * @param iconPath 水印图片路径
     * @param srcImgPath 源图片路径
     * @param targerPath 目标图片路径
     * @param degree 水印图片旋转角度
     */
    public static void markImageByIcon(String iconPath, String srcImgPath,
                                       String targerPath, Integer degree) {
        OutputStream os = null;
        try {
            Image srcImg = ImageIO.read(new File(srcImgPath));
            BufferedImage buffImg = new BufferedImage(srcImg.getWidth(null),
                    srcImg.getHeight(null), BufferedImage.TYPE_INT_RGB);
            // 得到画笔对象
            // Graphics g= buffImg.getGraphics();
            Graphics2D g = buffImg.createGraphics();

            // 设置对线段的锯齿状边缘处理
            g.setRenderingHint(RenderingHints.KEY_INTERPOLATION,
                    RenderingHints.VALUE_INTERPOLATION_BILINEAR);

            g.drawImage(srcImg.getScaledInstance(srcImg.getWidth(null), srcImg
                    .getHeight(null), Image.SCALE_SMOOTH), 0, 0, null);

            if (null != degree) {
                // 设置水印旋转
                g.rotate(Math.toRadians(degree),
                        (double) buffImg.getWidth() / 2, (double) buffImg
                                .getHeight() / 2);
            }
            // 水印图象的路径 水印一般为gif或者png的，这样可设置透明度
            ImageIcon imgIcon = new ImageIcon(iconPath);
            // 得到Image对象。
            Image img = imgIcon.getImage();
            float alpha = 0.65f; // 透明度
            g.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_ATOP,
                    alpha));
            // 表示水印图片的位置
            g.drawImage(img, 0, 0,buffImg.getWidth(),buffImg.getHeight(), null);
            g.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER));
            g.dispose();
            os = new FileOutputStream(targerPath);
            // 生成图片
            ImageIO.write(buffImg, "JPG", os);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (null != os){
                    os.close();
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}

```

效果如下

![3](http://typa.qingyun.run/img/3.jpg)

本次处理到此为止，后续更改再次更新

