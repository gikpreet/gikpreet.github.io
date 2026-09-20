---
title: AI Agent의 핵심 구성 요소
date: 2026-09-18 12:10:05 +0900
categories: [AI, AI Agent]
tags: [ai,agent]     # TAG names should always be lowercase
---

앞에서 살펴본 것 처럼 AI Agent는 단순한 대규모 언어 모델이 아닙니다. LLM이 언어를 이해하고 문제를 추론하는 핵심 역할을 담당한다면, AI Agent는 이러한 추론 능력을 실제 작업으로 연결하기 위해 여러 구성 요소를 하나의 시스템으로 통합합니다.

일반적인 AI Agent의 구조를 단순화하면 다음과 같이 표현할 수 있습니다.

![](/assets/img/2026-09-18-1/image01.png)

실제 AI Agent마다 내부 구조와 명칭은 다를 수 있습니다. 어떤 Agent는 별도의 Planning 모듈을 사용하고, 어떤 Agent는 LLM이 계획과 도구 선택을 동시에 수행합니다. Memory를 적극적으로 사용하는 Agent도 있고 현재 Context만을 중심으로 동작하는 Agent도 있습니다. 

하지만 대부분의 현대 AI Agent를 이해하는데는 몇 가지 공통적인 구성 요소가 있습니다. LLM, Context, Memory, Planning, Tools, Skills, Execution Environment, Observation, Evaluation, Orchestration 입니다.

이 요소들은 독립적으로 존재하기보다는 서로 연결되어 하나의 지속적인 작업 흐름을 만들어 냅니다.

## AI Agent에서 LLM

AI Agent의 중심에는 일반적으로 LLM이 있습니다.

LLM은 Agent가 현재 상황을 이해하고, 사용자의 목표를 해석하고, 무엇을 해야 하는지를 결정하도록 돕습니다. 이 때문에 AI Agent에서 LLM은 흔히 Reasoning Engine, 즉 추론 엔진의 역할을 한다고 설명할 수 있습니다.

만약, 사용자가 아래와 같은 요청을 실행했다고 합시다.

> 이 Java 프로젝트의 테스트가 실패하는 원인을 찾아서 수정해 줘.

이 요청에는 구체적인 작업 절차가 포함되어 있지 않습니다. 어떤 테스트를 실행해야 하는지, 어디에서 오류를 찾아야 하는지 사용자는 지정하지 않았습니다.

이 경우, Agent의 LLM이 질문을 받습니다. 우선 요청을 해석합니다.

GOAL
: Java 프로젝트의 테스트를 정상적으로 통과 시키는 것

그리고 현재 상황을 바탕으로 다음 행동을 판단할 수 있습니다.

```
프로젝트 구조를 확인해야 합니다.
     ↓ 
테스트 실행 방법을 확인해야 합니다.
     ↓ 
테스트를 실행해야 합니다.
     ↓ 
오류 메시지를 분석해야 합니다.
     ↓ 
관련 코드를 찾아야 합니다.
```

LLM은 단순히 문장을 만들어 내는 역할에 머무르지 않습니다. Agent 내부에서는 현재 상황과 목표 사이에서 다음 행동을 선택하는 의사결정의 중심으로 사용됩니다.

그렇다고 LLM 자체가 모든 작업을 수행하는 것은 아닙니다. LLM은 파일을 직접 읽는 파일 시스템도 아니며, 명령을 실행하는 터미널도 아니고, 웹사이트를 탐색하는 브라우저도 아닙니다.

LLM의 역할은 보다 근본적으로 다음과 같습니다.

> 현재 상황을 해석하고, 목표를 달성하기 위해 다음에 무엇을 해야 할지를 판단하는 것입니다.
{: .prompt-info }

따라서 Agent 전체를 인간의 작업 과정에 비유한다면 LLM은 두뇌에 가까운 구성 요소라고 할 수 있습니다.

## Context: 지금 알아야 하는 정보

LLM이 올바른 판단을 내리려면 현재 상황에 대한 정보가 필요합니다. 이러한 정보를 Context라고 합니다. Context는 Agent가 현재 작업을 수행하는 순간에 LLM에게 제공되는 정보의 집합입니다.

예를 들어, Java 프로젝트로 수정하고 있는 Agent라면 다음과 같은 정보가 Context에 포함될 수 있습니다.

