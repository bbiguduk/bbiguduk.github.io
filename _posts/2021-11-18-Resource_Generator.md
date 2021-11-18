---
title: "iOS Resource 편하게 관리하기"
categories:
  - Swift
  - Script
tags:
  - Resource
---

앱을 개발하면서 Image, Color, String 등 관리에 대한 고민을 많이 하게 됩니다.

현재 지마켓의 경우 String 은 Build Phase 에 스크립트를 통해 Localizable.strings 파일을 읽어 들여 struct 를 만들어서 관리를 하고 있습니다 (auto complete 로 오타/에러를 줄여줌).

Image 나 Color 도 asset 으로 관리를 하게 되면 불러올 때 이름을 string 으로 불러와 로드해야 되는 문제가 있습니다.

여기서 문제라고 하면... string 오타로 인한 버그, 실수가 발생할 수 있다는 의미입니다.



하지만 이미 많은 오픈소스로 이러한 문제를 해결한 라이브러리가 많이 존재합니다.

가장 유명한 2가지는 [Rswift](https://github.com/mac-cain13/R.swift) 와 [SwiftGen](https://github.com/SwiftGen/SwiftGen) 입니다.

살펴보면 해당 라이브러리는 기능도 다양하고 여러가지를 지원하는 것을 확인할 수 있습니다.



그러다 문득 굳이 저렇게 많은 기능을 사용하지 않는다면 차라리 compact 한 버전으로 존재하면 좋을 것 같다는 생각을 들어 직접 구현해 보기로 했습니다.

먼저 생각한 컨셉은

1. 가벼워야 함
2. 간단해야 함
3. 프로그램 설치 등 하지 않고 가능했으면 함 (필요하면 파일 하나 정도는....)
4. String 은 .strings 파일로 관리가 되어야 함
5. Image, Color 는 asset 으로 관리가 되어야 함

을 염두에 두고 만들었습니다.

기본적으로 Command line tool 을 이용해서 만들도록 하겠습니다!

## String

![Localizable](/images/Localizable.png)

기본적으로 `.strings` 을 보시면 위의 이미지와 같이 작성하도록 되어 있습니다.

해당 파일을 읽어서 패턴을 이용해 `String` 부분만 빼내어 사용하도록 하겠습니다. 자세한 내용은 아래에 더 자세히 작성하도록 하겠습니다.

## Image

![Image](/images/Image.png)

`Images.xc/assets` 을 보시면 2개의 이미지가 들어가 있는 것을 확인해 볼 수 있습니다. 좀 더 어떤 구조로 되어 있는지 확인하기 위해 Finder 에서 살펴 보도록 하겠습니다.

> Asset 이름은 꼭 Images 로 할 필요는 없습니다.

![ImageAsset1](/images/ImageAsset1.png)![ImageAsset2](/images/ImageAsset2.png)

`Images.xc/assets` 폴더를 열어 보면 추가한 이미지들의 폴더가 보입니다. `이미지이름.imageset` 형식으로 되어 있으며, 이제 이 형식을 가지고 만들어 보도록 하겠습니다. 자세한 내용은 아래에서 더 다루도록 하겠습니다.

## Color

![Color](/images/Color.png)

`Colors.xc/assets` 을 보시면 3개의 Color 가 들어가 있는 것을 확인할 수 있습니다. 좀 더 어떤 구조로 되어 있는지 확인하기 위해 Finder 에서 살펴 보도록 하겠습니다.

> Asset 이름은 꼭 Images 로 할 필요는 없습니다.

![ColorAsset1](/images/ColorAsset1.png)![ColorAsset2](/images/ColorAsset2.png)

`Colors.xc/assets` 폴더를 열어 보면 추가한 Color 들의 폴더가 보입니다. `Color이름.colorset` 형식으로 되어 있으며, 이제 이 형식을 가지고 만들어 보도록 하겠습니다. 자세한 내용은 아래에서 더 다루도록 하겠습니다.

## 프로젝트

기본적으로 `./Script -param1 param1 -param2 param2 ....` 형식으로 작성하고 싶어 Xcode 에서 Command Line Tool 프로젝트로 생성하여 시작해 보도록 하겠습니다.

![Project](/images/NewProject.png)

`./Script -param1 param1 -param2 param2 ....` 이 형식처럼 arguments 를 받으려면 보통은 `CommandLine.arguments` 를 사용하여 개발하지만, 제가 원하는 형식은 `-Key value` 형식이기 때문에 `UserDefaults` 를 확장하여 사용할 예정입니다.

먼저, 생각하고 있는 argument 는 아래와 같습니다.

* type: String, Image, Color 중 변환이 필요한 타입을 받는 argument
* path: 변환이 필요한 소스에 대한 path 를 지정합니다.
  * String: `.strings` 파일의 path
  * Image, Color: `.xc/assets` 파일의 path
* output: 변환이 된 `.swift` 파일의 path 를 지정합니다.
* enumName: auto complete 에 사용될 Enumeration 이름

> 저는 아래와 같이 동작하도록 만들어 보겠습니다.
>
> `./ResourceGenerator -type string -path ${SRCROOT}/xxxx.strings -output ${SRCROOT}/abc.swift -enumName Localizable`

```swift
extension UserDefaults {
    var arguments: (ResourceType, String, String, String) {
        guard let type = string(forKey: "type"),
              let resourceType = ResourceType(rawValue: type) else {
            fatalError("An type must be specified by \"-type\".")
        }
        guard let assetPath  = string(forKey: "path") else {
            fatalError("An asset catalog path must be specified by \"-path\".")
        }
        guard let outputPath = string(forKey: "output") else {
            fatalError("An output path must be specified by \"-output\".")
        }
        guard let enumName = string(forKey: "enumName") else {
            fatalError("An enum name must be specified by \"-enumName\".")
        }
        
        return (resourceType, assetPath, outputPath, enumName)
    }
}
```

위와 같이 `UserDefaults` 를 확장하여 arguments 를 Key, Value 형식으로 사용할 수 있게 되었습니다.

```swift
extension FileManager {
    func localizable(path: String) throws -> [String]? {
        let url = URL(fileURLWithPath: path)
        let localizables = try String(contentsOf: url, encoding: .utf8)
        let localizableArr = localizables.split(separator: "\n").map(String.init)
        
        var result: [String] = []
        let regex = try NSRegularExpression(pattern: "^[a-zA-Z]+(.*)[\\s]*=[\\s]*(.*)$")
        localizableArr.forEach {
            let resultRange = regex.firstMatch(in: $0, range: NSRange($0.startIndex..., in: $0))
            
            if let resultRange = resultRange {
                let matchString = String($0[Range(resultRange.range, in: $0)!])
                let matchkey = matchString.split(separator: "=").map {
                    $0.trimmingCharacters(in: .whitespaces)
                }
                
                result.append(matchkey[0])
            }
        }
        
        return result
    }
    
    func /assets(type: ResourceType, in/assetsPath path: String) throws -> [String]? {
        // let remove white spaces and dash from asset name. e.g My Image.imagesets, My-Image.imagesets into My_Image
        let normalize = { (asset: String) -> String in
            if let regex = try? NSRegularExpression(pattern: "\\s|-", options: .caseInsensitive){
                let range = NSRange(location: 0, length: asset.count)
                
                return regex.stringByReplacingMatches(in: asset, options: .withTransparentBounds, range: range, withTemplate: "_")
            }
            return asset
        }
        
        let subpaths = try subpathsOfDirectory(atPath: path)
        return subpaths.filter {
                $0.hasSuffix(type./assets)
            }
            .map {
                normalize(($0 as NSString).lastPathComponent.components(separatedBy: ".")[0])
            }
    }
}
```

`FileManager` 를 확장하여 각 타입에 맞는 파일을 파싱하는 것을 위와 같이 만들었습니다.

`String` 인 경우 정규식을 사용하여 Key 값을 가져오고, `Image`, `Color` 의 경우 폴더 내에 `.imageset`, `.colorset` 의 앞의 이름을 가져오도록 만들었습니다.

```swift
func makeSwift(_ type: ResourceType, _ target: [String], _ outputPath: String, _ enumName: String) -> Bool {
    let indent = "    " // indent is 4 spaces
    var file: String = ""
    
    // file header
    file += "// Generated by Resources.swift" + "\n"
    file += "\n"
    
    switch type {
    case .string:
        // +String extension
        file += "// MARK: - \(type.classStr) extension" + "\n\n"
        file += "extension \(type.classStr) {" + "\n"
        // Property
        file += indent + "init(localizableName: \(enumName)) {" + "\n"
        file += indent + indent + "self.init(NSLocalizedString(localizableName.rawValue, comment: \"\"))" + "\n"
        file += indent + "}" + "\n"
        file += "}" + "\n\n"
        // -String extension
    
        // +enum
        file += "// MARK: - " + enumName + "\n\n"
        file += "enum \(enumName): String {" + "\n"
        target.forEach {
            file += indent + "case \($0) = \"\($0)\"" + "\n"
        }
        file += "\n"
        file += indent + "var \(type.rawValue): \(type.classStr) {" + "\n"
        file += indent + indent + "return \(type.classStr)(localizableName: self.rawValue)!" + "\n"
        file += indent + "}" + "\n"
        file += "}" + "\n"
        // -enum
    case .image, .color:
        file += "import UIKit" + "\n"
        file += "\n"
        
        // +UIImage/UIColor extension
        file += "// MARK: - \(type.classStr) extension" + "\n\n"
        file += "extension \(type.classStr) {" + "\n"
        // Init
        file += indent + "convenience init!(assetName: \(enumName)) {" + "\n"
            file += indent + indent + "self.init(named: assetName.rawValue)" + "\n"
        file += indent + "}" + "\n"
        file += "}" + "\n\n"
        // -UIImage/UIColor extension
        
        // +enum
        file += "// MARK: - " + enumName + "\n\n"
        file += "enum \(enumName): String {" + "\n"
        target.forEach {
            file += indent + "case \($0) = \"\($0)\"" + "\n"
        }
        file += "\n"
        file += indent + "var \(type.rawValue): \(type.classStr) {" + "\n"
        file += indent + indent + "return \(type.classStr)(named: self.rawValue)!" + "\n"
        file += indent + "}" + "\n"
        file += "}" + "\n"
        // -enum
    }
    
    let data = file.data(using: String.Encoding.utf8, allowLossyConversion: false)
    return FileManager.default.createFile(atPath: outputPath, contents: data, attributes: nil)
}
```

위의 코드는 파일에서 원하는 Key 값을 통해 적절한 Swift 파일을 생성하는 내용입니다.

이제 한번 테스트를 진행해 보도록 하겠습니다.

```swift
// Strings
let (resourceType, path, output, enumName) = (ResourceType.string, "/Users/bojeong/Desktop/Boram/Test/ResourcesGenerator/ResourcesGenerator/Localizable.strings", "/Users/bojeong/Desktop/Boram/Test/ResourcesGenerator/ResourcesGenerator/Strings.swift", "Strings")

let fm = FileManager.default

guard let parse = fm.parser(type: resourceType, targetPath: path),
      !parse.isEmpty else {
    fatalError("\n[Error] No data is found and failed to export a file...\n")
}

let result = makeSwift(resourceType, parse, output, enumName)
let resultStr = result ? "Succeeded" : "Failed"
print("\n\(resultStr) to generate resource manager file at \(output).\n")
```

원래는 인자를 받아 실행이 되어야 하지만 테스트 용이기 때문에 강제로 값을 삽입하여 테스트를 진행 하도록 하겠습니다.

위와 같이 작성한 후 실행해 보면, 아래와 같이 `Strings.swift` 파일이 생성된 것을 확인할 수 있습니다.

![Strings](/images/Strings.png)

해당 파일을 열어보면

```swift
// Generated by Resources.swift

// MARK: - String extension

extension String {
    init(localizableName: Strings) {
        self.init(NSLocalizedString(localizableName.rawValue, comment: ""))
    }
}

// MARK: - Strings

enum Strings: String {
    case test = "test"
    case test2 = "test2"
    case test3 = "test3"

    var string: String {
        return String(localizableName: self.rawValue)!
    }
}
```

원하는 대로 잘 작성된 것을 확인할 수 있습니다.

이제 이 프로젝트가 앱 프로젝트 Build Phase 에서 실행될 수 있도록 빌드 결과물을 찾아 앱에 적용해 보도록 하겠습니다.

Build 를 하게되면 기본적으로 `/Build/Products/Debug/ProjectName` 에 위치하지만 스크립트를 통해 빌드가 완료되면 결과물을 프로젝트 폴더로 복사 하도록 해봅시다.

![BuildPhase](/images/RunScript.png)

빌드를 해봅시다.

![BuildResult](/images/BuildResult.png)

실행가능한 파일이 생성된 것을 볼 수 있습니다.

이제 이 파일을 앱에 원하는 위치에 복사를 하고 앱에서 Build Phase 에 Run Script 를 추가하여 Resource 를 관리하는 Swift 파일을 생성해 보겠습니다.

> 저는 여기서 Image Resource 에 대한 것만 테스트 하였습니다.

![AppRunScript](/images/AppRunScript.png)

모듈을 빌드 해봅시다.

![DomainImages](/images/DomainImages.png)

![ImagesSwift](/images/ImagesSwift.png)

위와 같이 파일이 잘 생성된 것을 확인할 수 있습니다.

위에 작성한 코드는 [GitHub](https://github.com/bbiguduk/ResourceGenerator) 에 올려져 있습니다.

감사합니다!