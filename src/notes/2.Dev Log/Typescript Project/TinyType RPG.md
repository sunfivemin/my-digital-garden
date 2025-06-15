# **TinyType RPG 🎮**

> 타입스크립트 OOP 실습 프로젝트 - 버튼을 누르면 누군가 맞습니다.

---

## 👾 프로젝트 소개

**TinyType RPG**는 타입스크립트의 클래스, 상속, 인터페이스, 모듈 시스템을 활용하여 만든 초간단 턴제 RPG 전투 시뮬레이터입니다. 플레이어가 버튼을 눌러 적을 공격하고, 로그가 UI에 출력되는 구조입니다.

본 프로젝트는 스터디용으로 개발되었으며, 타입스크립트의 기초부터 클래스 기반 OOP 실습에 집중했습니다.

---

## 🧑‍💻 팀 역할 분담

|**팀원**|**담당 시스템**|**역할 설명**|
|---|---|---|
|A|캐릭터 시스템|캐릭터 클래스 설계, 상태 관리, 데미지/힐 처리|
|B|전투 시스템|턴 관리, 전투 흐름, 플레이어 액션 처리|
|C|스킬/아이템 시스템|스킬/아이템 인터페이스 정의 및 실제 사용 로직 구현|
|E|AI 행동 시스템|적 AI 랜덤 행동, 독침/힐/버프/분노 행동 설계 및 구현|
|D|UI/게임 상태 관리 (공통)|UI 버튼, HP 바, 로그 제어, UI 렌더링, 게임 흐름 연결|

---

## 📂 폴더 구조

```
src/
├── ai/
│   └── EnemyAI.ts            # E (AI 행동 시스템)
├── battle/
│   └── BattleManager.ts      # B (전투 시스템)
├── character/
│   ├── Character.ts          # A (캐릭터 시스템)
│   ├── Player.ts             # A
│   └── Enemy.ts              # A
├── skill/
│   ├── Skill.ts              # C (스킬 시스템)
│   └── Fireball.ts           # C
├── item/
│   ├── Item.ts               # C (아이템 시스템)
│   └── HealthPotion.ts       # C
├── ui/
│   └── UIManager.ts          # D (UI 시스템)
├── main.ts                   # D (게임 흐름 연결)
├── index.html                # D
└── style.css                 # D
```

---

## 💡 역할 배분 기준

- 시스템 기반으로 캐릭터, 전투, 스킬/아이템, AI 행동, UI를 명확히 분리하여 협업
- UI 및 게임 흐름(main.ts)은 전체 팀원 리뷰로 협업
- AI의 랜덤 행동 패턴은 E가 전담하여 확장성과 난이도 조절 가능

---

## ✅ A (캐릭터 시스템 담당)

- **책임:** 캐릭터의 모든 생명 관리 기능 담당 (HP, 공격, 데미지, 힐)
- **구현:**
    - Character.ts: 기본 캐릭터 속성과 데미지 처리 (takeDamage, isAlive)
    - Player.ts: 힐 기능 구현 (heal())
    - Enemy.ts: Character 상속만
- **목표:** 모든 생명체의 HP, 공격, 생존 여부는 이곳에서만 관리
- A는 캐릭터 시스템 담당으로, 전투에 등장하는 모든 생명체의 상태 관리와 데미지 처리, 플레이어만의 힐 기능을 구현했습니다.
---

## ✅ B (전투 시스템 담당)
- **책임:** 턴 흐름 및 플레이어 액션 관리
- **구현:**
    - BattleManager.ts: 공격, 힐, 스킬, 아이템 액션 처리
    - 턴 관리 메서드: getCurrentTurn, startTurnLoop, stopTurnLoop
    - AI는 EnemyAI.ts를 호출만 함
- **특이사항:**
    - 플레이어/적의 속도에 따라 턴 게이지 방식 도입 (by 찬솔)
    - speed, currentTurnPoint, increaseTurnPoint, isTurn, setTurnChangedCallback 등 메서드 추가
