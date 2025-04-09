# playnvest-eliza
Play &amp; Invest with Eliza OS


AI Game & DeFi Assistant: Technical README

목표
게이머가 게임 플레이에만 집중하면서도, Eliza AI Agent가 Sui 기반 DeFi와 게임 아이템(모의/온체인) 투자를 자동화해주는 시스템을 구현합니다.

⸻

개요 (Overview)

이 프로젝트는 게임 자산(NFT/아이템)과 DeFi 투자(스테이킹, LP 등)를 AI 에이전트가 대화형으로 관리·실행해주는 AI Game & DeFi Assistant입니다. 사용자는 단순히 챗 UI로 “아이템 사줘”, “SUI 스테이킹 해줘” 등을 명령하면, Eliza OS 에이전트가 Sui 블록체인 및 게임 마켓(모의)과 연동하여 자동 거래를 처리합니다.
	•	Eliza OS: 대화형 AI 프레임워크 (플러그인 구조)
	•	Sui 플러그인: 스왑, 스테이킹 등 Sui DeFi 액션 자동 실행
	•	Game 플러그인: Mock 게임 마켓(아이템 조회·매매·가격 모니터링) 기능 제공
	•	LLM(OpenAI): 자연어 질의→명령 해석, 투자 전략/추천 생성
	•	Frontend: 채팅 UI, 포트폴리오 대시보드
	•	Backend: Eliza OS 서버, Mock 게임 데이터 관리, 예약 주문 스케줄링, DB 관리

⸻

시스템 아키텍처 (System Architecture)

1) 전체 흐름도 (High-Level Flowchart)

아래는 사용자→프론트엔드→백엔드(엘리자 OS)→플러그인→Sui 블록체인 & Mock 게임 마켓으로 이어지는 전체 데이터 흐름을 나타냅니다.

flowchart LR
    A[사용자 (게이머)] -->|채팅 입력| B((프론트엔드))
    B -->|REST/WebSocket| C((백엔드/Eliza 서버))

    C -->|LLM 질의| D[ElizaOS Core <br> + OpenAI API]
    D -->|액션 결정| E((Sui Plugin))
    D -->|액션 결정| F((Game Plugin))

    E -->|스왑, 스테이킹, LP| G[Sui 블록체인]
    F -->|매수/매도, 예약 주문| H[Mock Game Market <br>(DB/API)]
    
    H -->|아이템 목록 & 시세| F
    G -->|체인 잔고 & 거래결과| E

    D -->|결과(메시지)| C
    C -->|JSON 응답| B
    B -->|화면 갱신| A

	1.	사용자가 채팅 UI를 통해 명령 입력
	2.	프론트엔드가 해당 명령을 백엔드/Eliza 서버로 전송
	3.	ElizaOS Core가 OpenAI API로 텍스트 해석 후, Sui Plugin(DeFi 액션) 또는 Game Plugin(아이템 거래)을 호출
	4.	Sui 블록체인 / Mock 게임 마켓에서 실제 트랜잭션(스왑, 매수 등) 수행
	5.	결과를 다시 대화 형태로 사용자에게 안내하고, 프론트엔드 대시보드 갱신

⸻

2) 시퀀스 다이어그램 (Sequence Diagram)

아래는 예시로 사용자가 ‘아이템 X를 10 SUI 이하로 떨어지면 사줘’ 라고 명령하는 시나리오를 보여줍니다.

sequenceDiagram
    participant U as User
    participant FE as Frontend
    participant BE as Backend (Eliza OS)
    participant EO as ElizaOS Core
    participant GP as Game Plugin
    participant DB as Mock Game Market/DB

    U->>FE: "아이템 X, 10 SUI 이하로 떨어지면 자동 매수"
    FE->>BE: 전달 (API/WS)
    BE->>EO: 대화 해석 (OpenAI API)
    EO->>GP: setPriceAlert(itemId=X, threshold=10, type=BUY)
    GP->>DB: 예약 주문 등록

    Note over GP,DB: 백엔드 서버 내<br>주기적으로 시세 갱신/체크

    DB->>GP: "현재 아이템 X 가격이 9.5 SUI로 하락"
    GP->>EO: "자동 매수 조건 충족"
    EO->>BE: 트랜잭션 실행 명령
    GP->>DB: 실제 매수 처리 (주문 체결)

    BE->>FE: "매수 성공! 아이템 X 보유 완료"
    FE->>U: UI 업데이트/알림



⸻

주요 컴포넌트별 구성

(A) 프론트엔드 (Frontend)
	•	챗 UI: 사용자와 Eliza Agent가 텍스트로 상호작용
	•	포트폴리오 대시보드:
	•	DeFi(스테이킹, LP) 자산 현황
	•	게임 아이템(NFT) 목록, 가격, 예약 주문 상태
	•	기술 스택 제안: React/Vue + REST or WebSocket 통신

