我们的项目采用的是一套**基于角色（RBAC）的动态权限控制方案** 。核心流程是：在用户登录后，通过 **Zustand** 在全局状态中维护当前用户的角色和权限树 ；在路由层面，我们利用 **React.lazy** 实现懒加载 ，并使用**高阶组件（HOC）**封装了一个 `AuthWrapper` 拦截器，通过比对路由元数据中的 `roles` 字段与当前用户角色，实现页面级的访问控制 ；对于未授权的跳转，系统会统一重定向至 403 页面并记录访问日志，确保了系统整体的安全性和扩展性



1. **超级管理员 (Admin)**：拥有所有权限，负责配置流程模版、管理用户账号。
    
2. **部门经理 (Manager)**：拥有审批权限，可以查看本部门的数据看板。
    
3. **普通员工 (User)**：只能发起申请（如请假、采购），查看自己的申请进度。
    
4. **财务/人事专员 (Auditor)**：特定流程节点的审批人员。

通过比对路由元数据中的 `roles` 字段与当前用户角色，实现页面级的访问控制 代码实现：


这里就是说我们每个路由 有一个roles数组 保存哪些角色可以 访问该页面 如果用户角色不匹配 不在roles中 那么就重定向到403页面
```js
// 第 43-49 行：工作流设计器路由，roles=['admin']
<Route
  path="workflow"
  element={
    <AuthWrapper roles={['admin']}>
      <WorkflowDesigner />
    </AuthWrapper>
  }
/>

// 第 51-57 行：审批列表路由，roles=['admin', 'manager', 'auditor', 'user']
<Route
  path="approval"
  element={
    <AuthWrapper roles={['admin', 'manager', 'auditor', 'user']}>
      <ApprovalList />
    </AuthWrapper>
  }
/>

// 第 59-65 行：系统管理路由，roles=['admin']
<Route
  path="admin"
  element={
    <AuthWrapper roles={['admin']}>
      <AdminPanel />
    </AuthWrapper>
  }
/>
```