* 현재 목표
* 최근 사용자의 요청
* 프로젝트의 디렉토리 구조
* 현재 읽고 있는 Java 파일
* 최근 테스트 결과
* 오류 메시지
* 사용 가능한 Tool
* 작업중 생성된 중간 결과

이를 하나의 작업 공간(Workspace)로 생각하면 이해가 쉽습니다. 사람이 일을 할 때, 책상 위에는 현재 읽고 있는 문서와 노트, 관련 자료, 지금 해결하고 있는 문제에 필요한 정보가 있습니다. Context의 개념은 이와 비슷합니다.

![](/assets/img/2026-09-18-1/image02.png)

Context가 충분하지 않으면 LLM이 정확한 판단을 내리기 어렵습니다. 반대로 너무 많은 정보를 한꺼번에 제공하면 중요한 정보를 찾기 어려워지고 처리 비용도 증가할 수 있습니다.

Agent 설계에서는 단순히 많은 정보를 제공하는 것이 아니라 **현재 작업에 필요한 정보를 적절하게 선택하여 Context에 넣는 것**이 중요합니다. 이를 **Context Management**라고 합니다.

좋은 Agent는 모든 정보를 항상 가지고 있으려고 하지 않습니다. 현재 작업에 필요한 정보를 찾고, 불필요한 정보를 제외하고, 필요한 순간에 다시 가져오는 방식으로 Context를 관리합니다.

## Memory: 과거의 정보를 다시 사용하는 능력

Context가 현재 작업에 필요한 정보라면 Memory는 이전의 경험이나 정보를 나중에도 다시 사용할 수 있도록 저장하는 기능입니다. 이 둘은 비슷해 보이지만 중요한 차이가 있습니다.

Context
: 현재 작업 중
: 지금 LLM이 보고 있는 정보

Memory
: 과거에 저장된 정보
: 필요할 때 다시 불러오는 정보

예를 들어 어떤 Agent가 특정 Java 프로젝트를 며칠동안 계속 관리한다고 생각해보면, 첫날 Agent는 프로젝트를 분석하면서 다음과 같은 사실을 파악했습니다.

```
Project : student-management 
Language : Java 21 
Framework : Spring Boot 
Build Tool : Gradle 
Database : PostgreSQL 
Test : JUnit
```

며칠 후, 프로젝트를 수정하면서 중요한 설계 결정을 내립니다.

```
Repository 계층에서는 Optional<Student>를 반환합니다. 
Service 계층에서 존재하지 않는 Student에 대한 예외 처리를 수행합니다.
```

이런 정보를 매번 처음부터 다시 조사해야 한다면 아주 비효율적이고 시간도 낭비됩니다. 

Memory가 있는 Agent라면 이런 정보를 저장해두었다가 다음 작업에서 다시 사용할 수 있습니다.

![](/assets/img/2026-09-18-1/image03.png)

Memory는 단순히 모든 과거 대화를 저장하는 것을 의미하지 않습니다. 실용적인 Agent에서는 어떤 정보를 저장할 것인지, 언제 불러올 것인지, 얼마나 오래 보존할 것인지가 중요합니다. 그래서 Memory는 여러 종류로 나누어 설명하기도 합니다.

Short-term Memory
: 현재 작업 과정에서 잠시 유지해야 할 정보를 의미합니다.
Long-term Memory
: 여러 세션에 걸쳐 다시 사용할 가치가 있는 정보를 저장
Eposodic Memory
: 과거에 어떤 작업을 수행했고 어떤 결과가 있었는지와 같은 경험을 저장
Semantic Memory
: 프로젝트 구조, 사용자 선호, 도메인 지식 처럼 비교적 일반화된 정보를 저장

모든 Agent가 이런 구분을 명시적으로 구현하는 것은 아니지만, 장기간 사용되는 Agent에서는 Memory 관리가 중요한 설계 요소가 됩니다.

## Planning: 목표를 작업으로 나누는 과정

사용자가 Agent에게 전달하는 요청은 하나의 명령으로 실행하기에는 너무 클때가 있습니다. 다음과 같이 요청한다고 생각해봅시다.

```
Java와 PostgreSQL을 이용해서 학생관리 프로그램을 만들어 주세요.
```

이 목표를 한 번의 행동으로 처리할 수는 없습니다. 먼저 해야 할 일을 여러 개의 작은 작업으로 나누어야 합니다.

