### 引言

在现代前端开发中，用户交互体验的提升一直是开发者关注的重点。想象一下，当用户在网页上选中文本时，不仅可以复制，还能直接保存为图片，或者一键生成 `Markdown` 格式的文本。这一切都可以通过 `HTML2Canvas` 和 `Turndown` 两个强大的 `JavaScript` 库轻松实现。本篇博客将向你展示如何在前端实现这些功能，提升用户的互动体验。

### 功能点

-  **复制为图片**

  用户可以通过选择网页上的文本内容，然后点击弹出的选项，将其转换为图片并复制到剪贴板。这项功能使用了`HTML2Canvas`库来实现，它可以将网页元素渲染为`Canvas`对象，并进一步转换为图片。

-  **保存为图片**

  除了复制，用户还可以将生成的图片保存到本地。通过`HTML2Canvas`的`toDataURL`方法，我们可以生成一个`Base64`格式的图片，并使用浏览器的下载功能将其保存。

- **复制为`Markdown`**

  对于需要纯文本的用户来说，将网页内容转换为`Markdown`格式非常实用。我们使用`Turndown`库来实现这一功能，它能够将`HTML`格式的内容快速转换为`Markdown`文本，并直接复制到剪贴板。

### 使用技术

#### `html2canvas`

[`html2canvas`](https://html2canvas.hertzen.com/) 是一个强大的工具，可以将网页元素渲染为图片。通过它，我们可以轻松地将选中的网页内容转换为 `PNG` 图片，并保存或复制到剪贴板。

##### 配置项

| **属性名**   | **默认值**                | **描述**                                 |
| ------------ | ------------------------- | ---------------------------------------- |
| `useCORS`    | `false`                   | 是否尝试使用 `CORS` 从服务器加载图片     |
| `allowTaint` | `false`                   | 是否允许不同源的图片污染画布             |
| `scale`      | `window.devicePixelRatio` | 用于渲染的比例，默认为浏览器设备像素比率 |

#### `Turndown`

[`Turndown`](https://github.com/mixmark-io/turndown) 是一个用于将 `HTML` 转换为 `Markdown` 的 `JavaScript` 库。它通常用于将富文本内容从网页或其他 `HTML` 格式转换为纯文本 `Markdown` 格式，以便在不同平台上显示或存储。

#### `Window.getComputedStyle()`

[`Window.getComputedStyle()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/getComputedStyle)方法返回一个对象，该对象在应用活动样式表并解析这些值可能包含的任何基本计算后报告元素的所有 CSS 属性的值。私有的 CSS 属性值可以通过对象提供的 API 或通过简单地使用 CSS 属性名称进行索引来访问。

#### `Window.getSelection`

返回一个 [`Selection`](https://developer.mozilla.org/zh-CN/docs/Web/API/Selection) 对象，表示用户选择的文本范围或光标的当前位置。

#### `Selection`

`Selection` 对象表示用户选择的文本范围或插入符号的当前位置。它代表页面中的文本选区，可能横跨多个元素。文本选区由用户拖拽鼠标经过文字而产生。

`Selection` 对象所对应的是用户所选择的 [`ranges`](https://developer.mozilla.org/zh-CN/docs/Web/API/Range)（区域），俗称拖蓝。默认情况下，该函数只针对一个区域。

> “拖蓝”是中文网络用语，指用户用鼠标选择文本时，文本背景变成蓝色的现象。大多数操作系统和浏览器在用户选择文本时，会将选中的文本背景变为蓝色，以表示该区域被选中。这个过程被称为“拖蓝”或“拖选”，通常用来描述用户通过鼠标选择文本的行为，例如在网页上选择文字后进行复制或高亮操作。

##### 属性

- `isCollapsed`

  返回一个布尔值，用于判断选区的起始点和终点是否在同一个位置。

- `rangeCount`

  返回该选区所包含的连续范围的数量。

##### 方法

- `getRangeAt`

  返回一个包含当前选区内容的区域对象。

  > 在代码中，`const range = selection.getRangeAt(0)` 中的 `0` 是用来获取当前用户选择的文本范围 (`range`) 列表中的第一个范围对象。通常情况下，当用户在网页上选择文本时，只会创建一个 `Range` 对象。因此，使用 `getRangeAt(0)` 来获取这个范围对象是最常见和安全的做法。
  >
  > 在绝大多数的浏览器中，用户只能一次选择一个连续的文本范围（即一个 `Range` 对象）。因此，`getRangeAt(0)` 可以确保我们获取到当前的选区。如果用户有多个选区，`getRangeAt(0)` 仅获取第一个。
  >
  > 通常情况下，不会使用 `addRange` 添加多个 `Range` 对象到选区，除非你正在编写一个允许非连续选择的自定义逻辑。

#### `Range`

**`Range`** 接口表示一个包含节点与文本节点的一部分的文档片段。

##### 方法

- `getBoundingClientRect`

  返回一个 [`DOMRect`](https://developer.mozilla.org/zh-CN/docs/Web/API/DOMRect) 对象，其绑定了 `Range` 的整个内容。

- `cloneContents`

  返回一个复制 `Range` 中所有节点的[`文档片段`](https://developer.mozilla.org/zh-CN/docs/Web/API/DocumentFragment)。

### 核心实现

在实现过程中，首先需要监听用户的文本选择操作。当用户完成选择时，展示一个操作按钮，提供“复制为图片”、“保存为图片”以及“复制为 `Markdown`”的选项。以下是核心代码段：

```javascript
// 方法：将选中内容渲染为图片并复制到剪贴板
const copyAsImage = async () => {
  if (hiddenContent.value) {
    hiddenContent.value.style.display = 'block'; // 显示隐藏内容
    try {
      const canvas = await HTML2CanvasService(hiddenContent.value, { useCORS: true });
      const blob = await new Promise(resolve => canvas.toBlob(resolve, 'image/png'));
      if (blob) {
        navigator.clipboard.write([new ClipboardItem({ 'image/png': blob })]);
        ElMessage.success('图片已复制到剪贴板');
      }
    } catch (error) {
      ElMessage.error('转换为图片时出错');
    } finally {
      hiddenContent.value.style.display = 'none'; // 重新隐藏内容
    }
  }
};
```

这一段代码展示了如何使用 `HTML2Canvas` 将选中的网页内容渲染为图片并复制到剪贴板。用户可以轻松地将网页上的任意选中内容保存为图片并分享。

#### 保存图片到本地

通过稍微调整，我们还可以将选中的内容保存为本地 `PNG` 文件。这对于需要将内容进行本地存档的用户来说非常有用。

```javascript
// 方法：将选中内容保存为图片
const saveAsImage = async () => {
  if (hiddenContent.value) {
    hiddenContent.value.style.display = 'block';
    try {
      const canvas = await HTML2CanvasService(hiddenContent.value, { useCORS: true });
      const image = canvas.toDataURL('image/png');
      const link = document.createElement('a');
      link.href = image;
      link.download = `screenshot_${Date.now()}.png`;
      link.click();
      ElMessage.success('图片已保存');
    } catch (error) {
      ElMessage.error('保存图片时出错');
    } finally {
      hiddenContent.value.style.display = 'none';
    }
  }
};
```

在这段代码中，我们使用了 `canvas.toDataURL` 方法将 `Canvas` 对象转换为 `Base64` 编码的图片数据，然后通过动态创建一个 `<a>` 标签实现文件下载。

#### `Markdown` 转换

我们可以通过以下代码实现选中文本的 `Markdown` 转换，并复制到剪贴板：

```javascript
// 方法：将选中内容转换为 Markdown
const copyAsMarkdown = () => {
  if (hiddenContent.value) {
    const turnDownService = new TurnDownService();
    const markdownContent = turnDownService.turndown(hiddenContent.value.innerHTML);
    navigator.clipboard.writeText(markdownContent).then(() => {
      ElMessage.success('Markdown 已复制到剪贴板');
    });
  }
};
```

这段代码展示了如何使用 `Turndown` 将 HTML 内容转换为 `Markdown` 格式，并通过 `navigator.clipboard.writeText` 将结果复制到剪贴板中，用户可以方便地将其粘贴到任何 `Markdown` 编辑器中。

### 项目后续迭代

为了提升该工具的实用性，我计划进行以下几个方面的迭代：

- **`Tampermonkey[油猴]`插件：** 正在开发中，目标是将该功能集成到`Tampermonkey`脚本中，方便用户在各种网页中使用。
- **谷歌浏览器插件：** 也在进行中，最终希望通过浏览器扩展的形式，让用户能更轻松地访问和使用该工具。

### 总结

通过这篇博客，我们了解了如何使用`HTML2Canvas`和`Turndown`实现将网页内容转换为图片和`Markdown`格式的功能。该工具不仅提升了用户体验，也为开发者提供了一个轻量级的解决方案。如果你对这类功能有需求，欢迎尝试并进一步改进。

------

希望这个博客内容能够满足你的需求！

完整代码如下：

```vue
<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue';
import { ElMessage } from 'element-plus';
import HTML2CanvasService from 'html2canvas';
import TurnDownService from 'turndown';

const popoverOperate = ref(false);
const selectionPopover = ref<HTMLElement | null>(null);
const hiddenContent = ref<HTMLElement | null>(null);

/* 方法一：复制为图片 */
const copyAsImage = async () => {
  if (hiddenContent.value) {
    // 临时显示隐藏内容以便 html2canvas 渲染
    hiddenContent.value.style.display = 'block';
    try {
      const canvas = await HTML2CanvasService(hiddenContent.value, {
        useCORS: true,
        allowTaint: false
      });
      const blob = await new Promise<Blob | null>((resolve, reject) => canvas.toBlob(resolve, 'image/png'));
      if (blob) {
        navigator.clipboard.write([new ClipboardItem({ 'image/png': blob })]).then(() => {
          ElMessage({
            message: '图片已复制到剪贴板',
            type: 'success',
            grouping: true,
          });
        }).catch((error) => {
          ElMessage({
            message: '复制到剪贴板失败',
            type: 'error',
            grouping: true
          });
        });
      } else {
        ElMessage({
          message: '无法将 Canvas 转换为 Blob',
          type: 'error',
          grouping: true
        });
      }
      hiddenContent.value.style.display = 'none';
      popoverOperate.value = false;
    } catch (error) {
      ElMessage({
        message: '转换为图片时出错',
        type: 'error',
        grouping: true
      });
      hiddenContent.value.style.display = 'none';
    }
  }
};
/* 方法二：保存为图片 */
const saveAsImage = async () => {
  if (hiddenContent.value) {
    // 临时显示隐藏内容以便 html2canvas 渲染
    hiddenContent.value.style.display = 'block';
    // 等待所有图片加载完成
    const images = hiddenContent.value.getElementsByTagName('img');
    const promises = Array.from(images).map(img => {
      return new Promise((resolve, reject) => {
        if (img.complete) {
          resolve(img);
        } else {
          img.onload = () => resolve(img);
          img.onerror = reject;
        }
      });
    });
    try {
      await Promise.all(promises);
      const canvas = await HTML2CanvasService(hiddenContent.value, {
        useCORS: true,
        allowTaint: false
      });
      const image = canvas.toDataURL('image/png');
      const link = document.createElement('a');
      const timestamp = Date.now();
      const randomHash = Math.random().toString(36).substring(2, 8);
      const fileName = `image_${timestamp}_${randomHash}.png`;
      link.href = image;
      link.download = fileName;
      link.click();
      hiddenContent.value.style.display = 'none';
      popoverOperate.value = false;
      ElMessage({
        message: '图片已保存',
        type: 'success',
        grouping: true,
      });
    } catch (error) {
      ElMessage({
        message: '转换为图片时出错',
        type: 'error',
        grouping: true
      });
      hiddenContent.value.style.display = 'none';
    }
  }
}
/* 方法三：复制为Markdown */
const markdownContent = ref('');
const copyAsMarkdown = () => {
  if (hiddenContent.value) {
    const turnDownService = new TurnDownService();
    markdownContent.value = turnDownService.turndown(hiddenContent.value.innerHTML);
    navigator.clipboard.writeText(markdownContent.value).then(() => {
      ElMessage({
        message: 'Markdown 已复制到剪贴板',
        type: 'success',
        grouping: true,
      });
    });
    popoverOperate.value = false;
  }
};

// 获取文本节点外层的样式
const getParentStyles = (node: Node | null): Record<string, string> => {
  if (node && node.nodeType === Node.ELEMENT_NODE) {
    const element = node as HTMLElement;
    const computedStyle = window.getComputedStyle(element);
    console.log('computedStyle', computedStyle.display)
    return {
      color: computedStyle.color,
      fontSize: computedStyle.fontSize,
      fontWeight: computedStyle.fontWeight,
      fontFamily: computedStyle.fontFamily,
      backgroundColor: computedStyle.backgroundColor,
      padding: computedStyle.padding,
      margin: computedStyle.margin,
      border: computedStyle.border,
      textAlign: computedStyle.textAlign,
      width: computedStyle.width,
      // lineHeight: computedStyle.lineHeight
      // 你可以添加其他需要的样式属性
    };
  }
  return {};
};

const handleSelection = () => {
  const selection = window.getSelection();
  if (selection && selection.rangeCount > 0 && !selection.isCollapsed) {
    const range = selection.getRangeAt(0);
    const rect = range.getBoundingClientRect();
    const fragment = range.cloneContents();
    let commonAncestorContainer: any = range.commonAncestorContainer;
    if (commonAncestorContainer.nodeType === Node.TEXT_NODE) {
      commonAncestorContainer = commonAncestorContainer.parentElement;
    }
    const parentElement = range.startContainer.parentElement;
    const parentStyles = getParentStyles(parentElement);
    const container = document.createElement('div');
    // 复制选区到隐藏容器中
    if (hiddenContent.value) {
      hiddenContent.value.innerHTML = '';
      hiddenContent.value.appendChild(fragment);
      Object.assign(hiddenContent.value.style, parentStyles);
    }
    // 设置 popover 位置
    if (selectionPopover.value) {
      selectionPopover.value.style.position = 'absolute';
      selectionPopover.value.style.left = `${rect.right}px`;
      selectionPopover.value.style.top = `${rect.bottom + window.scrollY}px`;
      popoverOperate.value = true;
    }
  } else {
    popoverOperate.value = false;
  }
};

onMounted(() => {
  document.addEventListener('mouseup', handleSelection);
});

onBeforeUnmount(() => {
  document.removeEventListener('mouseup', handleSelection);
});
</script>

<template>
  <div class="container">
    <!-- 页面内容 -->
    <p>请选择这段文字并尝试复制为图片。</p>
    <p style="font-size: 35px;color: blueviolet">你可以选择任意文本，然后点击弹出的按钮将其转换为图片。</p>
    <el-image src="https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg" />
    <p>你可以选择任意文本，然后点击弹出的按钮将其转换为图片。</p>

    <el-popover placement="right" v-model:visible="popoverOperate">
      <div class="operate" @click="copyAsImage">复制为图片</div>
      <div class="operate" @click="saveAsImage">保存为图片</div>
      <div class="operate" @click="copyAsMarkdown">复制为Markdown</div>
      <template #reference>
        <span ref="selectionPopover"></span>
      </template>
    </el-popover>
    <!-- 隐藏的用于复制的容器 -->
    <div ref="hiddenContent" style="display:none;"></div>
  </div>
</template>

<style lang="scss" scoped>
.container {
  width: 500px;
  margin: 0 auto;
  padding: 20px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.12), 0 0 6px rgba(0, 0, 0, 0.04);
}
.content {
  padding: 20px;
  background-color: #f5f5f5;
  border: 1px solid #dcdcdc;
}
.operate {
  padding: 5px 0;
  cursor: pointer;
  font-size: 14px;
  &:hover {
    color: #409eff;
  }
}
</style>
```
