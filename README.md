프로젝트 1 — 자동화 도구 비교 구현
1. 주제
Google Forms 응답 → 점수에 따른 조건 분기 → Google Sheets 기록 + Gmail 알림

동일한 워크플로우를 Make와 Zapier에서 각각 구현한다.

전체 구조는 다음과 같다.

Google Forms 응답
       ↓
     Trigger
       ↓
응답 데이터 확인
       ↓
   조건 분기
   ┌──────┴──────┐
   ↓             ↓
80점 이상       80점 미만
   ↓             ↓
합격자 시트     불합격자 시트
기록            기록
   ↓             ↓
Gmail 알림      Gmail 알림

이 구조는 Trigger 1개, 조건 분기 1개, Action 2개 이상이라는 요구사항을 만족한다.

2. 테스트용 Google Form 만들기
Google Form을 하나 만든다.

제목:

자동화 테스트 점수 입력

질문은 다음처럼 만든다.

질문	유형
이름	단답형
이메일	단답형
점수	숫자

예시 데이터:

이름	이메일	점수
김민수	본인 이메일	90
이서연	본인 이메일	70

Google Form의 응답을 Google Sheets에 연결한다.

시트 이름은 다음처럼 만들어두면 좋다.

Form Responses
합격자
불합격자

3. Make 구현
워크플로우
Google Sheets
Watch New Rows
       ↓
Router
   ┌───┴───┐
   ↓       ↓
점수 >= 80  점수 < 80
   ↓       ↓
합격자 시트  불합격자 시트
   ↓       ↓
Gmail       Gmail

Make의 무료 플랜은 현재 월 1,000 credits를 제공하고, 시각적 no-code workflow builder와 Routers & Filters를 포함한다. 무료 플랜의 최소 실행 간격은 15분이다. 
M
Make
+1

Step 1. Scenario 생성
Make에 접속해서 새 Scenario를 만든다.

첫 번째 모듈:

Google Sheets → Watch New Rows

Google Sheets 계정을 연결하고 테스트용 Spreadsheet를 선택한다.

Step 2. Router 추가
Google Sheets 모듈 다음에 Router를 추가한다.

Router를 추가하면 다음과 같이 두 경로가 만들어진다.

             Router
            /      \
           /        \
       합격 경로    불합격 경로

Step 3. 합격 Filter
첫 번째 경로에 Filter를 설정한다.

조건:

점수 >= 80

예를 들어 Make에서:

Score
Greater than or equal to
80

으로 설정한다.

Step 4. 합격자 Google Sheets 기록
Filter 뒤에:

Google Sheets → Add a Row

를 추가한다.

합격자 시트를 선택하고 다음처럼 매핑한다.

이름 → Name
이메일 → Email
점수 → Score

Step 5. Gmail 알림
그 다음:

Gmail → Send an Email

을 추가한다.

예:

To: 응답자의 이메일

Subject:
[자동화 테스트] 합격 안내

Body:
안녕하세요.

테스트 결과 80점 이상으로 합격 처리되었습니다.

이름: {{이름}}
점수: {{점수}}

Step 6. 불합격 Filter
두 번째 Router 경로에는:

점수 < 80

조건을 넣는다.

Step 7. 불합격자 시트 기록
두 번째 경로에:

Google Sheets → Add a Row

를 추가한다.

이번에는 불합격자 시트에 기록한다.

이름 → Name
이메일 → Email
점수 → Score

Step 8. 불합격 Gmail
마지막으로 Gmail을 추가한다.

To: 응답자의 이메일

Subject:
[자동화 테스트] 결과 안내

Body:
안녕하세요.

테스트 결과 80점 미만으로 확인되었습니다.

이름: {{이름}}
점수: {{점수}}

4. Make 테스트
반드시 두 경로를 각각 실제 실행한다.

테스트 1
Google Form에:

이름: 김민수
이메일: 본인 이메일
점수: 90

입력.

기대 결과:

Google Sheets
      ↓
Router
      ↓
점수 >= 80
      ↓
합격자 Sheet 기록
      ↓
Gmail 발송

테스트 2
다시 Google Form에:

이름: 이서연
이메일: 본인 이메일
점수: 70

입력.

기대 결과:

