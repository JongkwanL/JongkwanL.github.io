---
title: Django Custom pagination
description: >
author: Jongkwan Lee
date: 2025-01-12 19:00:00 +0900
categories: [Django, Pagination]
tags: []
pin: false
math: true

media_subpath: '/posts/20250112'
---

## 1. Django 기본 Paginator 이해

장고에서 가장 간단하게 페이지네이션을 적용하려면, 다음처럼 `Paginator` 클래스를 사용하면 됩니다.

```python
# views.py
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger
from django.shortcuts import render
from .models import Post

def post_list_view(request):
    post_qs = Post.objects.all().order_by('-created_at')
    
    # Paginator 객체 생성 (페이지당 10개)
    paginator = Paginator(post_qs, 10)
    
    page = request.GET.get('page', 1)
    
    try:
        posts = paginator.page(page)
    except PageNotAnInteger:
        posts = paginator.page(1)
    except EmptyPage:
        posts = paginator.page(paginator.num_pages)

    return render(request, 'blog/post_list.html', {'posts': posts})
```

- **`Paginator(queryset, per_page)`**: queryset, list 등 iterable 객체와 한 페이지당 보여줄 개수를 인자로 받음  
- **`paginator.page(number)`**: 실제로 해당 `number`에 해당하는 데이터 목록(`Page` 객체)을 반환  
- 예외 처리: `PageNotAnInteger`, `EmptyPage` 예외를 적절히 처리  

### 템플릿 예시

```html
<!-- post_list.html -->
{% for post in posts %}
  <h2>{{ post.title }}</h2>
  <p>{{ post.content }}</p>
{% endfor %}

<div class="pagination">
  {% if posts.has_previous %}
    <a href="?page={{ posts.previous_page_number }}">이전</a>
  {% endif %}
  
  <span>현재 {{ posts.number }} / {{ posts.paginator.num_pages }}</span>
  
  {% if posts.has_next %}
    <a href="?page={{ posts.next_page_number }}">다음</a>
  {% endif %}
</div>
```

- `posts` 변수가 Page 객체이므로, `posts.has_previous`, `posts.has_next`, `posts.next_page_number`, `posts.previous_page_number` 등을 바로 사용 가능  
- `posts.paginator.num_pages`를 통해 전체 페이지 수를 가져올 수 있음  



## 2. 커스텀 Paginator가 필요한 상황

기본 Paginator로도 대부분의 상황을 처리할 수 있지만, 다음과 같은 요구사항이 있을 때는 **커스텀 Paginator**가 유용합니다.

1. **페이지 번호 대신 다른 식별자**(예: 해시값, 날짜, 슬러그 등)로 페이지네이션을 하고 싶을 때  
2. **페이지 목록을 일괄 계산**하기보다는 이전/다음 버튼만 사용하거나, infinite scroll처럼 **동적으로 데이터를 불러와야** 할 때  
3. `paginate_by`가 상황에 따라 **동적으로 달라져야** 할 때(예: 사용자 환경, 디바이스 종류에 따라 per_page 다르게 설정)  
4. 특정 조건(필터, 검색, 정렬 등)에 따라 여러 쿼리 파라미터를 함께 처리해야 할 때  


## 3. 가장 흔한 커스텀 방식: Paginator 상속하기

Django는 `django.core.paginator.Paginator` 클래스를 상속받아서 필요한 로직을 오버라이딩할 수 있게 해줍니다. 예를 들어, **페이지 번호 대신 특정 키**(timestamp, slug 등)를 사용해 다음 페이지를 찾고 싶다면, 아래처럼 구현할 수 있습니다.

### 3.1 예시: created_at 기준 ‘다음 페이지’ 구하기

게시물이 시간순으로 정렬되어 있다고 가정하고, **“이전 페이지” / “다음 페이지”** 형식의 링크만 제공하는 시나리오를 생각해 봅시다.

```python
# paginations.py (예: 커스텀 paginator 파일)
from django.core.paginator import Paginator

class CreatedAtPaginator(Paginator):
    """
    created_at 컬럼을 기준으로, 다음 데이터들을 가져오도록 커스터마이징한 Paginator
    """
    
    def __init__(self, object_list, per_page, **kwargs):
        super().__init__(object_list, per_page, **kwargs)

    def page_by_timestamp(self, timestamp):
        """
        - timestamp(예: 2025-01-01 10:00:00) 이후의 데이터를
          일정 개수(per_page)만큼 가져온다.
        - 만약 timestamp가 주어지지 않으면 가장 최신부터 시작.
        """
        if timestamp:
            # 특정 시점 이전(혹은 이후) 데이터만 필터링
            filtered_qs = self.object_list.filter(created_at__lt=timestamp)
        else:
            # timestamp가 없으면 전체 목록에서 상위 per_page만큼
            filtered_qs = self.object_list
        
        # 실제로 슬라이싱
        return filtered_qs.order_by('-created_at')[:self.per_page]
```

