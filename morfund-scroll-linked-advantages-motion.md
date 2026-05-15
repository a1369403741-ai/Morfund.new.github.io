# Morfund 六大核心优势｜滚动联动动效实现说明

> 用途：给 Codex 落实「六大核心优势」板块的滚动联动动效。  
> 目标：保持当前页面设计稿不变，只实现右侧卡片在遮罩容器内纵向滚动、左侧标题同步高亮的交互效果。

---

## 1. 交互目标

请实现「六大核心优势」板块的滚动联动动效，要求基于当前页面设计稿进行开发，不要重新设计版式。

左侧为 6 个优势标题，右侧为对应的内容卡片。页面滚动到该模块时，模块进入 sticky 状态；用户继续向下滚动时，右侧卡片在一个固定高度的遮罩容器中纵向滚动切换，左侧标题同步高亮。

用户点击左侧标题时，也需要平滑滚动到对应卡片位置。

---

## 2. 内容顺序

```text
1. 额度高
2. 币种灵活
3. 平台官方
4. 申请方便
5. 还款灵活
6. 入账便捷
```

---

## 3. 布局要求

1. 整个模块保持左右结构：
   - 左侧：优势标题列表
   - 右侧：卡片展示区

2. 左侧标题列表和右侧卡片区域需要整体垂直居中。

3. 右侧不要做原地替换，不要直接切换 `display` / `opacity`。

4. 右侧所有卡片需要上下排列在同一个 `track` 容器中。

5. `track` 外层需要一个固定高度的 `mask / overflow hidden` 容器。

6. 当前卡片始终在 `mask` 容器中垂直居中显示。

7. `mask` 容器高度比单张卡片高度大约 30%，让上一个 / 下一个卡片有更自然的进入和离开空间。

8. 去掉额外说明文字、调试框、绿色边框、进度说明区域。

---

## 4. 动效要求

1. 动效完全由鼠标滚动进度驱动，不使用弹性缓动。

2. 使用纯线性过渡，不要 `spring`、`bounce`、`easeOutBack`。

3. 向下滚动时：
   - `card track` 整体向上移动
   - 下一个卡片从下方进入 `mask` 容器
   - 上一个卡片从上方离开 `mask` 容器

4. 向上回退时：
   - `card track` 整体向下移动
   - 上一个卡片从上方回到中心
   - 当前卡片向下离开

5. 卡片 `opacity` 需要跟随滚动幅度连续变化：
   - 当前居中卡片 `opacity = 1`
   - 相邻进入 / 离开的卡片 `opacity` 根据距离从 `0 → 1` 或 `1 → 0`
   - 不允许等滚动结束后才突然切换透明度

6. 左侧标题高亮也跟随当前居中卡片变化：
   - 当前标题文字为白色
   - 未选中标题降低透明度
   - 当前标题可展开一行说明文字

7. 点击左侧标题时，页面需要平滑滚动到对应卡片的 `scroll progress`，而不是直接强制替换内容。

---

## 5. 实现方式建议

1. 设置 `sticky-section` 高度为内容数量对应的滚动高度，例如 6 个卡片可设置为 `600vh` 左右。

2. `sticky-inner` 使用：

```css
position: sticky;
top: 0;
height: 100vh;
```

3. 通过 `window scroll` 计算当前模块滚动进度：

```js
progress = passed / totalScrollable
```

4. 将 `progress` 映射为精确索引：

```js
exactIndex = progress * (items.length - 1)
```

5. 根据 `exactIndex` 移动右侧 `card track`：

```js
translateY = centerOffset - exactIndex * step
```

6. 计算基础参数：

```js
step = cardHeight + cardGap
centerOffset = (maskHeight - cardHeight) / 2
```

7. 每张卡片 opacity 计算方式：

```js
distance = Math.abs(index - exactIndex)
opacity = clamp(1 - distance / 1.3, 0, 1)
```

其中 `1.3` 用来扩大过渡范围，让透明度变化更自然。

8. 当前高亮标题：

```js
activeIndex = Math.round(exactIndex)
```

---

## 6. 核心伪代码

```js
const items = [...六个优势内容];

const progress = passed / totalScrollable;
const exactIndex = progress * (items.length - 1);
const activeIndex = Math.round(exactIndex);

const step = cardHeight + cardGap;
const centerOffset = (maskHeight - cardHeight) / 2;

cardTrack.style.transform =
  `translateY(${centerOffset - exactIndex * step}px)`;

cards.forEach((card, index) => {
  const distance = Math.abs(index - exactIndex);
  const opacity = clamp(1 - distance / 1.3, 0, 1);
  const scale = 0.982 + opacity * 0.018;

  card.style.opacity = opacity;
  card.style.transform = `scale(${scale})`;
});

tabs.forEach((tab, index) => {
  tab.classList.toggle("active", index === activeIndex);
});
```

---

## 7. 点击左侧标题逻辑

