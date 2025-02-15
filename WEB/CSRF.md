## CSRF (Cross Site Request Forgery) ##
CSRF는 웹 보안 취약점 중 하나로 공격자가 사용자의 인증 정보를 악용하여 사용자가 원하지 않는 요청을 보내게 만드는 공격이다.

### CSRF 공격 조건 ###
1. 사용자가 사이트에 접속 & 로그인하여 세션을 유지함
  - 서버는 사용자의 브라우저에 세션을 저장함
2. 사용자가 악성 사이트를 방문할 때, 사이트는 사용자의 인증 정보를 이용해 로그인 되어 있는 사이트에 비정상적인 요청을 보냄
3. 요청을 받은 서버는 유효한 사용자로부터 온 요청이라 판단하고 요청을 처리함

<br />

### CSRF 방어 방법 ###
1. CSRF 토큰 사용
사용자가 사이트와 상호작용할 때, 각 요청에 대해 고유한 토큰을 생성하고 이를 검증하는 방식으로 유효한 사용자인지 확인 가능
<br />
<br />
CSRF 토큰은 서버가 브라우저에게 발급하는 임의의 문자열로 공격자는 이 토큰을 알 수 없기 때문에 CSRF 공격을 방지할 수 있음
<br />
<br />
사용자가 웹 페이지를 요청하면 서버는 고유한 CSRF 토큰을 생성해서 사용자의 세션에 저장 및 페이지의 HTML 폼이나 요청 헤더에 포함시킴
<br />
<br />
사용자는 요청을 보낼 때 CSRF 토큰을 포함시켜서 보냄 (보통 폼의 숨겨진 필드로 포함되거나 AJAX 요청 헤더에 추가됨)
<br />
<br />
서버는 요청이 오면 CSRF 토큰을 추출해서 서버의 세션 저장소에 저장된 토큰과 비교하여 검증함
<br />
<br />
스프링 시큐리티는 기본으로 CSRF 옵션을 제공해줌

    ```java
    @Bean
        public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
            http
                .csrf()
                ...
        }
    ```

<br />
<br />

2. SameSite 쿠키 속성 설정
SameSite 속성은 쿠키가 언제 어떤 상황에서 전송될지를 제어하며 외부 사이트에서의 요청에 쿠키가 포함되지 않도록 함
<br />
<br />
SameSite 속성에는 Strict, Lax, None 세 가지가 있음
<br />
<br />
<b>Strict</b>는 쿠키가 오직 같은 사이트에서의 요청만 전송됨
<br />
외부 사이트에서의 요청에는 쿠키가 포함되지 않으므로 CSRF 공격을 방어할 수 있음
<br />
<br />
<b>Lax</b>는 기본 값으로 안전한 HTTP 메서드 요청에 대해서만 쿠키가 전송됨
<br />
다만, 링크 클릭과 같은 경우에는 외부 사이트에서도 쿠키가 포함될 수 있음
<br />
기본적인 CSRF 공격 방어
<br />
<br />
<b>None</b>은 모든 요청에 대해 쿠키가 전송됨
<br />
CSRF 공격에 대한 방어가 없음
<br />

    ```java
    @GetMapping("/test")
    public ResponseEntity<String> setCookie(HttpServletResponse response) {
        Cookie cookie = new Cookie("name", "value");

        cookie.setHttpOnly(true);
        cookie.setSecure(true);
        cookie.setPath("/");
        cookie.setSameSite("Strict");
        response.addCookie(cookie);

        return ResponseEntity.ok("쿠키 SameSite 설정");
    }

    ```
<br />
<br />

3. Referer 검증
HTTP 요청의 Referer 헤더는 현재 요청이 발생한 페이지의 URL을 포함하므로 요청이 어디서 왔는지 알 수 있음
<br />
<br />
서버는 요청의 Referer 헤더 값을 검사하여 허용된 도메인에서 왔는지 검증하고 유효하지 않은 도메인에서 온 요청이라면 거부함
<br />
<br />

    ```java
    @PostMapping("/test")
    public ResponseEntity<String> checkReferer(HttpServletRequest request) {
        // HTTP 요청의 Referer 헤더
        String referer = request.getHeader("Referer");

        // 허용된 도메인
        String allowedDomain = "https://도메인";

        // Referer 헤더 검증
        if (!referer.startsWith(allowedDomain)) {
            // "Invalid Referer" 예외
        }

        return ResponseEntity.ok("요청 수행");
    }
    ```