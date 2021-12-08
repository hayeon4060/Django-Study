# Ch 3. Blog App

1. 애플리케이션 설계-	포스트 리스트를 보여주고, 특정 포스트를 클릭하면 해당 글을 읽을 수 있는 기능을 개발
- 화면 UI 설계
* post_list.html : Post들 title, modify_date, description. 그리고 페이지 이동과 관련된 template 파일
 - post_detail.html : Post간 이동, 그리고 post의 content를 보여주는 template 파일
-	테이블 설계 : Post
 - id
 - title : CharField(50), 포스트 제목
 - slug : SlugField(50), Unique, 포스트 제목 별칭?
 - description : CharField(100), Blank, 포스트 내용 한 줄 설명
 - content : TextField, 포스트 내용
 - create_dt : DateTimeField, auto_now_add, 포스트 생성 날짜 (auto_now_add : 생성되는 순간 자동?)
 - modify_dt : DateTimeField, auto_now, 포스트 수정 날짜
-	로직 설계 : Ch 2의 URL-뷰-템플릿 간 처리 흐름대로. 원래는 꼭 필요한 과정이지만 지금은 이걸 쓰기에는 너무 쉬운 과정이라 넘어감
-	URL 설계
 - blog/post/ : ListView, DetailView를 상속
 - blog/archive/ : ArchiveView 관련 view를 상속
 - 해보면서 알아보자
-	코딩순서 : 2장과 같음, 뼈대  모델  URLconf  View  Template
