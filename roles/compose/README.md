# Docker Compose Ansible Role

## 介绍

该 Ansible 角色用于自动化部署和管理 Docker Compose 配置文件及其相关应用容器。它将根据模板渲染配置文件，并通过 Docker Compose 启动容器，简化了 Docker 应用的部署过程。

## 功能
- 创建远程服务器的目录环境
- 渲染并生成环境环境`.env`
- 渲染并生成 `docker-compose.yml` 配置文件。
- 将生成的配置文件部署到指定的远程服务器目录。
- 使用 Docker Compose 启动和管理容器服务。
- 支持将 Docker Compose 配置文件放入远程服务器的特定目录。

## 目录结构

```plaintext
roles/
├── docker/
│   ├── tasks/
│   │   └── main.yml        # 主要任务
│   ├── templates/
│   │   └── docker-compose.yml.j2  # Docker Compose 模板文件
│   ├── defaults/
│   │   └── main.yml        # 默认变量
│   ├── vars/
│   ├── handlers/
│   └── files/
```

## 安装

1. 将此角色添加到你的 Ansible 项目中：
   ```bash
   git clone https://your-repository-url roles/docker
   ```

2. 在你的 `playbook.yml` 中引用该角色：
   ```yaml
   ---
   - name: Deploy Docker Compose Application
     hosts: remote_servers
     become: true
     roles:
       - docker
   ```

## 变量

你可以在 `defaults/main.yml` 中配置以下变量：

```yaml
# 远程服务器上存放 Docker Compose 配置文件的目录
compose_dir: "/opt/docker/containers/compose"

# Docker Compose 配置文件的路径（模板）
compose_template: "docker-compose.yml.j2"

# 渲染后的 Docker Compose 配置文件路径
compose_file: "{{ compose_dir }}/docker-compose.yml"

# Docker Compose 项目的目录
project_src: "{{ compose_dir }}"
```

### 可选变量

这些变量可以根据需要进行设置：

```yaml
# 配置 Docker Compose 项目的环境变量目录
env_file: "{{ compose_dir }}/.env"

# Docker Compose 服务配置的其他目录路径
app_data_dir: "/opt/docker/appdata"
```
## 管理容器

### 示例
```yaml
- name: Test compose role
  hosts: localhost
  roles:
    - role: compose
      vars:
        compose_root_dir: /home/ubuntu/containers # 配置默认的容器管理目录
        compose_proxy_enable: true  # 选择true就配置网络
        compose_traefik_enable: false # 部署traefik时打开
        compose_apps:  # 需要开启的容器，容器的compose文件需要先放置再roles/file目录下
          - name: portainer  # 容器名称
            state: present  # present 表示编排部署容器，absent 表示删除容器
            rm_conf: false  # 是否删除容器保存在~/containers/appdata的持久化数据
```

## 使用示例

### Playbook 示例

这是一个使用该角色部署 Docker Compose 应用的完整 Playbook 示例：

```yaml
---
- name: Setup Docker Compose on remote server
  hosts: remote_servers
  become: true
  roles:
    - docker

- name: Deploy and start Docker Compose application
  hosts: remote_servers
  become: true
  tasks:
    - name: Render docker-compose.yml from template
      template:
        src: "docker-compose.yml.j2"
        dest: "/opt/docker/containers/compose/docker-compose.yml"
        mode: '0644'

    - name: Start Docker Compose application
      docker_compose:
        project_src: "/opt/docker/containers/compose"
        state: present
```

### Docker Compose 模板示例

在 `templates/docker-compose.yml.j2` 中，你可以自定义 Docker Compose 配置模板。例如：

```yaml
version: '3.7'

services:
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    security_opt:
      - no-new-privileges:true
    restart: unless-stopped
    networks:
      - traefik
    ports:
      - "5375:5375"
    volumes:
      - "{{ app_data_dir }}/qbittorrent:/config"
      - "{{ download_dir }}:/downloads"

networks:
  traefik:
    external: true
```

## 依赖

- **Ansible 2.9+**  
- **Docker 20.10+**  
- **Docker Compose 1.28+**

## 注意事项

- 在远程服务器上使用时，请确保已安装 Docker 和 Docker Compose。
- 如果你有特定的网络设置或额外的环境变量，可以在 `defaults/main.yml` 中进行配置。

## 贡献

欢迎提交 Issue 或 Pull Request 来改进该角色！对于任何问题或功能需求，请通过 GitHub Issues 提交。

## License

MIT License