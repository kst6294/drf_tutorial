# 시작하기

관리자가 시스템의 사용자와 그룹을 관리할 수 있게 하는 간단한 API를 개발해보자.

# 1. 프로젝트 세팅

가상환경 생성 (miniconda 기준)

```bash
conda create -n drf_tutorial python=3.8
pip install django
pip install djangorestframework
```

Django 프로젝트 및 app 생성

```bash
# create project
django-admin startproject tutorial . # 점을 찍는 이유는 현재 위치한 디렉토리를 프로젝트 디렉토리로 설정하기 위함

cd tutorial # 프로젝트가 생성후 tutorial 디렉토리로 이동

django-admin startapp quickstart 
```

데이터베이스 동기화

```bash
python manage.py migrate
```

Migrate 이 완료되면 초기 user 정보가 생성된다. 

# 2. Serializers

Serialzers 를 생성해야한다. `tutorial/quickstart` 해당 경로에 `[serializers.py](http://serializers.py)` 를 생성하자.

그리고 아래와 같이 코드를 입력.

```python
from django.contrib.auth.models import User, Group
from rest_framework import serializers

class UserSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = User
        fields = ['url', 'username', 'email', 'groups']

class GroupSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Group
        fields = ['url', 'name']
```

[참고] 위 코드에서는 `HyperlinkedModelSerializer` 를 사용했다. 고유키(Primary Key) 혹은 다양한 다른 관계들을 사용할 수 있지만, hyperlinking 이 프로젝트를 RESTful 하게 사용하기에 좋은 방법이다.

# 3. Views

`tutorial/quickstart/views.py` 를 열고 아래와 같이 코드를 작성하자.

```python
from django.contrib.auth.models import User, Group
from rest_framework import viewsets
from rest_framework import permissions
from tutorial.quickstart.serializers import UserSerializer, GroupSerializer

class UserViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows users to be viewed or edited.
    """
    queryset = User.objects.all().order_by('-date_joined')
    serializer_class = UserSerializer
    permission_classes = [permissions.IsAuthenticated]

class GroupViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows groups to be viewed or edited.
    """
    queryset = Group.objects.all()
    serializer_class = GroupSerializer
    permission_classes = [permissions.IsAuthenticated]
```

여러개의 views 를 작성하지 않고 공통으로 사용할 수 있도록 `Viewsets` class 사용해서 작성했다!

위 코드를 개별 view 로 쪼갤 수 있지만 viewsets 을 사용하는 것이 가독성을 높이는 좋은 방법이다.

# 4.URLs

`tutorial/urls.py` 를 열고 아래 코드를 작성하자.

```python
from django.urls import include, path
from rest_framework import routers
from tutorial.quickstart import views

router = routers.DefaultRouter()
router.register(r'users', views.UserViewSet)
router.register(r'groups', views.GroupViewSet)

# Wire up our API using automatic URL routing.
# Additionally, we include login URLs for the browsable API.
urlpatterns = [
    path('', include(router.urls)),
    path('api-auth/', include('rest_framework.urls', namespace='rest_framework'))
]
```

여러개의 views 가 아닌 viewsets 를 사용했기 때문에, router 클래스 사용으로 viewsets 를 설정해주어 API 호출을 위한 URL 설정이 자동으로 생성된다. 

만약 API URL 에 대한 관리가 필요하다면, 기존의 views 의 동작방식을 사용해서 각각 URL 설정을 해주는 방법을 사용해도 괜찮다.

# 5. Pagination

페이지네이션은 하나의 페이지나 return 에 있는 여러개의 객체를 관리할 수 있게 한다. 아래의 코드를 `tutorial/settings.py` 에 추가하면 된다.

```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10
}

INSTALLED_APPS = [
    ...
    'rest_framework',
]
```

# 6. Run

이제 해당 프로젝트를 시작해보면 된다.

```python
python manage.py runserver
```# drf_tutorial
