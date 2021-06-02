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

![Xcode comment shortcut](/images/Xcode_comment_documentation.png)

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

## Documentation 구성

### Summary

해당 Documentation 주석에 요약을 작성할 수 있으며, Documentation 주석의 첫째 줄에 해당합니다.

### Discussion

Documentation 주석에 첫째 줄은 Summary 를 나타내지만 빈줄을 추가하여 두번째 단락은 자세한 설명에 해당합니다.

```swift
/// Summary
///
/// Discussion
```

### Parameters, Returns, Throws

- Parameter

  Parameter 이름으로 구성이 되며 파라미터에 대한 설명을 기입할 수 있습니다. Parameter 가 여러개인 경우 - Parameters: 를 추가하고 - 으로 구분하여 여러 파라미터를 입력할 수 있습니다.

  ```swift
  /// - Parameters:
  ///		- param1: Description
  ///		- param2: Description
  ```

- Returns

  리턴하는 것에 대한 설명을 입력할 수 있습니다.

- Throws

  발생가능한 오류에 대해 입력할 수 있습니다.

### 그 밖의 구성

#### **Safety Information**

- Precondition
- Postcondition
- Requires
- Invariant
- Complexity
- Important
- Warning

#### **Metadata**

- Author
- Authors
- Copyright
- Date
- SeeAlso
- Since
- Version

#### **Notes**

- Attention
- Bug
- Experiment
- Note
- Remark
- ToDo

### MARK, TODO, FIXME

Section 을 나누기 위해 사용되며 시안성을 높이기 위해 사용됩니다. 주석을 작성하듯이 두 개의 슬래시 (`//`)  로 시작하여 작성되며 대시 (`-`) 를 추가하여 보기 쉽게 라인으로 구분되게 작성할 수도 있습니다.

> 출처: [NSHipster](https://nshipster.com/swift-documentation/)

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
```swift
/// 일반 함수, 파라미터 포함 함수, 리턴이 있는 함수, 파라미터 리턴이 있는 함수를 포함한 class
class Test {
    
    /// 일반 함수
    func testFunction() {
        
    }
    
    /// 파라미터가 포함되어 있는 함수
    /// - Parameter param: String 타입을 파라미터로 가집니다.
    func testFunctionWithParam(param: String) {
        
    }
    
    /// 리턴이 있는 함수
    /// - Returns: String 타입을 반환합니다.
    func testFunctionWithReturn() -> String {
        return ""
    }
    
    /// 파라미터와 리턴이 있는 함수
    /// - Parameter param: String 타입을 파라미터로 가집니다.
    /// - Returns: String 타입을 반환합니다.
    func testFunctionWithParamReturn(param: String) -> String {
        return ""
    }
}
```

해당 예제를 기반으로 Documentation을 만들어 보도록 하겠습니다.

```
$ jazzy --clean --author Boram
```

위 Command 로 문서를 만들어 봅시다.

<img src="https://github.com/bbiguduk/bbiguduk.github.io/blob/main/images/public_documentation.png?raw=true" alt="private Documentation" style="zoom:50%;" />

JAZZY 는 기본적으로 public 만 문서화 하기 때문에 예제를 문서화 하면 아무런 내용도 나타나지 않습니다.

> 기본적으로 public, open 접근제어자에 대해서만 문서가 생성이 되므로 강제로 접근제어자 변경이 필요합니다.

```
$ jazzy --clean --author Boram --min-acl private
```

위 Command 로 문서를 다시 만들어 봅시다.

<img src="https://github.com/bbiguduk/bbiguduk.github.io/blob/main/images/private_documentation.png?raw=true" alt="private Documentation" style="zoom:50%;" />

이제 모든 swift 소스에 대한 문서가 완성된 것을 확인할 수 있습니다. 근데 Documentation 주석을 작성하지 않은 소스도 문서화 되어 있는 것을 볼 수 있습니다. 이것을 생성하지 않으려면

> 기본적으로 모든 파일에 대한 문서가 생성이 되므로 Documentation 주석이 없는 부분이 생성되지 않게 하려면 추가 옵션을 지정해 주어야 합니다.

```
$ jazzy --clean --author Boram --min-acl private --skip-undocumented
```

위 Command 를 실행하게 되면

<img src="https://github.com/bbiguduk/bbiguduk.github.io/blob/main/images/skip_undocumentation.png?raw=true" alt="private Documentation" style="zoom:50%;" />

이제 깔끔하게 내가 추가한 Documentation 주석 부분만 문서화 됨을 확인할 수 있습니다.

아래는 Realm 에 나와 있는 Command line 예제이며 여러가지 [Option](https://github.com/realm/jazzy) 을 확인해 볼 수 있습니다.

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

## 
