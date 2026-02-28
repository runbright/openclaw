# Antigravity 시스템 아키텍처 및 동작 원리 가이드

이 문서는 Antigravity AI 에이전트의 내부 구조, 도구 연동 방식, 그리고 지식 관리 시스템에 대해 상세히 설명합니다.

---

## 1. 전체 시스템 아키텍처

Antigravity는 단순한 LLM을 넘어선 **자율적 에이전트 시스템**입니다. LLM이 '두뇌' 역할을 하며, 다양한 도구(Tools)와 MCP(Model Context Protocol)를 통해 실제 환경(파일 시스템, 터미널, 웹)과 상호작용합니다.

```mermaid
graph TD
    User((사용자 프롬프트)) --> Orchestrator[에이전트 오케스트레이터]
    
    subgraph Brain [두뇌 - LLM]
        Orchestrator --> Reasoning[추론 및 계획 수립]
        Reasoning --> Decision[도구 호출 결정]
    end

    subgraph Memory [기억 장치 - RAG & Context]
        Reasoning <--> KI[Knowledge Items - 장기 기억]
        Reasoning <--> Context[Persistent Context - 대화 및 작업 로그]
        Reasoning <--> Files[코드베이스/문서 Indexing]
    end

    subgraph Tools [도구 및 인터페이스]
        Decision --> FileTool[파일 시스템 도구]
        Decision --> TermTool[터미널/셸 도구]
        Decision --> WebTool[웹 브라우저/검색 도구]
        Decision --> MCP[MCP Servers]
        Decision --> GenTool[이미지 생성 도구]
    end

    FileTool --> Workspace[(로컬 워크스페이스)]
    TermTool --> Shell[실제 터미널 실행]
    WebTool --> Browser[Headless Browser / Search]
    MCP --> ExternalAPI[외부 리소스 및 데이터]
    
    Tools --> Observation[실행 결과 관찰]
    Observation --> Reasoning
```

---

## 2. 핵심 동작 원리: Reasoning Loop (Think-Act-Observe)

Antigravity는 사용자 요청을 완수할 때까지 다음 루프를 반복합니다.

1.  **Think (추론)**: 요청을 분석하고, 현재 상태를 파악하며, 다음 단계(Step)를 계획합니다.
2.  **Act (실행)**: 계획된 행동을 위해 가장 적합한 도구를 선택하여 실행합니다.
3.  **Observe (관찰)**: 도구 실행의 결과(성공/실패, 출력 내용, 오류 메시지)를 분석하여 지식으로 흡수합니다.

```mermaid
sequenceDiagram
    participant User as 사용자
    participant Agent as Antigravity Agent
    participant Env as 실행 환경 (FS/Terminal/Web)

    User->>Agent: 프롬프트 입력 (예: "버그 수정해줘")
    loop 지식 수집 및 분석
        Agent->>Env: 파일 구조 확인 (list_dir)
        Env-->>Agent: 디렉토리 목록 반환
        Agent->>Env: 코드 검색 (grep_search)
        Env-->>Agent: 검색 결과 반환
    end
    
    Note over Agent: 해결 계획 수립 (Thinking)

    loop 실행 및 검증
        Agent->>Env: 코드 수정 (replace_file_content)
        Env-->>Agent: 수정 완료
        Agent->>Env: 테스트 실행 (run_command)
        Env-->>Agent: 테스트 결과 (실패/성공)
        Note over Agent: 실패 시 원인 분석 후 재시도
    end

    Agent->>User: 작업 완료 보고 및 결과 요약
```

---

## 3. 주요 구성 요소 상세 설명

### 3.1 LLM & Orchestration
*   **지능**: 고성능 LLM을 사용하여 복잡한 코딩 문제 해결 및 아키텍처 설계를 수행합니다.
*   **Sub-agents**: 복잡도가 높은 작업(예: 브라우저 조작)은 전용 Sub-agent에게 위임하여 전문성을 높입니다.

### 3.2 도구 생태계 (Toolbox)
*   **파일 시스템(FS)**: 코드를 읽고(`view_file`), 구조를 파악하고(`view_file_outline`), 정밀하게 수정(`multi_replace_file_content`)합니다.
*   **터미널**: 실제 셸 명령어를 실행하여 빌드, 테스트, 배포 과정을 수행합니다.
*   **MCP (Model Context Protocol)**: 다양한 외부 서비스(GitHub, Google Drive, Slack 등)와 표준화된 방식으로 연동하여 리소스를 가져옵니다.

### 3.3 지식 관리 (Memory System)
*   **RAG (Retrieval-Augmented Generation)**: 단순히 문서를 찾는 수준을 넘어, 프로젝트의 전반적인 맥락을 실시간으로 검색하여 LLM의 context window를 최적화합니다.
*   **Knowledge Items (KIs)**: 과거의 중요한 결정 사항, 아키텍처 패턴, 해결된 버그 등을 데이터베이스화하여 동일한 실수를 반복하지 않도록 합니다.

### 3.4 워크플로우 (Workflows)
*   프로젝트 루트의 `.agents/workflows`에 정의된 가이드를 읽어, 복잡한 프로세스를 표준화된 절차에 따라 수행합니다.

---

## 4. LangGraph 및 기술 스택 활용 방식

*   **상태 관리**: 작업의 진행 상태를 그래프 구조로 관리하여, 어느 단계에서 오류가 발생했는지 추적하고 복구할 수 있습니다.
*   **에이전트 협업**: 상위 에이전트가 전략을 짜고, 하위 실행 유닛들이 각자의 도구를 사용하여 결과를 보고하는 계층적 구조를 가집니다.

---

*본 문서는 Antigravity 에이전트에 의해 자동 생성되었습니다.*
