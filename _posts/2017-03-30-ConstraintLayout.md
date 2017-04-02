---
layout: post
title: ConstraintLayout笔记
date: 2017-03-30
tags: android    
---

### ConstraintLayout 
ConstraintLayout靠Constraints来组织view，其内的view hierarchy只有一层。在布局的时候，通过构造以下3者的关系来入手：
1. ConstraintLayout（parent）
2. sibling view
3. guideline

### Anchor
每个view都有top, bottom, left(start),right(end)4个锚点，其中：
1. text-based views 会有 baseline anchor
2. guideline有 android:orientation="vertical/horizontal" 两种模式，vertical时有左右锚点，horizontal时有上下锚点

### Guideline
guideline是view的子类，只不过宽高为0，visibility为gone，只是为了布局提供瞄点而存在。guideline可以以**绝对值**或**百分比**的方式放置位置<br/>
**绝对值**：
```
layout_constraintGuide_begin
layout_constraintGuide_end
```
**百分比**
```
layout_constraintGuide_Percent
```

### Xml Format
```html
layout_constraint[SourceAnchor]_[TargetAnchor]="[TargetId]"

<Button
  android:id="@+id/button_cancel"
  …​ />

<Button
  android:id="@+id/button_next"
  app:layout_constraintStart_toEndOf="@+id/button_cancel"
  …​ />
```
### Bias
the bias attributes:
```
layout_constraintHorizontal_bias
layout_constraintVertical_bias
```

**app:layout_constraintVertical_bias="0.25"**Now the view is weighted with a 25/75 split of the available space on each axis.Technically, when no bias constraint is present, the bias is 0.5.

### 视图编辑器自动生成的属性
**Autoconnect**
```
tools:layout_editor_absoluteX
tools:layout_editor_absoluteY
```
保存当前状态，帮助视图编辑器绘制预览

**Inference**
```
tools:layout_constraint[Anchor]_creator
```
帮助视图编辑器区分哪些constraints是手动添加的，哪些是通过inference自动添加的

### View Measurements
* Exact
* Wrap Content
* Any Size (Measure to fill the available space for the attached constraints, Set layout_width or layout_height to 0dp)

**android:minWidth / android:minHeight** will be used by ConstraintLayout when its dimensions are set to WRAP_CONTENT.

### Ratio
```
layout_constraintDimensionRatio = "width:height"
layout_constraintDimensionRatio = "H,width:height" //H/W为需要动态调整的一方
layout_constraintDimensionRatio = "floatValue[width/height]"

<ImageView
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:src="@drawable/water"
    app:layout_constraintDimensionRatio="1:3"/>

<ImageView
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:src="@drawable/water"
    app:layout_constraintDimensionRatio="0.3"/>
```

### Visibility behavior
![](/images/android/constraint_visibility.png)<br/>
**Margins when connected to a GONE widget**<br/>
When a position constraint target's visibility is View.GONE, you can also indicates a different margin value to be used using the following attributes:
* layout_goneMarginStart
* layout_goneMarginEnd
* layout_goneMarginLeft
* layout_goneMarginTop
* layout_goneMarginRight
* layout_goneMarginBottom

### Chains
两个互相链接的view产生一个chain
![](/images/android/constraint_chain.png)<br/>
Chains are controlled by attributes set on chain head: the head is the left-most widget for horizontal chains, and the top-most widget for vertical chains.
![](/images/android/constraint_chain_head.png)<br/>

**Chain Style**<br/>
通过layout_constraintVertical_chainStyle
给chain设置不同的chain_style后，chain的行为会有不同的表现 (默认**CHAIN_SPREAD**)
![](/images/android/constraint_chain_style.png)<br/>

**Weighted chains**<br/>
chain默认平分可用空间，*layout_constraintHorizontal_weight* 和*layout_constraintVertical_weight* 以正比的关系分配剩余空间。

### 动态构造ConstraintLayout
**ConstraintSet**可以动态构造ConstraintLayout，具体参见：
[ConstraintSet Anadroid Api](https://developer.android.com/reference/android/support/constraint/ConstraintSet.html)