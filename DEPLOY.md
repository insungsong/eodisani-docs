# 어디사니 문서 배포 가이드

## 개요
이 저장소는 GitHub Pages를 통해 `https://insungsong.github.io/eodisani-docs/` 에 자동으로 배포됩니다.

## 배포 방법

### 1. 파일 수정
원하는 HTML 파일을 수정합니다.
```bash
# 예: marketing.html 수정
vim marketing.html
```

### 2. 변경사항 확인
로컬에서 변경사항을 확인하려면 브라우저로 HTML 파일을 직접 열거나, 간단한 웹 서버를 실행합니다.

```bash
# Python 3로 로컬 서버 실행
python3 -m http.server 8000

# 브라우저에서 확인
# http://localhost:8000/marketing.html
```

### 3. Git에 커밋
```bash
# 변경된 파일 확인
git status

# 파일 추가
git add marketing.html  # 또는 수정한 파일명

# 커밋
git commit -m "feat: 변경사항 설명"
```

### 4. GitHub에 푸시
```bash
# main 브랜치에 푸시
git push origin main
```

### 5. 배포 확인
푸시 후 1-2분 정도 기다리면 GitHub Pages가 자동으로 업데이트됩니다.
- 배포 페이지: https://insungsong.github.io/eodisani-docs/marketing.html
- GitHub Actions에서 배포 상태 확인: https://github.com/insungsong/eodisani-docs/actions

## 주요 페이지

- **마케팅 페이지**: `marketing.html`
  - URL: https://insungsong.github.io/eodisani-docs/marketing.html
  - 어디사니 앱 소개 및 다운로드 페이지

- **개인정보 처리방침**: `privacy.html`
  - URL: https://insungsong.github.io/eodisani-docs/privacy.html

- **이용약관**: `terms.html`
  - URL: https://insungsong.github.io/eodisani-docs/terms.html

- **고객지원**: `support.html`
  - URL: https://insungsong.github.io/eodisani-docs/support.html

- **문의하기**: `contact.html`
  - URL: https://insungsong.github.io/eodisani-docs/contact.html

## 원스텝 배포 스크립트

빠르게 배포하려면 아래 명령어를 사용하세요:

```bash
# 모든 변경사항 추가, 커밋, 푸시
git add .
git commit -m "feat: 문서 업데이트"
git push origin main
```

또는 한 줄로:

```bash
git add . && git commit -m "feat: 문서 업데이트" && git push origin main
```

## 롤백 방법

실수로 잘못된 내용을 배포한 경우:

```bash
# 이전 커밋으로 되돌리기
git log --oneline  # 커밋 히스토리 확인
git revert <commit-hash>  # 특정 커밋 되돌리기
git push origin main

# 또는 강제로 이전 버전으로 되돌리기 (주의!)
git reset --hard <commit-hash>
git push -f origin main
```

## 트러블슈팅

### 배포가 안 될 때
1. GitHub Actions 탭에서 에러 확인
2. GitHub Pages 설정 확인:
   - Settings > Pages > Source: "Deploy from a branch"
   - Branch: "main" 또는 "gh-pages"

### 캐시 문제
브라우저 캐시 때문에 업데이트가 안 보일 수 있습니다:
- Chrome/Edge: `Ctrl+Shift+R` (Mac: `Cmd+Shift+R`)
- Safari: `Cmd+Option+R`

## GitHub Pages 설정

이 저장소는 이미 GitHub Pages가 설정되어 있습니다. 
새 저장소에서 설정이 필요한 경우:

1. GitHub 저장소 > Settings
2. 왼쪽 메뉴에서 "Pages" 선택
3. Source: "Deploy from a branch" 선택
4. Branch: "main" 선택, 폴더: "/ (root)" 선택
5. Save 클릭

## 주의사항

- `main` 브랜치에 푸시하면 자동으로 배포되므로 신중하게 푸시하세요
- 민감한 정보(API 키, 비밀번호 등)는 절대 커밋하지 마세요
- HTML 파일에 문법 오류가 있으면 제대로 표시되지 않을 수 있습니다
- 이미지나 리소스 파일도 함께 커밋해야 합니다

## 연락처

문제가 있거나 질문이 있으면 저장소 이슈를 생성하세요.