![](/assets/img/2026-09-18-1/image04.png)

이렇게 큰 목표를 작은 작업으로 분해하는 과정을 **Task Decomposition**이라고 합니다.

Planning은 단순히 할 일에 대한 리스트를 만드는 것 만을 의미하지 않습니다. 각 작업 사이의 의존 관계도 판단해야 합니다.

예를 들어, 데이터베이스 설계르 ㄹ하지 않은 상태에서 Repository 구현을 시작하면 나중에 코드를 크게 수정해야 할 수도 있습니다. 따라서 Agent는 작업 순서를 고려해야 합니다.

```
요구사항 분석 
 ↓
데이터 모델 설계
 ↓ 
Database Schema 
 ↓ 
Repository 
 ↓ 
Service 
 ↓ 
API
 ↓ 
Test
```

Planning이 잘 이루어지면 Agent는 복잡한 문제를 단계적으로 해결할 수 있습니다. 

그러나 모든 Agent가 작업 시작 전에 완전한 계획을 먼저 만드는 것은 아닙니다. 어떤 Agent는 먼저 간단한 계획을 세운 뒤 작업 결과를 보면서 계속 계획을 수정합니다. 이를 **Replanning**이라고 합니다.

```
Plan 
  ↓ 
Execute 
  ↓ 
Observe 
  ↓ 
예상과 다름 
  ↓ 
Replan 
  ↓ 
Execute
```

실제 환경은 예상한대로 움직이지 않을 때가 많기 때문에 Replanning은 Agent의 중요한 능력입니다.

## Tools: 외부 환경에 대한 액션

LLM이 아무리 뛰어난 추론 능력을 가지고 있어도 외부 시스템과 상호작용할 방법이 없다면 실제로 할 수 있는 일은 제한됩니다. 

Agent가 외부 환경에 대한 액션을 위해 사용하는 기능을 일반적으로 **Tool**이라고 합니다. 개발 Agent라면 다음과 같은 Tool을 사용할 수 있습니다.

```
File Tool
 ├─ 파일 읽기
 ├─ 파일 쓰기 
 └─ 파일 검색 
Shell Tool
 ├─ 명령 실행 
 └─ 프로그램 실행 
Git Tool
 ├─ status 
 ├─ diff 
 ├─ commit 
 └─ branch 
Browser Tool
 ├─ 페이지 열기
 ├─ 링크 탐색
 └─ 데이터 입력 
Database Tool
 ├─ Query
 ├─ Insert 
 └─ Update
```

예를 들어, LLM이 다음과 같이 판단했을 때,

> "StudentRepository.java 파일을  확인해야 합니다."

Agent는 파일 읽기 Tool을 사용할 수 있습니다.

> read_file("StudentRepository.java")

Tool은 파일 내용을 Agent에게 반환하고. LLM은 그 결과를 분석합니다.

그리고 다음과 같이 판단할 수 있습니다.

> "findById() 메서드의 null 처리를 수정해야 합니다."

이번에는 파일 편집 Tool을 사용합니다. 그다음 테스트를 실행하기 위해 Shell Tool을 사용할 수 있습니다.

> run_terminal("mvn test")

즉, Agent의 작업은 다음과 같은 흐름을 가집니다.

![](/assets/img/2026-09-18-1/image05.png)

이 구조가 Tool-using Agent의 기본적인 동작 방식입니다.

## Skill: 반복 가능한 작업 방법

Tool이 하나의 개별 행동을 수행하는 기능이라면 Skill은 높은 수준의 개념입니다. Skill은 특정 종류의 작업을 수행하기 위한 **재사용 가능한 절차나 능력**이라고 볼 수 있습니다.

예를 들어, `run_terminal()`은 Tool 입니다. 하지만 다음과 같은 작업 절차는 skill이라고 볼 수 있습니다.

```
Java Test Debugging Skill 
1. 프로젝트 Build Tool 확인 
2. 테스트 실행 
3. 실패한 테스트 확인 
4. Stack Trace 분석 
5. 관련 코드 검색 
6. 문제 코드 수정 
7. 테스트 재실행 
8. 결과 검증
```

