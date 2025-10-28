# 简化修复方案

## 修复内容

### 1. ✅ 2FA错误提示
**方案**: 使用 `el-alert` 在表单顶部显示红色错误提示

**改动**:
- 添加 `errorMessage` 状态
- 在表单上方显示 `<el-alert>` 组件
- 修改 `httpClient.post` 支持错误回调
- 登录失败时显示错误消息

**效果**: 2FA验证码错误时，会在登录框顶部显示红色警告框

### 2. ✅ 用户菜单显示
**方案**: 重构用户菜单结构，使用wrapper包裹

**改动**:
```vue
<div class="user-menu-wrapper">
  <el-dropdown trigger="click">
    <span class="user-dropdown-trigger">
      <el-icon><User /></el-icon>
      <span class="username">{{ userStore.username }}</span>
      <el-icon><arrow-down /></el-icon>
    </span>
  </el-dropdown>
</div>
```

**CSS**:
- 使用 `gap: 8px` 简化间距
- 固定高度 `height: 60px`
- 使用 `display: flex` 确保对齐

### 3. ✅ 手动执行功能
**方案**: 简化 `ElMessageBox` 调用

**改动**:
```javascript
ElMessageBox({
  title: '手动执行任务',
  message: `确定要手动执行任务 "${item.name}" 吗？`,
  showCancelButton: true,
  confirmButtonText: '确定执行',
  cancelButtonText: '取消',
  type: 'warning'
}).then(() => {
  taskService.run(item.id, () => {
    ElMessage.success('任务已开始执行')
  })
}).catch(() => {})
```

## 测试步骤

1. 重启前端开发服务器
2. 测试登录 - 输入错误的2FA验证码，应该看到红色警告框
3. 登录成功后，检查右上角用户名是否正确显示
4. 点击"手动执行"按钮，应该弹出确认对话框

## 关键文件

- `web/vue/src/pages/user/login.vue` - 登录页面
- `web/vue/src/components/common/navMenu.vue` - 导航菜单
- `web/vue/src/pages/task/list.vue` - 任务列表
- `web/vue/src/utils/httpClient.js` - HTTP客户端
- `web/vue/src/api/user.js` - 用户API
