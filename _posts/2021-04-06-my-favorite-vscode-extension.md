---
title: 내가 좋아하는 vscode extension
categories:
  - vscode
tags:
  - vscode
---

사내에서 항상 vscode 내 계정 로그인에 실패했었는데, github로 시도한게 문제였는지 ms 계정으로 했더니 한번에 성공했다.
사실 그 일과는 하등 관계 없지만, 생각난 김에 내가 좋아하는(추천하는) vscode extension들.

#### C/C++

* Name: C/C++
* Id: ms-vscode.cpptools
* Description: C/C++ IntelliSense, debugging, and code browsing.
* Version: 1.2.2
* Publisher: Microsoft
* VS Marketplace Link: [https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools)

말이 필요없다. 사실 호불호를 논할 필요도 없다. 이게 없으면 vscode를 쓸 생각도 못했을거다. 좋다는 에디터/IDE들이 많지만 C를 지원하는건 많지 않았던 시기, vim밖에 답이 없나 싶었을 때 쯤 마지막이다 하는 마음으로 사용하기 시작했다. 몇년간의 기다림 끝에 Find all reference 기능이 추가되어서 지금은 정말 편해졌다. (이전엔 검색만으로 썼었다). 자 이제 Call hierarchy를 보여줘.
문제점이 있다면 자동 업데이트를 하는데 업데이트를 하는 local OA의 OS를 물고 들어가는 것 같다. 난 linux용을 remote에 설치해야하는 처지인데 항상 실패한다. 문제는 이거를 실패하고 나면 디버그를 할 수가 없다. 결국 자동 업데이트 옵션을 껏고 매번 VSIX를 받아서 수동으로 설치하고 있다. 원래는 remote에서 직접 인터넷을 물고들어가는건가 싶어서 테스트 해보고 싶기도 하다. 그냥 remote에 설치되어있다고 인식되면 알아서 remote의 OS로 업데이트 되면 될 것 같은데.

#### Python

