---
layout: single
title:  "[minimal-mistakes] page style customize"
category: Github-page
tag: Github-page
---

# [minimal-mistakes] page style customize

> minimal_mistakes를 사용해서 github blog를 운영하고 있다.
<br>다 좋은데.. 몇가지 나의 심기를 건드는 디자인들이 있어 이를 직접 Customize를 해보자. 😂

---

### inline 코드 색상 변경
1. inline 코드 블럭 색상
   - `_sass/minimal-mistakes/_variables.scss`에서 `$code-background-color` 색상 조절
2. inline 코드 내부 색상 변경
    - `_sass/minimal-mistakes/_base.scss` 에서 Line 163의 css selector에 color 속성 추가
    ```css
        p > code,
        a > code,
        li > code,
        figcaption > code,
        td > code {
          padding-top: 0.1rem;
          padding-bottom: 0.1rem;
          font-size: 0.8em;
          background: $code-background-color;
          border-radius: $border-radius;
          **color: #eb6060;**
          &:before,
          &:after {
            letter-spacing: -0.2em;
            content: "\00a0"; /* non-breaking space*/
          }
        }
    ```