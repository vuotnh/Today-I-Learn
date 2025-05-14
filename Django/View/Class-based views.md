`Class-based view` là triển khai các views bằng các python class (object), thay vì định nghĩa các function như `function-based views`.
Việc định nghĩa các view dưới dạng các class giúp tổ chức code khoa học hơn, tái sử dụng code hiệu quả hơn nhờ các ***đặc tính kế thừa và mixins (đa kế thừa)*** của OOPS.

Nếu chỉ sử dụng `function-based view` => mỗi api sẽ phải define một function riêng với một số logic code lặp lại => lặp code

Khi định nghĩa view dưới dạng các class => có thể định nghĩa các `generic view` thực hiện các tính năng lặp lại => các view khác sẽ kế thừa lại các tính năng đó.
```python
# Sử dụng function-based view
from django.http import HttpResponse
def my_view(request):
	if request.method == 'GET':
		# <view logic>
		return HttpResponse("result")

# Sử dụng class-based view
from django.http import HttpResponse
from django.views import View
class MyView(View):
	def get(self, request):
		# <view logic>
		return HttpResponse("result")
```

***Các class-based view đều kế thừa đối tượng django.views.View***
- Khi Django map URL tới một view => nó mong muốn nhận được một `callable function` chứ ko phải một class => không thể đưa view class vào URLconf một cách trực tiếp
- các class view sẽ có một phương thức đặc biệt tên `as_view()`. Phương thức này sẽ trả về một hàm (method) thuộc class đúng như Django mong muốn

### <span style="color: green">Lưu ý</span>
Khi URLconf gọi `MyView.as_view()`
- tạo ra một instance của class view
- gọi hàm `setup()` ở class `django.views.View` để khởi tạo các thuộc tính cần thiết như `request, args, kwargs,..`
- sau đó gọi tới hàm `dispatch()` ở `django.views.View` sẽ dựa vào loại HTTP method của request (GET, POST, PUT, ...). Nếu class có định nghĩa method tương ứng như `get()`, `post()`, ... thì `dispatch()` sẽ gọi đúng method đó.

Tuy nhiên, khi người dùng muốn đặt tên của method khác các HTTP method, hàm `dispatch()` của `django.views.View` sẽ không thể trả về đúng method mong muốn. Do đó ta có thể custom lại hàm này để trả về method với tên tùy biến.
```python
from django.views import View
from django.http import HttpResponseNotAllowed

class ActionView(View):
    def dispatch(self, request, *args, **kwargs):
        action = getattr(self, 'custom_action', None)
        # Nếu class có method nào trùng tên với action => gọi method đó
        if action and hasattr(self, action): 
            handler = getattr(self, action)
            return handler(request, *args, **kwargs)
        return super().dispatch(request, *args, **kwargs)

    @classmethod
    def as_view(cls, **initkwargs):
	    # Lấy ra biến action mà người dùng truyền vào
        action = initkwargs.pop('action', None) 
        view = super().as_view(**initkwargs)
        def wrapped_view(request, *args, **kwargs):
            self = cls(**initkwargs)
            if action:
	            # lưu tên của method vào custom_action
                self.custom_action = action 
            return self.dispatch(request, *args, **kwargs)
        return wrapped_view

class GuestAgentView(ActionView):
    def get_config(self, request, *args, **kwargs):
        return JsonResponse({"config": "some config here"})

# Khi sử dụng với URLconf
urlpatterns = [
    path("get-config/", GuestAgentView.as_view(action='get_config'), name='get-config'),
]
```

## Mixins

***Mixins*** là việc đa kế thừa các class cha, class con sẽ thừa hưởng đầy đủ các hành vi và thuộc tính của các class cha.
Việc sử dụng Mixins giúp giảm lặp code, tái sử dụng code.

Định nghĩa ra các `Generic View` sử dụng như các `mixins` để các class con kế thừa