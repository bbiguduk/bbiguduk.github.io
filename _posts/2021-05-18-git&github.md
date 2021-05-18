---
title: "Git & GitHub 란?"
categories:
  - Software Configuration Management
  - SCM
tags:
  - github
  - git
  - Git
  - GitHub
  - Version Control System
  - VCS
---

대부분 IT 회사에서는 개발한 프로젝트 및 소스관리를 위해 형상관리 툴을 사용하고 있습니다.

대부분은 Git을 사용하지만 저는 지금까지 SVN을 사용하여 Git에 대해 공부할 겸 정리를 해보도록 하겠습니다.

## Software Configuration Management (SCM)

소프트웨어 개발 및 유지보수를 위해 버전관리를 하는 SVN, Git 등을 나타냅니다.

## Git

Git은 소규모 프로젝트부터 대규모 프로젝트까지 모든것을 빠르고 효율적으로 처리하도록 설계된 오픈소스 분산 버전제어 시스템입니다.

### Git 설치

MAC 기준으로 작성하였습니다.

1. Homebrew 설치

  ```
  $ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  ```

2. Git 설치

  ```
  $ brew install git
  ```

3. Git 실행

  ```
  $ git
  ```

터미널에서 명령어를 쳐보면 여러가지 기능을 사용할 수 있습니다.

[git 명령어](/images/terminal_git.png)

## GitHub

[GitHub 로고](/images/GitHub_logo.png)

GitHub는 Git을 사용한 소프트웨어 개발 및 버전 관리를위한 인터넷 호스팅 제공 업체입니다. Git의 분산 버전 제어 및 소스 코드 관리 (SCM) 기능과 자체 기능을 제공합니다. 모든 프로젝트에 대한 액세스 제어 및 버그 추적, 기능 요청, 작업 관리, 지속적인 통합 및 Wiki와 같은 여러 협업 기능을 제공합니다.