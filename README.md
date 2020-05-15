# Django_Tutorial

## 1.기본설정
1. python -m venv 가상환경이름 (vscode의 경우 써줘야 )
2. source 가상환경이름/Scripts/activate
3. pip install django
4. django-admin startproject config .   
->장고 프로젝트 생성

## 2.웹 서버 시작하기
1. python manage.py runserver
2. python manage.py 8080   
->포트 변경(기본 8000)
3. python manage.py 0.0.0.0:8000/0:8000   ->IP직접 지정/같은 네트워크 망에서 접속 가능

## 3.앱 만들기
python manage.py startapp 앱이름

## 4.첫 번째 뷰 
* polls/views.py
```
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```
* polls/urls.py (직접 생성) :view를 호출하기 위한 url 연결
```
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
//index 뷰 연결 
```
path함수(route,view,kwargs,name)
1. route:주소
2. view:1의 주소로 접근했을때 호출 할 view
3. kwargs:view에 전달할 값들
4. name:route의 이름

```polls 폴더에 있는 urls.py는 앱의 라우팅만 담당한다 프로젝트의 메인 urls.py 파일에서 연결 해줘야 정상 동작```

* config/urls.py
```
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```
include는 다른 urls.py 파일을 참고할 수 있도록 해줌   
```ex)127.0.0.1:8000/polls/lsit 주소로 접속하면 polls/까지는 일치하기 때문에 잘라내고 list/부분만 polls/urls.py에서 찾아보는 방식으로 동작```

## 5.데이터베이스 만들기
python manage.py migrate   
->데이터베이스 만들고 초기화

``` settings.py TIME_ZONE=Asia/Seoul 설정하면 시간 바뀜```
## 6.모델 만들기
모델이란? 데이터베이스의 구조도
* polls/models.py
```
from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published') //사람이 읽기 쉬운 형태로 인자 전달 우리가 만든 홈페이지에는 date published로 나온다

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0) //기본값 0
```
1. 장고 모델은 models.Model 상속받아 만든다
2. 각 클래스의 변수들은 필드값을 갖는다
```ex)CharField=문자열 DateTimeField=날짜와 시간```

3. ForeignKey :다른 모델과의 관계를 만들기 위해 사용
```Choice모델이 ForeignKey로 Question모델을 갖는다는것은 Choice모델이 Question모델에 소속된다는것을 의미```
* config/settings.py (polls앱이 현재 프로젝트에 설치되어 있다고 알려줌) 
```
INSTALLED_APPS = [
    'polls.apps.PollsConfig', //'polls'가능
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```
python manage.py makemigrations polls   
->데이터베이스에 적용(Question/Choice모델을 만들었다는 사실을 알려줌,아직 반영x)

python manage.py migrate polls 0001   
->데이터베이스에 테이블을 생성하고 초기화

```1.models.py에서 모델을 변경 2.python manage.py makemigrations을 통해 변경사실에 대한 마이그레이션 생성 3.python manage.py migrate를 통해 변경사실 데이터베이스에 적용```

## 7.모델에 함수 추가하기
Question/Choice 모델에 __str__메소드추가  
->관리자 화면이나 쉘에서 객체를 출력할때 나타날 내용을 결정
```
class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text

class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```
```self란? 자기자신을 참조하는 객체```

* polls/models.py
```
import datetime 
from django.db import models
from django.utils import timezone

class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
```
## 8.관리자 페이지 확인하기
python manage.py createsuperuser   
->관리자 계정 생성
```/admin으로 들어가면 관리가 페이지 나옴```

관리자 페이지에는 그룹과 유저만 있는데 모델을 관리하려면 admin.py파일 수정해야 함
* polls/admin.py
```
from django.contrib import admin
from .models import Question

admin.site.register(Question)
```
Question 등록

## 9.여러가지 뷰 추가하기
우리가 만들어야 하는것
```1.투표 목록 2.투표 상세 화면 3.투표 기능 4.투표 결과```

