---
layout:     post
title:      Django使用笔记
subtitle:   Django
date:       2018-12-19
author:     owl city
header-img: img/django-person.png
catalog: true
tags:
    - Django
    - Python
    - Web Framework
---
## Django框架的搭建

## Django的使用

### 模型和数据库

#### Model

#### 进行查询
##### raw SQL
[raw_sql文档](https://docs.djangoproject.com/zh-hans/2.2/topics/db/sql/)

##### 使用Model进行数据库操作
- 创建Object
    - ```python
    from .models import Blog
    b = Blog(name=...)
    b. save()
    # 或者一步写完
    P = Blog.objects.create(xxx)
    ```

- 单表(单Model)更新记录

```python
# 相当于sql的update
b.name = 'xxx'
b.save()
```

- 对有外键关系Model进行更新和添加

```python
from blog.models import Blog, Entry, Author
# foreignkey
entry = Entry.objects.get(pk=1)
cheese_blog = Blog.objects.get(name="Cheddar Talk")
entry.blog = cheese_blog
entry.save()

# ManyToMany
joe = Author.objects.create(name="Joe")
entry.authors.add(joe)
```

- 精确查询

```python
# 下面两个是等价的
Blog.objects.get(id=12)
Bolg.objects.get(id__exact=12)

# 这样可以不区分大小写进行匹配
Blog.objects.get(name__iexact='AffA')
```

- Contains
类似于sql中的:`... LIKE '%LonL%'`

- 对同一model的不同字段进行比较
Django提供了F()表达式, F()表达式也支持四则等运算

```python
# 过滤评论数大于点赞数的条目
Entry.objects.filter(n_comments__gt=F('n_pingbacks'))
```

- Q objects
当查询条件较为复杂时,如有`&`或`|`条件时,可以使用`Q objects`,会将不同的objects合并和一个新的objects

```python
Poll.objects.get(
    Q(question__startswith='Who'),
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))
)
# 上述例子上当与这个sql:
# SELECT * from polls WHERE question LIKE 'Who%' AND (pub_date = '2005-05-02' OR pub_date = '2005-05-06')
```

**`Q`关键字可以和关键字查询一起使用,但是必须在关键字条件之前使用`Q`,不然会报错**


### URL设计篇
#### URL的获取
示例:

```python
from django.urls import path

from . import views

urlpatterns = [
    #...
    path('articles/<int:year>/', views.year_archive, name='news-year-archive'),
    #...
]

# 在模板中:
<a href="{% url 'news-year-archive' 2012 %}">2012 Archive</a>
{# Or with the year in a template context variable: #}
<ul>
{% for yearvar in year_list %}
<li><a href="{% url 'news-year-archive' yearvar %}">{{ yearvar }} Archive</a></li>
{% endfor %}
</ul>

# 在python代码中
from django.http import HttpResponseRedirect
from django.urls import reverse

def redirect_to_year(request):
    # ...
    year = 2006
    # ...
    return HttpResponseRedirect(reverse('news-year-archive', args=(year,)))
```

#### Class-based views
Django提供了使用面向对象的方式来管理我们的视图,这样可以结构化我们的代码以及能够重用一些功能

>Function-based views:基于函数的通用视图的问题在于，虽然它们很好地涵盖了简单的情况，但是除了一些简单的配置选项之外，没有办法扩展或定制它们，这限制了它们在许多实际应用中的有用性。

>Class-based views:基于类的泛型视图的创建目标与基于函数的泛型视图相同，以使视图开发更加容易。然而，通过使用mixins实现解决方案的方式提供了一个工具包，使得基于类的通用视图比基于函数的视图更具可扩展性和灵活性。
