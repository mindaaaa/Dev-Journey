[필수] 부스트캠프 웹・모바일은 2025년 6월 23일(월) 베이직을 시작으로 챌린지, 멤버십까지 약 8개월의 시간을 투입해야 합니다. 이 시간을 온전히 학습과 개발에 몰입할 때 지속 가능한 개발자로 성장할 수 있습니다. 소중한 8개월이라는 시간을 투입하기 전 나는 왜 개발자가 되고 싶은지, 그리고 개발자로 성장하는 여정에서 부스트캠프에서 보낼 8개월 동안 이루고 싶은 목표는 무엇인지 곰곰히 생각해보면 좋겠습니다. 개발자가 되고 싶은 이유와 부스트캠프 웹・모바일에서 이루고자 하는 목표를 구체적으로 서술해 주세요. 거창한 이유나 목표일 필요는 없습니다. 솔직한 나의 이야기와 목표를 작성해주세요. (공백 포함 800자 내외) 

### 📌 본문 (799자)
입학 당시 전자공학을 전공했지만, 학과 수업으로 접한 C언어에 재미를 느껴 1년간 휴학하며 다양한 언어와 도구(C, 파이썬, Unity 등)를 경험했습니다. 그 과정에서 브라우저에서 직접 결과를 확인하고, 사용자 경험을 빠르게 피드백받을 수 있는 웹 개발에 매력을 느꼈고 결국 전공을 컴퓨터학부로 바꿔 웹 개발을 본격적으로 공부하게 됐습니다.
하지만 파이썬 기반 수업이 많아 웹 개발자로서 깊이 있는 팀 경험을 쌓기 어려웠고, 협업 경험이 부족하다는 점이 늘 아쉬움으로 남았습니다.
이후 타 부트캠프 지원하면서 짧은 기간이지만 다른 개발자들과 코드 리뷰를 주고 받으며 협업을 통한 기술적 성장 가능성을 직접 체감한 경험이 있습니다. 단순한 문제 해결뿐 아니라, 디렉토리 구조를 나누는 기준, JSDoc을 활용한 코드 의도 전달 등을 새롭게 인식했고, 그 후로 실제 코드에 적용하고 있습니다. 최근에는 나만의 인터프리터 언어(MINT)를 직접 설계하며, 개발의 깊이를 넓혀가고 있습니다.
하지만 결국 제가 지향하는 개발자의 모습은 단독 구현에만 몰두하는 개발자가 아니라 실제 서비스를 만들어가는 팀의 일원입니다. 각자의 색으로 하나의 맵을 완성해나가듯, 동료들과 함께 방향을 찾아가는 과정이 제가 꿈꾸는 개발입니다.
부스트캠프에서는 이런 가치관을 바탕으로, 팀과 함께 실제 서비스 수준의 프로젝트를 경험하며 협업 감각을 키우고 싶습니다. 특히 코드 리뷰와 문제 해결 과정 속에서 기술뿐 아니라 소통과 설계 역량까지 함께 성장시키고 장기적으로는 사용자에게 가치 있는 경험을 제공하는 프론트엔드 개발자로 자리잡는 것이 목표입니다.


---

[필수] 부스트캠프 웹・모바일에서는 스스로 학습의 주체가 되는 태도가 중요합니다. 누군가 해주길 기다리지 않고 스스로 시도한 경험, 적당한 선에서 멈추지 않고 끈질기게 더 나은 방법을 찾은 경험, 주변사람에게 자신 있게 설명할 수 있을 만큼 무언가에 몰입한 경험 등 자기주도적으로 사고하고 행동한 경험을 구체적인 과정과 그 경험이 현재 나에게 미친 영향을 중심으로 서술해 주세요. 개발과 관련한 경험이 아니어도 괜찮으며, 결과적으로 실패한 경험도 좋습니다. 그 과정에서 어떻게 자기 주도적으로 고민하고 행동했는지, 무엇을 배웠는지가 드러나면 충분합니다. (공백 포함 800자 내외) 

