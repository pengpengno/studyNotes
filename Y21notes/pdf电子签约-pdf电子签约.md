---
title: pdf电子签约
date: 2021-09-22 08:40:50.193
updated: 2022-04-27 16:35:19.475
url: /archives/pdf电子签约
categories: 
tags: 
- 开发开发开发！！！
---






1.背景
在某些业务场景中，需要提供相关的电子凭证，比如网银/支付宝中转账的电子回单，签约的电子合同等。方便用户查看，下载，打印。目前常用的解决方案是，把相关数据信息，生成对应的pdf文件返回给用户。



本文源码：http://git.oschina.net/lujianing/java_pdf_demo

2.iText
iText是著名的开放源码的站点sourceforge一个项目，是用于生成PDF文档的一个java类库。通过iText不仅可以生成PDF或rtf的文档，而且可以将XML、Html文件转化为PDF文件。 

iText 官网：http://itextpdf.com/

iText 开发文档： http://developers.itextpdf.com/developers-home

iText目前有两套版本iText5和iText7。iText5应该是网上用的比较多的一个版本。iText5因为是很多开发者参与贡献代码，因此在一些规范和设计上存在不合理的地方。iText7是后来官方针对iText5的重构，两个版本差别还是挺大的。不过在实际使用中，一般用到的都比较简单，所以不用特别拘泥于使用哪个版本。比如我们在http://mvnrepository.com/中搜索iText，出来的都是iText5的依赖。

来个最简单的例子:

添加依赖:


<!-- https://mvnrepository.com/artifact/com.itextpdf/itextpdf -->
<dependency>
    <groupId>com.itextpdf</groupId>
    <artifactId>itextpdf</artifactId>
    <version>5.5.11</version>
</dependency>
测试代码:JavaToPdf

package com.lujianing.test;
 
import com.itextpdf.text.Document;
import com.itextpdf.text.DocumentException;
import com.itextpdf.text.Paragraph;
import com.itextpdf.text.pdf.PdfWriter;
 
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
 
/**
 * Created by lujianing on 2017/5/7.
 */
public class JavaToPdf {
 
    private static final String DEST = "target/HelloWorld.pdf";
 
 
    public static void main(String[] args) throws FileNotFoundException, DocumentException {
        Document document = new Document();
        PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(DEST));
        document.open();
        document.add(new Paragraph("hello world"));
        document.close();
        writer.close();
    }
}
运行结果:



3.iText-中文支持
iText默认是不支持中文的，因此需要添加对应的中文字体,比如黑体simhei.ttf

可参考文档:http://developers.itextpdf.com/examples/font-examples/using-fonts#1227-tengwarquenya1.java

测试代码:JavaToPdfCN

package com.lujianing.test;
 
import com.itextpdf.text.Document;
import com.itextpdf.text.DocumentException;
import com.itextpdf.text.Font;
import com.itextpdf.text.FontFactory;
import com.itextpdf.text.Paragraph;
import com.itextpdf.text.pdf.BaseFont;
import com.itextpdf.text.pdf.PdfWriter;
 
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
 
/**
 * Created by lujianing on 2017/5/7.
 */
public class JavaToPdfCN {
 
    private static final String DEST = "target/HelloWorld_CN.pdf";
    private static final String FONT = "simhei.ttf";
 
 
    public static void main(String[] args) throws FileNotFoundException, DocumentException {
        Document document = new Document();
        PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(DEST));
        document.open();
        Font f1 = FontFactory.getFont(FONT, BaseFont.IDENTITY_H, BaseFont.NOT_EMBEDDED);
        document.add(new Paragraph("hello world,我是鲁家宁", f1));
        document.close();
        writer.close();
    }
}
输出结果:



4.iText-Html渲染
在一些比较复杂的pdf布局中，我们可以通过html去生成pdf

可参考文档:http://developers.itextpdf.com/examples/xml-worker-itext5/xml-worker-examples

添加依赖:

<!-- https://mvnrepository.com/artifact/com.itextpdf.tool/xmlworker -->
<dependency>
    <groupId>com.itextpdf.tool</groupId>
    <artifactId>xmlworker</artifactId>
    <version>5.5.11</version>