* Name: Python
* Id: ms-python.python
* Description: Linting, Debugging (multi-threaded, remote), Intellisense, Jupyter Notebooks, code formatting, refactoring, unit tests, and more.
* Version: 2021.3.680753044
* Publisher: Microsoft
* VS Marketplace Link: [https://marketplace.visualstudio.com/items?itemName=ms-python.python](https://marketplace.visualstudio.com/items?itemName=ms-python.python)

말이 필요 없다2222. 사실 Python만 할거면 pycharm을 쓰는게 훨씬 편리한 것 같다. 하지만 지금은 C/C++도 같이 봐야 하는 상황이고, 개발도 환경도 제법 복잡한데 (원격 위에 도커 위에 가상 OS위에 도커 위에 TC다..ㅎ...) 그렇게 복잡한 경우, 대중적이지 않은 경우는 상용 툴들이 커버를 잘 못하더라. vscode는 일단 백그라운드에서 돌아가는건 다 리눅스 프로그램들이라 (형상 관리는 git 커맨드가 돌아가고 있다거나 디버그는 설정에 따라 gdb가 뒤에 돌아가고 있다거나) 나만 잘하면 그래도 상용 툴보다도 다양한 경우에 사용할 수 있다. 나만 잘하면 된다.

#### Dracula Official

* Name: Dracula Official
* Id: dracula-theme.theme-dracula
* Description: Official Dracula Theme. A dark theme for many editors, shells, and more.
* Version: 2.22.3
* Publisher: Dracula Theme
* VS Marketplace Link: [https://marketplace.visualstudio.com/items?itemName=dracula-theme.theme-dracula](https://marketplace.visualstudio.com/items?itemName=dracula-theme.theme-dracula)

이쁜 테마들 많지만 vscode에서는 이것을 사용한다. 회사에서도 집에서도. 다크 테마는 개인적인 취향이고, vscode에서 C/C++, Python, markdown 이렇게 세가지 언어를 사용하는데 세가지 다 훌륭하게 구분해준다. (은근 여러 언어 다 잘 해주는게 많지 않다.) 터미널에서는 nord 테마를 사용하지만 vscode는 여기에 정착.

#### gitlens / git graph

* Name: GitLens — Git supercharged
* Id: eamodio.gitlens
* Description: Supercharge the Git capabilities built into Visual Studio Code — Visualize code authorship at a glance via Git blame annotations and code lens, seamlessly navigate and explore * Git repositories, gain valuable insights via powerful comparison commands, and so much more
* Version: 11.3.0
* Publisher: Eric Amodio
* VS Marketplace Link: [https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)

* Name: Git Graph
* Id: mhutchie.git-graph
* Description: View a Git Graph of your repository, and perform Git actions from the graph.
* Version: 1.30.0
* Publisher: mhutchie
* VS Marketplace Link: [https://marketplace.visualstudio.com/items?itemName=mhutchie.git-graph](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens)

vscode에서 이 두개 안쓰는 사람 없을 것이라고 생각한다. 정말정말 편한 익스텐션. 특히 graph는 터미널에서 보는게 아무리 봐도 한눈에 들어오지 않아서 GUI가 훨신 편한 것 같다. 어지간한 커맨드는 커맨드라인에서 처리하는데, checkout 정도는 git graph 안에서 처리한다. 아, 새로운 브랜치 만들기도 vscode 내부 소스 컨트롤쪽에서 처리한다. 소스 컨트롤이 최근에 많이 업데이트 되어서 사용하기 훨씬 간편해졌다.

#### Remote Development

* Name: Remote Development
* Id: ms-vscode-remote.vscode-remote-extensionpack
* Description: An extension pack that lets you open any folder in a container, on a remote machine, or in WSL and take advantage of VS Code's full feature set.
* Version: 0.20.0
* Publisher: Microsoft
* VS Marketplace Link: [https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)

회사에선 개발환경 자체가 리눅스 머신이라 사용. 근데 ssh remote + docker 둘 다 쓰는건 안되더라. 그냥 build랑 run은 별도로 하고 있음. 집에서는 WSL이나 아니면 groom 서비스로 썼었는데... 각각의 이유로 그닥 편리하지는 않아서 결국 안쓰게 됨. 여러 remote 중첩해서 사용하는거 가능했으면 좋겠다.

#### Tabnine Autocomplete AI

* Name: Tabnine Autocomplete AI: JavaScript, Python, TypeScript, PHP, Go, Java, Ruby, C/C++, HTML/CSS, C#, Rust, SQL, Bash, Kotlin, React
* Id: tabnine.tabnine-vscode
* Description: 👩‍💻🤖 JavaScript, Python, Java, Typescript & all other languages - AI Code completion plugin. Tabnine makes developers more productive by auto-completing their code.
* Version: 3.2.16
* Publisher: TabNine
* VS Marketplace Link: [https://marketplace.visualstudio.com/items?itemName=TabNine.tabnine-vscode](https://marketplace.visualstudio.com/items?itemName=TabNine.tabnine-vscode)

코드 자동완성시켜주는 익스텐션. 주의할 점은 구조체의 멤버변수 같은거 존재하지 않는 애를 자동완성시켜줄 수 있으니 주의. 최근에 막 업데이트 하면서 내부 dependency 다운로드 실패가 자꾸 뜬다. 뭘 실패한건지라도 알려주든가. 오히려 구버젼이 더 편하고 좋았던듯. 아 그리고 마크다운은 이거 쓰면 좀 불편하긴 하다. 블로그 글을 머신러닝으로 자동완성 시켜준다니 얼마나 의미 없는 일이겠어.

#### Bookmarks

* Name: Bookmarks
* Id: alefragnani.bookmarks
* Description: Mark lines and jump to them
* Version: 13.0.4
* Publisher: Alessandro Fragnani
* VS Marketplace Link: [https://marketplace.visualstudio.com/items?itemName=alefragnani.Bookmarks](https://marketplace.visualstudio.com/items?itemName=alefragnani.Bookmarks)

말 그대로의 툴. 간편하고 사용하기 좋다. C/C++ 익스텐션의 code navigation이 완벽하진 않아서 자주 막히는 곳인데 자주 보게 되는 코드 부분들에 북마크 해두면 생산성 향상에 좋다.

#### Live Share

* Name: Live Share
* Id: ms-vsliveshare.vsliveshare
* Description: Real-time collaborative development from the comfort of your favorite tools.
* Version: 1.0.4070
* Publisher: Microsoft
* VS Marketplace Link: [https://marketplace.visualstudio.com/items?itemName=MS-vsliveshare.vsliveshare](https://marketplace.visualstudio.com/items?itemName=MS-vsliveshare.vsliveshare)

프로젝트를 구글 드라이브마냥 동시 편집이 가능하게 해준다. 스터디 때 사용하면서 이게 회사에서 가능하면 진짜 좋을텐데라고 생각했다.

#### Bracket Pair Colorizer

* Name: Bracket Pair Colorizer 2
* Id: coenraads.bracket-pair-colorizer-2
* Description: A customizable extension for colorizing matching brackets
* Version: 0.2.0
* Publisher: CoenraadS
* VS Marketplace Link: [https://marketplace.visualstudio.com/items?itemName=CoenraadS.bracket-pair-colorizer-2](https://marketplace.visualstudio.com/items?itemName=CoenraadS.bracket-pair-colorizer-2)

짝이 맞는 괄호마다 색을 바꿔준다. 괄호가 있는 언어를 쓴다면 무조건 써야 한다고 생각한다. 특히나 C처럼 다중으로 쓰이곤 하는 경우라면 더더욱.

#### Jira and bitbucket

* Name: Jira and Bitbucket (Official)
* Id: atlassian.atlascode
* Description: Bringing the power of Jira and Bitbucket to VS Code - With Atlassian for VS Code you can create and view issues, start work on issues, create pull requests, do code reviews, * start builds, get build statuses and more!
* Version: 2.8.6
* Publisher: Atlassian
* VS Marketplace Link: [https://marketplace.visualstudio.com/items?itemName=Atlassian.atlascode](https://marketplace.visualstudio.com/items?itemName=Atlassian.atlascode)

회사에서 JIRA를 사용하기 때문에 설치했다. 특이한 점은 최초 연동시에는 remote 연결 상태에서 하면 실패한다. 무조건 로컬에서 오픈 한 상태로 연결해야 함. 편리한 점은 봐야 할 JIRA가 여러개인 경우 (우리꺼, 고객꺼, 해외연구소꺼 이런식으로...) 하나하나 들어가는거보다 vscode에 모두 등록해놓고 필터링으로 해서 모든 JIRA에서 내 이슈들을 볼 수 있다는 것이다. 다만 간단하게 코멘트 달거나 상태 변경하는 것은 가능한데 좀 어려운 기능들은 그냥 페이지 가는게 속편하다. 어차피 url 복사해주기때문에 그것도 쉽다. 아, bug를 수정 시작하면서 branch로 따주고 이런 소소한 기능들도 편리하다. 정작 나는 하던대로 수동으로 브랜치 만들고 있긴 하다. TODO도 코드에 박으면 자동으로 Task 생성해줬던 것 같다.

그 외에 cpplint나 markdownlint같은 것도 쓰고 있고... 뭔가 더 있었는데 기억 나면 글 하나 더 쓰는걸로 :)
