# My blog

## 如何开发

### 安装 Hexo

```bash
npm install -g hexo-cli
```

### 新建文章

```bash
hexo new '文章名'
```

### 本地预览

```bash
hexo s
```

---

## 如何部署

### 生成静态文件

> 建议在 `hexo g` 之前执行 `hexo clean` 清除缓存

```bash
hexo g
```

### 部署至远程

> 需要在根目录下 `_config.yml` 中配置 `deploy` 的字段

```bash
hexo deploy
```
