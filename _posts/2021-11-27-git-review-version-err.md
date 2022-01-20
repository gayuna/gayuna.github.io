---
title: git review에서 `--preserve-merges was replaced by --rebase-merges`에러가 뜰 때
categories:
  - tinytip
tags:
  - git
---

#### 현상

`git review` 커맨드를 날렸을 때  

```
Errors running git rebase -p -i <리모트 브랜치명> fatal: --preserve-merges was replaced by --rebase-merges
It is likely that your change has a merge conflict. You may resolve itin the working tree now as described above and then run 'git review'again, or if you do not want to resolve it yet (note that the changecan not merge until the conflict is resolved) you may run 'git rebase--abort' then 'git review -R' to upload the change without rebasing.
```

라는 에러가 뜨면서 리뷰가 등록되지 않는다.  
`-R` 옵션을 주면 정상적으로 등록되며 컨플릭트도 없다.  
IntelliJ 내장 gerrit 플러그인에서는 문제 없이 작동한다.

#### 원인

git version이 2.34.0이었고 git review 버젼이 2.1.0이었다. git 2.34.0에서 rebase 옵션 중 `--preserve-merges`가 사라졌다. git review 내부에서 이 옵션을 쓰는데 git이 지원하지 않으니 생기는 문제였다. 이를 수정한 버젼이 git review 2.2.0이다.

#### 해결 방법

`git-review`를 최신 버젼으로 업데이트 한다.  
`brew install git-review`
