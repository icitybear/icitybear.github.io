---
title: "Tool5" #标题
date: 2026-05-07T17:05:58+08:00 #创建时间
lastmod: 2026-05-07T17:05:58+08:00 #更新时间
author: ["citybear"] #作者
categories: # 没有分类界面可以不填写
- tech
tags: # 标签
- 工具
- 测试
keywords: 
- 
description: "" #描述 每个文章内容前面的展示描述
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
draft: false # 是否为草稿
comments: true #是否展示评论 有自带的扩展成twikoo
showToc: true # 显示目录 文章侧边栏toc目录
TocOpen: true # 自动展开目录
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
showbreadcrumbs: true #顶部显示当前路径
cover:
    image: "" #图片路径：posts/tech/文章1/picture.png
    caption: "" #图片底部描述
    alt: ""
    relative: false

# reward: true # 打赏
mermaid: true #自己加的是否开启mermaid
---

## 1. 背景与痛点分析

UI 自动化测试长期以来面临着诸多挑战：
- **脚本设计复杂**：传统工具需要大量编码工作
- **调试耗时**：定位元素和等待问题消耗大量时间
- **维护成本高**：页面变化导致测试用例频繁失效
- **跨浏览器兼容性差**：不同浏览器需要不同的测试脚本
- **执行速度慢**：传统的 WebDriver 通信方式效率低下
  
Playwright 作为现代自动化测试工具，旨在彻底解决这些痛点，提供更稳定、快速、易用的测试解决方案。

## 2. Playwright 核心优势

Playwright 是一个由 Microsoft 开发的跨浏览器自动化工具，Python 版本提供了完整的 API 支持。

### 核心特点：
1. **真正的跨浏览器**：支持 Chromium、Firefox 和 WebKit
2. **Pythonic API**：符合 Python 开发习惯的简洁语法
3. **无需 WebDriver**：直接通过浏览器开发工具协议通信
4. **自动等待机制**：内置智能等待，测试更稳定
5. **强大的调试能力**：内置追踪、截图、视频录制
6. **网络控制**：完整的请求/响应拦截和模拟
7. **移动端支持**：原生移动浏览器模拟
8. **认证管理**：支持多种认证方式持久化
9. **并行执行**：原生支持测试并行化
10. **丰富的工具链**：代码生成、调试、追踪一体化