Google Sheets
      ↓
Router
      ↓
점수 < 80
      ↓
불합격자 Sheet 기록
      ↓
Gmail 발송

이 두 번의 실행 화면이 매우 중요하다.

Make에서 찍을 스크린샷
Make Scenario 전체 화면
Router와 두 개의 Filter가 보이는 화면
합격 경로 실행 결과
불합격 경로 실행 결과
합격자 Google Sheets
불합격자 Google Sheets
Gmail 수신 결과
5. Zapier 구현
여기서 중요한 문제가 하나 있다.

현재 Zapier Free 플랜은 월 100 tasks와 2-step Zap이 기본이고, Multi-step Zap은 Professional 이상이다. 또한 Free에서는 polling trigger가 15분 간격이다. 
Z
Zapier Help
+1

따라서 과제의 Trigger + Filter/Path + Action 2개 이상을 하나의 Zap으로 구현하려면 현재 무료 플랜만으로는 제한될 수 있다. Zapier의 Filter/Paths 기능 자체는 조건부/분기 로직을 제공하지만, 무료 플랜의 2-step 제한 때문에 과제 요구사항 전체를 무료로 구현하는 조합은 적합하지 않다. 
Z
Zapier Help
+1

따라서 Zapier는 14일 trial 또는 해당 기능이 활성화된 계정에서 구현하는 것을 추천한다. Zapier는 가입 시 유료 기능을 시험할 수 있는 14일 trial을 제공한다고 안내한다. 
Z
Zapier Help

6. Zapier 구조
Make와 완전히 동일한 논리로 만든다.

Google Sheets
New Spreadsheet Row
       ↓
Paths
   ┌───┴────┐
   ↓        ↓
80점 이상   80점 미만
   ↓        ↓
합격 Sheet   불합격 Sheet
   ↓        ↓
Gmail       Gmail

Zapier에서는 Paths를 사용하면 된다. Zapier 공식 설명에서도 Filter와 Paths를 조건부/분기 로직을 위한 기능으로 설명한다. 
Z
Zapier Help

7. Zapier 설정
Trigger
App:
Google Sheets

Event:
New Spreadsheet Row

Google Forms 응답이 들어오는 Google Sheets를 선택한다.

Paths
다음 단계에서:

Paths

추가.

Path A
이름:

합격

조건:

Score >= 80

Path B
이름:

불합격

조건:

Score < 80

Path A
Action 1:

Google Sheets
Create Spreadsheet Row

합격자 시트에 기록.

Action 2:

Gmail
Send Email

합격 메일 발송.

Path B
Action 1:

Google Sheets
Create Spreadsheet Row

불합격자 시트에 기록.

Action 2:

Gmail
Send Email

불합격 메일 발송.

8. Zapier 테스트
Make와 동일하게 두 번 테스트한다.

테스트 A
김민수
90점

→ 합격 Path 실행

테스트 B
이서연
70점

→ 불합격 Path 실행

스크린샷:

Zap 전체 구조
Trigger
Paths
합격 Path
불합격 Path
합격 실행 결과
불합격 실행 결과
Google Sheets 결과
Gmail 결과
프로젝트 1 비교 분석 보고서
아래 내용을 그대로 Markdown 보고서로 사용하면 된다.

작성


프로젝트 1. 자동화 도구 비교 구현
1. 프로젝트 개요
이번 프로젝트에서는 동일한 자동화 워크플로우를 Make와 Zapier에서 각각 구현하고 두 도구의 특징을 비교하였다.

구현한 자동화 업무는 Google Forms 응답을 확인하여 점수에 따라 합격자와 불합격자를 분류하고 Google Sheets에 기록한 뒤 Gmail로 결과를 알리는 작업이다.

전체 워크플로우는 다음과 같다.

Google Forms 응답
       ↓
Google Sheets
       ↓
조건 분기
   ┌───┴────┐
   ↓        ↓
80점 이상   80점 미만
   ↓        ↓
합격자 기록  불합격자 기록
   ↓        ↓
Gmail 알림  Gmail 알림

2. Trigger와 Action
Trigger
Trigger는 자동화 워크플로우를 시작시키는 이벤트이다.

