이건 **Tailwind CSS의 CLI 실행 파일이 .bin/tailwindcss로 설치되어 있을 때만 동작**해요.

그런데 지금은 **Next.js 프로젝트에서 Tailwind가 PostCSS 플러그인 형태로만 설치**되고

CLI는 **별도로 설치(tailwindcss-cli)해야만** 사용 가능한 구조가 된 경우였어요.

# 설치

npm install -D tailwindcss postcss autoprefixer

npm install -D tailwindcss-cli

  

# 초기 설정

npx tailwindcss-cli init -p


<p className="font-kkonghae text-2xl">이건 김콩해체야</p>