# 无人机制作教程

这是一个使用 MkDocs Material 构建的无人机制作教程站点，内容面向固定 Super 无人机配置：PX6C / V6C 类飞控、PX4 v1.13.3、F60 4in1 电调、Livox Mid-360、NUC Ubuntu 20.04 + ROS Noetic、Point-LIO-new 与 MAVROS 外部定位融合。

## 项目结构

```text
.
├── docs/                  # MkDocs 源码与站点资源
│   ├── index.md
│   ├── hardware/          # 硬件连线
│   ├── flight-debug/      # 飞控调试
│   ├── nuc/               # NUC 配置
│   ├── assets/            # 图片、图表、参数文件和数据资源
│   └── stylesheets/       # 自定义样式
├── mkdocs.yml             # MkDocs 配置
├── requirements.txt       # 构建依赖
├── .github/workflows/     # GitHub Pages 自动构建与部署
├── .gitignore
└── README.md
```

`docs/` 是唯一的页面源码与站点资源目录。GitHub Pages 不直接发布仓库根目录文件，而是通过 GitHub Actions 运行 MkDocs，把生成的 `site/` artifact 部署出去。

## 本地预览

安装依赖：

```bash
python -m pip install -r requirements.txt
```

启动本地预览：

```bash
mkdocs serve
```

默认访问地址：

```text
http://127.0.0.1:8000/
```

## 本地构建

严格构建并输出到临时验证目录：

```bash
mkdocs build --strict -d .verify-site
```

`.verify-site/` 已在 `.gitignore` 中忽略，不需要提交。

## GitHub Pages

仓库已包含 GitHub Actions 工作流：

```text
.github/workflows/pages.yml
```

推送到 `master` 后，工作流会自动：

1. 安装 `requirements.txt` 中固定的 MkDocs 依赖。
2. 执行 `mkdocs build --strict --site-dir site`。
3. 上传 `site/` 作为 GitHub Pages artifact。
4. 部署到 GitHub Pages。

GitHub 仓库的 Pages 设置需要使用 `GitHub Actions` 作为发布源。

## 参数文件

固定参数文件放在：

```text
docs/assets/params/Super01_Gate15SD.params
```

站点构建后会随 `assets/params/` 一起发布，可在“方案一：导入已有参数文件”页面中下载。

## 依赖版本

当前固定版本：

```text
mkdocs==1.6.1
mkdocs-material==9.7.6
```
