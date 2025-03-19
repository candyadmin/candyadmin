---
title: docker中运行chrome并使用代码控制
date: 2025-03-18 15:38:54
categories: docker
tag: chrome
---
#使用docker运行chrome
```shell
docker run -d -p 4444:4444 -p 5900:5900 --name selenium-chrome-debug selenium/standalone-chrome-debug
```
#使用java代码控制chrome
```java
package org.example;

import org.openqa.selenium.By;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.remote.RemoteWebDriver;
import java.net.MalformedURLException;
import java.net.URL;

public class SeleniumDockerTest {
    public static void main(String[] args) {
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--disable-dev-shm-usage");  // 解决 Chrome 崩溃问题
        options.addArguments("--no-sandbox");  // 避免 Chrome 在容器中受限制
//        options.addArguments("--user-data-dir=/home/seluser");  // 设置 Chrome 用户数据目录

        WebDriver driver = null;
        try {
            driver = new RemoteWebDriver(new URL("http://10.8.0.5:4444/wd/hub"), options);

            driver.get("https://www.jd.com");
            System.out.println("页面标题: " + driver.getTitle());
            WebElement body = driver.findElement(By.cssSelector("#J_cate > ul"));
            System.out.println("页面内容: " + body.getText());
            System.in.read();
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("错误: " + e.getMessage());
        } finally {
            if (driver != null) {
                driver.quit();
            }
        }
    }
}

```