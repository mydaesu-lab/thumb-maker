# 썸네일 10초 — 블로그·인스타·유튜브 썸네일 생성기

브라우저에서 10초 만에 썸네일을 만들고 원본 해상도 PNG로 저장하는 **단일 파일 웹앱**입니다.
서버가 없고, 업로드한 이미지와 입력한 문구는 기기 밖으로 나가지 않습니다.

- 파일: `index.html` (이 파일 하나로 동작)
- 외부 의존: Google Fonts + jsDelivr Pretendard CSS (글꼴만 CDN)
- 빌드 도구·프레임워크·백엔드: 없음

---

## 1. 기능

| 구분 | 내용 |
| --- | --- |
| 플랫폼 프리셋 | 네이버 블로그·인스타 정사각 1080×1080 / 유튜브 1280×720 / 인스타 카드뉴스 1080×1350 / OG·티스토리 1200×630 |
| 배경 | 그라디언트 16종 + 단색 8종, 그라디언트 각도, 랜덤 배경, 배경 이미지 업로드(cover 크롭·흐림·어둡게) |
| 글자 | 상단 배지(켜기/끄기), 제목(자동 줄바꿈·자동 크기 축소), 하단 문구·워터마크, 글자색·포인트색 |
| 글꼴 | Pretendard, Noto Sans KR, Black Han Sans, Jua, Do Hyeon, Noto Serif KR |
| 스타일 | 가로 정렬, 세로 위치, 글자 그림자, 형광펜 강조 |
| 내보내기 | 원본 해상도 PNG 다운로드, 클립보드 복사(지원 브라우저) |
| 기타 | 빠른 스타일 5종, 작업 내용 자동 저장(localStorage), 광고 영역 자리(숨기기 가능) |
| 검색·공유 | title·description·keywords, Open Graph·X 카드, JSON-LD(WebApplication + FAQPage), 사이트맵·robots.txt |
| 본문 콘텐츠 | 편집기 아래 “클릭률 높이는 썸네일 제작 팁 4가지” · “자주 묻는 질문 6개” · 서비스 소개·개인정보처리방침 모달 |

### 렌더링 방식
미리보기와 결과물이 **같은 `<canvas>`** 입니다. 캔버스 자체를 1080×1080 같은 실제 출력 해상도로 두고 CSS로만 줄여 보여주기 때문에,
html2canvas 같은 변환 라이브러리 없이 화면에서 본 그대로 고해상도 PNG가 나옵니다.

- 줄바꿈: 공백 기준 + 한 단어가 폭을 넘으면 글자 단위로 분할 (한글 대응)
- 자동 축소: 지정한 크기로 넘칠 때 6%씩 줄여 캔버스 안에 들어오게 함 (미리보기에 안내 표시)
- 흐림: 축소→확대 방식이라 `ctx.filter`를 지원하지 않는 브라우저에서도 동일하게 동작
- 글꼴: `document.fonts.load()` + `document.fonts.ready` 후 다시 그려 폭 측정 오차를 보정
- 형광펜: 글자색과 포인트색 대비가 낮으면 글자색을 자동으로 맞추고 토스트로 알림

---

## 2. 실행

```bash
# 방법 1: 그냥 열기
open index.html            # macOS
# 방법 2: 로컬 서버 (클립보드 복사는 https/localhost에서만 동작)
python3 -m http.server 8000
# → http://localhost:8000
```

`file://`로 열면 클립보드 이미지 복사가 브라우저 정책상 막힐 수 있습니다. PNG 다운로드는 항상 동작합니다.

---

## 3. 배포 (무료 호스팅)

### GitHub Pages (현재 배포됨)

- 저장소: https://github.com/mydaesu-lab/thumb-maker
- 사이트: https://mydaesu-lab.github.io/thumb-maker/
- 설정: `main` 브랜치 / 루트(`/`)에서 배포 (Deploy from a branch)

수정한 뒤 다시 올리는 방법:
```bash
git add . && git commit -m "update" && git push origin main
```

### Cloudflare Pages
대시보드 → Workers & Pages → Create → Pages → Connect to Git → 저장소 선택 → Build command 없음, Output directory `/` → Deploy.

### Vercel
`vercel` CLI 실행 후 프레임워크 없음(Other), 빌드 명령 비워두기, 출력 디렉터리 `.`.

> 커스텀 도메인을 붙이면 AdSense 승인과 공유에 유리합니다.

### 배포 직후 할 일 (검색 등록)

1. 주소는 `https://mydaesu-lab.github.io/thumb-maker/` 로 치환해 두었습니다 (`index.html`·`robots.txt`·`sitemap.xml`).
2. `index.html` `<head>` 의 `canonical` · `og:url` 은 주소가 정해져 주석을 풀어 두었습니다.
   `google-site-verification` · `naver-site-verification` 은 콘솔에서 코드를 받은 뒤 주석을 풀고 채웁니다.
3. **네이버 서치 어드바이저** — 사이트 등록 → 소유 확인(메타태그 또는 HTML 파일) → 사이트맵 제출
   (`sitemap.xml`) → 웹페이지 수집 요청.
4. **Google Search Console** — 속성 추가(URL 접두어) → 소유 확인 → Sitemaps 에 `sitemap.xml` 제출 →
   URL 검사 → 색인 생성 요청.
