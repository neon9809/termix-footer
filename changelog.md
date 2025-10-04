# Termix Footer 功能说明

## 概述

本修改为 Termix 项目添加了在登录页面显示自定义 footer 内容的功能。footer 内容通过 Docker 外部挂载的 `footer.html` 文件提供。

## 修改内容

### 1. HomepageAuth.tsx 修改

- 添加了 `footerContent` 状态来存储 footer 内容
- 添加了 `useEffect` 钩子在组件加载时获取 footer.html 内容
- 在登录表单下方添加了 footer 显示区域
- 使用 `dangerouslySetInnerHTML` 渲染 HTML 内容

### 2. Dockerfile 修改

- 添加了新的 VOLUME 声明：`VOLUME ["/usr/share/nginx/html/footer.html"]`
- 允许外部挂载 footer.html 文件

## 使用方法

### 1. 构建 Docker 镜像

```bash
cd Termix
docker build -f docker/Dockerfile -t termix:latest .
```

### 2. 创建 footer.html 文件

在宿主机上创建一个 `footer.html` 文件，例如：

```html
<div style="padding: 10px; border-top: 1px solid #333; margin-top: 10px;">
    <p style="margin: 0; color: #888; font-size: 12px;">
        © 2024 Your Company Name. All rights reserved. | 
        <a href="/privacy" style="color: #888; text-decoration: none;">Privacy Policy</a> | 
        <a href="/terms" style="color: #888; text-decoration: none;">Terms of Service</a>
    </p>
</div>
```

### 3. 运行 Docker 容器

使用以下命令运行容器，将宿主机的 footer.html 文件挂载到容器内：

```bash
docker run -d \
  --name termix \
  -p 8080:8080 \
  -v /path/to/your/footer.html:/usr/share/nginx/html/footer.html:ro \
  -v /path/to/data:/app/data \
  termix:latest
```

**注意：**
- 将 `/path/to/your/footer.html` 替换为宿主机上 footer.html 文件的实际路径
- 将 `/path/to/data` 替换为数据目录的实际路径
- `:ro` 表示只读挂载

### 4. 使用 docker-compose

也可以使用 docker-compose.yml 文件：

```yaml
version: '3.8'
services:
  termix:
    build:
      context: .
      dockerfile: docker/Dockerfile
    ports:
      - "8080:8080"
    volumes:
      - ./footer.html:/usr/share/nginx/html/footer.html:ro
      - ./data:/app/data
    environment:
      - NODE_ENV=production
```

然后运行：

```bash
docker-compose up -d
```

## 功能特性

1. **自动加载**：页面加载时自动获取 footer.html 内容
2. **错误处理**：如果 footer.html 不存在或无法加载，不会显示错误，只是不显示 footer
3. **HTML 支持**：支持完整的 HTML 内容，包括样式和链接
4. **响应式设计**：footer 区域会适应登录表单的宽度
5. **热更新**：修改宿主机上的 footer.html 文件后，刷新页面即可看到更新

## 安全注意事项

- footer.html 文件使用 `dangerouslySetInnerHTML` 渲染，请确保内容来源可信
- 建议使用只读挂载 (`:ro`) 来防止容器修改 footer 文件
- 避免在 footer.html 中包含可执行的 JavaScript 代码

## 自定义样式建议

为了与 Termix 的深色主题保持一致，建议在 footer.html 中使用以下样式：

```html
<div style="padding: 10px; border-top: 1px solid #333; margin-top: 10px; text-align: center;">
    <p style="margin: 0; color: #888; font-size: 12px; line-height: 1.4;">
        <!-- 你的 footer 内容 -->
    </p>
</div>
```

## 故障排除

1. **Footer 不显示**：
   - 检查 footer.html 文件是否存在于指定路径
   - 检查文件权限是否正确
   - 查看浏览器开发者工具的网络选项卡，确认 `/footer.html` 请求状态

2. **样式问题**：
   - 确保 footer.html 中的 CSS 样式与 Termix 主题兼容
   - 使用内联样式以避免样式冲突

3. **容器启动失败**：
   - 检查挂载路径是否正确
   - 确保宿主机上的 footer.html 文件存在

