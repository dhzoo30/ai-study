# Memory 가 필요한 이유
LLM 가 session 간 state 를 공유하지 않기 때문에.
-> 유한한 context window
-> preference, decision, progress 가 다음 세션으로 이어지지 않음
-> 경험에서 배울 수 없음. 같은 실수를 반복하고 성공한 절차를 재사용하기 어려움

Agent memory: 에이전트가 스스로 쌓고 고쳐나가는 state 저장소

# Context engineering 개념
## RAG
- 정적인 외부 지식 소스
- 상태가 바뀌지 않음 (index 가 미리 구축)
- Query 는 memory 방식과 공유, write 는 없음
## Context window
- 한 번의 호출에 넣는 토큰
- 상태가 바뀌지 않음 (호출마다 초기화)
- 작은 규모에서 메모리를 대체할 수 있음
## Continual learning
- 모델 가중치를 다룸
- 상태가 바뀜 (파라미터가 갱신)
- 이 개념을 차용한 parametric memory 도 있음
## Agent Memory
- agent 의 경험과 지식을 다룸
- 상태가 바뀜 (agent 가 스스로 쓰고 고침)

# Memory Types
## Working Memory 
 agent 의 active workspace 라고 생각하면 됨. 현재 agent 가 수행하고 있는 내용의 중간 과정, 결과등을 기록해놓는 것.
 e.g., Scrachpads, CoT buffer, Context
 Session 마다 초기화되는, 휘발되는 기억 (in-context history)

## Episodic Memory
Episodic memory 는 과거의 이벤트를 맥락과 시간으로 인덱싱 하는 memory type.
- Past interactions : 과거의 대화, 시도, 결과
- Successful trajectories: 높은 reward 를 받았던 action 들의 예시
- Failure cases: 실패했던 케이스에 대해 root-cause 와 함께 기록하여 추후 실패 방지
- Retrieval-augmented episodic recall: 주어진 task 에 대해 가장 비슷한 과거의 에피소드들을 가져와 context 에 추가함.
주로 episode 요약과 함께 임베딩되어 vector store 됨

## Semantic Memory
특정 에피소드(사건) 으로부터 알게되는 일반적인 사실이나 개념
- Factual knowledge: Entity, 특성, 관계 (e.g., 파리는 프랑스의 수도)
- Domain concepts: 정의, 용어, 온톨로지
- Knowledge graphs: structured 된 관계
Episodic memory 와의 차이는 context-independent 라는 것.

## Procedural Memory (Skills)
"How to do things" 를 기억하는 것. Skill 이나 자동화되는 패턴에 관한 기억
- Tool-use pattern
- Action sequences (e.g., 코드를 배포할땐 run tests -> build images -> push -> update menifest -> deploy)
- 

# Memory Architectures

## RAG 기반
Embedding Stores, Vector Databases
Re-ranking
Retrieval Stragies
## Summarization-Based
LLM 을 활용해서 정보를 요약/추출

## Graph-Based
Knowledge graph 
GraphRAG
Temporal Knowledge Graph

그 외 Key-Value Memory Networks, MemGPT 와 같이 Virtual Context Management 정도로 구분 가능

# Memory Operations
## Write
- 모든걸 기록할 수 없음. 중요도를 판단해서 저장해야함. 
- Contradiction detection: 새로운 사실이 기존의 사실과 충돌할 때 어떻게 처리할 것인지? (기존 사실을 덮어씀, 둘다 보존, 또는 user 가 결정하도록)
- 어떤 형식으로 할건지
	- Atomic facts
	- Structured notes
	- Summarized episodes
	- 그대로
## Read / Retrieve
- Query 의 경우 기존 RAG 방식과 많은 부분 공유
## Update
memory 간 충돌이 났을 때 어떻게 할것인가
Forgetting 을 어떻게 구현할 것인가
	- LRU
	- Importance-weighted
## Reflect
보다 고차원적인 작용. 적재된 여러 memory들로부터 High-level abstraction 을 추출해냄
- 예를들어 실패한 episode 들로부터 공통된 유저 패턴을 파악
# Multi-turn, Multi-agent
Multiple agent 시스템이 공통 task 를 처리할 때 coordination mechanism 을 구현해야 함
- Shared Memory Pool