5. 공유 미리보기 확인: 카카오톡 디버거(`https://developers.kakao.com/tool/debugger/sharing`),
   페이스북 공유 디버거에서 `og:title`·`og:image` 가 정상으로 나오는지 봅니다.
   (`og:image` 는 지금 `./preview.png` 상대 경로입니다. 미리보기가 안 나오면 절대 주소로 바꾸세요.)

> **robots.txt 위치 주의**: robots.txt 는 도메인 **최상위**에서만 읽힙니다. GitHub Pages 프로젝트
> 페이지(`<계정>.github.io/<저장소>/`)라면 저장소 안의 `robots.txt` 는 검색엔진이 요청하지 않습니다.
> 이때는 sitemap.xml 을 각 콘솔에서 직접 제출하면 되고, robots.txt 가 필요하면
> `<계정>.github.io` 저장소 루트에 두어야 합니다. 커스텀 도메인을 붙였다면 그 도메인 루트에 두면 됩니다.

---

## 4. 수익화 자리

`index.html`에서 `<!-- 광고 자리 ... -->` 주석과 `AD 광고 영역` 카드가 광고 위치입니다.

1. AdSense 승인 후 `index.html`의 아래 부분을 실제 광고 단위로 교체합니다.
   ```html
   <div class="ad-slot">
     <!-- 여기에 AdSense 광고 단위 스크립트를 붙여넣기 -->
   </div>
   ```
2. `<head>`에 AdSense 로더 스크립트를 추가합니다.
   ```html
   <script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-XXXXXXXXXXXXXXXX" crossorigin="anonymous"></script>
   ```
3. 광고 영역은 `숨기기` 버튼으로 접히므로 UX를 해치지 않습니다. 모바일에서는 미리보기 아래에만 노출됩니다.

승인 팁: 도구만 있는 사이트는 콘텐츠가 부족하다고 판단될 수 있습니다. 이 버전에는 이미
편집기 아래에 **팁 4가지 · FAQ 6개 · 서비스 소개 · 개인정보처리방침**이 들어 있어, 검토자가 볼 수 있는
본문과 고지가 갖춰져 있습니다.

- 개인정보처리방침은 푸터의 `개인정보처리방침` 버튼으로 열리며, `index.html#privacy` 주소로도 바로 열립니다.
  AdSense 의 ‘개인정보처리방침 URL’ 칸에는 `https://<주소>/#privacy` 를 넣을 수 있습니다.
- 서비스 소개는 `#about` 입니다.
- 심사에서 별도 페이지를 요구할 경우, 같은 내용으로 `privacy.html`·`about.html` 을 만들어 푸터 링크를
  그쪽으로 바꾸면 됩니다(내용은 모달에서 그대로 복사).
- 사이트 소개 모달의 `<!-- 배포 시: 운영자 연락처 -->` 자리에 이메일을 적어 두면 심사·문의에 도움이 됩니다.
- 광고가 붙은 뒤에도 **편집기·미리보기 위에는 광고를 넣지 마세요.** AdSense 정책상 콘텐츠와 구분되지 않는
  위치의 광고는 위험합니다. 지금 자리(미리보기 아래 카드)를 그대로 쓰면 됩니다.

추가 수익화 아이디어(코드에 자리만 잡아두면 됨):
- Pro 템플릿 팩: `STYLES` 배열에 항목을 추가하고 일부를 잠금 처리
- 제휴 링크: 무료 이미지 사이트(Unsplash·Pexels), 폰트 판매처, Canva 등
- "이 썸네일로 블로그 쓰기" 제휴 링크

---

## 5. 커스터마이징

| 무엇을 | 어디를 |
| --- | --- |
| 배경 프리셋 추가 | `GRADIENTS` / `SOLIDS` 배열에 `{ id, name, colors: [...] }` 추가 |
| 글꼴 추가 | `<head>`의 Google Fonts 링크에 family 추가 + `FONTS`에 항목 추가 |
| 플랫폼 프리셋 추가 | `PRESETS`에 `{ w, h }` 추가 + `PRESET_ORDER`에 키 추가 |
| 빠른 스타일 | `STYLES` 배열 (배경·글꼴·색·정렬·크기 묶음) |
| 기본 문구 | `DEFAULTS` 객체 |
| 여백 비율 | `buildLayout()`의 `padX / padTop / padBottom` (캔버스 대비 비율) |

색상·간격 등 UI 톤은 `<style>` 상단 `:root` 변수에서 바꿉니다.

---

## 6. 알려진 제한

- **클립보드 이미지 복사**: `https` 또는 `localhost`에서만 허용됩니다(브라우저 보안 정책). 미지원 시 안내 토스트가 뜨고 다운로드를 쓰면 됩니다.
- **글꼴 CDN 차단 환경**(사내망 등): 시스템 글꼴로 대체되어 줄바꿈 위치가 조금 달라질 수 있습니다.
- 아주 큰 사진(수천만 화소)을 올리면 브라우저 메모리를 많이 씁니다. 4K 이하를 권장.
- 저장은 브라우저 `localStorage`에만 됩니다(기기·브라우저별). 시크릿 모드에서는 저장되지 않습니다.

---

## 7. 파일

```
index.html   앱 전체 (HTML + CSS + JS, 약 92KB) — 편집기 · 미리보기 · 광고 자리 · 팁/FAQ · 푸터/모달
README.md    이 문서
preview.png  기본 설정 결과 예시 (720×720) — og:image 로도 사용
robots.txt   검색 로봇 안내 (도메인 최상위에서만 인식)
sitemap.xml  사이트맵 (배포 주소 반영됨)
```
