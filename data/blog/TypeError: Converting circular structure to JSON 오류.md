---
title: Converting circular structure to JSON 오류
date: '2023-10-25'
lastmod: '2023-10-25'
tags: ['JavaScript']
draft: false
summary: ''
---

회사에서 Resize 기능을 구현하던 중에 뒤늦게 오류를 발견했다.

```jsx
// 기존 코드
const items = [
      { value: "1080p", width: 1920, height: 1080 },
      { value: "720p", width: 1280, height: 720 },
      { value: "2x Resize", width: 2560, height: 1440 },
      { value: "4x Resize", width: 5120, height: 2880 },
      { value: "Custom" }
    ];


...

return (
    <Select defaultValue="Custom" options={items} onChange={handleChange} onClick={handleResizeClick}>
        {t("Resize")}
    </Select>
)
```
