---
layout: post
title: "记录GitHub Action托管项目的一次小小的排障"
date: 2025-02-07
summary: 今天更新了一下Home Assistant那篇blog，但是GitHub给我报错了，简单记录一下排障过程的学习过程。
categories: [学习分享]
tags: [学习]
---

## 推送新的blog时报错

![image-20250207221710476](https://weekly-liulei.oss-cn-beijing.aliyuncs.com/images/20250207221710607.png)

## 先说结论 

这个错误的核心问题是：项目中锁定的 sass-embedded 版本 1.70.0-aarch64-linux 被从 RubyGems 上删除了。因此，bundle install 无法找到该版本的 gem，导致安装失败。

## 解决办法 

**更新 Gemfile.lock 文件安装最新的可用的sass-embedded版本：**

```
rm Gemfile.lock
bundle install
```



补充一点小知识：

RubyGems 是 Ruby 语言的官方包管理系统，类似于 Python 的 pip 或 JavaScript 的 npm。它用于发布、管理和下载 Ruby 程序包（称为 gems）。任何开发者都可以将自己的 gem 发布到 RubyGems 上，但有时一些版本会被删除或弃用，原因包括：

​	1.	**安全问题**：如果发现某个版本存在安全漏洞，发布者可能会选择删除该版本。

​	2.	**兼容性问题**：如果某个版本不兼容某些环境或其他依赖库，可能会被删除或被标记为不推荐使用。

​	3.	**版本错误或损坏**：某些版本可能包含构建错误、依赖问题或其他技术问题，发布者可能会选择删除它。

​	4.	**发布者的选择**：某些版本可能被发布者决定停止维护或不再支持，因此也可能会被删除。

---

## 排障踩坑

这个报错看起来非常简单，但是我从一开始排查的时候，并没有直接找到问题的关键点。

**1. 依赖的 Gem 版本被删除，但没有及时发现**

​	•	**坑点**：错误信息提到 sass-embedded (1.70.0-aarch64-linux) 无法找到，但一开始可能没有立刻意识到是因为该版本被从 RubyGems 上移除。

​	•	**表现**：我最初以为是本地环境的问题，关注点在ruby版本上了，而不是依赖本身的问题。当我反复折腾ruby版本的时候，依赖也出了次声问题。

​	•	**改进**：

​	•	遇到类似问题时，可以第一时间去 [RubyGems 官网](https://rubygems.org/gems/sass-embedded) 查找该 gem 是否仍然存在。

​	•	直接运行 gem search sass-embedded --remote 查看可用版本。

**2.直接 bundle install，而不是 bundle update**

​	•	**坑点**：执行 bundle install 时，它会严格按照 Gemfile.lock 安装固定的 gem 版本。但如果某个 gem 版本被删除，就会导致安装失败。

​	•	**表现**：尝试 bundle install，但由于 sass-embedded (1.70.0-aarch64-linux) 已被删除，导致构建失败。

​	•	**改进**：

​	•	当遇到 **依赖不可用** 的问题时，可以先运行 bundle update sass-embedded，以获取最新可用版本。

​	•	如果 bundle update 仍然无法解决问题，可以尝试删除 Gemfile.lock，然后重新运行 bundle install。

**3.可能忽略了 Gemfile 的 platform 设置**

​	•	**坑点**：sass-embedded 这个 gem 可能是特定架构的（aarch64-linux），如果我们没有明确设置 platform，可能会遇到安装失败的问题。

​	•	**表现**：在 CI/CD 或不同平台（比如 x86_64-linux 和 aarch64-linux）之间切换时，某些 gem 可能无法兼容，导致意外的构建失败。

​	•	**改进**：

​	•	检查 Gemfile 中是否有 platform 限制，比如：

```bash
gem 'sass-embedded', platform: :ruby
```

​	•	在不同架构（如 x86_64 和 aarch64）上使用 bundle lock --add-platform aarch64-linux 确保兼容性。



## 经验教训

​	1.	**遇到 bundle install 失败，第一步先检查 Gemfile.lock 依赖是否仍然可用**。

​	2.	**不要被后续的 deprecation warning 误导，应该从最早的错误日志入手分析**。

​	3.	**适当使用 bundle update，并清理 Gemfile.lock，而不是盲目 bundle install**。

​	4.	**在 CI/CD 中清理缓存，避免使用已过期的 gem 版本**。

​	5.	**如果 gem 依赖于特定架构（如 aarch64-linux），要注意 Gemfile 的 platform 兼容性**。