Agent가 처음 접하는 작업이라면 각 단계를 그때그때 추론해야 할 수 있습니다. 그러나 이런 작업 방식을 skill로 가지고 있다면 비슷한 문제가 발생했을 때 재사용할 수 있습니다. 이를 구조적으로 표현하면 다음과 같습니다.

![](/assets/img/2026-09-18-1/image06.png)

Tool과 skill의 차이를 간단하게 표현하면 다음과 같습니다.

> Tool은 무엇을 할 수 있는지를 정의하고, Skill은 그것을 어떻게 조합해서 일을 해결할지를 정의합니다.
{: .prompt-info }

최근 Agent 시스템에서 Skill이 중요하지는 이유도 여기에 있습니다. Agent가 모든 문제를 매번 처음부터 해결하는 대신, 효과적이었던 작업 방법을 다시 사용할 수 있다면 효율성과 일관성을 높일 수 있습니다.

## Execution Environment - 실제적인 수행 공간

Agent가 Tool을 사용한다고 해서 모든 Tool이 LLM 내부에서 실행되는 것은 아닙니다. 실제 명령이나 프로그램은 **Execution Environment**, 즉 실행 환경에서 동작합니다.

Coding Agent를 예로 들면, 실행 환경은 사용자의 컴퓨터일 수도 있고, Docker Container나 가상 머신일 수도 있으며, 별도의 Cloud Sandbox일 수도 있습니다.

![](/assets/img/2026-09-18-1/image07.png)

살행 환경은 Agent의 능력뿐만 아니라 안전성과도 밀접하게 관련됩니다. 

예를 들어 Agent를 사용자의 실제 윤영 서버에서도 바로 실행하면 매우 강력하지만 위험도 높아집니다. 반대로 별도의 Sandbox 안에서 실행하면 Agent가 실수하더라도 몀향을 제한할 수 있습니다.

실제 Agent 시스템에서는 다음과 같은 실행 방식을 사용할 수 있습니다.

```
Local Machine 
Virtual Machine 
Docker Container 
Cloud Sandbox 
Remote Server 
Isolated Workspace
```

Agent에게 얼마나 강력한 실행 환경을 제공할 것인가는 시스템의 목적과 보안 요구사항에 따라 결정해야 합니다.

## Observation: 실행 결과를 다시 받아들이는 과정

Agent에게 tool을 제공하는 것만으로는 충분하지 않습니다. Tool을 실행한 다음 **무슨 일이 일어났는지 확인하는 과정**이 필요합니다. 이를 **Observation**이라고 합니다.

예를 들어, Agent가 다음 명령을 실행했다고 합시다.

```bash
mvn test
```

실행 결과가 다음과 같이 반환됩니다.

```
Tests run: 12 

Failures: 1 

StudentServiceTest.findStudent NullPointerException
```

이 정보가 Observation입니다. Agent는 Observation을 다시 Context에 넣고 LLM에게 전달합니다. 

LLM은 결과를 해석합니다.

```
"StudentService의 findStudent 메서드에서 null 관련 문제가 발생한 것으로 보입니다."
```

그리고 다음 행동을 결정합니다.

```
StudentService.java 읽기
```

이 과정을 구조적으로 표현하면 다음과 같습니다.

![](/assets/img/2026-09-18-1/image08.png)

Agent가 단순한 자동화 스크립트와 다른 이유 중 하나가 바로 이러한 **관찰과 재판단의 구조**에 있습니다.

## Evaluation: 목표를 달성했는지 판단

Agent는 단순히 여러 행동을 수행하는 것이 목적이 아닙니다. 사용자가 요청한 목표가 실제로 달성되었는지 판단해야 합니다. 이를 **Evaluation**이라고 합니다.

예를 들어, 목표가 다음과 같다고 할 때

```
모든 Java 테스트를 통과시키세요
```

Agent가 코드를 수정한 뒤 테스트를 실행합니다.

```
Tests run: 12 
Failures: 0 
Errors: 0 
BUILD SUCCESS
```

이 결과를 통해 목표가 달성되었다고 판단할 수 있습니다. 그러나 목표가 추상적인 경우에는 평가가 어려울 수도 있습니다.

예를 들어 다음과 같은 요청이 있다고 생각해봅시다.

```
이 코드를 더 읽기 쉽게 개선해 주세요
```

이 경우 단순히 프로그램이 실행되는지만으로 작업 완료 여부를 판단하기 어렵습니다. 코드 스타일, 복잡도, 중복, 가독성등의 기준을 함께 평가해야 할 수 있습니다. 

