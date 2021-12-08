# Ch 3. Blog App

## 1. 애플리케이션 설계 
#### 포스트 리스트를 보여주고, 특정 포스트를 클릭하면 해당 글을 읽을 수 있는 기능을 개발
- 화면 UI 설계
   - post_list.html : Post들 title, modify_date, description. 그리고 페이지 이동과 관련된 template 파일
   - post_detail.html : Post간 이동, 그리고 post의 content를 보여주는 template 파일
- 테이블 설계 : Post
   - id
   - title : CharField(50), 포스트 제목
   - slug : SlugField(50), Unique, 포스트 제목 별칭? 자동 생성 됨. 자세한 설명언 뒤에
   - description : CharField(100), Blank, 포스트 내용 한 줄 설명
   - content : TextField, 포스트 내용
   - create_dt : DateTimeField, auto_now_add, 포스트 생성 날짜 (auto_now_add : 생성되는 순간 자동?)
   - modify_dt : DateTimeField, auto_now, 포스트 수정 날짜
- 로직 설계 : Ch 2의 URL-뷰-템플릿 간 처리 흐름대로. 원래는 꼭 필요한 과정이지만 지금은 이걸 쓰기에는 너무 쉬운 과정이라 넘어감
- URL 설계
   - blog/post/ : ListView, DetailView를 상속
   - blog/archive/ : ArchiveView 관련 view를 상속
- 코딩순서 : 2장과 같음, 뼈대 --> 모델 --> URLconf --> View --> Template. 이 순서를 지킬 것

---

## 2. 코딩
### 2.2. 모델 코딩
#### 포스트 테이블에 필요한 여러 코드들
- 앞에서는 테이블 정의만 설계했는데, 여기서는 필요한 메소드도 함께 정의한다.
- reverse() : urls.py에 정의한 URL name을 인수로 받아서 URL 패턴을 반환하는 함수.
- title : CharField, verbose_name은 컬럼에 대한 별칭을 나타냄.
- slug : 제목의 별칭이다. 특정 포스트를 검색할 때 기본키 대신 사용됨…?
   - 페이지나 포스트를 설명하는 핵심 단어 집합
   - 페이지나 포스트 제목에서 조사, 전치사, 쉼표, 마침표 등 빼고, 띄어쓰기는 하이픈 대체
   - 슬러그를 URL에 사용함으로써, 검색 엔진의 정확도를 높여주고 빨리 찾게 해줌.
   - 수동으로 슬러그를 만드는 것보다, 이미 만들어놓은 데이터/함수를 이용해서 slug를 만들어라.
- blank=True : 빈칸도 가능하다.
- TextField : 여러 줄 입력이 가능
- DateTimeField에서 auto_now_add=True : 객체가 생성될 때의 시각을 자동 기록
- DateTimeField에서 auto_now : 객체가 데이터베이스에 저장될 때의 시각 자동 기록. 즉, 변경될 때.
- 필드 속성 외 필요한 파라미터는 Meta 내부 클래스로 정의 한다.
- 테이블의 별칭 : 단수, 복수로 가질 수 있는데 따로 설정 가능
- db내에 저장되는 테이블 이름을 지정할 수 있음
- 모델 객체의 리스트 출력 시에 modify_dt 컬럼을 기준으로 내림차순 정렬




#### 그 외 새로 정의되는 메소드들
- 새로 정의되는 메소드들
   - get_absolute_url() : 메소드가 정의된 객체를 지칭하는 URL을 반환함, 메소드 내에서 reverse()를 호출한다.
   - get_previous() : 메소드 내에서 장고의 내장 함수인 get_previous_by_modify_dt()를 호출. modify_dt 기준으로 최신 포스트를 반환?
   - get_next() : -modify_dt 컬럼을 기준으로 다음 포스트를 반환. 마찬가지로 장고 내장함수를 호출
   - get_previous_by_modify_dt()와 같은 내장함수들은 column이름을 어떻게 만드느냐에 따라서 다른 이름이 된다(modify_dt라는 column이 있기에 이러한 함수가 만들어짐)
- 위에 정의한 Post 테이블이 admin 사이트에 보이도록, admin.py 수정
   - Post 객체를 보여줄 때 id, title, modify_dt를 보이도록
   - modify_dt 컬럼을 사용하는 필터 사이드바 보여주도록
   - 검색박스 표시, 입력된 단어는 title과 content 컬럼에서 검색하도록
   - slug 필드는 title 필드를 사용해서 미리 채워지도록 한다.
- makemigrations, migrate 진행하고 확인



### 2.3. URLconf 코딩