# Training Memory Systems with RL
Memory operation (read, write, update reflect) 를 언제 적절히 호출할지 RL 로 학습시킬 수 있음
- memory 가 정답을 포함하면 reward
- 올바른 memory 를 retrieve 하면 reward
- 불필요한 write 를 안하면 reward
등등
보통 가장 중요한 문제는 "무엇을 기억할지" 결정하는 문제

# Evaluating Memory Systems
memory 성능 평가는 대체로 쉽지 않고, benchmark 신뢰도도 그렇게 높은 편은 아닌 것 같음 그래도 LongMemEval, LOCOMO, InfiniteBench 과 같은 benchmark 존재
1. Information extraction
2. Multi-session reasoning
3. Temporal reasoning
4. Knowledge updates 
5. Abstention - 기억이 없을때 모른다고 하기
### Metrics
Memory Recall
Memory Precision
Latency
Token efficiency
Answer accuracy
Faithfulness
Personalization quality
Contradiction rate

# Timeline 별 정리

## 초기  (2023)
문제의식: Context engineering. 한 에피소드에서 누적된 성공,실패 시도를 agent 가 반영할 수 있는가. Context window 를 벗어났을 때 맥락을 유지시킬 수 있는가
## Reflexion (2023)
 가장 단순한 경험 메모리. 과제를 시도한 뒤 결과 피드백을 받고 "왜 실패했고 다음엔 어떻게 할 지"에 대한 반성문을 에피소트 버퍼에 저장. 다음 시도의 프롬프트에 그 반성문을 같이 넣어 성능을 올림. 한 과제 안에서만 유효 (장기기억이 아님). 이후 방법들이 개선되어 과제간에 누적시키는 방향으로 확장 (ReasoningBank, ACE)
## LangChain BufferMemory (2023)
 대화기록을 그대로, 또는 줄여서 다음 프롬프트에 붙이는 가장 기초적인 방식. 
- 전체대화를 그대로 붙이기
- 최근 n turns 붙이기
- 턴마다 LLM 이 누적 요약을 갱신
- Token limit 넘치면 요약
 메모리라기보다는 컨텍스트 관리에 가까움.

## 중기 (2024~)
문제의식: 인지과학을 접목하기 시작. Memory 를 하나의 layer 로서 library 화 할 수 있는가. LLM 이 스스로 대화기록에서 사실관계를 추출해 장기기억 메모리를 만들 수 있는가. 시간에 따라 바뀌는 사실관계를 반영할 수 있는가

## Mem0 (2024~)
사실 추출과 벡터화하여 저장하는 것을 library 화 
LLM 이 사실 추출, 중복 정리
```
m.add("우리 팀 스탠드업은 화요일 10시야", user_id="minji")
# → 기존 기억 10개 조회 → LLM 추출 1회 → 해시 중복 제거 → 벡터 저장 → 엔티티 연결
# → {"results": [{"id": "3f2a…", "memory": "User의 팀 스탠드업은 매주 화요일 10시", "event": "ADD"}]}

m.add("스탠드업이 목요일로 바뀌었어", user_id="minji")
# → 화요일 기억이 LLM에 보이지만 연산은 ADD뿐 → 새 기억 추가, 둘 다 남음

m.search("스탠드업 언제야?", filters={"user_id": "minji"})
# → 목요일(0.66), 화요일(0.64) 둘 다 반환. 최신 값 판단은 앱의 몫
```

최근에는 계속 업데이트하여 graph 기반 memory, memory compression 등도 지원한다고 함.

# 최근
## MemGPT -> Letta(2024)
Mem0 와 같은 방식은 모델이 저장할지 결정하는게 아니라 모든 메시지에 대해 별도 추출 호출이 항상 돌아감. MemGPT 는 에이전트가 주도적으로 스스로 기억을 파일에 쓰고 수정함

## Dreaming

사람이 꿈을 꾸듯이, agent 가 background process 로 스스로 memory 를 정리하여 개선함.
저장된 session log 들을 기반으로 반복 오류, 유저 선호 등을 찾아 memory 를 재구성함
Claude Dreams, ChatGPT Dreaming 에 구현되어있음.