* polls/views.py
```
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```
detail,result,vote 뷰 추가   
```%s에는 question_id의 값이 들어감 파이썬은 ,대신 %사용```

* polls/urls.py
```
from django.urls import path
from . import views

urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
    # ex: /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    # ex: /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```
뷰가 동작하도록 url연결   
```<>:변수/이 부분에 해당하는값을 뷰에 인자로 전달한다```
* polls/views.py
```
from django.http import HttpResponse
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)
```
```Question모델의 pub-date값 역순(최신순)으로 5개까지만  정렬하고 ,로 붙여서 출력```

html 파일은 템플릿에 저장한다   
->polls/templates/polls 폴더 만들기
* polls/templates/polls/index.html
```
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

```1.{% %} :if문이나 for문 2.{% for i in array %} :array 배열안에 있는 값 i,array 길이만큼 반복 3.{{i}} :i값 출력 4.{% endfor %} :닫아줘야 한다```

탬플릿 불러오기 위해 polls/views.py 수정
* polls/views.py (밑에 더 간단한거 있음)
```
from django.http import HttpResponse
from django.template import loader //탬플릿 불러오기 위해 loader import
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    template = loader.get_template('polls/index.html') //loader를 이용해 index.html을 불러오고 투표 목록 전달
    context = {
        'latest_question_list': latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```
```context:탬플릿에서 쓰이는 변수명과 파이썬 객체를 연결하는 사전형 값```

* render를 이용해 수정
```
from django.shortcuts import render //render import
from .models import Question

def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```
```render메소드는 request,탬플릿 이름,사전형 객체를 인자로 받는다```

## 10.404오류 일으키기
* polls/views.py (밑에 더 간단한거 있음)
```
from django.http import Http404 //Http404를 이용하면 상세 정보를 불러올 수 있는 투표 항목이 없을 경우 404 오류 발생
from django.shortcuts import render
from .models import Question

def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question}) //detail.html 탬프릿 사용
```
* get_object_or_404()로 수정
```
from django.shortcuts import get_object_or_404, render

def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```
```get_object_or_404 메소드는 Django 모델을 첫번째 인자로 받고 get 함수에 인자를 넘긴다 객체 존재하지 않으면 404 발생```

* polls/templates/polls/detail.html
```
<h1>{{ question.question_text }}</h1>
<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
{% endfor %}
</ul>
```
```question.choice_set.all :Question 모델을 foreign key로 참조하는 모든 Choice모델을 가져온다```

## 11.하드 코딩된 URL 없애기
```<a href="/polls/{{ question.id }}/">처럼 href 속성 값을 직접 써주는 방식으로 해둘 경우 나중에 주소를 다른 형태로 바꾸려면 변경하기 귀찮음```

```<a href="{% url 'detail' question.id %}">```

url 템플릿 태그를 사용해 주소를 만들어 출력
detail이라는 이름을 가진 URL형식을 찾아 URL을 만들어 출력 urls.py 전체를 검색해 찾는다(path에서 인수 이름을 정의했기 때문)

## 12.URL네임스페이스 설정
네임스페이스란? 분리된 경로를 만드는 개념  
ex)detail이라는 주소 이름을 가진 뷰가 여러 앱에 있을 경우 장고는 어느 뷰의 URL을 만들지 알 수가 없기때문에 네임스페이스를 설정해 각각의 뷰가 어느 앱에 속한것인지 구분   
* polls/urls.py (urls.py에 설정)
```app_name='polls'```
* polls/templates/polls/index.html(템플릿도 수정)   
```<a href="{% url 'polls:detail' question.id %}">```

## 13.간단한 폼 만들기
* polls/templates/polls/detail.html
```
<h1>{{ question.question_text }}</h1>
{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'polls:vote' question.id %}" method="post"> //form 태그 사용하여 사용자가 답변 선택하고 전달
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
{% endfor %}
<input type="submit" value="Vote">
</form>
```
```1.사용자가 선택한 항목의 번호를 vote뷰에 전달 2.forloop.counter :반복문의 횟수를 출력 3.input의 id와 label의 for는 같은값을 넣어준다(레이블 연결하면 텍스트만 눌러도 체크 됨) 4.csrf_token :서버로 들어온 요청이 사이트 내부에서 온 것인지 확인```
* polls/views.py
```
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from .models import Choice, Question