### 官方支持的语言：
1. Node.js（TypeScript/JavaScript）— 原生版本，功能最全
2. [Python](https://github.com/microsoft/playwright-python)
3. [Go](https://github.com/playwright-community/playwright-go) 社区有非官方的第三方绑定，比如 playwright-go，但它不是 Microsoft 官方维护的，更新可能滞后，API 覆盖也不完整。
4. Java .NET（C#）

## 3. 环境安装与配置

### 基础安装

```bash
# 使用国内镜像加速安装
pip install playwright -i https://pypi.tuna.tsinghua.edu.cn/simple

# 安装浏览器环境（推荐只安装 Chromium 以节省空间）
playwright install --with-deps chromium

# 安装测试框架支持
pip install pytest-playwright allure-pytest pytest-html
```


### 验证安装

```python
from playwright.sync_api import sync_playwright

def test_basic():
    with sync_playwright() as p:
        browser = p.chromium.launch(headless=False)
        page = browser.new_page()
        page.goto("https://www.baidu.com")
        print(f"页面标题: {page.title()}")
        browser.close()

if __name__ == "__main__":
    test_basic()
```


## 4. 基础使用与脚本录制

### UI 脚本录制

Playwright 提供了强大的脚本录制功能，可以快速生成测试代码：

```bash
# 启动脚本录制工具
playwright codegen https://www.baidu.com

# 指定浏览器和输出文件
playwright codegen --target python -o test_baidu.py https://www.baidu.com
```


### 录制生成的示例脚本

```python
from playwright.sync_api import sync_playwright, expect

def test_baidu_search():
    with sync_playwright() as p:
        # 启动浏览器
        browser = p.chromium.launch(headless=False)
        context = browser.new_context()
        page = context.new_page()

        # 导航到百度首页
        page.goto("https://www.baidu.com/")

        # 定位搜索框并输入关键词
        search_box = page.locator("#kw")
        search_box.fill("playwright")
        
        # 点击搜索按钮
        search_button = page.locator("#su")
        search_button.click()

        # 等待结果加载并验证
        expect(page.locator('//*[@id="3"]/div/h3/a/div/div/p/span/span')).to_have_text("playwright(英语单词) - 百度百科")

        # 截图保存
        page.screenshot(path="search_result.png")

        # 关闭浏览器
        browser.close()

if __name__ == "__main__":
    test_baidu_search()
```


## 5. Pytest 集成实战

### 安装 Pytest 插件

```bash
pip install pytest-playwright
```


### 基础测试用例

```python
import pytest
from playwright.sync_api import Page, expect

class TestLogin:
    """登录功能测试"""
    
    def test_successful_login(self, login_page: Page):
        """测试成功登录"""
        login_page.fill("#username", "testuser")
        login_page.fill("#password", "password123")
        login_page.click("#login-btn")
        
        expect(login_page).to_have_url("https://example.com/dashboard")
        expect(login_page.locator(".welcome-message")).to_contain_text("欢迎")
    
    def test_login_failure(self, login_page: Page):
        """测试登录失败"""
        login_page.fill("#username", "wronguser")
        login_page.fill("#password", "wrongpass")
        login_page.click("#login-btn")
        
        expect(login_page.locator(".error-message")).to_be_visible()
        expect(login_page.locator(".error-message")).to_contain_text("用户名或密码错误")
```


## 6. 高级特性与最佳实践

### 6.1 页面对象模型 (Page Object Pattern)

```python
# tests/pages/login_page.py
from playwright.sync_api import Page, expect

class LoginPage:
    """登录页面对象模型"""
    
    def __init__(self, page: Page):
        self.page = page
        self.username_input = page.locator("#username")
        self.password_input = page.locator("#password")
        self.login_button = page.locator("#login-btn")
        self.error_message = page.locator(".error-message")
        self.success_message = page.locator(".welcome-message")
    
    def navigate(self):
        """导航到登录页面"""
        self.page.goto(f"{TestConfig.BASE_URL}/login")
    
    def login(self, username: str, password: str):
        """执行登录操作"""
        self.username_input.fill(username)
        self.password_input.fill(password)
        self.login_button.click()
    
    def assert_login_success(self):
        """验证登录成功"""
        expect(self.page).to_have_url(f"{TestConfig.BASE_URL}/dashboard")
        expect(self.success_message).to_be_visible()
    
    def assert_login_error(self, expected_message: str):
        """验证登录失败"""
        expect(self.error_message).to_be_visible()
        expect(self.error_message).to_contain_text(expected_message)
```


### 6.2 网络请求拦截与模拟

```python
import json
from playwright.sync_api import Route

def test_api_mocking(page: Page):
    """API 请求模拟测试"""
    
    def handle_route(route: Route):
        # 拦截 API 请求并返回模拟数据
        route.fulfill(
            status=200,
            contentType="application/json",
            body=json.dumps({"name": "测试用户", "id": 1, "email": "test@example.com"})
        )
    
    # 注册路由拦截
    page.route("**/api/user/*", handle_route)
    
    page.goto("https://example.com/profile")
    expect(page.locator(".user-name")).to_contain_text("测试用户")

def test_network_monitoring(page: Page):
    """网络请求监控"""
    
    # 监听所有网络请求
    requests = []
    page.on("request", lambda request: requests.append(request.url))
    
    page.goto("https://example.com")
    
    # 验证关键请求是否发送
    assert any("/api/user" in url for url in requests), "用户API请求未发送"
```


### 6.3 文件上传下载处理

```python
def test_file_upload(page: Page):
    """文件上传测试"""
    # 文件上传
    with page.expect_file_chooser() as fc_info:
        page.locator("input[type='file']").click()
    file_chooser = fc_info.value
    file_chooser.set_files("test.pdf")
    
    # 验证上传成功
    expect(page.locator(".upload-success")).to_be_visible()

def test_file_download(page: Page):
    """文件下载测试"""
    # 监听下载事件
    with page.expect_download() as download_info:
        page.locator("#download-btn").click()
    
    download = download_info.value
    # 保存下载文件
    download.save_as(f"downloads/{download.suggested_filename}")
    print(f"下载文件: {download.path()}")
```


### 6.4 错误处理与重试机制

```python
import logging
import time
from functools import wraps

def retry_on_failure(max_attempts=3, delay=1):
    """失败重试装饰器"""
    def decorator(test_func):
        @wraps(test_func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return test_func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise e
                    logging.warning(f"测试失败，第{attempt + 1}次重试。错误: {e}")
                    time.sleep(delay)
            return None
        return wrapper
    return decorator

class TestFlakyFeatures:
    @retry_on_failure(max_attempts=3)
    def test_flaky_api_integration(self, page: Page):
        """处理不稳定的 API 集成测试"""
        page.goto("https://example.com/dashboard")
        expect(page.locator(".api-data")).to_be_visible(timeout=10000)
```


### 6.5 性能测试与监控

```python
def test_performance_metrics(page: Page):
    """性能监控与指标收集"""
    
    # 启动性能监控
    page.context.tracing.start(
        screenshots=True, 
        snapshots=True, 
        sources=True
    )
    
    # 导航到页面并等待加载完成
    page.goto("https://example.com")
    page.wait_for_load_state("networkidle")
    
    # 获取性能指标
    metrics = page.evaluate("""() => {
        const { performance } = window;
        const navEntry = performance.getEntriesByType('navigation')[0];
        return {
            domContentLoaded: navEntry.domContentLoadedEventEnd - navEntry.domContentLoadedEventStart,
            loadComplete: navEntry.loadEventEnd - navEntry.loadEventStart,
            firstPaint: performance.getEntriesByName('first-paint')[0].startTime,
            firstContentfulPaint: performance.getEntriesByName('first-contentful-paint')[0].startTime,
        };
    }""")
    
    # 断言性能指标
    assert metrics["firstContentfulPaint"] < 3000, "首屏渲染时间过长"
    assert metrics["domContentLoaded"] < 2000, "DOM 加载时间过长"
    
    # 停止追踪并保存
    page.context.tracing.stop(path="trace.zip")
```


### 6.6 跨浏览器测试配置

```python
import pytest

@pytest.mark.parametrize("browser_name", ["chromium", "firefox", "webkit"])
def test_cross_browser(login_page: Page, browser_name):
    """跨浏览器兼容性测试"""
    login_page.login("testuser", "password123")
    expect(login_page.page).to_have_url("https://example.com/dashboard")
```


## 7. 可视化测试报告输出

### 安装与配置

```bash
# 安装 Allure
pip install allure-pytest

# 或者安装 HTML 报告
pip install pytest-html
```


### 测试报告配置

```python
# pytest.ini 配置文件
[pytest]
addopts = 
    --tb=short
    --alluredir=allure-results
    --html=report.html
    --self-contained-html
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
```


### 生成测试报告

```python
# run_tests.py
import pytest
import os

if __name__ == "__main__":
    # 运行测试并生成报告
    pytest.main([
        "tests/",
        "--alluredir", "allure-results",
        "--html", "report.html",
        "--self-contained-html"
    ])
    
    # 生成 Allure 报告
    os.system("allure generate allure-results -o allure-report --clean")
```


### 运行测试

```bash
# 运行测试并生成报告
python run_tests.py

# 或者直接使用 pytest
pytest --alluredir=allure-results --html=report.html

# 查看 Allure 报告
allure serve allure-results
```


## 8. Jenkins 持续集成

### GitHub Actions 配置

```yaml
# .github/workflows/playwright.yml
name: Playwright Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.8, 3.9, 3.10]
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}
    
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        playwright install --with-deps chromium
    
    - name: Run tests
      run: |
        pytest --alluredir=allure-results --html=report.html
      env:
        HEADLESS: true
        BASE_URL: ${{ secrets.TEST_BASE_URL }}
    
    - name: Upload test results
      uses: actions/upload-artifact@v3
      with:
        name: test-results-${{ matrix.python-version }}
        path: |
          allure-results/
          report.html
        retention-days: 30
    
    - name: Publish Allure Report
      uses: simple-elf/allure-report-action@master
      if: always()
      with:
        allure_results: allure-results
        allure_report: allure-report
        keep_reports: 20
```


### Jenkinsfile 配置

```groovy
pipeline {
    agent any
    
    tools {
        python 'python3'
    }
    
    stages {
        stage('Setup') {
            steps {
                sh 'python -m pip install --upgrade pip'
                sh 'pip install -r requirements.txt'
                sh 'playwright install --with-deps chromium'
            }
        }
        
        stage('Test') {
            steps {
                sh 'pytest --alluredir=allure-results --html=report.html'
            }
        }
        
        stage('Report') {
            steps {
                allure includeProperties: false, jdk: '', results: [[path: 'allure-results']]
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: '.',
                    reportFiles: 'report.html',
                    reportName: 'HTML Report'
                ])
            }
        }
    }
    
    post {
        always {
            archiveArtifacts artifacts: 'allure-results/,report.html', fingerprint: true
        }
    }
}
```


## 9. 优势总结与最佳实践

### 🎯 核心优势总结

1. **稳定的测试**：自动等待机制大幅提升测试稳定性
2. **极速执行**：比传统 Selenium 快 2-3 倍
3. **Pythonic API**：符合 Python 开发习惯，学习成本低
4. **强大的工具链**：代码生成、调试、追踪一体化
5. **丰富的报告**：截图、视频、追踪文件一应俱全
6. **跨浏览器支持**：真正的多浏览器兼容性测试
7. **移动端测试**：原生支持移动端浏览器模拟

## 官方资源与学习材料

- **Python 文档**: https://playwright.dev/python/docs/intro
- **API 参考**: https://playwright.dev/python/docs/api/class-playwright
- **GitHub 仓库**: https://github.com/microsoft/playwright-python

