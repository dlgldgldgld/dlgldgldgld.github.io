---
layout: single
title:  "class 변수, class method, static method?"
category: Python
tag: Python
---

# class 변수, class method
- **class 변수**
  - class 내부의 가장 첫번째 indent에서 선언할 수 있다.
  - 객체를 생성하지 않고도 Class의 name을 통해서만 접근 가능.

- **class method**
  - `@classmethod` decorator 추가를 통해 선언이 가능하다.
  - 해당 구문이 있든 없든 객체를 생성하지 않고도 사용은 가능하나 결과값이 다르다. 아래 소스를 기준으로 @classmethod가 있을때와 없을때 차이가 존재한다.
    - @classmethod를 추가했을 경우 : first 객체가 print_pages의 cls로 넘어가지 않는다.
        ```
        Python Pratice book
        first page
        second page
        third page
        ```
    - @classmethod를 추가하지 않았을 경우 : first 객체가 print_pages의 cls 변수로 넘어가게 된다.
        ```
        Python Pratice book
        second page
        third page
        ``` 
- **example 코드**
    ```python
    from operator import attrgetter

    class Page : 
        book_title = 'Python Pratice book' # class 변수
        def __init__(self, num, content):
            self.num = num
            self.content = content

        def output(self):
            return f'{self.content}'
        
        @classmethod # class method
        def print_pages(cls, *pages):
            print(cls.book_title)
            pages = list(pages)
            for page in sorted(pages, key=attrgetter('num')):
                print(page.output())


    first = Page(1,'first page')
    second = Page(2,'second page')
    third = Page(3,'third page')

    Page.print_pages(first, second, third)
    ```

# static method
- class method와 거의 동일한 방식으로 선언이 가능하다.
  - `@staticmethod`를 통해 선언한다.
- 스태틱 메서드는 단순한 함수와 같다고 보면 된다.
- 적극적으로 사용할 이유는 그다지 많지 않다고한다.

- **example 코드**
    ```python
        class Page : 
        def __init__(self, num, content):
            self.num = num
            self.content = content
        @staticmethod
        def check_blank(page):
            return bool(page.content)

        first = Page(1,'')
        Page.check_blank(first)
    ```