---
title: 터미널 개선 작업
categories:
  - tinytip
tags:
  - iTerm2
  - git
  - zsh
---

기존에는 vscode에서 그래프같은건 git graph를 띄어놓고 실제 작업은 아래 터미널창에서 커맨드로 했었다.  
그런데 IntelliJ를 쓰기 시작했는데 이거의 git 화면이 도저히 적응이 되지 않는거다.  
적응이 될 때 까지는 그냥 터미널 켜서 커맨드 라인에서 쓸 것 같아서 좀 더 편의성을 높여보았다. vim 쓰기 싫어서 혼자 vscode 연구해서 쓰던 내게 이런 일이... tmux같이 원래도 쓰고 있던 애들은 제외.

#### fzf 설치

파일 검색이 너무 힘들어서 깔았다.

```zsh
brew install fzf
$(brew --prefix)/opt/fzf/install
```

쓰는 김에 사람들이 쓰는거 조합해서 내 입맛에 맞게 좀 더 커스텀했다.

```zsh
alias pf="fzf --layout=reverse --height=50% --preview '([[ -f {} ]] && (bat --style=numbers --color=always {} || cat {})) || ([[ -d {} ]] && (tree -C {} | less)) || echo {} 2> /dev/null | head -200'"
```

검색도 편해지고, `ctrl` + `R` 했을 때도 편하게 되었다.

#### fzf 이용해서 git alias 추가

커맨드라인에서 하자니 가장 불편했던게 브랜치 체크아웃이랑 로그 보는거라 해당 부분을 개선할 수 있는 방법을 찾았다.

```
[alias]
        co = "!f() { args=$@; if [ -z \"$args\" ]; then branch=$(git branch --all | grep -v HEAD | fzf --preview 'echo {} | cut -c 3- | xargs git log --color --graph' | cut -c 3-); git checkout $(echo $branch | sed 's#remotes/[^/]*/##'); else git checkout $args; fi }; f"
        logs = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%C(bold blue)<%an>%Creset' --abbrev-commit
        ls = !git logs | fzf -m --ansi --layout=reverse --preview=\"echo {} | sed 's/-.*$//' | egrep -o '[0-9a-f]+' | xargs git show --color=always\" | sed 's/-.*$//' | egrep -o '[0-9a-f]+'
```

remote branch 들을 보면서 커서로 이동하면서 엔터로 체크아웃, log를 커서로 이동하면서 diff를 볼 수 있게 되었다.

#### neoVim 설치

건드리기 시작한김에 조금 더 해줬다. 굳이 바꿔줄 필요는 없을 것 같았지만 바꿔보았다.

#### SpaceVim 설치

플러그인 모임. 이걸 하고 나니 Vim 화면이 많이 달라져서 스스로도 낯가리는 중. 근데 낯선거지 괜찮은 것 같아서 (하다못해 page up/down 때 부드럽게 움직이는거라든가) 적응 할 생각. 모든 테마를 nord로 바꾸는 사람인데 내장 테마중에 nord가 있어서 좋았다.

그 외에는 IntelliJ에서 gitignore된 부분은 배경색과 글자색이 거의 같아서 잘 안보이는 문제가 있었다. 근데 ignore된거 표시를 아얘 지우는건 아닌 것 같아서 글자색을 잘 보이는걸로 바꿨다. 깃 이슈도 있던데 다들 코멘트로 어디서 색 바꿨는지만 공유하고 공식이 뭔가 패치를 해주지는 않는듯...

참고
* https://www.mimul.com/blog/gui-free-tips-git-command/
* https://subicura.com/mac/dev/terminal-apps.html
