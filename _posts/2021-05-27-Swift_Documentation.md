---
title: "Swift Documentation"
categories:
  - Documentation
tags:
  - Swift
  - Documentation
---

업무를 진행하면서 개발에 집중 뿐만 아니라 추후 문서 작업까지 한번에 해결하기 위해 Documentation 작업 제안. 검토된 내용에 대해 기술하였습니다. 

## Documentation Comments 작성 방법

Xcode 내에 기본적으로 Documentation 주석을 달 수 있는 단축키 (```option(⌥) + command(⌘) + /```)가 존재합니다 (아래 그림 참조).

![Xcode comment shortcut](./images/Xcode_comment_documentation.png)

단축키를 이용하여 주석을 달 경우 한 줄 Documentation 주석 (```///```)이 달리게 됩니다.

```swift
/// 이것은 한 줄 Documentation 주석입니다.
/// 여러줄로 작성 시 라인 앞에 /// 항상 추가해주어야 합니다.
struct Test {
  let testProperty1: String
  let testProperty2: String
}
```

여러줄을 작성하기 위해선 기존 여러줄 주석에서 시작부분에 별표문자를 하나 추가하하여 작성합니다 (```/** ... */```).

```swift
/**
	이것은 여러줄 Documentation 주석입니다.
	해당 주석 기호 안에 여러줄 작성이 가능합니다.
*/
struct Test {
  let testProperty1: String
  let testProperty2: String
}
```

Documentation 주석은 Markdown 규칙이 적용됩니다.

- 단락은 빈 줄로 구분됩니다.
- 목록을 나타내려면 (`-`, `+`, `*`, `•`) 로 나타냅니다.
- 순서가 있는 목록을 나타내려면 숫자 (`1`, `2`, ....) 와 마침표 (`.`)나 숫자와 닫는 괄호 (`1)`) 로 나타냅니다.
- 헤더와 같은 큰 제목 표시는 `#`, `=`, `_` 로 표시합니다.
- 링크와 이미지 모두 표기 가능하고 Xcode 에서 확인할 수 있습니다.



## Documentation Tools

Apple Documentation 처럼 만들어 주는 툴이 2가지 정도 존재하고 있습니다. JAZZY, SwiftDoc 이 있으며, 해당 툴들을 사용하면 소스 내에 작성된 Documentation 주석을 기반으로 문서를 생성해 줍니다.

### Documentation Tool 종류

#### JAZZY

<img src="https://github.com/realm/jazzy/blob/master/images/logo.jpg?raw=true" alt="JAZZY Logo" style="zoom:50%;" />

모바일 데이터베이스 오픈소스 인 Realm 에서 만든 툴로 Swift와 Objective-C 모두 지원 가능하며, command-line 명령어로 동작합니다.

[JAZZY Github URL](https://github.com/realm/jazzy)

#### SwiftDoc

Swift 만 지원하며, command-line 명령어로 동작합니다.

[SwiftDoc Github URL](https://github.com/SwiftDocOrg/swift-doc)

> Swift와 Objective-C 모두 지원하는 JAZZY 를 기준으로 설명하도록 하겠습니다.

### JAZZY 사용법

#### 설치

1. Xcode Command-line tool 이 설치되어 있어야 합니다.

   ```
   $ xcode-select --install
   ```

2. JAZZY 설치

```
$ [sudo] gem install jazzy
```

> 오류사항 대처방법
>
> - Permission Error
>
> ```
> ERROR:  While executing gem ... (Gem::FilePermissionError)
>     You don't have write permissions for the /Library/Ruby/Gems/2.6.0 directory.
> ```
>
> 참조 사이트: [Gem File Permission Error](https://jojoldu.tistory.com/288)

#### 사용법

기본적으로 ```jazzy``` 명령어로 쉽게 문서를 만들 수 있습니다. 하지만 이 명령어로 문서를 생성할 경우 ```open``` 과 ```public``` 에 대한 것만 생성이 됩니다. 여러가지 옵션을 추가하는 것이 가능하며 아래에 설명되어 있습니다.

- ```internal```, ```fileprivate```, ```private``` 문서화 추가
  ```
  --min-acl [internal | fileprivate | private]
  ```
- 기존에 생성한 문서를 지우고 새로운 문서를 생성
  ```
  --clean
  ```
- 작성자 지정
  ```
  --author Boram
  ```
- 테마지정
  ```
  --theme [apple | fullwidth | jony]
  ```
> 테마명
>
> ```apple```: [--theme apple](https://realm.io/docs/swift/latest/api/)
>
> ```fullwidth```: [--theme fullwidth](https://reduxkit.github.io/ReduxKit/)
> 
> ```jony```: [--theme jony](https://harshilshah.github.io/IGListKit/)

#### Example
- Swift
  ```
  jazzy \
    --clean \
    --author Realm \
    --author_url https://realm.io \
    --github_url https://github.com/realm/realm-cocoa \
    --github-file-prefix https://github.com/realm/realm-cocoa/tree/v0.96.2 \
    --module-version 0.96.2 \
    --build-tool-arguments -scheme,RealmSwift \
    --module RealmSwift \
    --root-url https://realm.io/docs/swift/0.96.2/api/ \
    --output docs/swift_output \
    --theme docs/themes
  ```
- Objective-C
  ```
  jazzy \
    --objc \
    --clean \
    --author Realm \
    --author_url https://realm.io \
    --github_url https://github.com/realm/realm-cocoa \
    --github-file-prefix https://github.com/realm/realm-cocoa/tree/v2.2.0 \
    --module-version 2.2.0 \
    --build-tool-arguments --objc,Realm/Realm.h,--,-x,objective-c,-isysroot,$(xcrun --show-sdk-path),-I,$(pwd) \
    --module Realm \
    --root-url https://realm.io/docs/objc/2.2.0/api/ \
    --output docs/objc_output \
    --head "$(cat docs/custom_head.html)"
  ```

