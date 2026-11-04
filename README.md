# zk-guard

知课客户端的远端配置文件，只放两个 JSON 和一个控制台页面，不含任何用户数据。

| 文件 | 作用 |
| --- | --- |
| `guard.json` | 全局可用状态：`enabled` / `minBuild` / `updated` |
| `hot.json` | 界面热更新补丁：`v` / `css` / `js` / `flags` |
| `index.html` | 控制台（GitHub Pages），写入令牌只保存在本机浏览器 |

控制台：https://pwhbut.github.io/zk-guard/

客户端每 3 分钟读一次 `guard.json`，按顺序尝试 jsDelivr → GitHub API → raw。
三个源都取不到时沿用上一次结果（从未成功过则按可用处理）。