```js
tab.addEventListener("click", () => {
  const targetY =
    sectionTop + totalScrollable * (index / (items.length - 1));

  window.scrollTo({
    top: targetY,
    behavior: "smooth"
  });
});
```

---

## 8. CSS 结构建议

```css
.sticky-section {
  height: 620vh;
  position: relative;
}

.sticky-inner {
  position: sticky;
  top: 0;
  min-height: 100vh;
  display: flex;
  align-items: center;
}

.content-grid {
  display: grid;
  grid-template-columns: 380px minmax(0, 1fr);
  gap: 150px;
  align-items: center;
}

.mask-shell {
  position: relative;
  height: 455px;
  overflow: hidden;
  border-radius: 20px;
  mask-image: linear-gradient(
    180deg,
    transparent 0%,
    rgba(0,0,0,.12) 5%,
    rgba(0,0,0,.55) 13%,
    #000 24%,
    #000 76%,
    rgba(0,0,0,.55) 87%,
    rgba(0,0,0,.12) 95%,
    transparent 100%
  );
}

.card-track {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  gap: 34px;
  will-change: transform;
}

.card {
  flex: 0 0 350px;
  min-height: 350px;
  will-change: opacity, transform;
}
```

---

## 9. 注意事项

1. 不要使用 `display: none` 切换卡片。
2. 不要使用 React state 直接替换右侧卡片内容。
3. 不要使用弹性缓动。
4. 不要保留调试文字、绿色边框或说明容器。
5. 保持当前页面视觉风格，只实现滚动联动动效。
6. 移动端可以降级为横向 Tab + 单卡片展示，避免 sticky 长滚动影响体验。

---

## 10. 给 Codex 的完整启动命令

```text
请实现「六大核心优势」板块的滚动联动动效，要求基于当前页面设计稿进行开发，不要重新设计版式。

交互目标：
左侧为 6 个优势标题，右侧为对应的内容卡片。页面滚动到该模块时，模块进入 sticky 状态；用户继续向下滚动时，右侧卡片在一个固定高度的遮罩容器中纵向滚动切换，左侧标题同步高亮。用户点击左侧标题时，也需要平滑滚动到对应卡片位置。

内容顺序：
1. 额度高
2. 币种灵活
3. 平台官方
4. 申请方便
5. 还款灵活
6. 入账便捷

布局要求：
1. 整个模块保持左右结构：左侧优势标题列表，右侧卡片展示区。
2. 左侧标题列表和右侧卡片区域需要整体垂直居中。
3. 右侧不要做原地替换，不要直接切换 display / opacity。
4. 右侧所有卡片需要上下排列在同一个 track 容器中。
5. track 外层需要一个固定高度的 mask / overflow hidden 容器。
6. 当前卡片始终在 mask 容器中垂直居中显示。
7. mask 容器高度比单张卡片高度大约 30%，让上一个 / 下一个卡片有更自然的进入和离开空间。
8. 去掉额外说明文字、调试框、绿色边框、进度说明区域。

动效要求：
1. 动效完全由鼠标滚动进度驱动，不使用弹性缓动。
2. 使用纯线性过渡，不要 spring、bounce、easeOutBack。
3. 向下滚动时：card track 整体向上移动，下一个卡片从下方进入 mask 容器，上一个卡片从上方离开 mask 容器。
4. 向上回退时：card track 整体向下移动，上一个卡片从上方回到中心，当前卡片向下离开。
5. 卡片 opacity 需要跟随滚动幅度连续变化：当前居中卡片 opacity = 1，相邻进入 / 离开的卡片 opacity 根据距离从 0 → 1 或 1 → 0。
6. 左侧标题高亮也跟随当前居中卡片变化：当前标题文字为白色，未选中标题降低透明度，当前标题可展开一行说明文字。
7. 点击左侧标题时，页面需要平滑滚动到对应卡片的 scroll progress，而不是直接强制替换内容。

实现方式建议：
1. 设置 sticky-section 高度为内容数量对应的滚动高度，例如 6 个卡片可设置为 600vh 左右。
2. sticky-inner 使用 position: sticky; top: 0; height: 100vh。
3. 通过 window scroll 计算当前模块滚动进度：progress = passed / totalScrollable。
4. 将 progress 映射为精确索引：exactIndex = progress * (items.length - 1)。
5. 根据 exactIndex 移动右侧 card track：translateY = centerOffset - exactIndex * step。
6. step = cardHeight + cardGap。
7. centerOffset = (maskHeight - cardHeight) / 2。
8. 每张卡片 opacity 计算方式：distance = Math.abs(index - exactIndex)，opacity = clamp(1 - distance / 1.3, 0, 1)。
9. 当前高亮标题：activeIndex = Math.round(exactIndex)。

注意：
不要使用 display none 切换卡片；不要使用 React state 直接替换右侧卡片内容；不要使用弹性缓动；不要保留调试文字、绿色边框或说明容器；保持当前页面视觉风格，只实现滚动联动动效。
```
