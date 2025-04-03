- 安装依赖
	```
	pkg update
	
	pkg install golang git make nodejs
	
	npm install -g pm2
	```
- 克隆并进入仓库
	```
	git clone https://github.com/binwiederhier/ntfy
	
	cd ntfy
	```
- 创建构建脚本
	```
	cat > build_termux.sh << 'EOF'
	#!/data/data/com.termux/files/usr/bin/bash
	
	export GOPATH=$HOME/go
	export CGO_ENABLED=1
	export GOOS=android
	export GOARCH=arm64  # 对于大多数现代手机，如果是较旧设备可能需要使用arm
	
	# 设置版本信息
	VERSION=$(git describe --tag 2>/dev/null || echo "dev")
	COMMIT=$(git rev-parse --short HEAD 2>/dev/null || echo "unknown")
	DATE=$(date +%s)
	
	# 构建ntfy二进制文件
	go build \
	  -o ntfy \
	  -tags sqlite_omit_load_extension,osusergo,netgo \
	  -ldflags "-s -w -X main.version=$VERSION -X main.commit=$COMMIT -X main.date=$DATE"
	
	echo "ntfy构建完成！"
	EOF
	
	chmod +x build_termux.sh
	```
- 运行构建脚本
	- `.build_termux.sh`
- 创建配置文件
	```
	mkdir -p ~/.config/ntfy
	
	cat > ~/.config/ntfy/server.yml << 'EOF'
	base-url: http://localhost:8327
	listen-http: :8327
	cache-file: /data/data/com.termux/files/home/.cache/ntfy/cache.db
	attachment-cache-dir: /data/data/com.termux/files/home/.cache/ntfy/attachments
	auth-file: /data/data/com.termux/files/home/.config/ntfy/auth.db
	auth-default-access: read-write
	EOF
	
	# 创建缓存目录
	mkdir -p ~/.cache/ntfy/attachments
	
	# 复制构建好的ntfy二进制文件到termux 的bin目录
	cp ntfy $PREFIX/bin/
	```
- 直接运行服务
	```
	# 运行
	ntfy serve --config ~/.config/ntfy/server.yml
	```
- 使用`pm2`来运行
	- 创建`vi ~/.config/ntfy/ecosystem.config.js`
		```
		module.exports = {
		  apps: [{
		    name: "ntfy",
		    script: "ntfy",
		    args: "serve --config /data/data/com.termux/files/home/.config/ntfy/server.yml",
		    watch: false,
		    autorestart: true,
		    restart_delay: 5000,
		    max_memory_restart: "100M",
		    exp_backoff_restart_delay: 100,
		    max_restarts: 10,
		    min_uptime: "10s",
		    out_file: "~/.pm2/logs/ntfy-out.log",
		    error_file: "~/.pm2/logs/ntfy-error.log",
		    merge_logs: true,
		    time: true
		  }]
		};
		```
	- 启动
		```
		pm2 start ~/.config/ntfy/ecosystem.config.js
		```