따라서 Agent 시스템에서는 가능한 한 목표를 **검증 가능한 상태**로 정의하는 것이 좋습니다.

```
코드를 좋게 만들어 주세요
```

보다는

``` 
"기능은 그대로 유지하면서 
중복 코드를 제거하고, 
모든 기존 테스트가 통과하도록 
리팩터링해 주세요."
```

가 좋습니다. 목표가 명확할수록 Agent도 작업 완료 여부를 보다 정확하게 판단할 수 있습니다.

## Orchstration: 모든 구성 요소를 연결

AI Agent에는 LLM, Memory, Tools, Skills, Planning 등 여러 구성 요소가 존재합니다. 이 요소들이 각각 잘 동작하더라도 적절한 순서로 연결되지 않으면 하나의 Agent로 작동할 수 없습니다.

이들을 조정하고 흐름을 관리하는 기능을 **Orchstration** 이라고 합니다. Orchstration은 일종의 지휘자 역할을 합니다.

![](/assets/img/2026-09-18-1/image09.png)

Orchstrator는 현재 작업 상태를 관리하고, LLM에게 어떤 정보를 전달할지 결정하고, Tool 실행 결과를 다시 LLM에게 보내고, 작업을 계속할지 종료할지 결정합니다.

Multi-Agent 시스템에서는 Orchstrator의 역할이 더욱 중요해집니다. 에를 들어, 여러 Agent가 협업하는 경우 다음과 같은 구조가 만들어질 수 있습니다.

![](/assets/img/2026-09-18-1/image10.png)

이 경우 누가 어떤 작업을 수행할 것인지, 결과를 누구에게 전달할 것인지, 실패한 작업을 다시 누구에게 보낼 것인지 등을 관리해야 합니다.

따라서 Agent가 복잡해질수록 Orchstration의 중요성도 커집니다.

## Agent Loop: 구성 요소들이 실제로 연결되는 방식

실제 Agent 구성에서는 이 구성요소들이 따로 움직이지 않습니다. 하나의 반복적인 작업 흐름으로 연결됩니다. 사용자가 목표를 전달하는 순간부터 작업이 완료될 때 까지의 전체 흐름은 다음과 같이 표현될 수 있습니다.

![](/assets/img/2026-09-18-1/image01.png)

이 구조가 바로 Agent Loop의 핵심입니다.

Agent의 핵심은 처음부터 모든 것을 정확하게 예측하는 데 있지 않습니다. 실제로 행동하고, 그 결과를 관찰하고, 필요한 경우 판단을 수정하는 데 있습니다. 이 점에서 Agent는 기존의 고정된 프로그램과 다른 특징을 가집니다.

전통적인 프로그램이 미리 정의된 절차를 따르는 경우가 많다면 Agent는 실행 중 얻은 정보를 바탕으로 다음 행동을 바꿀 수 있습니다.

전총적인 Agent의 Workflow가 다음과 같다면,

```
A → B → C → D
```

Agent는 다음과 같은 형태를 가질 수 있습니다.

![](/assets/img/2026-09-18-1/image11.png)

이런 동적 의사 결정 구조가 Agentic System의 중요한 특징입니다.

## 각 구성 요소의 의존

AI Agent의 성능을 결정하는 것은 단순히 강력한 LLM 하나가 아닙니다. 

* 아무리 뛰어난 모델을 사용하더라도 필요한 Tool이 없으면 실제 행동을 수행할 수 없습니다.
* Tool이 많더라도 적절한 권한 관리가 없다면 위험할 수 있습니다.
* Memory가 있어도 잘못된 정보를 저장하면 이후 판단을 오히려 방해할 수 있습니다.
* Planning이 지나치게 복잡하면 단순한 작업에서도 불필요하게 많은 단계를 거칠 수 있습니다.
* Context 관리가 제대로 이루어지지 않으면 중요한 정보가 LLM에 전달되지 않을 수도 있습니다.

따라서 Agent의 성능은 다음과 같은 관계로 이해하는 것이 좋습니다.

![](/assets/img/2026-09-18-1/image12.png)

좋은 Agent는 특정 요소 하나가 뛰어난 시스템이라기보다, 각 구성 요소가 목적에 맞게 균형 있게 연결된 시스템이라고 할 수 있습니다.

