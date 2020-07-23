# django的三种时间字段（DateField,DateTimeField,TimeField）

创建django的model时，有DateTimeField、DateField和TimeField三种类型可以用来创建日期字段，其值分别对应着datetime()、date()、time()三中对象。这三个field有着相同的参数auto_now和auto_now_add，表面上看起来很easy，但实际使用中很容易出错，下面是一些注意点。

## DateTimeField.auto_now

这个参数的默认值为false，设置为true时，能够在保存该字段时，将其值设置为当前时间，并且每次修改model，都会自动更新。因此这个参数在需要存储“最后修改时间”的场景下，十分方便。需要注意的是，设置该参数为true时，并不简单地意味着字段的默认值为当前时间，而是指字段会被“强制”更新到当前时间，你无法程序中手动为字段赋值；如果使用django再带的admin管理器，那么该字段在admin中是只读的。

## DateTimeField.auto_now_add

这个参数的默认值也为False，设置为True时，会在model对象第一次被创建时，将字段的值设置为创建时的时间，以后修改对象时，字段的值不会再更新。该属性通常被用在存储“创建时间”的场景下。与auto_now类似，auto_now_add也具有强制性，一旦被设置为True，就无法在程序中手动为字段赋值，在admin中字段也会成为只读的。

![img](F:\typoraImg\1195739-20171125144246078-1086668689.png)

 

## admin中的日期时间字段

auto_now和auto_now_add被设置为True后，这样做会导致字段成为editable=False和blank=True的状态。editable=False将导致字段不会被呈现在admin中，blank=Ture表示允许在表单中不输入值。此时，如果在admin的fields或fieldset中强行加入该日期时间字段，那么程序会报错，admin无法打开；如果在admin中修改对象时，想要看到日期和时间，可以将日期时间字段添加到admin类的readonly_fields中：

```
class YourAdmin(admin.ModelAdmin):
    readonly_fields = ('save_date', 'mod_date',)
admin.site.register(Tag, YourAdmin)
```

 

## 如何将创建时间设置为“默认当前”并且可修改

那么问题来了。实际场景中，往往既希望在对象的创建时间默认被设置为当前值，又希望能在日后修改它。怎么实现这种需求呢？

django中所有的model字段都拥有一个default参数，用来给字段设置默认值。可以用default=timezone.now来替换auto_now=True或auto_now_add=True。timezone.now对应着django.utils.timezone.now()，因此需要写成类似下面的形式：

```
from django.db import models
import django.utils.timezone as timezone
class Doc(models.Model):
    add_date = models.DateTimeField('保存日期',default = timezone.now)
    mod_date = models.DateTimeField('最后修改日期', auto_now = True)
```