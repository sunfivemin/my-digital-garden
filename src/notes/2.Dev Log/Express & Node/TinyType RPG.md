# TinyType RPG 🎮

타입스크립트 OOP 실습 프로젝트 버튼을 누르면 누군가 맞습니다.

---

## 👾 프로젝트 소개
  
**TinyType RPG**는 타입스크립트의 클래스, 상속, 인터페이스, 모듈 시스템을 활용하여 만든
초간단 턴제 RPG 전투 시뮬레이터입니다.
플레이어가 버튼을 눌러 적을 공격하고, 로그가 UI에 출력되는 구조입니다.

본 프로젝트는 스터디용으로 개발되었으며,
타입스크립트의 기초부터 클래스 기반 OOP 실습에 집중했습니다.


---

## 🧑‍💻 팀 역할 분담

| 팀원 | 담당 시스템 | 코드 책임 예시 | 역할 설명 |

| ---- | ------------------------ | ----------------------------------------------------------------------------- | ---------------------------------------------------------------- |

| A | 캐릭터 시스템 | `character/Character.ts`, `character/Player.ts`, `character/Enemy.ts` | 캐릭터 클래스 설계, 상태 관리, 데미지/힐 처리 |

| B | 전투 시스템 | `battle/BattleManager.ts` | 턴 관리, 전투 흐름, 플레이어 액션 처리 |

| C | 스킬/아이템 시스템 | `skill/Skill.ts`, `skill/Fireball.ts`, `item/Item.ts`, `item/HealthPotion.ts` | 스킬/아이템 인터페이스 정의 및 실제 사용 로직 |

| E | AI 행동 시스템 | `ai/EnemyAI.ts` | 적 AI 랜덤 행동, 독침/힐 등 행동 패턴 설계 및 구현 |

| D | UI/게임 상태 관리 (공통) | `ui/UIManager.ts`, `main.ts`, `index.html`, `style.css` | UI 버튼, HP 바, 로그 제어, UI 렌더링, 게임 흐름 연결 (공통 협업) |

  

---

## 📂 폴더 구조

```plaintext

src/
├── ai/
│ └── EnemyAI.ts # E (AI 행동 시스템)
├── battle/
│ └── BattleManager.ts # B (전투 시스템)
├── character/
│ ├── Character.ts # A (캐릭터 시스템)
│ ├── Player.ts # A
│ └── Enemy.ts # A
├── skill/
│ ├── Skill.ts # C (스킬 시스템)
│ └── Fireball.ts # C
├── item/
│ ├── Item.ts # C (아이템 시스템)
│ └── HealthPotion.ts # C
├── ui/
│ └── UIManager.ts # D (UI 시스템)
├── main.ts # D (게임 흐름 연결)
├── index.html # D
└── style.css # D

```

  

## 💡 역할 배분 기준

- 시스템 기반으로 캐릭터, 전투, 스킬/아이템, AI 행동, UI를 명확히 분리하여 효율적인 협업 및 발표 준비
- UI 및 게임 흐름(`main.ts`)은 전체 팀원 리뷰 및 협업 (공통 작업으로 진행)
- 적 AI 랜덤 행동 패턴(`enemyAction`)은 E가 전담하여 확장성과 난이도 조절 가능

---

## 🧩 주요 기술

- **TypeScript (ES6 Module)**
- **OOP 구조화 (클래스, 상속, 인터페이스)**
- **Parcel (빠른 번들링 & 개발 서버)**
- **HTML + CSS (간단 UI)**
- **모듈 기반 구조 관리**


---

## 🚀 실행 방법
```bash
# 의존성 설치
npm install

# 개발 서버 실행
npm run dev
```



| **팀원**                | **해야 할 일 (구체적)**                                                                                                                        | **담당 파일**                                                             |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| A (캐릭터 시스템)           | Character.ts에서 **공통 캐릭터 로직 구현**Player.ts에서 **힐 기능 구현 (heal 메서드)**Enemy.ts는 Character 상속만                                                | Character.ts, Player.ts, Enemy.ts                                     |
| B (전투 시스템)            | BattleManager.ts에서 **플레이어 액션 관리 (playerAction())**턴 관리, 전투 흐름만 전담AI는 **E가 만든 EnemyAI.ts 호출만**                                           | battle/BattleManager.ts                                               |
| C (스킬/아이템 시스템)        | Skill.ts 인터페이스 정의Fireball.ts 스킬 클래스 구현Item.ts 인터페이스 정의HealthPotion.ts 아이템 클래스 구현                                                        | skill/Skill.ts, skill/Fireball.ts, item/Item.ts, item/HealthPotion.ts |
| E (AI 행동 시스템)         | EnemyAI.ts 생성적 AI 패턴 (decideAction()) 독립 구현랜덤 평타/독침/힐 로직 설계**BattleManager에서는 호출만 하게 구조 유지**                                            | ai/EnemyAI.ts                                                         |
| D (UI 시스템 & 게임 흐름 공통) | UIManager.ts에서 **버튼 상태, HP 바, 로그 관리 전담**main.ts에서는 **게임 흐름만 연결 (UIManager, BattleManager 호출만)**index.html, style.css로 **UI 뼈대와 스타일 관리** | ui/UIManager.ts, main.ts, index.html, style.css                       |
> “저희는 시스템별로 역할을 명확히 나누었습니다.
> A는 캐릭터 시스템을 담당하여 기본 캐릭터와 플레이어의 힐 기능을 구현,
> B는 전투 시스템으로 BattleManager를 관리하여 플레이어 액션과 턴 흐름 담당,
> C는 스킬과 아이템 시스템을 전담하여 스킬과 아이템의 인터페이스 및 실제 로직 개발,