이번 프로젝트에서는 Google Sheets에 새로운 Google Forms 응답이 추가되는 것을 Trigger로 사용하였다.

Action
Action은 Trigger가 발생한 이후 자동으로 수행되는 작업이다.

이번 프로젝트에서는 다음 Action을 사용하였다.

Google Sheets에 데이터 기록
Gmail 이메일 발송
3. 조건 분기
조건 분기는 입력된 데이터의 조건에 따라 서로 다른 작업을 수행하도록 만드는 기능이다.

이번 프로젝트에서는 점수를 기준으로 다음과 같이 분기하였다.

점수 >= 80 → 합격 경로
점수 < 80  → 불합격 경로

실제 테스트에서도 90점 데이터를 입력하여 합격 경로를 실행했고, 70점 데이터를 입력하여 불합격 경로를 실행하였다.

4. Make 구현
Make에서는 Google Sheets 모듈을 Trigger로 사용하고 Router를 통해 두 개의 경로로 분기하였다.

Google Sheets
Watch New Rows
       ↓
     Router
    /      \
   /        \
Score >= 80  Score < 80
   ↓          ↓
합격자 Sheet  불합격자 Sheet
   ↓          ↓
Gmail        Gmail

Make의 Free 플랜은 월 1,000 credits를 제공하며 시각적 워크플로우 빌더와 Routers & Filters를 지원한다.

5. Zapier 구현
Zapier에서는 Google Sheets의 새로운 행을 Trigger로 사용하고 Paths를 이용하여 점수에 따른 두 개의 경로를 구성하였다.

Google Sheets
New Spreadsheet Row
       ↓
     Paths
    /      \
   /        \
Score >= 80  Score < 80
   ↓          ↓
합격자 Sheet  불합격자 Sheet
   ↓          ↓
Gmail        Gmail

Zapier의 Filter와 Paths는 조건에 따라 특정 Action을 실행하거나 여러 경로로 워크플로우를 분기할 수 있도록 지원한다.

6. 도구 비교
비교 항목	Make	Zapier
UI/UX	시각적인 노드 기반 구조	단계 중심의 직관적인 인터페이스
설정 난이도	처음에는 다소 학습 필요	비교적 직관적
조건 분기	Router와 Filter를 이용해 시각적으로 구성	Filter와 Paths를 이용
무료 플랜	월 1,000 credits	월 100 tasks
무료 플랜의 워크플로우	Routers & Filters 사용 가능	Free는 2-step 제한
실행 주기	Free에서 최소 15분 간격	Free polling trigger는 15분 간격
실행 로그	Scenario 실행 단위로 확인 가능	Task History에서 실행 확인
복잡한 워크플로우	복잡한 분기 구조에 강점	단순한 자동화에 특히 편리
학습 난이도	중간	낮음
시각적 설계	강점	상대적으로 단순

7. Make의 장점
첫 번째 장점은 시각적인 워크플로우 구성이다.

여러 개의 분기와 모듈을 하나의 화면에서 확인할 수 있기 때문에 복잡한 자동화 구조를 설계할 때 유리하다.

두 번째 장점은 Router와 Filter를 이용한 조건 분기이다.

여러 조건을 가진 자동화 흐름을 시각적으로 표현하기 쉽다는 점이 장점이다.

세 번째 장점은 무료 플랜에서도 Router와 Filter를 사용할 수 있다는 점이다.

8. Make의 단점
처음 사용하는 경우 모듈, Router, Filter, Mapping 등의 개념을 익혀야 하기 때문에 Zapier보다 학습 난이도가 높게 느껴질 수 있다.

또한 복잡한 Scenario를 구성하면 화면에 많은 모듈이 배치되어 전체 구조를 관리하는 것이 어려워질 수 있다.

9. Zapier의 장점
Zapier는 자동화 단계를 순서대로 구성하는 방식이 직관적이어서 처음 자동화를 접하는 사용자도 비교적 쉽게 사용할 수 있다.

또한 다양한 외부 서비스와 연결하여 업무 자동화를 구성하기 편리하다.

Filter와 Paths를 통해 조건에 따른 자동화도 구현할 수 있다.

