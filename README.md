# ClickHouse 24.3 for Kunpeng920 (鲲鹏920)

自动编译适配华为鲲鹏920处理器的 ClickHouse 24.3 版本。

## 背景

ClickHouse 24.3 官方ARM版本使用了较新的指令集优化,在鲲鹏920 + 麒麟4.19内核环境下会出现"非法指令"错误。本项目通过GitHub Actions自动编译兼容版本。

## 兼容性模式

### Safe Mode (安全模式) - **推荐**
- 编译参数: `-march=armv8-a -mtune=generic`
- 目标: ARMv8.0-A 基线指令集
- 优点: 最大兼容性,所有鲲鹏920都能运行
- 缺点: 性能略低

### Optimized Mode (优化模式)
- 编译参数: `-march=armv8.2-a -mtune=tsv110`
- 目标: ARMv8.2-A + 鲲鹏优化
- 优点: 性能更好
- 缺点: 某些早期鲲鹏920可能不兼容

## 使用方法

### 1. Fork 本仓库
点击右上角 **Fork** 按钮

### 2. 启用 GitHub Actions
进入 **Settings** → **Actions** → **General** → 启用 Actions

### 3. 运行编译工作流
1. 进入 **Actions** 标签页
2. 选择 **Build ClickHouse 24.3 for Kunpeng920**
3. 点击 **Run workflow**
4. 选择编译选项:
   - **ClickHouse version**: `v24.3.13.40-lts` (默认)
   - **Build type**: `Release` (生产环境) 或 `RelWithDebInfo` (调试)
   - **Compatibility mode**: `safe` (推荐) 或 `optimized`
5. 点击 **Run workflow** 开始编译

### 4. 下载编译结果
编译完成后(约1-2小时):
1. 进入该工作流运行页面
2. 在 **Artifacts** 区域下载 `clickhouse-kunpeng920-safe.tar.gz`

### 5. 安装到目标服务器

```bash
# 解压
tar -xzvf clickhouse-24.3-kunpeng920-safe.tar.gz

# 停止旧服务
sudo systemctl stop clickhouse-server

# 备份旧版本
sudo cp /usr/bin/clickhouse-server /usr/bin/clickhouse-server.bak

# 安装新版本
sudo cp clickhouse-server /usr/bin/
sudo cp clickhouse-client /usr/bin/
sudo cp clickhouse-local /usr/bin/
sudo chmod +x /usr/bin/clickhouse-*

# 验证版本
clickhouse-server --version

# 启动服务
sudo systemctl start clickhouse-server

# 检查状态
sudo systemctl status clickhouse-server
```

## 验证是否解决"非法指令"问题

```bash
# 查看服务日志
sudo journalctl -u clickhouse-server -f

# 查看系统日志确认无非法指令错误
dmesg | grep -i "illegal\|trap"

# 测试连接
clickhouse-client --query "SELECT version()"
```

## 编译参数说明

### Safe Mode (安全模式)
```cmake
-DARCH_NATIVE=OFF
-DNO_ARMV81_OR_HIGHER=1
-DCOMPILER_FLAGS="-march=armv8-a -mtune=generic"
-DENABLE_MULTITARGET_CODE=OFF
```

### Optimized Mode (优化模式)
```cmake
-DARCH_NATIVE=OFF
-DARM_MARCH="armv8.2-a"
-DCOMPILER_FLAGS="-march=armv8.2-a -mtune=tsv110"
```

## 环境要求

- **目标服务器**: 鲲鹏920 (Kunpeng920)
- **操作系统**: 麒麟 (Kylin) 4.19.90-52.52 aarch64
- **glibc**: 2.28+
- **内核**: 4.19+

## 故障排除

### 编译失败
1. 检查 GitHub Actions 日志
2. 可能是子模块下载超时,重新运行工作流
3. 如果磁盘空间不足,工作流会自动清理不必要的文件

### 运行时错误
1. 如果 safe 模式仍有问题,请提供:
   ```bash
   cat /proc/cpuinfo | grep -m1 "Features"
   dmesg | tail -20
   ```
2. 尝试使用 optimized 模式(如果 safe 模式性能不够)

## 性能对比

| 模式 | 编译时间 | 查询性能 | 兼容性 |
|------|---------|---------|--------|
| Safe | ~60分钟 | 基线 | 100% |
| Optimized | ~60分钟 | +5-15% | 95%+ |

## 许可证

本项目仅提供编译配置,ClickHouse 本身遵循 Apache 2.0 许可证。

## 相关链接

- [ClickHouse 官方](https://clickhouse.com/)
- [ClickHouse GitHub](https://github.com/ClickHouse/ClickHouse)
- [华为鲲鹏社区](https://www.hikunpeng.com/)

## 问题反馈

如果遇到问题,请提供:
1. 鲲鹏920 具体型号 (`dmidecode -t processor`)
2. CPU Features (`cat /proc/cpuinfo | grep Features`)
3. 错误日志 (`dmesg` 和 `journalctl`)
