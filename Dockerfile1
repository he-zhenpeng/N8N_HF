# Stage 1: Node builder
FROM node:22-slim as nodebuilder

RUN npm install -g pnpm pm2 n8n

# Stage 2: rclone
FROM rclone/rclone:latest as rclone

# Stage 3: Final image
FROM python:3.13-slim-bullseye

ENV DEBIAN_FRONTEND=noninteractive

# 仅安装必要依赖
RUN apt-get update && apt-get install -y \
    curl zsh git sudo unzip tzdata openssl nginx jq \
    netcat sshpass procps wget ca-certificates \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# 拷贝 node 工具和 rclone
COPY --from=nodebuilder /usr/local/bin/node /usr/local/bin/
COPY --from=nodebuilder /usr/local/lib/node_modules /usr/local/lib/node_modules
COPY --from=nodebuilder /usr/local/bin/npm /usr/local/bin/
COPY --from=nodebuilder /usr/local/bin/n8n /usr/local/bin/n8n
COPY --from=rclone /usr/local/bin/rclone /usr/bin/rclone

RUN ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm

# 安装 code-server
RUN curl -fsSL https://code-server.dev/install.sh | sh -s -- --version=4.23.0-rc.2

# 创建 coder 用户
RUN useradd -m -s /bin/zsh coder && echo 'coder ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

USER coder
ENV HOME=/home/coder \
    PATH=/home/coder/.local/bin:$PATH

COPY --chown=coder start_server.sh $HOME
COPY --chown=coder apps.conf $HOME
COPY --chown=coder nginx.conf $HOME

RUN chmod +x $HOME/start_server.sh && \
    mkdir -p $HOME/.local/share/code-server/User && \
    echo '{ "workbench.colorTheme": "Default Dark Modern" }' > $HOME/.local/share/code-server/User/settings.json

WORKDIR /home/coder

EXPOSE 5700

CMD ["sh", "-c", "/home/coder/start_server.sh"]
