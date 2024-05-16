---
title: gitlab-tips.md
date: 2024-05-16 16:17:56
tags: [gitlab, ci/cd, ChatGPT]
---

# gitlab stages 如何并行

在 GitLab CI/CD 中，你可以通过定义作业（jobs）并指定它们属于同一个阶段（stage），来实现这些作业在该阶段并行执行。默认情况下，同一个阶段中的作业会并行执行，前提是你有足够的并发运行器（runner）来支持并行执行。如果运行器数量不足，作业将会排队等待可用的运行器。

以下是一个简单的 `.gitlab-ci.yml` 文件示例，展示了如何设置并行作业：

```yaml
stages:
  - build
  - test

job1:
  stage: build
  script:
    - echo "This is job1"

job2:
  stage: build
  script:
    - echo "This is job2"

test1:
  stage: test
  script:
    - echo "This is test1"

test2:
  stage: test
  script:
    - echo "This is test2"
```

在这个例子中，`job1` 和 `job2` 都属于 `build` 阶段，它们将会并行执行（前提是有足够的运行器）。同样的，`test1` 和 `test2` 都属于 `test` 阶段，也会并行执行。

如果你想要更细粒度地控制作业的并行性，你可以使用 GitLab 的 `parallel` 关键字。这允许你将单个作业分成多个实例，它们可以并行运行。例如：

```yaml
test:
  stage: test
  script:
    - echo "This is a parallel test"
  parallel: 3
```

在这个例子中，`test` 作业将会启动三个并行的实例。

要注意的是，GitLab Runner 的并发数是可以配置的。在 Runner 的配置文件中，你可以设置 `concurrent` 的值来限制并发作业的数量。如果你的并发数设置得太低，即使你在 `.gitlab-ci.yml` 中配置了并行作业，它们也可能不会真正并行执行。

确保你的 GitLab Runner 配置可以支持你期望的并行作业数量，这样你才能充分利用 GitLab CI/CD 的并行执行能力。


