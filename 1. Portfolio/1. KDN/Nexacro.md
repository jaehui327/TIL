# 📘 Nexacro

`넥사크로(Nexacro)`는 기업용 UI 프레임워크로, HTML5 기반 웹 애플리케이션을 **윈도우 애플리케이션처럼 구성**할 수 있게 도와주는 툴입니다.  
주로 공공기관, 금융권 등의 내부 시스템에서 사용됩니다.

---

## 🔧 기본 구조

넥사크로 프로젝트는 다음과 같은 주요 컴포넌트로 구성됩니다:

- **Form (.xfdl)**: 화면 단위 구성요소
- **Script (.xjs)**: 자바스크립트 로직 정의
- **Dataset**: 화면-서버 간 데이터 송수신 객체
- **Component**: UI 구성 요소 (Button, Grid 등)
- **Global Variables**: 전역 데이터 보관 영역

---

## 🗂 프로젝트 구조 예시

```
/nexacro_project/
│
├── main.xfdl             # 메인 화면
├── login.xfdl            # 로그인 화면
├── common/
│   ├── lib_script.xjs    # 공통 스크립트
│   └── config.xjs        # 전역 변수, 환경 설정
├── theme/
│   └── default.xtheme    # 테마 파일
└── service/
    └── form_data.xml     # 데이터 정의 파일
```

---

## 📱 Form 개발 기본

### 폼에서 사용되는 주요 함수

```javascript
// on_init: 초기 변수 선언 및 이벤트 연결
this.on_init = function (obj, e) {
  this.addEventHandler("onload", this.form_onload, this);
};

// onload: 화면 로드 시 실행
this.form_onload = function (obj, e) {
  trace("화면 로드됨");
};

// onclose: 폼 종료 시 실행
this.form_onclose = function (obj, e) {
  trace("폼 종료됨");
};
```

### 이벤트 연결 방법

```javascript
this.addEventHandler("onclick", this.btn_click, this);
```

또는 디자인 화면 우측 이벤트 탭에서 매핑 가능

---

## 📦 Dataset 사용

### 1. 생성 및 바인딩

```javascript
// 생성
var dsUser = new Dataset("dsUser", this);
this.addChild("dsUser", dsUser);

// 바인딩
this.grid01.set_binddataset("dsUser");
```

### 2. 서버 통신 연동

```javascript
this.transaction(
  "getUserInfo", // svcID
  "/user/info", // URL
  "", // input
  "dsUser=out_ds", // output
  "param1=value1", // arguments
  "fnCallback" // callback function
);
```

---

## 🔄 서버 연동 구조

### 요청 흐름

```
[Component] ↔ [Dataset] ↔ [Transaction] ↔ [Backend]
```

서버는 대부분 Nexacro 전용 Gateway 혹은 Spring Controller와 연결됩니다.  
Nexacro → JSON 또는 XML → BFF → DB 로 이어지는 구조가 일반적입니다.

---

## 🎨 주요 컴포넌트 활용

### Grid

```xml
<Grid id="grdUser" binddataset="dsUser" autoenter="select" selecttype="row">
    <Formats>
        <Format id="default">
            <Columns>
                <Column size="100"/>
                <Column size="150"/>
            </Columns>
            <Rows>
                <Row size="24"/>
            </Rows>
            <Band id="body">
                <Cell col="0" text="bind:USER_ID"/>
                <Cell col="1" text="bind:USER_NAME"/>
            </Band>
        </Format>
    </Formats>
</Grid>
```

### Button

```xml
<Button id="btnSearch" text="조회" onclick="btnSearch_onclick"/>
```

---

## 🧠 팁 & 베스트프랙티스

| 항목                | 설명                                                 |
| ------------------- | ---------------------------------------------------- |
| Dataset 초기화      | `this.ds.clearData();`                               |
| trace 디버깅        | `trace("디버깅 로그");`                              |
| 공통 처리 함수화    | `lib_script.xjs`에 `gfn` prefix로 공통 함수 등록     |
| 글로벌 변수 접근    | `application.gdsUserInfo.getColumn(0, "user_id")`    |
| 팝업 호출           | `this.open("popup", "popup::PopupForm.xfdl", args);` |
| Component 동적 제어 | `this["btnSearch"].set_text("다시 조회");`           |

---

## 🧪 디버깅 방법

- **trace()**: F12 개발자 도구 콘솔에서 로그 출력
- **브레이크 포인트**: Nexacro Studio 디버그 모드
- **Form Preview**: 스튜디오 내 [실행 > 미리보기]

---

## 📚 확장 개념

| 개념            | 설명                                    |
| --------------- | --------------------------------------- |
| MDI             | 다중 문서 인터페이스 (탭 기반 UI)       |
| ChildFrame      | 팝업 또는 탭 내 화면 구성               |
| 전역 Dataset    | 사용자 정보, 설정 등 모든 화면에서 참조 |
| Theme/Style     | CSS처럼 공통 UI 스타일 지정 가능        |
| External Script | 공통 스크립트를 외부 .xjs로 분리 관리   |

---

## 📌 결론

Nexacro는 진입 장벽이 있지만, 구조화된 개발과 대용량 업무 UI에 적합한 프레임워크입니다.  
이 문서를 바탕으로 `Form`, `Dataset`, `Component`, `Transaction` 등의 개념을 잘 이해하고, 재사용 가능한 공통 함수와 구성 방식으로 효율적인 업무 화면을 구성할 수 있습니다.
