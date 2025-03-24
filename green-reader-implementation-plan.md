# Green Reader 实现方案

## 项目概述

将 Dark Reader 浏览器扩展改造为 Green Reader，用于为网站提供绿色护眼模式，保护用户眼睛健康。

## 可行性分析

经过对 Dark Reader 代码库的分析，改造为 Green Reader 是完全可行的。Dark Reader 的核心功能是通过修改网页的颜色来创建暗色主题，我们可以利用相同的技术架构，但修改颜色处理算法，将其转变为绿色护眼模式。

## 需要修改的主要部分

### 1. 项目名称和品牌标识

- **manifest.json**: 修改扩展名称、描述和图标路径
- **本地化文件**: 修改 `_locales` 目录下的描述文本
- **图标文件**: 替换 `icons` 目录下的图标，从深色主题改为绿色主题

### 2. 颜色处理算法

#### 核心文件：

- **generators/modify-colors.ts**: 修改颜色转换逻辑，从暗色模式转变为绿色护眼模式
- **generators/utils/matrix.ts**: 修改颜色矩阵转换，增加绿色滤镜效果
- **utils/color.ts**: 可能需要修改颜色处理函数
- **defaults.ts**: 修改默认颜色配置，使用适合护眼的绿色色调

### 3. 默认配置修改

- **defaults.ts**: 修改 `DEFAULT_COLORS` 和 `DEFAULT_THEME`，设置默认的绿色护眼参数
  - 调整亮度、对比度、灰度和棕褐色参数，以获得最佳的绿色护眼效果
  - 可能需要增加一个新的参数来控制绿色的强度

### 4. UI 界面调整

- **ui/popup/components**: 修改弹出窗口组件，突出绿色护眼特性
- **ui/controls/color-picker**: 可能需要调整颜色选择器，增加绿色预设

## 具体实现方案

### 颜色处理算法修改

在 `generators/utils/matrix.ts` 中，我们可以修改 `Matrix.invertNHue()` 方法，将其改为绿色滤镜矩阵：

```typescript
// 原暗色模式矩阵
invertNHue(): matrix5x5 {
    return [
        [0.333, -0.667, -0.667, 0, 1],
        [-0.667, 0.333, -0.667, 0, 1],
        [-0.667, -0.667, 0.333, 0, 1],
        [0, 0, 0, 1, 0],
        [0, 0, 0, 0, 1],
    ];
},

// 改为绿色护眼模式矩阵
greenEyeCare(): matrix5x5 {
    return [
        [0.8, 0.2, 0.0, 0, 0],
        [0.1, 0.9, 0.0, 0, 0],
        [0.1, 0.2, 0.7, 0, 0],
        [0, 0, 0, 1, 0],
        [0, 0, 0, 0, 1],
    ];
},
```

在 `createFilterMatrix` 函数中，将模式判断从暗色模式改为绿色护眼模式：

```typescript
export function createFilterMatrix(config: Theme): matrix5x5 {
    let m: matrix5x5 = Matrix.identity();
    if (config.sepia !== 0) {
        m = multiplyMatrices(m, Matrix.sepia(config.sepia / 100));
    }
    if (config.grayscale !== 0) {
        m = multiplyMatrices(m, Matrix.grayscale(config.grayscale / 100));
    }
    if (config.contrast !== 100) {
        m = multiplyMatrices(m, Matrix.contrast(config.contrast / 100));
    }
    if (config.brightness !== 100) {
        m = multiplyMatrices(m, Matrix.brightness(config.brightness / 100));
    }
    // 修改这里，从 invertNHue 改为 greenEyeCare
    if (config.mode === 1) {
        m = multiplyMatrices(m, Matrix.greenEyeCare());
    }
    return m;
}
```

### 默认配置修改

在 `defaults.ts` 中，修改默认颜色配置：

```typescript
export const DEFAULT_COLORS = {
    darkScheme: {
        // 修改为适合绿色护眼的背景色
        background: '#e8f5e9',
        text: '#1b5e20',
    },
    lightScheme: {
        background: '#f1f8e9',
        text: '#33691e',
    },
};

export const DEFAULT_THEME: Theme = {
    // 其他配置保持不变
    // ...
    // 调整以下参数以获得最佳的绿色护眼效果
    brightness: 100,
    contrast: 95,
    grayscale: 0,
    sepia: 15, // 增加一点棕褐色可以减少蓝光
    // ...
};
```

### 本地化文件修改

修改 `_locales/en.config` 等文件中的描述：

```
@extension_description
Green eye-care mode for every website. Take care of your eyes, use green theme for comfortable browsing.

@store_listing
This eye-care extension enables green mode by creating eye-friendly themes for websites on the fly. Green Reader adjusts colors to reduce eye strain and make them comfortable to read for long periods.

You can adjust the brightness, contrast, green filter intensity, and other settings to find the most comfortable reading experience for your eyes.

...
```

## 实施步骤

1. 修改项目名称和描述（manifest.json 和本地化文件）
2. 创建新的图标文件，替换原有图标
3. 修改颜色处理算法，实现绿色护眼效果
4. 调整默认配置参数
5. 更新 UI 界面，突出绿色护眼特性
6. 测试不同网站上的效果并优化参数

## 预期效果

Green Reader 将为用户提供舒适的绿色护眼模式，减少眼睛疲劳，特别适合长时间阅读和工作的用户。通过调整绿色的强度、亮度和对比度，用户可以找到最适合自己的护眼设置。