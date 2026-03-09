# 宝宝性别竞猜 - Firebase 多人在线版配置指南

## 📋 功能说明

✅ **多人实时在线投注** - 所有人的投注实时同步，无需刷新页面
✅ **数据永久保存** - 存储在 Firebase 云端，不会丢失
✅ **零后端部署** - 只需要配置 Firebase，无需自己搭建服务器
✅ **完全免费** - Firebase 免费套餐足够使用

---

## 🚀 快速开始（10分钟配置）

### 第一步：创建 Firebase 项目

1. 访问 [Firebase 控制台](https://console.firebase.google.com/)
2. 点击 "添加项目" 或 "Add project"
3. 输入项目名称（例如：baby-bet），点击继续
4. 关闭 Google Analytics（可选），点击 "创建项目"
5. 等待项目创建完成

### 第二步：创建 Realtime Database

1. 在 Firebase 控制台左侧菜单，点击 "Realtime Database"
2. 点击 "创建数据库" 按钮
3. 选择数据库位置：
   - 选择 **美国（us-central1）** 或 **新加坡（asia-southeast1）**（推荐中国用户）
4. 设置安全规则：
   - 暂时选择 "以**测试模式**启动"（稍后会修改规则）
5. 点击 "启用"

### 第三步：获取 Firebase 配置

1. 在 Firebase 控制台，点击左上角 ⚙️ 齿轮图标
2. 选择 "项目设置"
3. 滚动到底部，找到 "您的应用" 部分
4. 点击 **</> Web** 图标（添加 Web 应用）
5. 输入应用昵称（例如：baby-bet-web），点击 "注册应用"
6. 复制显示的配置代码，找到类似下面的配置对象：

```javascript
const firebaseConfig = {
  apiKey: "AIzaSyXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  authDomain: "your-project.firebaseapp.com",
  databaseURL: "https://your-project-default-rtdb.firebaseio.com",
  projectId: "your-project",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "123456789012",
  appId: "1:123456789012:web:xxxxxxxxxxxxx"
};
```

### 第四步：修改 HTML 文件

1. 打开 `baby-bet-online.html` 文件
2. 找到代码中的这一部分（约第 382-390 行）：

```javascript
// TODO: 替换为你的 Firebase 配置
const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
    databaseURL: "https://YOUR_PROJECT_ID-default-rtdb.firebaseio.com",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_PROJECT_ID.appspot.com",
    messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
    appId: "YOUR_APP_ID"
};
```

3. **完整替换**成你从 Firebase 复制的配置
4. 保存文件

### 第五步：设置数据库安全规则（重要！）

1. 返回 Firebase 控制台 -> Realtime Database
2. 点击 "规则" 标签页
3. 将规则替换为以下内容：

```json
{
  "rules": {
    "bets": {
      ".read": true,
      ".write": true,
      "$betId": {
        ".validate": "newData.hasChildren(['name', 'gender', 'amount', 'time', 'timestamp'])"
      }
    }
  }
}
```

4. 点击 "发布" 按钮

**安全说明：**
- 上述规则允许所有人读写 `bets` 节点（适合内部小范围使用）
- 如果需要更严格的权限控制，请参考后面的"高级配置"章节

### 第六步：测试

1. 双击打开 `baby-bet-online.html` 文件（在浏览器中打开）
2. 页面顶部应该显示 "✅ 在线同步"
3. 填写投注信息并提交
4. 打开另一个浏览器窗口（或用手机打开），应该能实时看到刚才的投注

---

## 🌐 部署到网上（可选）

### 方法1：部署到 Firebase Hosting（推荐）

```bash
# 1. 安装 Firebase CLI
npm install -g firebase-tools

# 2. 登录 Firebase
firebase login

# 3. 初始化项目
firebase init hosting

# 4. 选择你刚创建的 Firebase 项目
# 5. 设置 public 目录为当前目录
# 6. 配置为单页应用：No
# 7. 不覆盖 index.html

# 8. 部署
firebase deploy
```

部署成功后，你会获得一个类似 `https://your-project.web.app` 的网址

### 方法2：部署到 GitHub Pages

1. 将 `baby-bet-online.html` 重命名为 `index.html`
2. 上传到 GitHub 仓库
3. 在仓库设置中启用 GitHub Pages
4. 访问 `https://你的用户名.github.io/仓库名/`

---

## 🔒 高级安全配置（推荐生产环境使用）

如果你想限制只有特定的人可以操作，可以使用以下安全规则：

### 限制写入（只允许添加，不允许删除）

```json
{
  "rules": {
    "bets": {
      ".read": true,
      ".write": false,
      "$betId": {
        ".write": "!data.exists()"
      }
    }
  }
}
```

### 限制删除（需要密码）

修改 HTML 中的删除和清空功能，添加密码验证：

```javascript
window.deleteBet = async function(betId) {
    const password = prompt('请输入管理密码：');
    if (password !== '你的密码') {
        alert('密码错误！');
        return;
    }
    // ... 原有的删除代码
};

window.clearAllBets = async function() {
    const password = prompt('请输入管理密码：');
    if (password !== '你的密码') {
        alert('密码错误！');
        return;
    }
    // ... 原有的清空代码
};
```

---

## ❓ 常见问题

### 1. 页面显示 "❌ 连接失败"

**原因：** Firebase 配置错误

**解决方法：**
- 检查 `firebaseConfig` 是否正确复制
- 确保 `databaseURL` 字段存在且正确
- 打开浏览器开发者工具（F12）查看详细错误信息

### 2. 提交投注后没有反应

**原因：** 数据库规则未配置

**解决方法：**
- 检查 Firebase 控制台 -> Realtime Database -> 规则
- 确保 `.write` 权限为 `true`

### 3. 数据无法实时同步

**原因：** 网络问题或浏览器缓存

**解决方法：**
- 刷新页面（Ctrl+F5 强制刷新）
- 检查网络连接
- 清除浏览器缓存

### 4. Firebase 免费套餐够用吗？

**答案：** 完全够用！

Firebase 免费套餐（Spark Plan）包括：
- 1 GB 存储空间
- 10 GB/月 数据传输
- 100 个同时在线连接

对于一个投注游戏，即使有 100 人参与也完全没问题。

### 5. 如何备份数据？

在 Firebase 控制台 -> Realtime Database：
1. 点击右上角三个点
2. 选择 "导出 JSON"
3. 下载备份文件

---

## 📱 移动端优化

页面已经做了响应式设计，直接在手机浏览器打开即可。

为了更好的体验，可以添加到主屏幕：
- iOS：Safari -> 分享 -> 添加到主屏幕
- Android：Chrome -> 菜单 -> 添加到主屏幕

---

## 🎨 自定义修改

### 修改颜色主题

在 CSS 部分（第 15-17 行）修改背景色：

```css
background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
```

### 修改页面标题

在 HTML 部分（第 235 行）：

```html
<h1>宝宝性别竞猜</h1>
<p>猜猜是男孩还是女孩？押注赢大奖！</p>
```

---

## 📞 技术支持

如果遇到问题，可以：
1. 查看浏览器控制台（F12）的错误信息
2. 查看 [Firebase 官方文档](https://firebase.google.com/docs/database)
3. 检查 Firebase 控制台中的数据库使用情况

---

## ✅ 配置检查清单

- [ ] Firebase 项目已创建
- [ ] Realtime Database 已启用
- [ ] Firebase 配置已复制到 HTML 文件
- [ ] 数据库安全规则已设置
- [ ] 在浏览器中测试，显示 "✅ 在线同步"
- [ ] 提交测试投注成功
- [ ] 多个窗口能实时看到数据同步

全部完成后，就可以分享链接给朋友们一起玩了！🎉
