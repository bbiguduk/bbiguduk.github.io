---
title: "Implementing Modern Collection Views"
categories:
  - Swift
  - iOS
  - UIKit
  - Translation
tags:
  - DiffableDataSource
  - CompositionalLayout
---
> [Apple Developer](https://developer.apple.com/documentation/uikit/views_and_controls/collection_views/implementing_modern_collection_views)에 작성된 샘플 코드를 번역한 내용입니다.  
> [노션](https://sun-brain-f5e.notion.site/Implementing-Modern-Collection-Views-ab9be6de55b14760906a9cc1cefed17c)에도 동일하게 작성되어 있습니다.

앱에 혼합 레이아웃 (compositional layout) 을 가져오고 비교가능한 데이터 소스 (diffable data source) 로 유저 인터페이스 업데이트를 간소화 합니다.

## 개요 (**Overview)**

콜렉션 뷰는 유연한 시각적 배열로 데이터를 나타낼 수 있습니다. 이 샘플 앱은 레이아웃의 여러 타입을 어떻게 생성하고 콜렉션 뷰에서 데이터를 어떻게 관리하는지 보여줍니다. 샘플 앱은 2가지 기술에 집중합니다:

- 혼합, 유연, 그리고 빠른 콜렉션 뷰 레이아웃의 타입인 혼합 레이아웃 (compositional layout) 으로 모든 콘텐츠에 대한 시각적 배열을 만들 수 있습니다.
- 비교가능한 데이터 소스 (diffable data source), 콜렉션 뷰의 데이터와 유저 인터페이스에 대해 간편하고 효과적인 업데이트 관리를 제공하는 데이터 소스의 특별한 타입입니다.

### 샘플 코드 프로젝트 구성 (**Configure the Sample Code Project)**

Xcode 에서 샘플 코드 프로젝트를 실행하려면 먼저 iOS 또는 macOS 중 선택해야 합니다.

iOS 에서 예제를 보려면:

1. Modern Collection Views 타겟을 선택합니다.
2. Scheme 메뉴에서 앱을 실행하기 위해 iOS 시뮬레이터를 선택합니다.

macOS 에서 예제를 보려면:

1. Modern Collection Views Mac 타겟을 선택합니다.
2. Scheme 메뉴에서 My Mac 을 선택합니다.
3. 타겟에 Build Settings 에서 Signing & Capabilities > Signing Certificate 에서 Sign to Run Locally 를 선택합니다.
4. 앱을 실행하고 Example 메뉴에서 예제를 이동합니다.

여기서 표시된 코드 예제는 iOS 타겟이지만 macOS 타겟에 대한 `.swift` 파일은 macOS-equivalent 예제에서 찾을 수 있습니다.

### 그리드 레이아웃 생성 (**Create a Grid Layout)**

Grid 예제는 다섯개의 동일한 크기의 아이템의 열을 만들기 위해 분수의 크기 (fractional sizing) 을 사용하여 그리드 레이아웃을 어떻게 생성하는지 보여줍니다. `.fractionalWidth(0.2)` 를 사용하여 그룹의 너비의 20% 로 아이템들의 가로 그룹을 생성합니다. 다섯개 아이템의 각 열은 그리드를 생성하기 위해 하나의 섹션에서 여러번 반복됩니다.

```swift
let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(0.2),
                                     heightDimension: .fractionalHeight(1.0))
let item = NSCollectionLayoutItem(layoutSize: itemSize)

let groupSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                      heightDimension: .fractionalWidth(0.2))
let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize,
                                                 subitems: [item])

let section = NSCollectionLayoutSection(group: group)

let layout = UICollectionViewCompositionalLayout(section: section)
return layout
```

### 아이템에 공간 추가 (**Add Spacing Around Items)**

Inset Items Grid 예제는 Grid 예제의 레이아웃으로 구성되어 있고 [`contentInsets`](https://developer.apple.com/documentation/uikit/nscollectionlayoutitem/3199084-contentinsets) 프로퍼티를 사용하여 아이템 주변에 어떻게 공간을 추가하는지 보여줍니다. 여기서 이 프로퍼티는 각 아이템 주변에 동일한 공간을 적용합니다.

```swift
let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(0.2),
                                     heightDimension: .fractionalHeight(1.0))
let item = NSCollectionLayoutItem(layoutSize: itemSize)
item.contentInsets = NSDirectionalEdgeInsets(top: 5, leading: 5, bottom: 5, trailing: 5)
```

### 행 레이아웃 생성 (**Create a Column Layout)**

Two-Column Grid 예제는 [`horizontal(layoutSize:subitem:count:)`](https://developer.apple.com/documentation/uikit/nscollectionlayoutgroup/3213854-horizontal) 의 `count` 파라미터에서 아이템 갯수를 지정하여 2행 레이아웃 그룹을 어떻게 생성하는지 보여줍니다. 이 접근방식은 그룹에 포함된 아이템의 갯수를 명확하게 알 수 있습니다. 이러한 경우 `count` 파라미터는 `itemSize` 보다 우선순위가 높고 아이템 크기는 아이템의 지정된 갯수에 맞게 자동으로 계산됩니다.

```swift
let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                     heightDimension: .fractionalHeight(1.0))
let item = NSCollectionLayoutItem(layoutSize: itemSize)

let groupSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                      heightDimension: .absolute(44))
let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitem: item, count: 2)
let spacing = CGFloat(10)
group.interItemSpacing = .fixed(spacing)
```

### 섹션별 레이아웃 (**Display Distinct Layouts Per Section)**

Distinct Sections 예제는 동일한 콜렉션 뷰 레이아웃의 다른 섹션에서 어떻게 다른 레이아웃 배열을 나타내는지 보여줍니다. 섹션에서 다른 레이아웃을 생성하려면 섹션 프로바이더 (section provider) 를 가진 혼합 레이아웃 (compositional layout) 이 필요합니다. 섹션 프로바이더 (section provider) 에서 코드는 구성중인 섹션을 결정하고 각 섹션에 대해 다른 레이아웃을 나타내기 위해 섹션의 인덱스 (`sectionIndex`) 에 접근합니다.

```swift
let layout = UICollectionViewCompositionalLayout { (sectionIndex: Int,
    layoutEnvironment: NSCollectionLayoutEnvironment) -> NSCollectionLayoutSection? in

    guard let sectionLayoutKind = SectionLayoutKind(rawValue: sectionIndex) else { return nil }
    let columns = sectionLayoutKind.columnCount

		// 그룹은 요청된 행의 갯수에 맞게 아이템의 너비를 자동으로 계산합니다.
		// 그래서 widthDimension 은 무시됩니다.
    let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                         heightDimension: .fractionalHeight(1.0))
    let item = NSCollectionLayoutItem(layoutSize: itemSize)
    item.contentInsets = NSDirectionalEdgeInsets(top: 2, leading: 2, bottom: 2, trailing: 2)

    let groupHeight = columns == 1 ?
        NSCollectionLayoutDimension.absolute(44) :
        NSCollectionLayoutDimension.fractionalWidth(0.2)
    let groupSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                          heightDimension: groupHeight)
    let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitem: item, count: columns)

    let section = NSCollectionLayoutSection(group: group)
    section.contentInsets = NSDirectionalEdgeInsets(top: 20, leading: 20, bottom: 20, trailing: 20)
    return section
}
return layout
```

### 다른 환경에서 레이아웃 (**Display Distinct Layouts in Different Environments)**

Adaptive Sections 예제는 환경에 따른 레이아웃을 어떻게 생성하는지 보여줍니다. 이 예제에서 행의 갯수는 화면 사이즈에 따라 변경됩니다. 새로운 환경에 레이아웃을 생성하려면 섹션 프로바이더 (section provider) 가 있는 혼합 레이아웃 (compositional layout) 이 필요합니다. 섹션 프로바이더 (section provider) 에서 코드는 현재 레이아웃 환경에서 사용가능한 공간 (`layoutEnvironment.container.effectiveContentSize`) 에 접근하고 가능한 너비를 기반으로 다른 행의 갯수를 나타냅니다.

```swift
let layout = UICollectionViewCompositionalLayout {
    (sectionIndex: Int, layoutEnvironment: NSCollectionLayoutEnvironment) -> NSCollectionLayoutSection? in
    guard let layoutKind = SectionLayoutKind(rawValue: sectionIndex) else { return nil }

    let columns = layoutKind.columnCount(for: layoutEnvironment.container.effectiveContentSize.width)

    let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(0.2),
                                         heightDimension: .fractionalHeight(1.0))
    let item = NSCollectionLayoutItem(layoutSize: itemSize)
    item.contentInsets = NSDirectionalEdgeInsets(top: 2, leading: 2, bottom: 2, trailing: 2)

    let groupHeight = layoutKind == .list ?
        NSCollectionLayoutDimension.absolute(44) : NSCollectionLayoutDimension.fractionalWidth(0.2)
    let groupSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                           heightDimension: groupHeight)
    let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitem: item, count: columns)

    let section = NSCollectionLayoutSection(group: group)
    section.contentInsets = NSDirectionalEdgeInsets(top: 20, leading: 20, bottom: 20, trailing: 20)
    return section
}
return layout
```

### 아이템에 뱃지 추가 (**Add Badges to Items)**

Item Badges 예제는 콜렉션 뷰의 아이템에 뱃지와 같은 보조 뷰를 어떻게 추가하는지 보여줍니다. 뱃지에 대한 보조 아이템을 생성하고 보조 아이템을 전달하여 각 아이템 우측 최상단에 뱃지를 추가하고 생성합니다. 데이터 소스는 [`supplementaryViewProvider`](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource/3255142-supplementaryviewprovider) 로 각 뱃지를 구성합니다.

```swift
let badgeAnchor = NSCollectionLayoutAnchor(edges: [.top, .trailing], fractionalOffset: CGPoint(x: 0.3, y: -0.3))
let badgeSize = NSCollectionLayoutSize(widthDimension: .absolute(20),
                                      heightDimension: .absolute(20))
let badge = NSCollectionLayoutSupplementaryItem(
    layoutSize: badgeSize,
    elementKind: ItemBadgeSupplementaryViewController.badgeElementKind,
    containerAnchor: badgeAnchor)

let itemSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(0.25),
                                     heightDimension: .fractionalHeight(1.0))
let item = NSCollectionLayoutItem(layoutSize: itemSize, supplementaryItems: [badge])
item.contentInsets = NSDirectionalEdgeInsets(top: 5, leading: 5, bottom: 5, trailing: 5)
```

### 섹션에 헤더와 푸터 추가 (**Add Headers and Footers to Sections)**

Section Headers/Footers 예제는 콜렉션 뷰의 각 섹션에 헤더와 푸터를 어떻게 추가하는지 보여줍니다. 헤더와 푸터를 나타내는 아이템을 생성하고 섹션의 [`boundarySupplementaryItems`](https://developer.apple.com/documentation/uikit/nscollectionlayoutsection/3199089-boundarysupplementaryitems) 로 아이템을 설정합니다.

```swift
let headerFooterSize = NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                             heightDimension: .estimated(44))
let sectionHeader = NSCollectionLayoutBoundarySupplementaryItem(
    layoutSize: headerFooterSize,
    elementKind: SectionHeadersFootersViewController.sectionHeaderElementKind, alignment: .top)
let sectionFooter = NSCollectionLayoutBoundarySupplementaryItem(
    layoutSize: headerFooterSize,
    elementKind: SectionHeadersFootersViewController.sectionFooterElementKind, alignment: .bottom)
section.boundarySupplementaryItems = [sectionHeader, sectionFooter]
```

예제에서 헤더와 푸터에 콘텐츠와 디자인을 구성하기 위해 `SupplementaryRegistration` 을 사용합니다.

```swift
let headerRegistration = UICollectionView.SupplementaryRegistration
<TitleSupplementaryView>(elementKind: SectionHeadersFootersViewController.sectionHeaderElementKind) {
    (supplementaryView, string, indexPath) in
    supplementaryView.label.text = "\(string) for section \(indexPath.section)"
    supplementaryView.backgroundColor = .lightGray
    supplementaryView.layer.borderColor = UIColor.black.cgColor
    supplementaryView.layer.borderWidth = 1.0
}
```

콜렉션 뷰는 비교가능한 데이터 소스 (diffable data source) 의 [`supplementaryViewProvider`](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource/3255142-supplementaryviewprovider) 에서 구성된 헤더와 푸터를 얻기 위해 위에서 설정한 `SupplementaryRegistration` 을 사용합니다.

```swift
dataSource.supplementaryViewProvider = { (view, kind, index) in
    return self.collectionView.dequeueConfiguredReusableSupplementary(
        using: kind == SectionHeadersFootersViewController.sectionHeaderElementKind ? headerRegistration : footerRegistration, for: index)
}
```

### 섹션에 헤더 고정 (**Pin Section Headers to Sections)**

Pinned Section Headers 예제는 섹션에 헤더를 어떻게 고정하는지 보여줍니다. 이 방법은 헤더는 스크롤 하는 동안 섹션에 보여지고 푸터는 제자리에 유지됩니다. 이 예제는 헤더의 [`pinToVisibleBounds`](https://developer.apple.com/documentation/uikit/nscollectionlayoutboundarysupplementaryitem/3199044-pintovisiblebounds) 프로퍼티를 `true` 로 하고 [`zIndex`](https://developer.apple.com/documentation/uikit/nscollectionlayoutsupplementaryitem/3199114-zindex) 의 값을 `1` 보다 크게하여 섹션을 스크롤 하는 동안 항상 최상단에 보여지게 압니다.

```swift
let sectionHeader = NSCollectionLayoutBoundarySupplementaryItem(
    layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                      heightDimension: .estimated(44)),
    elementKind: PinnedSectionHeaderFooterViewController.sectionHeaderElementKind,
    alignment: .top)
let sectionFooter = NSCollectionLayoutBoundarySupplementaryItem(
    layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                      heightDimension: .estimated(44)),
    elementKind: PinnedSectionHeaderFooterViewController.sectionFooterElementKind,
    alignment: .bottom)
sectionHeader.pinToVisibleBounds = true
sectionHeader.zIndex = 2
section.boundarySupplementaryItems = [sectionHeader, sectionFooter]
```

예제는 헤더와 푸터의 콘텐츠와 디자인을 구성하기 위해 보조 등록 (supplementary registration) 을 사용합니다. 콜렉션 뷰는 비교가능한 데이터 소스 (diffable data source) 의 [`supplementaryViewProvider`](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource/3255142-supplementaryviewprovider) 에서 구성된 헤더와 푸터를 얻기위해 보조 등록 (supplementary registration) 을 사용합니다.

```swift
dataSource.supplementaryViewProvider = { (view, kind, index) in
    return self.collectionView.dequeueConfiguredReusableSupplementary(
        using: kind == PinnedSectionHeaderFooterViewController.sectionHeaderElementKind ? headerRegistration : footerRegistration, for: index)
}
```

### 섹션 백그라운드 꾸미기 (**Decorate Sections with Backgrounds)**

Section Background Decoration 예제는 섹션 배경을 추가하여 섹션을 어떻게 구분하는지 보여줍니다. [`background(elementKind:)`](https://developer.apple.com/documentation/uikit/nscollectionlayoutdecorationitem/3199051-background) 를 사용하여 아이템을 꾸며 섹션 백그라운드를 생성합니다. 그런 다음에 섹션의 [`decorationItems`](https://developer.apple.com/documentation/uikit/nscollectionlayoutsection/3199091-decorationitems) 프로퍼티로 아이템 꾸미기를 설정합니다.

```swift
let sectionBackgroundDecoration = NSCollectionLayoutDecorationItem.background(
    elementKind: SectionDecorationViewController.sectionBackgroundDecorationElementKind)
sectionBackgroundDecoration.contentInsets = NSDirectionalEdgeInsets(top: 5, leading: 5, bottom: 5, trailing: 5)
section.decorationItems = [sectionBackgroundDecoration]
```

그리고 다음 코드와 같이 `register(_:forDecorationViewOfKind:)` 를 사용하여 백그라운드 뷰를 등록합니다.

```swift
let layout = UICollectionViewCompositionalLayout(section: section)
layout.register(
    SectionBackgroundDecorationView.self,
    forDecorationViewOfKind: SectionDecorationViewController.sectionBackgroundDecorationElementKind)
return layout
```

### 그룹을 중첩하여 커스텀 레이아웃을 생성 (**Create Custom Layouts by Nesting Groups)**

Nested Groups 예제는 그룹 안에 그룹을 중첩하여 유연한 레이아웃을 어떻게 생성하는지 보여줍니다. 2개의 아이템을 가지는 세로 그룹을 생성하고 가로 부모 그룹에 하나의 아이템을 가지는 그룹을 혼합합니다.

```swift
let leadingItem = NSCollectionLayoutItem(
    layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(0.7),
                                      heightDimension: .fractionalHeight(1.0)))
leadingItem.contentInsets = NSDirectionalEdgeInsets(top: 10, leading: 10, bottom: 10, trailing: 10)

let trailingItem = NSCollectionLayoutItem(
    layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                      heightDimension: .fractionalHeight(0.3)))
trailingItem.contentInsets = NSDirectionalEdgeInsets(top: 10, leading: 10, bottom: 10, trailing: 10)
let trailingGroup = NSCollectionLayoutGroup.vertical(
    layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(0.3),
                                      heightDimension: .fractionalHeight(1.0)),
    subitem: trailingItem, count: 2)

let nestedGroup = NSCollectionLayoutGroup.horizontal(
    layoutSize: NSCollectionLayoutSize(widthDimension: .fractionalWidth(1.0),
                                      heightDimension: .fractionalHeight(0.4)),
    subitems: [leadingItem, trailingGroup])
```

### 섹션 가로 스크롤 (**Scroll Sections Horizontally)**

Orthogonal Sections 예제는 세로 스크롤 레이아웃에서 가로 스크롤하는 섹션을 어떻게 생성하는지 보여줍니다. 섹션의 기본 레이아웃은 수직으로 콘텐츠를 구성하기 때문에 섹션의 [`orthogonalScrollingBehavior`](https://developer.apple.com/documentation/uikit/nscollectionlayoutsection/3199094-orthogonalscrollingbehavior) 프로퍼티를 [`UICollectionLayoutSectionOrthogonalScrollingBehavior.none`](https://developer.apple.com/documentation/uikit/uicollectionlayoutsectionorthogonalscrollingbehavior/none) 이외의 값으로 설정합니다. 이 케이스에서는 기본적으로 세로 스크롤 레이아웃이기 때문에 섹션은 가로로 스크롤 됩니다.

```swift
section.orthogonalScrollingBehavior = .continuous
```

### 가로 스크롤과 페이징 동작 선택 (**Choose Horizontal Scrolling and Paging Behavior)**

Orthogonal Section Behaviors 예제는 [`UICollectionLayoutSectionOrthogonalScrollingBehavior`](https://developer.apple.com/documentation/uikit/uicollectionlayoutsectionorthogonalscrollingbehavior) 에 대한 옵션을 보여줍니다. 섹션의 레이아웃은 다른 스크롤 동작을 보여주고 스크롤 옵션과 페이징 옵션의 차이점을 보여줍니다. 이 경우에 기본적으로 레이아웃은 세로 스크롤 이므로 섹션은 자체는 가로로 스크롤 됩니다.

```swift
case continuous, continuousGroupLeadingBoundary, paging, groupPaging, groupPagingCentered, none
func orthogonalScrollingBehavior() -> UICollectionLayoutSectionOrthogonalScrollingBehavior {
    switch self {
    case .none:
        return UICollectionLayoutSectionOrthogonalScrollingBehavior.none
    case .continuous:
        return UICollectionLayoutSectionOrthogonalScrollingBehavior.continuous
    case .continuousGroupLeadingBoundary:
        return UICollectionLayoutSectionOrthogonalScrollingBehavior.continuousGroupLeadingBoundary
    case .paging:
        return UICollectionLayoutSectionOrthogonalScrollingBehavior.paging
    case .groupPaging:
        return UICollectionLayoutSectionOrthogonalScrollingBehavior.groupPaging
    case .groupPagingCentered:
        return UICollectionLayoutSectionOrthogonalScrollingBehavior.groupPagingCentered
    }
}
```

### 콜렉션 뷰에서 데이터 업데이트 (**Update Data in a Collection View)**

Mountains Search 예제는 사용자가 데이터를 필터링할 때 콜렉션 뷰에서 어떻게 데이터와 유저 인터페이스를 업데이트 하는지 보여줍니다. 산 이름의 목록을 보여주고 산의 이름을 기반으로 서치바에서 필터링를 할 수 있습니다.

`mountainsCollectionView` 인 콜렉션 뷰는 각 산의 데이터로 부터 생성된 아이템을 가진 하나의 섹션으로 구성되어 있습니다. 이 예제는 `MountainsController` 에 정의된 `Mountain` 구조체로 각 데이터를 가지고 있습니다. 데이터를 관리하기 위해 이 예제에서는 `Mountain` 타입의 아이템과 하나의 섹션을 가지는 비교가능한 데이터 소스 (diffable data source) 를 사용합니다. 비교가능한 데이터 소스 (diffable data source) 가 생성될 때 `mountainsCollectionView` 에 연결하고 콜렉션 뷰에서 산의 이름을 보여줄 `LabelCell` 의 셀 타입을 등록합니다. 그런 다음에 산의 이름을 가진 셀을 구성합니다.

`performQuery(with:)` 메서드는 데이터와 유저 인터페이스를 업데이트 합니다. 이 메서드는 현재 타이핑 된 필터 텍스트와 해당 텍스트를 포함하는 산의 목록을 생성합니다. 그런 다음에 필터링 된 새로운 스냅샷 (snapshot) 을 사용하여 데이터를 나타냅니다. 스냅샷 (snapshot) 은 이전과 동일하게 하나의 섹션을 포함하지만 모든 산을 나타내는 아이템이 아닌 필터링 된 산의 아이템만 포함합니다.

그런 다음에 이 메서드는 비교가능한 데이터 소스 (diffable data source) 에 스냅샷 (snapshot) 의 데이터를 적용하기 위해 [`apply(_:animatingDifferences:completion:)`](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource/3375795-apply) 을 호출합니다. 비교가능한 데이터 소스 (diffable data source) 는 이전 상태와 새로운 상태의 차이점을 계산하여 새로운 상태의 데이터를 스냅샷 (snapshot) 에 저장하고 유저 인터페이스를 트리거하여 새로운 상태를 나타냅니다.

```swift
func performQuery(with filter: String?) {
    let mountains = mountainsController.filteredMountains(with: filter).sorted { $0.name < $1.name }

    var snapshot = NSDiffableDataSourceSnapshot<Section, MountainsController.Mountain>()
    snapshot.appendSections([.main])
    snapshot.appendItems(mountains)
    dataSource.apply(snapshot, animatingDifferences: true)
}
```

### 여러 섹션에서 데이터 업데이트 (**Update Data in Multiple Sections)**

Settings: Wi-Fi 예제는 여러 종류의 섹션과 아이템을 사용하는 테이블 뷰에서 데이터와 유저 인터페이스를 어떻게 업데이트 하는지 보여줍니다. iOS 설정에 Wi-Fi 페이지를 재생성하고 사용자가 Wi-Fi 스위치를 on / off 할 수 있고 현재 Wi-Fi 네트워크를 볼 수 있습니다.

`updateUI(animated:)` 메세드는 Wi-Fi 사용여부를 기반으로 섹션과 아이템을 나타냅니다. Wi-Fi 가 사용 가능하지 않다면 이 메서드는 `.config` 섹션과 아이템 만 스냅샷 (snapshot) 에 추가합니다. Wi-Fi 가 사용 가능하다면 이 메서드는 `.networks` 섹션과 아이템을 추가합니다.

```swift
let configItems = configurationItems.filter { !($0.type == .currentNetwork && !controller.wifiEnabled) }

currentSnapshot = NSDiffableDataSourceSnapshot<Section, Item>()

currentSnapshot.appendSections([.config])
currentSnapshot.appendItems(configItems, toSection: .config)

if controller.wifiEnabled {
    let sortedNetworks = controller.availableNetworks.sorted { $0.name < $1.name }
    let networkItems = sortedNetworks.map { Item(network: $0) }
    currentSnapshot.appendSections([.networks])
    currentSnapshot.appendItems(networkItems, toSection: .networks)
}

self.dataSource.apply(currentSnapshot, animatingDifferences: animated)
```

### 점진적으로 데이터 업데이트 (**Update Data Incrementally)**

Insertion Sort Visualization 예제는 데이터가 초기 상태에서 최종 상태로 변경될 때 진행률을 표시하여 어떻게 데이터를 점진적으로 업데이트 하는지 보여줍니다. 무작위 순서로 색깔을 보여주고 순서대로 정렬합니다.

이 예제와 다른 비교가능한 데이터 소스 (diffable data source) 의 가장 큰 차이점은 이 예제는 데이터의 상태를 업데이트 하기 위해 빈 스냅샷 (snapshot) 을 생성하지 않습니다. 대신에 `performSortStep()` 메서드는 `dataSource.snapshot()` 을 사용하여 콜렉션 뷰의 데이터의 현재 상태를 조회합니다. 그런 다음에 스냅샷 (snapshot) 의 한 부분만 수정하여 단계별로 정렬합니다.

```swift
// 데이터 소스에서 UI 의 현재 상태를 얻습니다.
var updatedSnapshot = dataSource.snapshot()

// 각 섹션에서 필요하면 다음 정렬 단계를 수행합니다.
updatedSnapshot.sectionIdentifiers.forEach {
    let section = $0
    if !section.isSorted {
				
				// 정렬 알고리즘 단계입니다.
        section.sortNext()
        let items = section.values

				// 새로운 정렬된 아이템으로 섹션의 아이템을 대신합니다.
        updatedSnapshot.deleteItems(items)
        updatedSnapshot.appendItems(items, toSection: section)

        sectionCountNeedingSort += 1
    }
}
```

### 간단한 리스트 레이아웃 생성 (**Create a Simple List Layout)**

Simple List 예제는 모든 화면 사이즈에 적용되는 기본 리스트 레이아웃을 어떻게 생성하는지 보여줍니다. 시스템에 정의된 리스트 모양 중 하나로 구성합니다. 그런 다음에 [`list(using:)`](https://developer.apple.com/documentation/uikit/uicollectionviewcompositionallayout/3600951-list) 에 구성을 넘겨 리스트 레이아웃을 생성합니다. 이 방법은 하나의 섹션에 목록 스타일로 된 혼합 레이아웃 (compositional layout) 을 생성합니다.

```swift
let config = UICollectionLayoutListConfiguration(appearance: .insetGrouped)
return UICollectionViewCompositionalLayout.list(using: config)
```

### 리스트 모양 선택 (**Choose a List Appearance)**

List Appearances 예제는 [`UICollectionLayoutListConfiguration.Appearance`](https://developer.apple.com/documentation/uikit/uicollectionlayoutlistconfiguration/appearance) 에 대한 각 옵션을 보여줍니다. 네비게이션 바에서 이름을 탭하면 각 리스트 스타일로 변경됩니다. 각 리스트는 리스트의 펼침 / 접음 을 나타내기 위해 [`UICollectionLayoutListConfiguration.HeaderMode.firstItemInSection`](https://developer.apple.com/documentation/uikit/uicollectionlayoutlistconfiguration/headermode/firstiteminsection) 헤더모드를 사용합니다.

```swift
var config = UICollectionLayoutListConfiguration(appearance: self.appearance)
config.headerMode = .firstItemInSection
```

### 리스트 셀 커스텀 (**Customize List Cells)**

List with Custom Cells 예제는 커스텀 리스트 셀을 어떻게 구성하는지 보여줍니다. 예제는 하나의 셀에 여러 서브뷰를 합친 [`UICollectionViewListCell`](https://developer.apple.com/documentation/uikit/uicollectionviewlistcell) 의 커스텀 하위클래스인 `CustomListCell` 을 사용합니다. 콘텐츠 구성을 사용하여 뷰의 모양과 콘텐츠를 설정합니다.

`updateConfiguration(using:)` 메서드는 셀의 초기 모양과 콘텐츠를 설정합니다. 시스템은 셀의 구성상태가 변경될 때마다 셀의 모양을 새로운 상태로 업데이트 하기 위해 호출합니다. 리스트 콘텐츠를 구성하기위해 현재상태에 대한 기본 콘텐츠 구성을 가져옵니다.

```swift
var content = defaultListContentConfiguration().updated(for: state)
```

그런 다음에 구성의 값을 커스텀하고 `listContentView` 의 `configuration` 프로퍼티에 구성을 할당합니다. 

이미지뷰와 라벨의 경우 `updateConfiguration(using:)` 메서드는 현재 상태에 대한 셀 기본 구성값을 조회하고 `valueConfiguration` 에 저장합니다. 시스템 스타일의 일관성을 유지하기 위해 커스텀 뷰의 구성에서 미리 구성된 기본 스타일과 메트릭스를 복사합니다.

```swift
categoryIconView.tintColor = valueConfiguration.imageProperties.resolvedTintColor(for: tintColor)
categoryIconView.preferredSymbolConfiguration = .init(font: valueConfiguration.secondaryTextProperties.font, scale: .small)
```

콜렉션 뷰에 커스텀 셀 하위클래스를 등록하기위해 이 예제에서는 셀 등록 (cell registration) 을 사용합니다. 셀 등록 (cell registration) 은 해당 아이템의 데이터를 각 셀에 구성합니다. 또한 확장을 나타내는 (disclosure indicator) 셀 악세서리 (cell accessory) 를 추가합니다.

```swift
let cellRegistration = UICollectionView.CellRegistration<CustomListCell, Item> { (cell, indexPath, item) in
    cell.updateWithItem(item)
    cell.accessories = [.disclosureIndicator()]
}
```

비교가능한 데이터 소스 (diffable data source) 는 셀이 덱 (dequeue) 할 때 셀 등록 (cell registration) 을 사용합니다.

```swift
return collectionView.dequeueConfiguredReusableCell(using: cellRegistration, for: indexPath, item: item)
```

### 여러 섹션 타입의 레이아웃 (**Build a Layout with Multiple Section Types)**

Emoji Explorer 예제는 섹션에 여러 타입의 혼합 레이아웃 (compositional layout) 을 어떻게 생성하는지 보여줍니다. 예제는 하나의 혼합 레이아웃 (compositional layout) 에서 가로 스크롤 섹션, 개요 섹션, 그리고 목록 섹션을 포함합니다. `createLayout()` 메서드는 각 섹션을 제공하는 섹션 프로바이더 (section provider) 를 정의합니다.

가장 상단 섹션은 [`orthogonalScrollingBehavior`](https://developer.apple.com/documentation/uikit/nscollectionlayoutsection/3199094-orthogonalscrollingbehavior) 프로퍼티를 설정하여 가로 스크롤이 가능합니다.

```swift
section.orthogonalScrollingBehavior = .continuousGroupLeadingBoundary
```

개요 섹션은 [`UICollectionLayoutListConfiguration.Appearance.sidebar`](https://developer.apple.com/documentation/uikit/uicollectionlayoutlistconfiguration/appearance/sidebar) 리스트 모양을 사용합니다.  데이터 구조 계층을 생성하기위해 섹션 스냅샷 (snapshot) 을 사용합니다.

```swift
let rootItem = Item(title: String(describing: category), hasChildren: true)
outlineSnapshot.append([rootItem])
let outlineItems = category.emojis.map { Item(emoji: $0) }
outlineSnapshot.append(outlineItems, to: rootItem)
```

리스트 섹션은 [`UICollectionLayoutListConfiguration.Appearance.insetGrouped`](https://developer.apple.com/documentation/uikit/uicollectionlayoutlistconfiguration/appearance/insetgrouped) 리스트 모양을 사용합니다. 이 섹션에 대한 구성은 즐겨찾기를 표기하기 위해 각 셀에 스와이프 액션을 추가합니다.

```swift
configuration.leadingSwipeActionsConfigurationProvider = { [weak self] (indexPath) in
    guard let self = self else { return nil }
    guard let item = self.dataSource.itemIdentifier(for: indexPath) else { return nil }
    return self.leadingSwipeActionConfigurationForListCellItem(item)
}
```

각 섹션은 셀의 타입을 구성하기위한 셀 등록 (cell registration) 을 가집니다. 콜렉션 뷰는 각 섹션에 나타낼 구성된 셀을 덱 (dequeue) 하기위해 이러한 등록을 사용합니다.

```swift
switch section {
case .recents:
    return collectionView.dequeueConfiguredReusableCell(using: gridCellRegistration, for: indexPath, item: item.emoji)
case .list:
    return collectionView.dequeueConfiguredReusableCell(using: listCellRegistration, for: indexPath, item: item)
case .outline:
    if item.hasChildren {
        return collectionView.dequeueConfiguredReusableCell(using: outlineHeaderCellRegistration, for: indexPath, item: item.title!)
    } else {
        return collectionView.dequeueConfiguredReusableCell(using: outlineCellRegistration, for: indexPath, item: item.emoji)
    }
}
```

### 값 셀 리스트 생성 (**Create a Value Cell List)**

Emoji Explorer - List 예제는 값 셀 스타일을 사용하는 셀은 리스트에서 어떻게 생성되는지 보여줍니다. 리스트에 각 셀에 기본값 셀 스타일을 사용하여 콘텐츠 구성을 적용합니다.

```swift
var contentConfiguration = UIListContentConfiguration.valueCell()
contentConfiguration.text = emoji.text
contentConfiguration.secondaryText = String(describing: emoji.category)
cell.contentConfiguration = contentConfiguration
```