## AI Agent의 구성 요소를 하나의 비유로 이해

AI Agent의 구조가 복잡하게 느껴진다면 사람의 작업 과정에 비유해서 이해할 수 있습니다. 어떤 개발자가 프로그램의 오류를 수정한다고 생각해 보겠습니다.

* 개발자는 먼저 문제를 이해합니다. 이것은 LLM의 추론 역할과 비슷합니다.
* 현재 코드와 오류 메시지를 책상 위에 놓고 살펴봅니다. 이것은 Context에 해당합니다.
* 이전에 비슷한 오류를 어떻게 해결했는지 떠올립니다. 이것은 Memory와 비슷합니다.
* 어떤 순서로 문제를 조사할지 정합니다. 이것은 Planning입니다.
* IDE, 터미널, 검색 도구를 사용합니다. 이것들은 Tools입니다.
* 익숙한 디버깅 절차를 활용합니다. 이것은 Skills와 비슷합니다.
* 프로그램을 실행합니다. 이것이 Execution입니다.
* 오류 메시지를 확인합니다. 이것이 Observation입니다.
* 문제가 해결되었는지 판단합니다. 이것이 Evaluation입니다.
* 그리고 이 모든 작업을 하나의 흐름으로 관리합니다. 이것이 Orchestration입니다.

이를 한눈에 정리하면 다음과 같습니다.

```
AI Agent                사람의 작업 과정

LLM                 →   사고와 판단
Context             →   현재 책상 위의 자료
Memory              →   기억과 경험
Planning            →   작업 계획
Tools               →   손과 도구
Skills              →   숙련된 작업 방법
Execution           →   실제 행동
Observation         →   결과 확인
Evaluation          →   성공 여부 판단
Orchestration       →   전체 작업 관리
```

이 비유를 기억하면 복잡한 Agent 시스템도 비교적 쉽게 이해할 수 있습니다.

## 정리

AI Agent는 LLM 하나로 이루어진 시스템이 아닙니다.

LLM은 Agent의 핵심적인 추론 엔진이지만, 실제 목표를 수행하려면 현재 상황을 제공하는 Context, 과거의 정보를 활용하는 Memory, 복잡한 목표를 작은 작업으로 나누는 Planning, 외부 환경과 상호작용하는 Tools, 반복 가능한 작업 방법을 제공하는 Skills, 실제 작업이 이루어지는 Execution Environment가 함께 필요합니다.

또한 Agent는 행동의 결과를 Observation을 통해 다시 받아들이고, Evaluation을 통해 목표 달성 여부를 판단합니다. 이러한 구성 요소의 실행 흐름을 관리하는 역할을 Orchestration이 담당합니다.

결국 AI Agent의 전체 구조는 다음과 같이 요약할 수 있습니다.

```
                GOAL
                  │
                  ▼
               CONTEXT
                  │
          ┌───────┴───────┐
          ▼               ▼
        MEMORY            LLM
                           │
                           ▼
                       PLANNING
                           │
                           ▼
                   TOOLS / SKILLS
                           │
                           ▼
                       EXECUTION
                           │
                           ▼
                      OBSERVATION
                           │
                           ▼
                      EVALUATION
                           │
                     목표 달성?
                      /       \
                    Yes        No
                     │          │
                     ▼          ▼
                 COMPLETE    REPLAN
```

따라서 AI Agent를 이해할 때 가장 중요한 것은 각각의 구성 요소를 개별적인 기능으로만 보는 것이 아닙니다.

AI Agent의 핵심은 LLM, Context, Memory, Planning, Tools, Skills, Execution, Observation이 하나의 반복적인 작업 흐름으로 연결된다는 데 있습니다.

LLM이 무엇을 해야 할지 판단하는 능력을 제공한다면, Tools는 실제로 행동할 수 있는 능력을 제공합니다. Memory는 과거의 경험을 현재에 연결하고, Planning은 큰 목표를 실행 가능한 단계로 바꾸며, Observation과 Evaluation은 실행 결과를 다음 판단으로 되돌려 줍니다.

이 모든 요소가 하나의 순환 구조를 이루면서 비로소 AI는 단순히 질문에 대답하는 시스템을 넘어, 목표를 받아 실제 작업을 수행하는 Agent로 발전하게 됩니다.