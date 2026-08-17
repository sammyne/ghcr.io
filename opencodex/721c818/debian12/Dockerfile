ARG OPENCODE_VERSION=1.18.18
ARG CLIPROXYAPI_VERSION=v7.2.132

# ---- 阶段 1：opencode（npm 淘宝源，不碰 GitHub）----
FROM node:24.19.0-bookworm-slim AS opencode
ARG OPENCODE_VERSION
# 换源：注释下一行回落 registry.npmjs.org
ENV npm_config_registry=https://registry.npmmirror.com
WORKDIR /out
RUN npm install -g "opencode-ai@${OPENCODE_VERSION}"
RUN set -eux; \
    BIN="$(readlink -f "$(command -v opencode)")"; \
    MAGIC="$(head -c 4 "$BIN" | od -An -tx1 | tr -d ' \n')"; \
    [ "$MAGIC" = "7f454c46" ] \
      || { echo "FATAL: opencode at $BIN is not a self-contained ELF binary" >&2; exit 1; }; \
    cp "$BIN" /out/opencode; \
    /out/opencode --version

# ---- 阶段 2：CLIProxyAPI（go install + goproxy.cn，不碰 GitHub）----
FROM golang:1.26.6-bookworm AS cliproxyapi
ARG CLIPROXYAPI_VERSION
# 换源：注释下一行回落 proxy.golang.org,direct
ENV GOPROXY=https://goproxy.cn,direct
WORKDIR /out
RUN GOBIN=/out go install "github.com/router-for-me/CLIProxyAPI/v7/cmd/server@${CLIPROXYAPI_VERSION}"
RUN set -eux; \
    mv /out/server /out/CLIProxyAPI; \
    test -x /out/CLIProxyAPI

# ---- 阶段 3：provider 包预热（@ai-sdk/openai，npmmirror）----
FROM node:24.19.0-bookworm-slim AS prewarm
# 换源：注释下一行回落 registry.npmjs.org
ENV npm_config_registry=https://registry.npmmirror.com
WORKDIR /out
RUN printf '{"name":"opencode-prewarm","private":true}\n' > package.json
RUN npm install "@ai-sdk/openai@4.0.42" --no-audit --no-fund

# ---- 阶段 4：rc（制品暂存：聚合所有应用制品，供最终阶段单层拷贝）----
# 约束：/staging 内路径不得与 debian 系统目录（etc/var/usr 等）冲突，COPY --from=rc 会整层覆盖到镜像根
FROM debian:bookworm-20260803-slim AS rc
WORKDIR /staging
COPY --from=opencode /out/opencode usr/local/bin/opencode
COPY --from=cliproxyapi /out/CLIProxyAPI usr/local/bin/CLIProxyAPI
COPY --from=prewarm /out opt/opencode-prewarm
COPY entrypoint.sh usr/local/bin/entrypoint.sh
# healthcheck 脚本内联（quoted heredoc 零展开，awk/$ 原样落盘），消除独立构建上下文文件
RUN cat > usr/local/bin/healthcheck.sh <<'EOF'
#!/usr/bin/env bash
set -uo pipefail

# 与 entrypoint.sh 的 key_from_config 同一解析规则
KEY="$(awk '/^api-keys:/{f=1;next} f && /^  - /{sub(/^  - */,""); gsub(/"/,""); print; exit}' /root/config.yaml 2>/dev/null || true)"

# 主判定：带鉴权打 /v1/models
if [ -n "$KEY" ] && curl -fsS -m 4 -H "Authorization: Bearer $KEY" http://127.0.0.1:8317/v1/models >/dev/null 2>&1; then
  exit 0
fi

# 兜底：代理端口可连即视为进程存活
(exec 3<>/dev/tcp/127.0.0.1/8317) 2>/dev/null
EOF
RUN set -eux; \
    chmod +x usr/local/bin/entrypoint.sh usr/local/bin/healthcheck.sh \
              usr/local/bin/opencode usr/local/bin/CLIProxyAPI; \
    bash -n usr/local/bin/healthcheck.sh; \
    ./usr/local/bin/opencode --version; \
    ./usr/local/bin/CLIProxyAPI --help > /dev/null 2>&1 || true; \
    test -d opt/opencode-prewarm/node_modules/@ai-sdk
# xdg-open 哑垫片：容器内无浏览器，opencode web 启动会 spawn 它，缺失则崩溃
RUN printf '#!/bin/sh\nexit 0\n' > usr/local/bin/xdg-open && chmod +x usr/local/bin/xdg-open

# ---- 最终阶段 ----
FROM debian:bookworm-20260803-slim
# 换源：注释下面整个 RUN 回落 deb.debian.org（bookworm 自带 debian.sources）
RUN set -eux; \
    rm -f /etc/apt/sources.list.d/debian.sources; \
    printf 'deb http://mirrors.aliyun.com/debian bookworm main\n\
deb http://mirrors.aliyun.com/debian bookworm-updates main\n' > /etc/apt/sources.list
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends ca-certificates curl; \
    rm -rf /var/lib/apt/lists/*
COPY --from=rc /staging/ /
ENV OPENCODE_PORT=4096
EXPOSE 4096
WORKDIR /root
HEALTHCHECK --interval=30s --timeout=5s --start-period=30s --retries=3 \
  CMD ["/usr/local/bin/healthcheck.sh"]
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