(B) 백엔드 (Backend)
	•	Eliza OS 서버 구동
	•	LLM 연동 (OpenAI Key)
	•	플러그인(Sui, Game) 로드
	•	Mock 게임 마켓(DB/API)
	•	아이템 목록, 가격 변동 로직, 매수/매도 API
	•	예약 주문 스케줄링(조건 충족 시 자동 체결)
	•	DB
	•	사용자 지갑/자산 상태, 거래 이력 기록
	•	예약 주문/거래 로그 관리

(C) Eliza OS & 플러그인 (Agent Core)
	1.	ElizaOS Core
	•	시스템 프롬프트 & 캐릭터 설정 (게임·DeFi 전문)
	•	사용자의 자연어 입력→OpenAI API로 분석→플러그인 액션 결정
	2.	Sui Plugin
	•	Sui 블록체인 상의 스왑, 스테이킹, LP, 송금 등 DeFi 액션 처리
	3.	Game Plugin
	•	Mock 게임 마켓과 상호작용 (아이템 매수/매도, 예약 주문, 이벤트 정보)
	•	백엔드의 /api/game-market/...을 호출하거나 DB에 직접 접근

(D) (선택) Sui Move 스마트 컨트랙트
	•	MVP 단계에서는 기존 스테이킹/DEX 프로토콜 호출만으로 충분
	•	게임 아이템을 온체인으로 하려면 Move 컨트랙트 개발 (추후 확장 가능)

⸻

설치 및 실행 방법 (Setup & Run)
	1.	프로젝트 클론

git clone https://github.com/your-repo/ai-game-defi-assistant.git
cd ai-game-defi-assistant


	2.	의존성 설치

pnpm install
# 또는 npm install


	3.	환경변수 설정
	•	.env 파일에 아래 정보 기재

OPENAI_API_KEY=sk-...
SUI_NETWORK=testnet
SUI_WALLET_PRIVATE_KEY=...
# 기타 DB, 포트 설정 등


	4.	백엔드(ElizaOS) 서버 실행

pnpm start:server
# or npm run start:server

	•	Eliza Agent가 구동되며, Sui & Game 플러그인이 로드됨
	•	Mock 게임 마켓 API도 함께 구동 or 별도 프로세스로 구동 (구현 구조에 따라)

	5.	프론트엔드 실행

pnpm start:client
# or npm run start:client

	•	브라우저 http://localhost:3000에서 챗 UI 접속

⸻

개발 로드맵 (Dev Roadmap)
	1.	기본 ElizaOS + Sui Plugin 연동
	•	Sui 테스트넷에서 토큰 스왑/전송 실험
	2.	Mock 게임 마켓
	•	간단한 In-memory 아이템 DB, 가격 변동, 매수/매도 API
	•	예약 주문(가격 조건) 처리 스케줄러
	3.	Game Plugin 개발
	•	listItems(), buyItem(), setPriceAlert() 등 액션을 Eliza에서 호출 가능하게
	•	대화형 예시 (“아이템 X 사줘”)를 LLM에 프롬프트 설정
	4.	프론트엔드
	•	채팅창 + 포트폴리오 대시보드 (DeFi+Game 자산)
	•	실시간 알림/상태 갱신
	5.	자동화 & 리밸런싱
	•	DeFi APY 체크, 스테이블 풀/LP 보상 재투자
	•	게임 아이템 가격 변동에 따른 자동 매수/매도
	6.	고급 기능(추후)
	•	Move 컨트랙트로 실제 온체인 NFT 거래
	•	멀티게임/멀티체인 확장, etc.

⸻

추가 참고사항 (Notes)
	•	보안: 자동 거래 시스템이므로, 지갑 키 접근 권한·한도 설정, 프로토콜 감사 여부 등을 신중히 검토.
	•	Mock vs On-chain: 해커톤/프로토타입 레벨에선 Mock 게임 마켓으로 빠르게 결과물을 제작. 나중에 진짜 온체인 또는 실제 게임 API로 교체 가능.
	•	LLM 한계: 대규모 언어 모델이 생성하는 전략이 무조건 최적은 아니므로, 리스크 표시와 사용자 승인 절차는 꼭 필요.

⸻

라이선스 (License)
	•	오픈소스 라이선스(예: MIT) 표기를 추가하세요.
	•	외부 종속성(Eliza OS, OpenAI API 등)의 라이선스도 준수해야 합니다.

⸻

결론

위 구조대로 프로젝트를 설계·구현하면, 게이머가 게임 자산과 Sui DeFi를 대화형 AI를 통해 손쉽게 운용하는 시스템이 탄생합니다. MVP 단계에서는 Mock 마켓(아이템 DB)만으로도 자동 매수/매도 + 스테이킹 기능을 시연할 수 있으며, 추후 온체인 NFT, 멀티 체인/게임 확장 등으로 발전시킬 수 있습니다.

문의/피드백:
	•	프로젝트 이슈 트래커나 팀 커뮤니티 채널을 통해 문의해주세요.
	•	더욱 흥미로운 아이디어나 협업 제안, 버그 리포트 등을 환영합니다.

Happy hacking! 게임도 즐기고 자산도 성장시키는 멋진 도전을 응원합니다.
