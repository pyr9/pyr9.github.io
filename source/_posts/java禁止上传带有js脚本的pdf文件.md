---
title: java禁止上传带有js脚本的pdf文件
date: 2025-01-24 10:54:10
tags: xss跨站漏洞
categories: 工具类
---

防止上传的pdf文件中，包含危害的Java script，导致的XSS漏洞

# 1. 引入依赖

```xml
        <dependency>
            <groupId>org.apache.pdfbox</groupId>
            <artifactId>pdfbox</artifactId>
            <version>2.0.27</version>
        </dependency>
```

# 2. 编写工具类

```java
package com.siemens.mmf.sith.utils;

import lombok.extern.slf4j.Slf4j;
import org.apache.pdfbox.cos.COSArray;
import org.apache.pdfbox.cos.COSDictionary;
import org.apache.pdfbox.cos.COSName;
import org.apache.pdfbox.cos.COSString;
import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.pdmodel.PDPage;
import org.apache.pdfbox.pdmodel.PDPageTree;
import org.apache.pdfbox.pdmodel.PDResources;
import org.apache.pdfbox.pdmodel.font.PDFont;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.util.regex.Pattern;

@Slf4j
public class PdfFileUtil {
    private static final Pattern JAVASCRIPT_PATTERN = Pattern.compile("(?i)javascript|alert\\(|document\\.|window\\.");

    /**
     * 检查PDF页面对象和页面中的字体资源 是否含有恶意的 JavaScript代码
     */
    public static boolean hasXSS(MultipartFile multipartFile) {
        try {
            PDDocument document = PDDocument.load(multipartFile.getInputStream());
            PDPageTree pages = document.getPages();
            for (PDPage page : pages) {
                if (hasXSS(page.getCOSObject())) {
                    return true;
                }
                PDResources resources = page.getResources();
                if (resources != null) {
                    for (COSName fontName : resources.getFontNames()) {
                        PDFont font = resources.getFont(fontName);
                        if (font != null && hasXSS(font.getCOSObject())) {
                            return true;
                        }
                    }
                }
            }
        } catch (IOException e) {
            log.error("read line exception" + e.getMessage());
        }
        return false;
    }

    /**
     * 检查JavaScript并递归检查嵌套对象
     */
    private static boolean hasXSS(COSDictionary obj) {
        if (obj.containsKey(COSName.JAVA_SCRIPT)) {
            return true;
        }
        // 递归地检查字典中的所有嵌套对象
        for (COSName key : obj.keySet()) {
            Object value = obj.getItem(key);
            if (value instanceof COSDictionary) {
                if (hasXSS((COSDictionary) value)) {
                    return true;
                }
            } else if (value instanceof COSArray) {
                if (checkArrayHasXSS((COSArray) value)) {
                    return true;
                }
            }
        }
        return false;
    }


    private static boolean checkArrayHasXSS(COSArray array) {
        for (int i = 0; i < array.size(); i++) {
            Object element = array.get(i);
            if (element instanceof COSString && hasXSS(((COSString) element).getString())) {
                return true;
            } else if (element instanceof COSDictionary) {
                if (hasXSS((COSDictionary) element)) {
                    return true;
                }
            } else if (element instanceof COSArray) {
                if (checkArrayHasXSS((COSArray) element)) {
                    return true;
                }
            }
        }
        return false;
    }

    private static boolean hasXSS(String value) {
        return JAVASCRIPT_PATTERN.matcher(value).find();
    }

}

```

测试文件地址：[pdf](https://oicu0619.github.io/poc.pdf)

