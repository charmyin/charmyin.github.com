---
layout: post
title: "Load properties file to beans in spring mvc"
description: "Load properties file to beans in spring mvc"
category: [springmvc]
tags: [springmvc, java]
---

---------------------------------------

####Load properties file to beans in spring mvc

**There are a few different ways to do it. I do the following. In app context:**

````<util:properties id="myProps" location="classpath:app.properties" />````

**Make sure you have the following at the top of your file to include the "util" namespace:**

        xmlns:util="http://www.springframework.org/schema/util"
        xsi:schemaLocation= "... http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-3.0.xsd"

I usually put my properties files in src/main/resources. As long as they're on the classpath, you're good. Then, in your controller, you can annotate your field/method with:

        @Controller
        public class MyController {

            @Value("#{myProps['uploadFolder']}")
            private String uploadFolder

            @Value("#{myProps['downloadFolder']}")
            private String downloadFolder

            @RequestMapping("myPage")
            public String loadPage(ModelMap m) {
                m.addAttribute("uploadFolder", uploadFolder);
                m.addAttribute("downloadFolder", downloadFolder);
                return "myPage";
            }

            public void setUploadFolder(String uploadFolder) {
                this.uploadFolder = uploadFolder;
            }
            public void setDownloadFolder(String downloadFolder) {
                this.downloadFolder = downloadFolder;
            }
        }

**Then, in your JSP:**

Download folder: ````${downloadFolder}````
Upload folder: ````${uploadFolder}````
HTH. Let me know if you have any questions.
