---
title: git commit에서 There was a problem with the editor 'vi'와 함께 커밋 실패
categories:
  - tinytip
tags:
  - git
---

기본 에디터가 vi로 설정되어 있는데 이게 제대로 동작을 안하는 것 같았다. `git config --global core.editor /usr/bin/vim`을 이용해 vim으로 변경하면서 해결됨.