</dependency> 
添加模板:template.html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8"/>
    <title>Title</title>
    <style>
        body{
            font-family:SimHei;
        }
        .red{
            color: red;
        }
    </style>
</head>
<body>
<div class="red">
    你好，鲁家宁
</div>
</body>
</html>
测试代码:JavaToPdfHtml

package com.lujianing.test;
 
import com.itextpdf.text.Document;
import com.itextpdf.text.DocumentException;
import com.itextpdf.text.pdf.PdfWriter;
import com.itextpdf.tool.xml.XMLWorkerFontProvider;
import com.itextpdf.tool.xml.XMLWorkerHelper;
import com.lujianing.test.util.PathUtil;
 
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.charset.Charset;
 
/**
 * Created by lujianing on 2017/5/7.
 */
public class JavaToPdfHtml {
 
    private static final String DEST = "target/HelloWorld_CN_HTML.pdf";
    private static final String HTML = PathUtil.getCurrentPath()+"/template.html";
    private static final String FONT = "simhei.ttf";
 
 
    public static void main(String[] args) throws IOException, DocumentException {
        // step 1
        Document document = new Document();
        // step 2
        PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(DEST));
        // step 3
        document.open();
        // step 4
        XMLWorkerFontProvider fontImp = new XMLWorkerFontProvider(XMLWorkerFontProvider.DONTLOOKFORFONTS);
        fontImp.register(FONT);
        XMLWorkerHelper.getInstance().parseXHtml(writer, document,
                new FileInputStream(HTML), null, Charset.forName("UTF-8"), fontImp);
        // step 5
        document.close();
    }
}
输出结果:


需要注意:

1.html中必须使用标准的语法，标签一定需要闭合

2.html中如果有中文，需要在样式中添加对应字体的样式

 

5.iText-Html-Freemarker渲染
在实际使用中，html内容都是动态渲染的，因此我们需要加入模板引擎支持，可以使用FreeMarker/Velocity，这里使用FreeMarker举例

添加FreeMarke依赖:

<!-- https://mvnrepository.com/artifact/org.freemarker/freemarker -->
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.19</version>
</dependency>
添加模板:template_freemarker.html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8"/>
    <title>Title</title>
    <style>
        body{
            font-family:SimHei;
        }
        .blue{
            color: blue;
        }
    </style>
</head>
<body>
<div class="blue">
    你好，${name}
</div>
</body>
</html>
测试代码:JavaToPdfHtmlFreeMarker

package com.lujianing.test;
 
import com.itextpdf.text.Document;
import com.itextpdf.text.DocumentException;
import com.itextpdf.text.pdf.PdfWriter;
import com.itextpdf.tool.xml.XMLWorkerFontProvider;
import com.itextpdf.tool.xml.XMLWorkerHelper;
import com.lujianing.test.util.PathUtil;
 
import freemarker.template.Configuration;
import freemarker.template.Template;
 
import java.io.ByteArrayInputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.StringWriter;
import java.io.Writer;
import java.nio.charset.Charset;
import java.util.HashMap;
import java.util.Map;
 
/**
 * Created by lujianing on 2017/5/7.
 */
public class JavaToPdfHtmlFreeMarker {
 
    private static final String DEST = "target/HelloWorld_CN_HTML_FREEMARKER.pdf";
    private static final String HTML = "template_freemarker.html";
    private static final String FONT = "simhei.ttf";
 
    private static Configuration freemarkerCfg = null;
 