- B는 전투 시스템을 맡아 플레이어의 액션, 턴 게이지 기반 턴 흐름, AI 호출 등을 담당했습니다.

---

## ✅ C (스킬/아이템 시스템 담당)

- **책임:** 스킬 및 아이템 기능 정의 및 구현
- **구현:**
    - Skill.ts: 스킬 인터페이스
    - Fireball.ts: 파이어볼 스킬 구현
    - Item.ts: 아이템 인터페이스
    - HealthPotion.ts: 회복 포션 구현
- **목표:** BattleManager는 타입만 보고 스킬/아이템 사용 가능
- C는 스킬과 아이템 시스템을 맡아 인터페이스 정의와 실제 로직을 구현했습니다.

---

## ✅ E (AI 행동 시스템 담당)

- **책임:** 슬라임의 랜덤 AI 행동 정의 및 강화
- **기존 스킬:** 힐, 독침 공격
- **추가 스킬:** 더블 어택, 방어 상승, 분노 (체력 10 미만시 발동)
- **구현:**
    - EnemyAI.ts에서 decideAction 메서드 통해 랜덤 행동 결정
    - poisonAttack, doubleAttack, increaseDefense, struggle() 등 독립 로직 설계
    - 슬라임의 능력치 강화 (버프 효과, 스킬 확률 향상)
- E는 적 슬라임의 행동을 정의하는 EnemyAI.ts를 전담했고, 독침·힐·더블어택·분노 등 스킬을 설계했습니다.

---

## ✅ D (UI/게임 흐름 관리 - 공통)

- **책임:** UI 및 게임 흐름 관리 (main.ts)
- **구현:**
    - UIManager.ts: 버튼, 로그, HP 바 렌더링 관리
    - main.ts: UIManager, BattleManager 연결만 담당
    - index.html, style.css: UI 구성 및 스타일링
- D는 UI와 게임 흐름을 맡아 UIManager.ts에서 렌더링을, main.ts에서는 흐름 연결을 전담했습니다.

---

## 🧩 주요 기술 스택

- TypeScript (ES6 모듈 시스템)
- OOP 구조화 (클래스, 상속, 인터페이스)
- Parcel (빠른 번들링 및 핫리로드 개발 서버)
- HTML/CSS 기반 UI
- 모듈별 파일 분리로 구조 명확화

---

## 🚀 실행 방법

```bash
npm install     # 의존성 설치
npm run dev     # 개발 서버 실행
```

---

## 💬 최종 정리 요약

- **A는 생명체 시스템 관리만.** (HP, 데미지, 힐)
- **B는 전투 흐름만.** (턴 게이지, 액션만 담당. AI는 호출만)
- **C는 아이템/스킬 시스템만.** (표준 인터페이스 기반으로)
- **E는 적 AI 결정만.** (슬라임 행동 독립 구현)
- **D는 UI와 게임 흐름 연결만.** (UIManager와 main.ts 중심 구조)

---

> “저희는 시스템별로 역할을 명확히 나누었습니다
> A는 캐릭터 시스템을, B는 전투 시스템을, C는 아이템/스킬 시스템을
> E는 적 AI 시스템을, D는 UI와 흐름 연결을 전담했습니다.”


## 🔗 더 자세한 내용은?

이 프로젝트의 **초기 세팅은 [sunfivemin/tinytype-rpg](https://github.com/sunfivemin/tinytype-rpg)** 저장소에서 시작되었으며,  
**본격적인 기능 구현과 시스템 개발은 [tslearners/typeScriptStudy](https://github.com/chan8919/typeScriptStudy)** 레포지토리에서 이어졌습니다.

👉 초기 저장소: [https://github.com/sunfivemin/tinytype-rpg](https://github.com/sunfivemin/tinytype-rpg)  
👉 전체 구현: [https://github.com/chan8919/typeScriptStudy](https://github.com/chan8919/typeScriptStudy)