10. Zapier의 단점
현재 Free 플랜에서는 2-step Zap 제한이 있기 때문에 Trigger와 여러 Action을 포함하는 복잡한 과제에는 제약이 있다.

따라서 이번 과제처럼 Trigger 이후 조건 분기와 여러 Action이 필요한 경우에는 유료 플랜 또는 trial이 필요할 수 있다.

11. 어떤 상황에 적합한가?
Make는 여러 단계의 Action과 복잡한 Router/Filter를 포함하는 자동화를 시각적으로 설계해야 할 때 적합하다고 판단하였다.

반면 Zapier는 비교적 단순한 업무를 빠르게 자동화하거나 처음 자동화 도구를 사용하는 사용자에게 적합하다고 판단하였다.

12. 최종 평가
이번 구현을 통해 Trigger는 자동화의 시작 이벤트이고 Action은 Trigger 이후 수행되는 작업이라는 것을 확인하였다.

또한 Filter와 Router/Paths를 이용하면 입력 데이터의 조건에 따라 서로 다른 작업을 수행할 수 있다는 것을 실제로 확인하였다.

두 도구 모두 코딩 없이 자동화를 구성할 수 있지만, Make는 복잡한 시각적 Workflow 구성에 강점이 있고 Zapier는 직관적인 설정과 빠른 구축에 강점이 있다고 판단하였다.

이번 프로젝트에서는 조건 분기가 포함된 복잡한 Workflow를 구성해야 했기 때문에 개인적으로는 Make가 더 적합하다고 판단하였다.
프로젝트 2 — 자유 주제
프로젝트 2는 이메일 업무 자동화로 가자.

주제:

중요 이메일 자동 분류 및 Google Sheets 기록

실제 업무와 연결하기 쉽고, 프로젝트 1에서 사용한 Google Sheets/Gmail을 재활용할 수 있어서 가장 간단하다.

워크플로우
Gmail
새 이메일 수신
      ↓
    Trigger
      ↓
이메일 제목/내용 확인
      ↓
   Filter
      ↓
"긴급" 또는 "중요"
      ↓
Google Sheets 기록
      ↓
Gmail 알림

여기서 중요한 점은 과제에서 조건 분기 1개 이상을 요구하므로 Filter만 사용하기보다는 Router를 사용하는 편이 더 안전하다.

따라서 최종적으로는:

Gmail 새 메일
      ↓
    Trigger
      ↓
    Router
   /      \
  /        \
중요메일   일반메일
  ↓          ↓
Sheet       Sheet
기록        기록
  ↓
Gmail 알림

이렇게 구성한다.

프로젝트 2 제출 문서
작성


프로젝트 2. 중요 이메일 자동 분류 및 기록
1. 자동화할 반복 업무
업무 중 이메일을 확인한 후 중요한 이메일을 찾아 별도로 기록하고 팀 또는 본인에게 다시 알림을 보내는 작업을 자동화한다.

기존에는 이메일을 직접 확인하고 중요한 이메일을 판단한 뒤 Google Sheets에 내용을 기록해야 했다.

이 과정을 자동화하여 특정 조건에 해당하는 이메일이 도착하면 자동으로 Google Sheets에 기록하고 알림을 보내도록 구성하였다.

2. 선정한 자동화 도구
선정 도구는 Make이다.

Make를 선정한 이유는 Router와 Filter를 이용하여 이메일을 조건별로 분류하는 과정을 시각적으로 구성하기 쉽기 때문이다.

또한 Free 플랜에서 Router와 Filter를 사용할 수 있어 과제의 조건 분기 요구사항을 구현하기에 적합하다.

3. 워크플로우
Gmail
새 이메일 수신
       ↓
     Trigger
       ↓
     Router
    /      \
   /        \
중요 이메일  일반 이메일
   ↓          ↓
Google Sheets Google Sheets
중요메일 기록 일반메일 기록
   ↓
Gmail 알림

4. Trigger
Trigger는 Gmail에 새로운 이메일이 수신되는 이벤트이다.

새 이메일이 들어오면 자동화 Workflow가 실행된다.

5. 조건 분기
Router를 이용하여 이메일의 제목 또는 내용에 특정 키워드가 포함되어 있는지 확인한다.

예를 들어 다음 키워드를 중요 이메일 기준으로 설정하였다.