    static {
        freemarkerCfg =new Configuration();
        //freemarker的模板目录
        try {
            freemarkerCfg.setDirectoryForTemplateLoading(new File(PathUtil.getCurrentPath()));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
 
 
    public static void main(String[] args) throws IOException, DocumentException {
        Map<String,Object> data = new HashMap();
        data.put("name","鲁家宁");
        String content = JavaToPdfHtmlFreeMarker.freeMarkerRender(data,HTML);
        JavaToPdfHtmlFreeMarker.createPdf(content,DEST);
    }
 
 
    public static void createPdf(String content,String dest) throws IOException, DocumentException {
        // step 1
        Document document = new Document();
        // step 2
        PdfWriter writer = PdfWriter.getInstance(document, new FileOutputStream(dest));
        // step 3
        document.open();
        // step 4
        XMLWorkerFontProvider fontImp = new XMLWorkerFontProvider(XMLWorkerFontProvider.DONTLOOKFORFONTS);
        fontImp.register(FONT);
        XMLWorkerHelper.getInstance().parseXHtml(writer, document,
                new ByteArrayInputStream(content.getBytes()), null, Charset.forName("UTF-8"), fontImp);
        // step 5
        document.close();
 
    }
 
    /**
     * freemarker渲染html
     */
    public static String freeMarkerRender(Map<String, Object> data, String htmlTmp) {
        Writer out = new StringWriter();
        try {
            // 获取模板,并设置编码方式
            Template template = freemarkerCfg.getTemplate(htmlTmp);
            template.setEncoding("UTF-8");
            // 合并数据模型与模板
            template.process(data, out); //将合并后的数据和模板写入到流中，这里使用的字符流
            out.flush();
            return out.toString();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                out.close();
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
        return null;
    }
}
输出结果:


目前为止，我们已经实现了iText通过Html模板生成Pdf的功能，但是实际应用中，我们发现iText并不能对高级的CSS样式进行解析，比如CSS中的position属性等，因此我们要引入新的组件

6.Flying Saucer-CSS高级特性支持
Flying Saucer is a pure-Java library for rendering arbitrary well-formed XML (or XHTML) using CSS 2.1 for layout and formatting, output to Swing panels, PDF, and images.

Flying Saucer是基于iText的，支持对CSS高级特性的解析。

添加依赖:

<!-- https://mvnrepository.com/artifact/org.xhtmlrenderer/flying-saucer-pdf -->
<dependency>
    <groupId>org.xhtmlrenderer</groupId>
    <artifactId>flying-saucer-pdf</artifactId>
    <version>9.1.5</version>
</dependency>
 
<!-- https://mvnrepository.com/artifact/org.xhtmlrenderer/flying-saucer-pdf-itext5 -->
<dependency>
    <groupId>org.xhtmlrenderer</groupId>
    <artifactId>flying-saucer-pdf-itext5</artifactId>
    <version>9.1.5</version>
</dependency>
添加模板:template_freemarker_fs.html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8"/>
    <title>Title</title>
    <style>
        body{
            font-family:SimHei;
        }
        .color{
            color: green;
        }
        .pos{
            position:absolute;
            left:200px;
            top:5px;
            width: 200px;
            font-size: 10px;
        }
    </style>
</head>
<body>
<img src="logo.png" width="600px"/>
<div class="color pos">
    你好，${name}
</div>
</body>
</html>
测试代码:JavaToPdfHtmlFreeMarker

package com.lujianing.test.flyingsaucer;
 
 
import com.itextpdf.text.DocumentException;
import com.itextpdf.text.pdf.BaseFont;
import com.lujianing.test.util.PathUtil;
import freemarker.template.Configuration;
import freemarker.template.Template;
import org.xhtmlrenderer.pdf.ITextFontResolver;
import org.xhtmlrenderer.pdf.ITextRenderer;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.StringWriter;
import java.io.Writer;
import java.util.HashMap;
import java.util.Map;
 
/**
 * Created by lujianing on 2017/5/7.
 */
public class JavaToPdfHtmlFreeMarker {
 
    private static final String DEST = "target/HelloWorld_CN_HTML_FREEMARKER_FS.pdf";
    private static final String HTML = "template_freemarker_fs.html";
    private static final String FONT = "simhei.ttf";
    private static final String LOGO_PATH = "file://"+PathUtil.getCurrentPath()+"/logo.png";
 
    private static Configuration freemarkerCfg = null;
 
    static {
        freemarkerCfg =new Configuration();
        //freemarker的模板目录
        try {
            freemarkerCfg.setDirectoryForTemplateLoading(new File(PathUtil.getCurrentPath()));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
 
    public static void main(String[] args) throws IOException, DocumentException, com.lowagie.text.DocumentException {
        Map<String,Object> data = new HashMap();
        data.put("name","鲁家宁");
        String content = JavaToPdfHtmlFreeMarker.freeMarkerRender(data,HTML);
        JavaToPdfHtmlFreeMarker.createPdf(content,DEST);
    }
 
    /**
     * freemarker渲染html
     */
    public static String freeMarkerRender(Map<String, Object> data, String htmlTmp) {
        Writer out = new StringWriter();
        try {
            // 获取模板,并设置编码方式
            Template template = freemarkerCfg.getTemplate(htmlTmp);
            template.setEncoding("UTF-8");
            // 合并数据模型与模板
            template.process(data, out); //将合并后的数据和模板写入到流中，这里使用的字符流
            out.flush();
            return out.toString();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                out.close();
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
        return null;
    }
 
    public static void createPdf(String content,String dest) throws IOException, DocumentException, com.lowagie.text.DocumentException {
        ITextRenderer render = new ITextRenderer();
        ITextFontResolver fontResolver = render.getFontResolver();
        fontResolver.addFont(FONT, BaseFont.IDENTITY_H, BaseFont.NOT_EMBEDDED);
        // 解析html生成pdf
        render.setDocumentFromString(content);
        //解决图片相对路径的问题
        render.getSharedContext().setBaseURL(LOGO_PATH);
        render.layout();
        render.createPDF(new FileOutputStream(dest));
    }
}
 输出结果:



 

在某些场景下，html中的静态资源是在本地，我们可以使用render.getSharedContext().setBaseURL()加载文件资源,注意资源URL需要使用文件协议 "file://"。

对于生成的pdf页面大小，可以用css的@page属性设置。

 

7.PDF转图片
在某些场景中，我们可能只需要返回图片格式的电子凭证，我们可以使用Jpedal组件，把pdf转成图片

添加依赖:

<!-- https://mvnrepository.com/artifact/org.jpedal/jpedal-lgpl -->
<dependency>
    <groupId>org.jpedal</groupId>
    <artifactId>jpedal-lgpl</artifactId>
    <version>4.74b27</version>
</dependency>
测试代码:JavaToPdfImgHtmlFreeMarker

package com.lujianing.test.flyingsaucer;
 
 
import com.itextpdf.text.DocumentException;
import com.itextpdf.text.pdf.BaseFont;
import com.lujianing.test.util.PathUtil;
import freemarker.template.Configuration;
import freemarker.template.Template;
import org.jpedal.PdfDecoder;
import org.jpedal.exception.PdfException;
import org.jpedal.fonts.FontMappings;
import org.xhtmlrenderer.pdf.ITextFontResolver;
import org.xhtmlrenderer.pdf.ITextRenderer;
import java.awt.image.BufferedImage;
import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.StringWriter;
import java.io.Writer;
import java.util.HashMap;
import java.util.Map;
 
import javax.imageio.ImageIO;
 
/**
 * Created by lujianing on 2017/5/7.
 */
public class JavaToPdfImgHtmlFreeMarker {
 
    private static final String DEST = "target/HelloWorld_CN_HTML_FREEMARKER_FS_IMG.png";
    private static final String HTML = "template_freemarker_fs.html";
    private static final String FONT = "simhei.ttf";
    private static final String LOGO_PATH = "file://"+PathUtil.getCurrentPath()+"/logo.png";
    private static final String IMG_EXT = "png";
 
    private static Configuration freemarkerCfg = null;
 
    static {
        freemarkerCfg =new Configuration();
        //freemarker的模板目录
        try {
            freemarkerCfg.setDirectoryForTemplateLoading(new File(PathUtil.getCurrentPath()));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
 
 
    public static void main(String[] args) throws IOException, DocumentException, com.lowagie.text.DocumentException {
        Map<String,Object> data = new HashMap();
        data.put("name","鲁家宁");
 
        String content = JavaToPdfImgHtmlFreeMarker.freeMarkerRender(data,HTML);
        ByteArrayOutputStream pdfStream = JavaToPdfImgHtmlFreeMarker.createPdf(content);
        ByteArrayOutputStream imgSteam = JavaToPdfImgHtmlFreeMarker.pdfToImg(pdfStream.toByteArray(),2,1,IMG_EXT);
 
        FileOutputStream fileStream = new FileOutputStream(new File(DEST));
        fileStream.write(imgSteam.toByteArray());
        fileStream.close();
 
    }
 
 
    /**
     * freemarker渲染html
     */
    public static String freeMarkerRender(Map<String, Object> data, String htmlTmp) {
        Writer out = new StringWriter();
        try {
            // 获取模板,并设置编码方式
            Template template = freemarkerCfg.getTemplate(htmlTmp);
            template.setEncoding("UTF-8");
            // 合并数据模型与模板
            template.process(data, out); //将合并后的数据和模板写入到流中，这里使用的字符流
            out.flush();
            return out.toString();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                out.close();
            } catch (IOException ex) {
                ex.printStackTrace();
            }
        }
        return null;
    }
 
    /**
     * 根据模板生成pdf文件流
     */
    public static ByteArrayOutputStream createPdf(String content) {
        ByteArrayOutputStream outStream = new ByteArrayOutputStream();
        ITextRenderer render = new ITextRenderer();
        ITextFontResolver fontResolver = render.getFontResolver();
        try {
            fontResolver.addFont(FONT, BaseFont.IDENTITY_H, BaseFont.NOT_EMBEDDED);
        } catch (com.lowagie.text.DocumentException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 解析html生成pdf
        render.setDocumentFromString(content);
        //解决图片相对路径的问题
        render.getSharedContext().setBaseURL(LOGO_PATH);
        render.layout();
        try {
            render.createPDF(outStream);
            return outStream;
        } catch (com.lowagie.text.DocumentException e) {
            e.printStackTrace();
        } finally {
            try {
                outStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return null;
    }
 
    /**
     * 根据pdf二进制文件 生成图片文件
     *
     * @param bytes   pdf二进制
     * @param scaling 清晰度
     * @param pageNum 页数
     */
    public static ByteArrayOutputStream pdfToImg(byte[] bytes, float scaling, int pageNum,String formatName) {
        //推荐的方法打开PdfDecoder
        PdfDecoder pdfDecoder = new PdfDecoder(true);
        FontMappings.setFontReplacements();
        //修改图片的清晰度
        pdfDecoder.scaling = scaling;
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        try {
            //打开pdf文件，生成PdfDecoder对象
            pdfDecoder.openPdfArray(bytes); //bytes is byte[] array with PDF
            //获取第pageNum页的pdf
            BufferedImage img = pdfDecoder.getPageAsImage(pageNum);
 
            ImageIO.write(img, formatName, out);
        } catch (PdfException e) {
            e.printStackTrace();
        } catch (IOException e){
            e.printStackTrace();
        }
 
        return out;
    }
}
输出结果：


Jpedal支持将指定页Pdf生成图片，pdfDecoder.scaling设置图片的分辨率(不同分辨率下文件大小不同) ，支持多种图片格式，具体更多可自行研究

8.总结
对于电子凭证的技术方案，总结如下:

1.html模板+model数据，通过freemarker进行渲染，便于维护和修改

2.渲染后的html流，可通过Flying Saucer组件生成pdf文件流，或者生成pdf后再转成jpg文件流

3.在Web项目中，对应的文件流，可以通过ContentType设置，在线查看/下载，不需通过附件服务



9.纯前端解决方案
还有一种解决方案是使用PhantomJS

git地址: https://github.com/ariya/phantomjs 

PhantomJS 是一个基于 WebKit 的服务器端 JavaScript API。它全面支持web而不需浏览器支持，其快速，原生支持各种Web标准： DOM 处理, CSS 选择器, JSON, Canvas, 和 SVG。 PhantomJS 可以用于 页面自动化 ， 网络监测 ， 网页截屏 ，以及 无界面测试 等。

具体方法可自行查询。