def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```
```1.request.POST['choice'] :선택된 설문의 ID를 문자열로 반환 POST자료에 choice 없으면 keyerror 2.HttpResponseRedirect에서 reverse 사용: URL을 하드코딩 하지 않도록 도와줌 -> /polls/question_id/results반환 3.폼 POST성공적 전송등의 redirect 상황에서 사용 4.selected_choice.save() :db에 저장```

* polls/views.py
```
def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```

* polls/templates/polls/results.html
```
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```
pluralize :복수형으로 만들어줌   
두명 이상 동시에 투표->결과 잘못 나옴(경쟁 상태)
```selected_choice.votes=F('votes')+1사용 ```

## 14.제너릭뷰 사용하기
제너릭뷰란? 장고에서 미리 준비한 뷰
* polls/views.py를 제너릭뷰로 수정 (함수형->클래스형)
```
from django.views import generic

class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list' //템플릿에 넘겨줄때 사용할 변수 이름 지정

    def get_queryset(self): //메소드 
        return Question.objects.order_by('-pub_date')[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'


class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'
```
```1.DetailView:객체 하나에 대한 상세 정보 2.ListView:조건에 맞는 여러 객체```
* polls/urls.py수정
```
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```
```1.클래스 뷰를 사용할때는 꼭 as_view()를 붙여야 한다 2.question_id->pk :PrimeKey를 담을 변수명 (DetailView에서 사용)```

## 15.정적 파일 사용
정적 파일이란? css,js같은 파일   
polls/static/polls 폴더 만들기
* css 불러오도록 html 수정
```
{% load static %} //정적 파일의 절대URL생성

<link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}">
```

## 16.관리자 폼 커스터마이징
* polls/admin.py
```
class QuestionAdmin(admin.ModelAdmin): //ModelAdmin을 상속받는 클래스
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date']}),
    ] //폼을 fieldset으로 분할

admin.site.register(Question, QuestionAdmin) //register의 두번째 인자로 클래스 전달
```
```fieldsets 튜플의 첫번째 요소 :fieldsets 제목```

* choice도 등록
```
class ChoiceInline(admin.StackedInline):
    model = Choice 
    extra = 3 //3개 추가

class QuestionAdmin(admin.ModelAdmin):
    fieldsets = [
        (None,               {'fields': ['question_text']}),
        ('Date information', {'fields': ['pub_date'], 'classes': ['collapse']}),
    ]
    inlines = [ChoiceInline]

admin.site.register(Question, QuestionAdmin)
```
* 인라인 아이템을 테이블 형식으로 보여주기(StackedInline->TabularInline)

* 목록에 보이는 항목 변경 (list_display 클래스 
변수 추가)
```
list_display = ('question_text', 'pub_date', 'was_published_recently')
```
* was_published_recently 출력 모양 변경(models.py 수정)
```
class Question(models.Model):
    # ...
    def was_published_recently(self):
        now = timezone.now()
        return now - datetime.timedelta(days=1) <= self.pub_date <= now
    was_published_recently.admin_order_field = 'pub_date' //정렬 기준
    was_published_recently.boolean = True //true면 아이콘
    was_published_recently.short_description = 'Published recently?' //항목의 헤더 이름
```
* 검색/필터 기능 추가
```
list_filter=['pub_date']
search_fields=['question_text]
```

//배프의 파이썬 웹 프로그래밍 튜토리얼 따라하기
