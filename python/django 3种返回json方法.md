## django 3种返回json方法

#### 1.手动组装字典返回

```
from django.http import JsonResponse, HttpResponse
from django.shortcuts import render
from app01.models import Book


# Create your views here.

def get_book(request):
    all_book = Book.objects.all()
    d = []
    for i in all_book:
        d.append({'name': i.name})
    return JsonResponse(d, safe=False)
```

#### 2.JsonResponse返回

```
def get_book2(request):
    from django.forms.models import model_to_dict
    all_book = Book.objects.all()
    d = []
    for i in all_book:
        d.append(model_to_dict(i))           # <-------针对一个对象()
    return JsonResponse(d, safe=False) # 非字典要设置成false
```

一般自己的系统会从别的系统获取数据, 这里应该也仅限于展示, 所以JsonResponse还是有很多实用场景

```
def booapi(request):
    from django.core.serializers import serialize
    book_list = [
        {'id': 1, 'name': 'ptyhon'},
        {'id': 2, 'name': 'go'},
    ]
    import json
    return HttpResponse(json.dumps(book_list), content_type='application/json')
```

#### 3.django自带的serializers返回

这个好像只能针对queryset操作,即本地db里的数据,不能操作从其他系统api获取到的list ,dict等

```
def get_book3(request):
    from django.core.serializers import serialize
    d = serialize('json', Book.objects.all()) # <-------针对一个queryset,[{}, {}]

    # return HttpResponse(d)
    return HttpResponse(d)
return render(request, 'myapp/index.html', {'foo': 'bar',}, content_type='application/xhtml+xml')

return HttpResponse(t.render(c, request), content_type='application/xhtml+xml')

return HttpResponse(json.dumps(data), content_type='application/json', status=400)

JsonResponse = HttpResponse+content-type
```

model转dict方法
https://mp.weixin.qq.com/s/7gPLaCESHAB0dLgq7qZq5Q
![img](F:\typoraImg\1312420-20181013131306037-2126256603.png)
使用类的__dict__方法

https://stackoverflow.com/questions/21925671/convert-django-model-object-to-dict-with-all-of-the-fields-intact
![img](F:\typoraImg\1312420-20181122113502110-1868059184.png)

http://www.liujiangblog.com/course/django/171
![img](F:\typoraImg\1312420-20181122113540370-1823313393.png)