- ROOT_URLCONF, APP_URLCONF의 두 개 파일에 코딩할 것.
- mysite/urls.py, blog/urls.py 두 개의 파일에 코딩하고, 북마크 앱도 그렇게 바꾸겠다.
- include : include 안에 들어간 문자열의 urls.py로 그 처리를 위임한다.
- bookmark 앱
   - app_name = bookmark : namespace를 bookmark로 지정. URL패턴의 이름을 정할 때 URL 패턴이름이 충돌되지 않도록 한다. bookmark에도 index, detail이라는 이름의 URL패턴이 있고, blog에도 그런 이름의 URL패턴이 있다면 둘이 충돌할 것. 그래서 앱네임을 지정해서 구분해준다.
   - path는 2장에서 만들었던 URL 패턴에서, mysite.urls로 옮겨진 ‘bookmark/’를 제외한 것. 나머지는 똑같음
   - URL 패턴 이름이 달라졌기 때문에 템플릿 파일에서 URL 패턴 이름(ex. index, detail 등)을 사용했다면 수정해줘야 한다.
   - \<li>\<a href="{% url 'detail' bookmark.id %}">{{ bookmark }}\</a>\</li> 이부분 --> url ‘detail’ 이부분을, url ‘bookmark:detail’ 로 바꿔주어야 함.
- blog 앱
   - re_path? regex를 사용해서 re_path라는 이름을 붙인 듯




### 2.4. View 코딩

- ListView에서 객체의 정렬 순서는 modify_dt를 기준으로 내림차순(왜? model 작성할 때 ordering 어트리뷰트를 그렇게 짰으니까)
- DetailView : re_path에서, slug를 View로 넘겨준다. View는 DetailView(PostDV)로 들어가며, 이 View에 할당된 template로 slug를 다시 넘겨준다. template는 아마도 post_detail.html일 듯.
- ArchiveView : 객체 리스트를 가져와서, 지정한 날짜 필드를 기준으로 최신 객체를 먼저 출력한다.
- 기준이 되는 날짜 필드는 modify_dt 컬럼을 사용한다.
- YearArchiveView : 테이블로부터 날짜 필드의 연도를 기준으로 객체 리스트를 가져오고, 그 객체들이 속한 월을 리스트로 출력한다. 연도 파라미터는 URLconf에서 view로 넘겨준다.
- 변경연도가 YYYY인 포스트를 검색해서, 그 포스트들의 변경 월을 출력 - 출력한다는 게 뭔 소리? 어디에 출력? 터미널에? (답 : template에 출력하는 코드가 있었음)
- make_object_list 속성이 True면, 해당 연도에 해당하는 객체 리스트를 만들어서 템플릿에 넘겨준다.  object_list 컨텍스트 변수를 템플릿에서 사용 가능. 이게 False면 대체 무슨 객체를 template에 넘겨준다는 거지?
- MonthArchiveView : 날짜 필드의 연월을 기준으로 객체 리스트를 가져옴. 연 월 파라미터는 URLconf에서 추출해서 view로 넘겨줌. 기준 날짜 필드는 modify_dt 사용
- DayArchiveView : 연 월 일. 나머지는 같음
- TodayArchiveView : 날짜 필드가 오늘인 객체 리스트를 가져온다. 오늘 날짜를 기준 연월일로 지정한다는 점 외에는 DayArchiveView와 같다.
- YearArchiveView, MonthArchiveView, DayArchiveView 각각이. 기준 연 월 일을 어떻게 구분할 수 있는지? URL 패턴에 기록된 year, month, day 말고는 전해주는 다른 방법이 없는데. URL 패턴에 기록된 2019라는 숫자가 year를 나타내는 것인지, 아닌지를 어떻게 YearArchiveView가 알고 그 year에 해당하는 객체들을 출력하는 건지?


### 2.5. Template 코딩
#### post_all.html
- post.get_absolute_url
   - post 객체의 메소드를 호출. /blog/post/slug단어/ 와 같은 형식의 URL 패턴이 나올 것.
   - models.py에서 코딩했음,
   - 한 개의 post 객체가 주어지면, models.py에서 코딩한 ‘blog:post_detail’과 args(slug)를 합쳐서 URL 패턴을 리턴한다.
   - 즉, post.get_absolute_url 은 그 post의 post_detail URL 패턴을 리턴함.
- post.modify_dt|date:”N d, Y” : post의 modify_dt 속성값을 N d, Y 형태로 출력
- page_obj : page 객체가 들어 있는 컨텍스트 변수. 하나의 페이지를 말하는 듯.
- page_obj.has_previous : 이전 페이지가 있는지 없는지를 Boolean
- page_obj.number는 현재 페이지 번호, page_obj.paginator.num_pages는 총 페이지 개수
- url이 ?page=3 이런 식으로 짜이면 알아서 page=3으로 넘어가져?