이렇게 커스텀 Paginator를 만들면, `page_by_timestamp` 메서드를 호출해서 **생성일(created_at)** 기준으로 원하는 개수만큼 데이터를 가져올 수 있습니다.

> **주의**: 기본 Paginator와는 동작 방식이 다르므로, `page()` 메서드를 사용하는 대신 직접 정의한 `page_by_timestamp()` 같은 커스텀 메서드를 사용해야 합니다.

### 3.2 뷰에서 사용하기

```python
# views.py
from django.shortcuts import render
from .models import Post
from .paginations import CreatedAtPaginator

def post_list_by_timestamp(request):
    timestamp = request.GET.get('timestamp', None)
    
    post_qs = Post.objects.all()  # 정렬은 paginator에서 처리
    paginator = CreatedAtPaginator(post_qs, 10)
    
    # 커스텀 메서드 호출
    posts = paginator.page_by_timestamp(timestamp)
    
    # 다음 페이지용 timestamp (가장 오래된 데이터의 created_at)
    next_timestamp = posts.last().created_at.isoformat() if posts.exists() else None
    
    context = {
        'posts': posts,
        'next_timestamp': next_timestamp,
    }
    return render(request, 'blog/post_list.html', context)
```

템플릿에서는 `next_timestamp`를 사용해 `?timestamp={{ next_timestamp }}` 형태로 “더 보기” 링크나 AJAX 콜을 구현할 수 있습니다.


## 4. Django Rest Framework에서의 Custom Pagination

Django Rest Framework(DRF)를 쓴다면, **클래스 기반**으로 제공되는 여러 Pagination 클래스를 상속하여 훨씬 쉽게 커스터마이징이 가능합니다.

- `PageNumberPagination`: 페이지 번호 기반(기본 Paginator와 유사)  
- `LimitOffsetPagination`: `limit`, `offset` 파라미터를 사용  
- `CursorPagination`: 커서(고유 식별자)를 사용하여 보안성과 무결성을 높임  

예를 들어, **페이지 번호와 한 페이지 사이즈를 동적으로 제어**하고 싶다면, `PageNumberPagination`를 상속 받아서 `page_size_query_param`을 지정할 수도 있습니다.

```python
# paginations.py
from rest_framework.pagination import PageNumberPagination

class CustomPageNumberPagination(PageNumberPagination):
    page_size = 10  # 기본 페이지 당 개수
    page_size_query_param = 'page_size'  # 사용자가 ?page_size=20 형태로 요청 가능
    max_page_size = 100  # 최대 허용 페이지 사이즈
```

이후 `settings.py`에서 전역으로 적용하거나, 개별 뷰/뷰셋에서 `pagination_class`로 지정하면 됩니다.

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'myapp.paginations.CustomPageNumberPagination',
    'PAGE_SIZE': 10,
}
```

또는 특정 뷰셋에서만 사용:

```python
# views.py
from rest_framework import viewsets
from .models import Post
from .serializers import PostSerializer
from .paginations import CustomPageNumberPagination

class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.all()
    serializer_class = PostSerializer
    pagination_class = CustomPageNumberPagination
```


## 5. 정리

1. **기본 `Paginator`**  
   - `Paginator`와 `Page` 객체를 통해 간단히 페이지네이션 구현 가능  
   - 예외 처리(`PageNotAnInteger`, `EmptyPage`)도 필수  
2. **커스텀 Paginator**  
   - `Paginator` 클래스를 상속받아 필요한 메서드를 오버라이딩하거나, 전혀 다른 로직(`page_by_timestamp`, `page_by_slug` 등)을 만들 수 있음  
   - **페이지 번호 대신 다른 기준**(시간, 해시, 슬러그)을 사용하는 경우에 유용  
3. **Django Rest Framework**  
   - `PageNumberPagination`, `LimitOffsetPagination`, `CursorPagination` 등 다양한 기본 페이징 전략이 있음  
   - 상속하여 `page_size_query_param`, `max_page_size` 등 쉽게 커스터마이징 가능  

프로젝트 요구사항에 따라 **페이지 번호** 기반의 전통적인 페이지네이션을 쓸 수도 있고, **커서 기반** 또는 **무한 스크롤(Infinite Scroll)** 구조처럼 좀 더 특별한 방식을 구현할 수도 있습니다. 결국 **장고의 기본 Paginator**를 잘 이해하면, 필요한 로직을 얼마든지 확장해 **맞춤형 페이지네이션**을 구현할 수 있습니다.