### 📌 본문 (754자)
Git을 꽤 오래 써왔지만 명령어만 복붙하는 수준에 머물렀습니다. 협업 과정에서 브랜치를 나누고 머지하는 역할을 맡게 되면, 내부 동작 원리를 이해하지 못한 채 겉핥기식으로 사용하는 방식이 팀에 혼란을 줄 수 있다는 불안감이 들었습니다. 특히 Git을 리드하는 입장에서 깊이 없는 판단이 부담스럽게 느껴졌고 단순 사용자보다는 원리를 이해하고 팀에 설명할 수 있는 사람이 되고 싶었습니다. 그래서 Git이 내부 동작 구조를 이해해보자는 목표로 미니 Git 구현을 시작했습니다.
Pro Git과 Git Internals를 참고해 Git이 어떤 객체를 만들어 파일을 저장하고 연결하는지 구조적으로 파악해 나갔습니다. blob, tree, commit 객체가  SHA-1 해시를 통해 어떻게 서로를 참조하며 연결되는지를 흐름도로 정리해가며 손으로 따라갔고, init → add → commit → branch → checkout → log 순으로 구현하며 내부 로직을 하나씩 익혀나갔습니다.
포인터 기반 구조에서는 해피 케이스만으로 동작을 신뢰하기 어렵다는 점을 뒤늦게 인지했고, 테스트 코드를 도입했으나 초기 설계에서 CLI 환경과 테스트 환경을 분리하지 않아 통합에 큰 어려움을 겪었습니다. 결국 각 명령어에 basePath 인자를 주입하는 방식으로 CLI와 Jest 환경을 분리하며 해결했습니다.
이 과정에서 테스트 환경까지 고려한 초기 설계의 중요성을 배웠고, 단순 구현을 넘어서 구조적 설계와 유지보수성까지 고민하는 개발 습관을 갖게 되었습니다.

---

[필수] 부스트캠프 웹・모바일에서는 커뮤니티 안에서 성장하는 태도가 중요합니다. 나의 생각을 정리해서 동료와 커뮤니케이션한 경험, 동료의 의견을 경청하고 피드백한 경험, 팀으로 협력하며 혼자일 때보다 더 나은 결과물을 만든 경험 등 열린 사고로 동료와 협력한 경험을 구체적인 과정과 그 경험이 현재 나에게 미친 영향을 중심으로 서술해 주세요. 개발과 관련한 경험이 아니어도 괜찮으며, 결과적으로 실패한 경험도 좋습니다. 그 과정에서 어떻게 열린 사고로 동료와 협력했는지, 무엇을 배웠는지가 드러나면 충분합니다. (공백 포함 800자 내외)
### 📌 본문 (800자)
소프트웨어공학 수업에서 외국인 팀원 1명을 포함한 3인 팀으로 관광지에서 다양한 포즈 가이드를 화면에 오버레이할 수 있는 카메라 앱을 기획한 적이 있습니다. 저는 기획을, 외국인 팀원이 UI 디자인(Figma)을, 다른 팀원은 분석을 맡았습니다.
언어 장벽이 있는 팀이었기 때문에 기획 내용을 쉬운 단어로 풀어 수업 후 직접 만나 설명하며 협업에 힘썼습니다.
'심플한 UI, 직관적인 UX'라는 핵심 가치를 설정한 덕분에, 디자인 방향성과 주요 기능에 대한 피드백을 빠르게 조율할 수 있었습니다. 예를 들어, 처음에는 오버레이 기능만으로 충분하다고 생각했지만, 팀원이 인스타그램이나 칼로리 앱처럼 사용자 간 공유와 반응이 서비스 확장에 기여한다는 점을 근거로 커뮤니티 기능을 제안했습니다. 단순한 기능 추가가 아닌 사용자 경험 확장이라는 점에 팀 모두가 공감했고, 논의를 거쳐 로그인 기능과 게시 기능을 기획에 반영하게 되었습니다.
최종 발표에서는 반영된 기능을 어떻게 효과적으로 전달할지가 주요 과제였습니다. 같은 수업을 듣는 학생들이 실제 구매자 역할을 하며 구매 여부를 선택하는 방식이었기 때문에, 제가 발표를 맡아 핵심 가치를중심으로 직관적으로 전달하는 전략을 제안하고 실행했습니다. 불필요한 설명은 줄이고 간결한 기획안으로 기능과 장점을 명확히 강조한 결과, 가장 많은 학생들의 선택을 받아 최고 판매율을 기록했습니다.
이 경험을 통해 좋은 아이디어는 수용하고, 핵심 가치를 중심으로 전략을 세우는 일이 팀 성과에 직결된다는 점을 체감했고, 이후 팀 프로젝트에서도 기획·설계 단계에서 이 전략을 적극 활용하고 있습니다.