긴급
중요
마감

조건에 해당하면 중요 이메일 경로로 이동한다.

조건에 해당하지 않으면 일반 이메일 경로로 이동한다.

6. Action
중요 이메일 경로에서는 Google Sheets에 다음 정보를 기록한다.

수신 시간
발신자
제목
본문
분류

그 다음 Gmail을 이용하여 중요 이메일이 수신되었다는 알림을 보낸다.

7. 실제 실행 테스트
Test Case 1 — 중요 이메일
테스트 이메일 제목:

[긴급] 과제 제출 마감 확인

기대 결과:

Gmail 수신
   ↓
Trigger 실행
   ↓
Router
   ↓
중요 이메일 경로
   ↓
Google Sheets 기록
   ↓
Gmail 알림

Test Case 2 — 일반 이메일
테스트 이메일 제목:

안녕하세요. 일반 안내입니다.

기대 결과:

Gmail 수신
   ↓
Trigger 실행
   ↓
Router
   ↓
일반 이메일 경로
   ↓
일반메일 Sheet 기록

8. 기대 효과
이 자동화를 통해 이메일을 직접 확인하고 중요한 메일을 따로 기록하는 반복 작업을 줄일 수 있다.

특히 긴급하거나 마감과 관련된 이메일을 자동으로 분류하기 때문에 중요한 업무를 놓칠 가능성을 줄일 수 있다.

9. 보안
제출용 스크린샷에는 이메일 주소, 계정 정보, API Key, 토큰, 비밀번호 등의 민감한 정보가 노출되지 않도록 확인한다.

필요한 경우 이메일 주소는 다음과 같이 마스킹한다.

example***@gmail.com

API Key나 Token이 화면에 표시되는 경우에는 반드시 *** 등으로 가린다.

프로젝트 2에서 찍을 화면
다음 5장 정도면 충분하다.

Make 전체 Scenario

Gmail
Router
Google Sheets
Gmail
Router 조건 화면

중요 이메일 조건이 보이게
중요 이메일 실행 화면

중요 경로가 실제로 실행된 것이 보이게
Google Sheets 결과

실제 데이터가 들어간 것이 보이게
Gmail 결과

실제 알림 메일이 도착한 것이 보이게
최종 제출 폴더 구조
이렇게 정리하면 깔끔하다.

📁 노코드_자동화_과제
│
├── 📄 프로젝트1_자동화도구_비교분석.md
├── 📄 프로젝트2_자동화설계_구현.md
│
├── 📁 프로젝트1_Make
│   ├── 01_Make_전체화면.png
│   ├── 02_Make_Router.png
│   ├── 03_Make_합격실행.png
│   ├── 04_Make_불합격실행.png
│   ├── 05_합격자_Sheet.png
│   └── 06_불합격자_Sheet.png
│
├── 📁 프로젝트1_Zapier
│   ├── 01_Zapier_전체화면.png
│   ├── 02_Zapier_Paths.png
│   ├── 03_Zapier_합격실행.png
│   ├── 04_Zapier_불합격실행.png
│   ├── 05_합격자_Sheet.png
│   └── 06_불합격자_Sheet.png
│
└── 📁 프로젝트2
    ├── 01_전체워크플로우.png
    ├── 02_Router조건.png
    ├── 03_실행결과.png
    ├── 04_GoogleSheets.png
    └── 05_Gmail.png

제출 전 체크리스트
 프로젝트 1에 Make 사용
 프로젝트 1에 Zapier 사용
 두 도구에서 동일한 Workflow 구현
 Trigger 1개 이상
 Action 2개 이상
 Filter/Router 1개 이상
 Make에서 합격 경로 실제 실행
 Make에서 불합격 경로 실제 실행
 Zapier에서 합격 경로 실제 실행
 Zapier에서 불합격 경로 실제 실행
 프로젝트 1 비교 항목 5개 이상
 프로젝트 2 반복 업무 정의
 프로젝트 2 도구 선정 이유 작성
 프로젝트 2 실제 Trigger 실행
 프로젝트 2 실행 결과 캡처
 API Key/Token/비밀번호 제거
 이메일 주소 마스킹
 실행 로그가 실제 실행임을 확인
