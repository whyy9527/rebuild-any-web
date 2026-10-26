---
name: webgl-game-control-acceptance
description: |
  验收 Canvas/WebGL 游戏与 DOM 控件的一致性。用于桌面模型看得见却点不到、前后台错位、拖动幽灵或输入锁残留、多色标识被遮挡，以及游戏 E2E 通过后用户仍指出这些问题。补充游戏开发流程，不评价关卡是否好玩。
---

# WebGL 游戏视觉与输入验收

## 触发与目标

目标是让可见物体、实际命中区域和手势生命周期一致。只验证 JSON 坐标相等、按钮中心可点或正常松手，是会漏掉用户问题的近似结果。

遵循仓库根 AGENTS.md 的无头运行要求。先读消费项目的当前维护入口和用户反馈；本技能补充已有开发/复刻流程，不接管 Goal 或新增任务状态。

## 可执行检查

### 可见模型与触点

- 用用户实际窗口的 CSS 视口和 DPR 复现；截图物理像素不等于 CSS 像素。手机横屏不能替代桌面窗口。覆盖实际解锁容量和窗口缩放往返，不把未解锁格子强行显示。
- 相机拟合、模型缩放完成后再投影命中区域与地面格子；使用模型角点的屏幕包围框作为起点。若相机变化影响队列间距，重新计算模型位置和 DOM 位置，不能只移动其中一层。
- 实际点击模型上部、主体和边缘，并验证选中的对象 ID。检查相邻控件、换色台和视口的完整矩形关系；中心命中不能证明整块模型可点。
- 同状态截图与真实操作共同判断对齐，不以布局函数自己的输出证明绘制正确。

### 中断的拖动

只在源按钮处理 pointerup，丢失 pointer capture 后会残留拖影、选择锁或被拖动抑制的视觉更新。

- 按 pointerId 跟踪主手势。完成/取消在 window 捕获阶段接收，并处理源控件 lostpointercapture；副触点不能替换主手势。
- 按物体当前 ID 查找操作对象，避免沿用生产、耗尽或重排前的数组下标。
- 取消时先清空手势所有权，再释放 capture，清理幽灵/高亮/预览及暂存的输入锁，并同步被推迟的模型更新。正常松手只完成一次。
- 分别检查原生 releasePointerCapture 后松手、鼠标移回时 buttons 不含主键、触摸取消、失焦、缩放、源物体耗尽和结算。失焦/暂停策略遵循当前游戏规则，不用固定超时取消仍有效的手势。
- 每次中断后检查没有误扣资源或误部署，再做一次正常操作证明可继续。显式 dispatch 的失焦事件只是生命周期 fixture，不冒充真实 OS 切换。

### 颜色与代价反馈

- “材质含有第二色”不证明玩家能辨认。对比同状态旧/新页面截图，检查实际可见面积、遮挡及小屏表现；完整外壳分色比被主体遮住的细条更适合表达多色能力，但不是所有游戏都必须采用此造型。
- 需要像素检查时从页面真实截图读取颜色面积。阈值和颜色范围按案例制定，验证旧问题会失败；不要把方块杀手的 8% 或特定色相范围推广成通用审美标准。
- 展示当前资源代价时，检查目标在继续行动后的预览更新。执行后的历史结果提示不能被新的实时预览覆盖。
- 等待原生 API 的实际后置状态，例如 document.fullscreenElement；locator.click 完成不等于异步全屏已经进入。

## 复用现有案例

独立私有仓库的[方块杀手工具](https://github.com/xinxiayiyan/block-killer/tree/main/tools)提供可执行样例；从该仓库根运行：

```sh
node tools/desktop-check.mjs private/qa/desktop-new <candidate-url>
node tools/drag-check.mjs private/qa/drag-new <candidate-url>
node tools/multicolor-check.mjs private/qa/color-new <candidate-url>
```

这些驱动只可直接验收方块杀手。其它游戏需要适配对象、输入和断言，不能沿用选择器后自报通过。按案例 README 准备依赖与构建；容量/解锁 fixture 与实际赚取成长分别记录。

已验证来源：[桌面触点](https://github.com/xinxiayiyan/block-killer/blob/main/evidence/desktop-controls.json)、[拖动恢复](https://github.com/xinxiayiyan/block-killer/blob/main/evidence/drag-recovery.json)、[多色与点击](https://github.com/xinxiayiyan/block-killer/blob/main/evidence/multicolor-click.json)。旧触点、丢失捕获和细条辨识的失败回执与修复复验均保留。此技能的格式/引用可校验；自动发现命中率和跨案例收益尚未验证。

## 禁止的捷径

- 不因截图黑屏或缺少依赖就自动弹出有头浏览器；先按仓库规则检查真实页面渲染与运行依赖。
- 不覆盖用户存档来准备 QA，不把解锁 fixture 冒充赚点流程，不削弱胜负/扣费断言来掩盖失败。
- 不为了修界面命中而顺带改核心规则，不把脚本通关、颜色面积或评审赞同当作真人趣味性与审美认可。
- 案例反馈和游戏参数留在案例；不要把一次截图要求推广为全局用户偏好。
