---
layout: post
title: "Get file's mime type by path"
description: "Get file's mime type by path~"
category: [springmvc]
tags: [springmvc, java]
---

---------------------------------------

####Load properties file to beans in spring mvc
In Java 7 you can now just use [Files.probeContentType(path)](http://docs.oracle.com/javase/7/docs/api/java/nio/file/Files.html#probeContentType%28java.nio.file.Path%29)

**eg:**

    try {
            String mimeType = Files.probeContentType(FileSystems.getDefault().getPath(filePath+fileName));
            response.setContentType(mimeType);
            IOUtils.copy(inputStream, response.getOutputStream());
        } finally {
            IOUtils.closeQuietly(inputStream);
        }
