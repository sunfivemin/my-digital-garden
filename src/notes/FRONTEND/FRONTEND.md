---
sticker: emoji//2763-fe0f
---


#### **1) `custom-head.html` 다시 생성하기**

1. **Obsidian Vault 경로 확인**
    
    - 현재 `public` 폴더 내에 `custom-head.html`이 있어야 합니다.
    - 사라졌다면 `public` 폴더 안에서 다시 생성해야 합니다.
2. **파일 만들기**
    
    - `public` 폴더 안에서 새로운 파일을 생성하고 이름을 `custom-head.html`로 지정하세요.
3. **내용 입력 (기본 예제)**
    
    html
    
    복사편집
    
    `<!DOCTYPE html> <html lang="ko"> <head>     <meta charset="UTF-8">     <meta name="viewport" content="width=device-width, initial-scale=1.0">     <meta name="description" content="My Digital Garden">     <title>Custom Head</title>      <!-- FontAwesome CDN -->     <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">      <!-- 추가 CSS -->     <link rel="stylesheet" href="lib/styles/obsidian.css">     <link rel="stylesheet" href="lib/styles/main-styles.css">     <link rel="stylesheet" href="lib/styles/global-variable-styles.css"> </head> <body> </body> </html>`
    
4. **파일 저장**
    
    - `custom-head.html`을 `public/` 폴더 안에 배치하고 저장합니다.
5. **Git 커밋 및 배포**
    
    - 파일이 정상적으로 추가되었는지 확인 후 다음 명령을 실행합니다.
    
    sh
    
    복사편집
    
    `git add public/custom-head.html git commit -m "Add custom-head.html" git push origin main`
    

---

### **2. `obsidian.css` 수정 방법**

현재 `obsidian.css`가 한 줄로 표시되는 이유는 **압축된(minified) 파일**이기 때문입니다. 보기 쉽게 만들려면 **포맷(Beautify)** 해야 합니다.

#### **1) CSS 파일 보기 쉽게 변환 (VS Code 사용)**

- `obsidian.css` 파일을 열고 전체 선택 (`Cmd + A` / `Ctrl + A`)
- `Shift + Alt + F` (VS Code에서 코드 자동 정렬)

#### **2) 온라인 CSS Beautifier 사용**

1. CSS Beautifier 같은 사이트에 접속합니다.
2. `obsidian.css`의 내용을 복사하여 붙여넣기합니다.
3. "Beautify" 버튼을 눌러 가독성을 높인 후, 다시 저장합니다.

#### **3) Git 업데이트**

수정한 후 Git을 사용해 다시 배포합니다.

sh

복사편집

`git add public/lib/styles/obsidian.css git commit -m "Fix obsidian.css formatting" git push origin main`

---

### **3. 아이콘 관련 문제 수정 (`clickable-icon` 문제 해결)**

현재 아이콘이 이상하게 보이는 이유는 **FontAwesome이나 Obsidian 내장 아이콘이 적용되지 않았기 때문**일 수 있습니다.

#### **1) 아이콘이 깨지는 문제 해결**

- `custom-head.html`에 FontAwesome을 추가하면 해결될 가능성이 있습니다.
- 이미 추가했다면 `obsidian.css`에서 `clickable-icon` 클래스를 확인하세요.

#### **2) `clickable-icon` 클래스 수동 설정**

- `obsidian.css`에서 다음 스타일을 추가하세요.
    
    css
    
    복사편집
    
    `.clickable-icon {     font-family: "Font Awesome 6 Free";     font-weight: 900;     display: inline-block;     width: 24px;     height: 24px;     text-align: center;     line-height: 24px; } .clickable-icon.sidebar-collapse-icon:before {     content: "\f104"; /* 왼쪽 화살표 */     font-family: "Font Awesome 6 Free"; } .clickable-icon.collapse-tree-button:before {     content: "\f107"; /* 아래쪽 화살표 */     font-family: "Font Awesome 6 Free"; }`
    
- 이렇게 하면 **사이드바 열고 닫기 아이콘**과 **트리 구조 접기 아이콘**이 변경됩니다.

---

### **4. Netlify에서 커스텀 설정 유지하기**

Netlify에서 특정 파일이 사라지는 이유는 **자동 빌드 시 특정 파일을 무시하거나 덮어쓰는 문제** 때문일 가능성이 큽니다.

#### **1) `netlify.toml` 파일 수정**

Netlify 배포 시 `custom-head.html`이 유지되도록 설정합니다.

1. `netlify.toml` 파일을 열고 아래 내용을 추가하세요.

    
    `[[redirects]] from = "/" to = "/index.html" status = 200 force = true  [build] publish = "public" command = "cp custom-head.html public/custom-head.html && cp -r notes public/notes"  [[headers]] for = "/*" [headers.values] Access-Control-Allow-Origin = "*"`
    
2. `git add .` -> `git commit -m "Fix Netlify settings"` -> `git push origin main` 실행 후 Netlify에서 다시 배포하세요.