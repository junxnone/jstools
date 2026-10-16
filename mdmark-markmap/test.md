# Markmap Blocks Demo

This file mixes regular prose, an unrelated code block, and two
` ```markmap ` blocks. Only the `markmap` blocks should be rendered as
mindmaps below.

```js
// not a markmap block - should be ignored
console.log("hello");
```

## First markmap: Skills

```markmap
---
title: Skills
markmap:
  spacingHorizontal: 40
  spacingVertical: 10
  initialExpandLevel: 2
---

- ML/DL
  - Tools
    - Scikit-Learn
    - Pytorch
    - Tensorflow
  - Tasks
    - Classification
    - Detection
    - Segmentation
- Linux
  - Ubuntu
  - Docker
- Python
  - OpenCV
  - Numpy
```

Some text between the two markmap blocks.

```markmap
- 3D
  - PCL
    - Filter
    - ICP
    - Measurement
  - Visualization
- Hardware
- CMake
```
