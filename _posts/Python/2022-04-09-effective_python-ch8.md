---
layout: single
title:  "[파이썬 코딩의 기술 : Effective PYTHON 2ND 요약 및 코드 정리] CHAPTER 8. 강건성과 성능"
category: Python
tag: Python
---

## 65. try/except/else/finally의 각 블록을 잘 활용하라

### finally 블록
- try에 들어가서 except를 하더라도 무조건 실행되는 block.
- 주로 파일 핸들을 안전하게 닫기 위해 사용되는 문법이다.

### else 블록
- try에서 예외를 발생시키지 않으면 실행되는 블록
- 아래 예제와 같이 코드를 분리하여 가독성을 높이고자 할때 사용되는 방식이다.

```python
import json

def load_json_key(data, key):
    try:
        print('* JSON 데이터 읽기')
        result_dict = json.loads(data)
    except ValueError as e:
        print('* ValudError 처리')
        raise KeyError(key) from e
    else:
        print('* 키 검색')
        return result_dict[key]
```