> E는 AI 행동 시스템으로 EnemyAI.ts를 따로 만들어 슬라임의 행동을 랜덤 패턴으로 구현했고,
> D는 UI 시스템과 게임 흐름을 관리하여 UIManager.ts로 UI를 전담,
> main.ts는 BattleManager, UIManager를 연결하는 흐름만 담당했습니다.”



---

## **✅ A (캐릭터 시스템 담당)**
- **책임:** 캐릭터의 모든 생명 관리 기능 담당 (HP, 공격, 데미지, 힐)
- **구현:**
    - Character.ts: 기본 캐릭터 속성과 데미지 처리 (takeDamage(), isAlive())
    - Player.ts: 플레이어 전용 힐 기능 (heal())
    - Enemy.ts: Character 상속만
- **목표:** 플레이어나 슬라임의 HP, 공격, 생존 여부를 여기서만 관리
- **발표 대사:** “A는 캐릭터 시스템 담당으로, 전투에 등장하는 모든 생명체의 상태 관리와 데미지 처리, 플레이어만의 힐 기능을 구현했습니다.”

---
## **✅ B (전투 시스템 담당)**

- **책임:** 플레이어 액션 관리, 턴 관리, 전투 흐름만 전담
- **구현:**
    - BattleManager.ts: 플레이어의 공격, 힐, 스킬, 아이템 액션 처리
    - 턴 관리 (playerAction(), getCurrentTurn(), isBattleOver())
    - AI 행동은 **E가 만든 EnemyAI.ts만 호출**
- **목표:** 플레이어가 한 행동만 처리, 적 AI는 E 호출만
- **발표 대사:** “B는 전투 시스템을 맡아서 플레이어의 액션과 턴 관리, 전투 흐름을 담당했습니다. 적 AI 로직은 E가 만든 EnemyAI.ts를 호출만 합니다.”

---

## **✅ C (스킬/아이템 시스템 담당)**

- **책임:** 스킬/아이템 인터페이스 정의 및 구체 클래스 구현
- **구현:**
    - Skill.ts: 스킬 인터페이스
    - Fireball.ts: 파이어볼 스킬
    - Item.ts: 아이템 인터페이스
    - HealthPotion.ts: 회복 포션
- **목표:** BattleManager가 Skill, Item 타입만 보고 사용할 수 있게끔 표준화
- **발표 대사:** C는 스킬과 아이템 시스템을 맡아서 인터페이스를 정의하고, Fireball과 HealthPotion 클래스를 구현했습니다.

---

## **✅ E (AI 행동 시스템 담당)**
- **책임:** 적 AI 행동만 단독 관리 (랜덤 행동 패턴)
- **구현:**
    - EnemyAI.ts: 슬라임의 랜덤 행동 결정 (decideAction())
    - 독침, 힐, 평타 판단만 담당
    - BattleManager는 이걸 호출만 하게 설계
- **목표:** BattleManager는 적 AI 행동을 몰라도 AI가 알아서 판단
- **발표 대사:** “E는 AI 행동 시스템을 전담하여 EnemyAI.ts에서 슬라임의 행동을 랜덤하게 결정했습니다. BattleManager는 AI 호출만 합니다.”

---

## **✅ D (UI/게임 상태 관리, 공통)**
- **책임:** UI만 전담 + 게임 흐름 연결
- **구현:**
    - UIManager.ts: 버튼, HP바, 로그 관리
    - main.ts: BattleManager와 UIManager 연결만
    - index.html, style.css: UI 뼈대와 스타일

- **목표:** 게임 UI와 상태 관리 분리 (게임 흐름 연결만 main.ts)
- **발표 대사:** “D는 UI 시스템과 게임 흐름 연결을 맡아서 UIManager.ts에서 버튼과 HP바, 로그 관리를 전담하고, main.ts에서는 BattleManager와 UIManager를 연결하는 흐름만 담당했습니다.”

---

## 💡 정리
    - **A는 생명체 관리만. 전투 X.**
    - **B는 전투만. AI는 호출만.**
    - **C는 아이템/스킬 틀과 사용 로직.**
    - **E는 적 AI 행동 결정만. BattleManager에 안 넣음.**
    - **D는 UI만. 게임 흐름은 main.ts에서 BattleManager/AI 연결만.**
