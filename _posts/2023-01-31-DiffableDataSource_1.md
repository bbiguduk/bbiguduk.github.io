---
title: "Updating Collection Views Using Diffable Data Sources"
categories:
  - Swift
  - iOS
  - UIKit
  - Translation
tags:
  - DiffableDataSource
---
> [Apple Developer](https://developer.apple.com/documentation/uikit/views_and_controls/collection_views/updating_collection_views_using_diffable_data_sources)에 작성된 샘플 코드를 번역한 내용입니다.  
> [노션](https://sun-brain-f5e.notion.site/Updating-Collection-Views-Using-Diffable-Data-Sources-df43341db2174423b5c5ff6d1a2924c1)에도 동일하게 작성되어 있습니다.

식별자 (identifier) 를 포함한 비교가능한 데이터 소스 (diffable data source) 를 사용하여 콜렉션 뷰의 데이터를 표시하고 업데이트 하는데 간소화 합니다.

## 개요 (**Overview)**

*콜렉션 뷰* 는 섹션 (section) 과 아이템 (item) 의 형식으로 데이터를 표시하고 콜렉션 뷰로 데이터를 나타내는 앱은 콜렉션 뷰에 섹션과 아이템을 삽입할 수 있습니다. 앱은 섹션과 아이템을 삭제 또는 이동에 대한 처리가 필요할 수도 있습니다. 예를 들어 샘플 프로젝트에 있는 앱은 콜렉션 뷰로 레시피를 나타내고 앱을 이용하여 레시피를 추가 및 삭제할 수 있고 레시피를 즐겨찾기로 표시할 수 있습니다. 이러한 동작을 위해 샘플 앱은 콜렉션 뷰에서 데이터 삽입, 삭제, 이동, 그리고 업데이트를 처리합니다.

앱에서 콜렉션 뷰를 사용할 때, [`UICollectionViewDataSource`](https://developer.apple.com/documentation/uikit/uicollectionviewdatasource) 프로토콜을 채택한 커스텀한 데이터 소스를 생성할 수 있습니다. 현재 콜렉션 뷰에서 정보를 최신으로 유지하기 위해 무슨 데이터가 변경되었는지 확인하고 변경사항을 기반으로 업데이트를 수행해야 하는데 이러한 프로세스는 삽입, 삭제, 그리고 이동을 신중하게 다뤄야 합니다.

이러한 복잡한 프로세스를 피하기위해 샘플 앱은 [`UICollectionViewDiffableDataSource`](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource) 객체를 사용합니다. *비교가능한 데이터 소스 (diffable data source)* 는 콜렉션 뷰에서 각 섹션과 아이템을 식별할 수 있는 식별자를 저장합니다. 이런 식별자는 안전하고, 변경되지 않음을 의미합니다. 반대로 `UICollectionViewDataSource` 를 준수하는 커스텀한 데이터 소스는 안전하지 않은 *인덱스 (indice)* 와 *인덱스 경로 (index path)* 를 사용합니다. 데이터 소스가 콜렉션 뷰의 콘텐츠를 추가, 삭제, 그리고 재정렬에 의해 변경될 수 있는 섹션과 아이템의 위치를 나타냅니다. 그러나 식별자를 가진 비교가능한 데이터 소스 (diffable data source) 는 콜렉션 뷰 내에 위치에 대한 정보가 없어도 섹션 또는 아이템을 참조할 수 있습니다.

> **NOTE**
이 샘플은 데이터를 나타내기 위해 콜렉션 뷰를 사용하지만 테이블 뷰 또한 동일한 컨셉입니다. 테이블 뷰로 비교가능한 데이터 소스를 사용하는 것에 대한 자세한 내용은 [`UITableViewDiffableDataSource`](https://developer.apple.com/documentation/uikit/uitableviewdiffabledatasource) 를 참조하세요.
> 

식별자로 값을 사용하려면 데이터 타입은 [`Hashable`](https://developer.apple.com/documentation/swift/hashable) 프로토콜을 준수해야 합니다. 해시를 사용하면 빠르고 효과적인 조회를 제공하는 [`Set`](https://developer.apple.com/documentation/swift/set), [`Dictionary`](https://developer.apple.com/documentation/swift/dictionary), 그리고 스냅샷 (snapshot) ([`NSDiffableDataSourceSnapshot`](https://developer.apple.com/documentation/uikit/nsdiffabledatasourcesnapshot) 과 [`NSDiffableDataSourceSectionSnapshot`](https://developer.apple.com/documentation/uikit/nsdiffabledatasourcesectionsnapshot) 의 인스턴스) 과 같은 키로 값을 사용하는 데이터 콜렉션을 허용합니다. `Hashable` 타입은 `Equatable` 프로토콜도 준수하기 때문에 식별자는 동등성에 대한 구현도 필요합니다. 더 자세한 내용은 `Equatable` 을 참조하세요.

식별자는 `Hashable` 과 `Equatable` 을 준수하기 때문에 비교가능한 데이터 소스 (diffable data source) 는 현재 스냅샷 (snapshot) 과 다른 스냅샷 (snapshot) 을 비교할 수 있습니다. 이러면 콜렉션 뷰 내에서 섹션과 아이템의 차이점을 기반으로 삽입, 삭제, 그리고 이동이 가능하므로 일괄 업데이트 하는 코든느 필요하지 않습니다.

> **Important**
동일한 두 식별자는 항상 같은 해시값을 가져야 합니다. 그러나 그 반대는 같을 필요가 없습니다. 동일한 해시 값을 가진 두 값은 반드시 같을 필요가 없습니다. 이러한 상황을 *해시 충돌 (hash collision)* 이라고 합니다. 효율성을 증가시키려면 동일하지 않은 식별자는 다른 해시 값을 가지도록 해야 합니다. 피할 수 없는 해시 충돌 (hash collision) 은 괜찮지만 최소한의 충돌 수를 유지해야 합니다. 그렇지 않으면 데이터 콜렉션에서 조회의 성능은 저하됩니다.
> 

### 비교가능한 데이터 소스 정의 (**Define the Diffable Data Source)**

샘플 프로젝트에서 `RecipeListViewController` 는 콜렉션 뷰로 레시피의 목록을 나타냅니다. 컨트롤러는 레시피를 나타내기 전에 비교가능한 데이터 소스 (diffable data source) 에 저장하기 위한 인스턴스 변수 (instance variable) 에 정의합니다.

```swift
private var recipeListDataSource: UICollectionViewDiffableDataSource<RecipeListSection, Recipe.ID>!
```

`RecipeListViewController` 는 섹션 식별자 타입으로 `RecipeListSection`, 그리고 아이템 식별자 타입으로 `Recipe.ID` 로 `recipeListDataSource` 로 선언합니다. 이러한 식별자 타입은 값이 포함된 타입을 데이터 소스에게 알려줍니다.

`recipeListDataSource` 는 `RecipeListSection` 을 사용하는 섹션 식별자 타입은 `Int` 타입의 원시값을 가지는 열거형 (enumeration) 입니다 (Swift 에서 `Int` 는 hashable 입니다). 각 열거형 케이스는 콜렉션 뷰의 섹션을 나타냅니다. 샘플에서 레시피의 목록을 나타내는 `main` 인 하나의 섹션만 있습니다.

```swift
private enum RecipeListSection: Int {
    case main
}
```

아이템 식별자 타입은 `recipeListDataSource` 에서 `Recipe.ID` 를 사용합니다. 이 타입은 `Recipe` 구조체에 있으며, 아래와 같이 정의되어 있습니다:

```swift
struct Recipe: Identifiable, Codable {
    var id: Int
    var title: String
    var prepTime: Int   // In seconds.
    var cookTime: Int   // In seconds.
    var servings: String
    var ingredients: String
    var directions: String
    var isFavorite: Bool
    var collections: [String]
    fileprivate var addedOn: Date? = Date()
    fileprivate var imageNames: [String]
}
```

이 구조체는 `id` 프로퍼티를 포함해야 하는 구조체를 요구하는 [`Identifiable`](https://developer.apple.com/documentation/swift/identifiable) 프로토콜을 준수합니다. `Identifiable` 을 준수함으로써, `Recipe` 구조체는 구조체에서 `id` 프로퍼티의 구현을 기반으로 연관된 타입 `ID` 를 자동으로 사용할 수 있습니다. 그리고 이 타입은 hashable 이기 때문에 샘플 앱은 아이템 식별자 타입으로 `Recipe.ID` 를 사용할 수 있습니다.

> **NOTE**
`Recipe` 구조체는 `Hashable` 프로토콜을 준수하지 않습니다. 레시피 구조체는 완벽하지 않고 비교가능한 데이터 소스 (diffable data source) 와 스냅샷 (snapshot) 에 저장된 아이템은 레시피 *식별자(identifier)* (각 레시피를 제공하기 위한 `Recipe.ID` 값) 이므로 hashable 하지 않아도 됩니다.
> 

`recipeListDataSource` 에 대해 아이템 식별자 타입으로 `Recipe.ID` 를 사용하는 것은 데이터 소스와 데이터 소스에 적용 된 모든 스냅샷 (snapshot) 에 완벽한 레시피 데이터가 아닌 `Recipe.ID` 값만 포함된다는 의미입니다. 이러한 접근 방식은 식별자 타입은 간단하고 hashable 타입이므로 콜렉션 뷰에서 레시피를 나타낼 때 최상의 성능을 위해 비교가능한 데이터 소스 (diffable data source) 를 최적화 합니다.

### 비교가능한 데이터 소스 구성 (**Configure the Diffable Data Source)**

비교가능한 데이터 소스 (diffable data source) 에서 데이터를 가지는 콜렉션 뷰가 나타나기 전에 샘플 앱은 데이터 소스를 구성합니다. 앱은 [`UICollectionViewDiffableDataSource`](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource) 인스턴스를 생성하고 콜렉션 뷰에 대한 셀을 구성하고 반환하는 클로저인 *셀 프로바이더 (cell provider)* 를 설정합니다.

`RecipeListViewController` 는 `configureDataSource()` 라는 이름의 메서드에서 `recipeListDataSource` 를 구성합니다. 뷰 컨트롤러 (view controller) 는 이 메서드를 [`viewDidLoad()`](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621495-viewdidload) 메서드에서 호출합니다.

`configureDataSource()` 메서드는 셀 등록 (cell registration) 을 호출하고 레시피 데이터로 각 셀을 구성하는 핸들러 클러저를 제공합니다.

> **NOTE**
셀 등록 (cell registration) 에 대한 아이템 타입은 비교가능한 데이터 소스 (diffable data source) 에서 사용하는 아이템 식별자 타입과 동일하지 않아도 됩니다.
> 

다음으로 `configureDataSource()` 는 [`UICollectionViewDiffableDataSource`](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource) 의 인스턴스를 생성하고 셀 프로바이더 (cell provider) 클로저를 정의합니다. 클로저는 레시피의 식별자를 받습니다. 그런다음 데이터 스토어에서 식별자를 이용하여 레시피를 검색하고 셀 등록 (cell registration) 의 핸들러 클로저에 레시피 구조체를 전달합니다.

```swift
private func configureDataSource() {
		// 비교가능한 데이터 소스 (diffable data source) 에서 사용 할
		// 셀 등록 (cell registration) 을 생성합니다.
    let recipeCellRegistration = UICollectionView.CellRegistration<UICollectionViewListCell, Recipe> { cell, indexPath, recipe in
        var contentConfiguration = UIListContentConfiguration.subtitleCell()
        contentConfiguration.text = recipe.title
        contentConfiguration.secondaryText = recipe.subtitle
        contentConfiguration.image = recipe.smallImage
        contentConfiguration.imageProperties.cornerRadius = 4
        contentConfiguration.imageProperties.maximumSize = CGSize(width: 60, height: 60)
        
        cell.contentConfiguration = contentConfiguration
        
        if recipe.isFavorite {
            let image = UIImage(systemName: "heart.fill")
            let accessoryConfiguration = UICellAccessory.CustomViewConfiguration(customView: UIImageView(image: image),
                                                                                 placement: .trailing(displayed: .always),
                                                                                 tintColor: .secondaryLabel)
            cell.accessories = [.customView(configuration: accessoryConfiguration)]
        } else {
            cell.accessories = []
        }
    }

		// 비교가능한 데이터 소스 (diffable data source) 와
		// 비교가능한 데이터 소스 (diffable data source) 의 셀 프로바이더 (cell provider) 를 생성합니다.
    recipeListDataSource = UICollectionViewDiffableDataSource(collectionView: collectionView) {
        collectionView, indexPath, identifier -> UICollectionViewCell in
				// `identifier` 는 `Recipe.ID` 의 인스턴스 입니다.
				// 데이터 스토어에서 레시피를 조회하기 위해 사용합니다.
        let recipe = dataStore.recipe(with: identifier)!
        return collectionView.dequeueConfiguredReusableCell(using: recipeCellRegistration, for: indexPath, item: recipe)
    }
}
```

### 식별자로 비교가능한 데이터 소스 로드 (**Load the Diffable Data Source with Identifiers)**

비교가능한 데이터 소스 (diffable data source) 가 구성된 `RecipeListViewController` 는 레시피로 콜렉션 뷰를 나타내기 위해 데이터 소스에서 데이터의 초기 로드를 수행하기 위한 `loadRecipeData()` 메서드를 호출합니다. 이 메서드는 레시피 식별자의 목록을 조회하고 [`NSDiffableDataSourceSnapshot`](https://developer.apple.com/documentation/uikit/nsdiffabledatasourcesnapshot) 의 인스턴스를 생성합니다. 그런다음 스냅샷 (snapshot) 에 `main` 섹션과 레시피 식별자를 추가합니다. 마지막으로 데이터 소스에 스냅샷 (snapshot) 을 적용하고 차이점 계산 또는 변경사항 애니메이션 없이 스냅샷 (snapshot) 의 데이터에 상태를 반영하기 위해 콜렉션 뷰를 재설정하기 위해 `applySnapshotUsingReloadData(_:)` 를 호출합니다.

```swift
private func loadRecipeData() {
		// All Recipes 또는 Favorites 와 같이 사이드 바에 선택된 걸 기준으로
		// 레시피 식별자의 목록을 조회합니다.
    guard let recipeIds = recipeSplitViewController.selectedRecipes?.recipeIds()
    else { return }
    
		// 새로운 스냅샷 (snapshot) 에 추가된 레시피 식별자를 통해 콜렉션 뷰를 업데이트 하고
		// 비교가능한 데이터 소스 (diffable data source) 에 스냅샷 (snapshot) 을 적용합니다.
    var snapshot = NSDiffableDataSourceSnapshot<RecipeListSection, Recipe.ID>()
    snapshot.appendSections([.main])
    snapshot.appendItems(recipeIds, toSection: .main)
    recipeListDataSource.applySnapshotUsingReloadData(snapshot)
}
```

> **Important**
각 아이템 식별자는 스냅샷 (snapshot) 내에서 유니크 해야 합니다. 그 결과로 스냅샷 (snapshot) 내에서 여러 위치에 나타날 수 없습니다. 섹션 식별자도 마찬가지입니다. 유니크 해야 하고 스냅샷 (snapshot) 내에서 여러 위치에 나타날 수 없습니다.
> 

### 아이템 삽입, 삭제, 그리고 이동 (**Insert, Delete, and Move Items)**

사람들은 샘플 앱 사용 시 2가지 타입으로 레시피 데이터를 변경할 수 있습니다:

- 레시피 추가 또는 삭제 또는 재정렬과 같은 데이터 변경
- 레시피의 이름 또는 즐겨찾기 마킹 변경과 같은 아이템의 속성 변경

데이터 콜렉션에 대한 변경을 처리하기 위해 앱은 현재 상태를 나타내는 새로운 스냅샷 (snapshot) 을 생성하고 비교가능한 데이터 소스 (diffable data source) 에 적용합니다. 데이터 소스는 변경사항을 파악하기 위해 새로운 스냅샷 (snapshot) 과 현재 스냅샷 (snapshot) 을 비교합니다. 그런 다음에 변경사항을 기준으로 콜렉션 뷰에 적절한 삽입, 삭제, 그리고 이동을 수행합니다.

비교가능한 데이터 소스 (diffable data source) 는 현재 스냅샷 (snapshot) 과 새로운 스냅샷 (snapshot) 의 변경사항을 파악할 수 있지만 데이터 콜렉션의 변경을 모니터링하지 않습니다. 대신에 새로운 스냅샷 (snapshot) 을 적용하기 위해 데이터 변경사항을 감지하고 비교가능한 데이터 소스 (diffable data source) 에 변경사항을 알리는 건 앱이 수행합니다.

> **NOTE**
앱은 앱의 다른 파트에서 데이터 변경사항을 알리기 위해 [`NotificataionCenter`](https://developer.apple.com/documentation/foundation/notificationcenter) 와 [Combine](https://developer.apple.com/documentation/combine) 과 같은 메카니즘을 사용할 수 있습니다. 이 샘플 앱은 `NotificationCenter` 를 사용합니다.
> 

예를 들어 누군가 레시피를 추가 또는 삭제한 후에 앱의 다른 파트에 레시피 변경사항을 알리기 위해 샘플은 `selectedRecipesDidChange` 알림을 사용합니다. 알림을 받기 위해 `RecipeListViewController` 는 선택기 (selector) 인 `selectedRecipesDidChange(_:)` 으로 알림 관찰자 (notification observer) 를 추가합니다.

```swift
NotificationCenter.default.addObserver(
    self,
    selector: #selector(selectedRecipesDidChange(_:)),
    name: .selectedRecipesDidChange,
    object: nil
)
```

`selectedRecipesDidChange(_:)` 는 `loadRecipeData()` 와 유사하지만 [`applySnapshotUsingReloadData(_:)`](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource/3804469-applysnapshotusingreloaddata) 을 사용하는 대신에 알림에서 제공하는 선택된 레시피 식별자의 목록을 적용하기 위해 [`apply(_:animatingDifferences:)`](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource/3795617-apply) 를 사용합니다.

[`apply(_:animatingDifferences:)`](https://developer.apple.com/documentation/uikit/uicollectionviewdiffabledatasource/3795617-apply) 메서드는 데이터를 나타내는 전체를 재설정하는 대신에 콜렉션 뷰에 필요한 부분만 업데이트를 수행합니다. 그리고 `animatingDifferences` 가 `true` 이므로 변경사항이 적용될 때 애니메이션이 적용됩니다.

```swift
@objc
private func selectedRecipesDidChange(_ notification: Notification) {
		// 알림의 `userInfo` 딕셔너리에서 선택된 레시피 식별자의 스냅샷 (snapshot) 을 생성하고
		// 비교가능한 데이터 소스 (diffable data source) 에 적용합니다.
    guard
        let userInfo = notification.userInfo,
        let selectedRecipeIds = userInfo[NotificationKeys.selectedRecipeIds] as? [Recipe.ID]
    else { return }
    
    var snapshot = NSDiffableDataSourceSnapshot<RecipeListSection, Recipe.ID>()
    snapshot.appendSections([.main])
    snapshot.appendItems(selectedRecipeIds, toSection: .main)
    recipeListDataSource.apply(snapshot, animatingDifferences: true)

		// 샘플 앱의 디자인은 상세 뷰 컨트롤러에 나타난 선택된 레시피가 새로운 스냅샷 (snapshot) 에는
		// 존재하지만 스냅샷 (snapshot) 을 적용하기 전인 콜렉션 뷰에서는 존재하지 않을 수 있습니다.
		// 예를 들어 즐겨찾기 레시피의 목록이 나타나는 동안 사용자는 `isFavorite` 버튼을 탭하여
		// 선택된 레시피를 즐겨찾기 해제 할 수 있습니다. 즐겨찾기 목록에서 선택된 레시피는 삭제 됩니다.
		// 다시 버튼을 탭하면 레시피는 목록에 다시 나타납니다. 이 시나리오에서 앱은 레시피를 다시 선택해야
		// 콜렉션 뷰에서 선택된 레시피로 나타납니다.
    selectRecipeIfNeeded()
}
```

### 기존 아이템 업데이트 (**Update Existing Items)**

기존 아이템의 속성 변경을 처리하기 위해 앱은 비교가능한 데이터 소스 (diffable data source) 에 현재 스냅샷 (snapshot) 을 조회하고 스냅샷 (snapshot) 에 [`recofigureItems(_:)`](https://developer.apple.com/documentation/uikit/nsdiffabledatasourcesnapshot/3804468-reconfigureitems) 또는 [`reloadItems(_:)`](https://developer.apple.com/documentation/uikit/nsdiffabledatasourcesnapshot/3375783-reloaditems) 를 호출합니다. 그런 다음 특정 아이템의 업데이트를 나타내기 위해 비교가능한 데이터 소스 (diffable data source) 에 스냅샷 (snapshot) 을 적용합니다.

다시 비교가능한 데이터 소스 (diffable data source) 가 아닌 앱에서 데이터 변경사항을 감지합니다.

예를 들어 사용자가 레시피를 즐겨찾기로 체크할 때 처럼 레시피의 변경에 대해 앱의 다른 파트에 알리려면 샘플에서는 `recipeDidChange` 알림을 보냅니다. `RecipeListViewController` 는 선택기 (selector) 로 `recipeDidChange(_:)` 을 가지는 관찰자를 사용하여 알림을 수신합니다.

```swift
NotificationCenter.default.addObserver(
    self,
    selector: #selector(recipeDidChange(_:)),
    name: .recipeDidChange,
    object: nil
)
```

`recipeDidChange` 알림은 하나의 레시피 변경에 대한 데이터를 나타냅니다. 하나의 레시피만 변경되었으므로 콜렉션 뷰에서 보여지는 전체 레시피에 대해 업데이트를 할 필요가 없습니다. 대신에 샘플은 변경된 레시피를 나타내는 셀만 업데이트 합니다. 예를 들어 사용자가 즐겨찾기로 레시피를 체크하면 레시피 옆에 하트 아이콘이 나타납니다. 그리고 사용자가 레시피를 즐겨찾기에서 해제하면 하트는 사라집니다.

최신 레시피 데이터로 셀을 업데이트 하기위해 `recipeDidChange(_:)` 메서드는 알림에서 제공된 레시피 식별자가 비교가능한 데이터 소스 (diffable data source) 에 포함되어 있는지 확인합니다. 그런 다음에 데이터 소스에 현재 스냅샷 (snapshot) 에서 조회하고 레시피 식별자를 전달하여 [`reconfigureItems(_:)`](https://developer.apple.com/documentation/uikit/nsdiffabledatasourcesnapshot/3804468-reconfigureitems) 를 호출합니다. 이 호출은 레시피 식별자로 식별된 셀에 나타내진 데이터를 업데이트 하라고 지시합니다. 마지막으로 `recipeDidChange(_:)` 업데이트 된 스냅샷 (snapshot) 을 데이터 소스에 적용합니다.

```swift
@objc
private func recipeDidChange(_ notification: Notification) {
    guard
				// `userInfo` 딕셔너리에서 `recipeId` 를 얻습니다.
        let userInfo = notification.userInfo,
        let recipeId = userInfo[NotificationKeys.recipeId] as? Recipe.ID,
				// 데이터 소스가 레시피를 포함하고 있는지 확인합니다.
        recipeListDataSource.indexPath(for: recipeId) != nil
    else { return }
    
		// 비교가능한 데이터 소스 (diffable data source) 의 현재 스냅샷 (snapshot) 을 얻습니다.
    var snapshot = recipeListDataSource.snapshot()
		// 콜렉션 뷰에 나타난 레시피 데이터를 업데이트 합니다.
    snapshot.reconfigureItems([recipeId])
    recipeListDataSource.apply(snapshot, animatingDifferences: true)
}
```

비교가능한 데이터 소스 (diffable data source) 는 업데이트 된 스냅샷 (snapshot) 과 현재 스냅샷 (snapshot) 을 비교하고 차이점을 적용합니다. 이런 경우 변경된 레시피를 나타내는 아이템을 재구성하라고 요청합니다. 이 요청을 수행하기 위해 업데이트 된 레시피를 조회하고 최신 레시피 데이터로 셀을 구성하는 셀 프로바이더 (cell provider) 클로저를 호출합니다. 그리고 `animatingDifferences` 가 `true` 이므로 스냅샷 (snapshot) 이 적용될 때 콜렉션 뷰는 하트 아이콘이 보이거나 사라지는 셀의 변경사항이 애니메이션 됩니다.

### 가벼운 데이터 구조체로 스냅샷 만들기 (**Populate Snapshots with Lightweight Data Structures)**

식별자를 저장하는 다른 방법은 가벼운 데이터 구조체로 비교가능한 데이터 소스 (diffable data source) 와 스냅샷 (snapshot) 을 채우는 것입니다. 데이터 구조체 방식은 편리하고 빠른 프로토타입 또는 속성 변경이 없는 아이템을 나타내는 것과 같은 일부 상황에 적합할 수 있지만 상당한 제한과 모순을 가져옵니다. 예를 들어 [`Hashable`](https://developer.apple.com/documentation/swift/hashable) 과 `Equatable` 구현은 변경될 수 있는 구조체의 모든 프로퍼티에 적용되어야 합니다. 구조체의 데이터가 변경되면 이전 버전과 동일하지 않게 되고 비교가능한 데이터 소스 (diffable data source) 는 새로운 스냅샷 (snapshot) 이 적용될 때 변경된 점을 파악할 때 사용합니다.

샘플은 이런 접근방식을 사용하여 사이드 바에 아이템을 노출합니다. `SidebarViewController` 에서 `SidebarItem` 구조체는 `title` 과 `type` 을 갖는 사이드 바 아이템의 프로퍼티를 정의합니다.

```swift
private struct SidebarItem: Hashable {
    let title: String
    let type: SidebarItemType
    
    enum SidebarItemType {
        case standard, collection, expandableHeader
    }
}
```

이런 프로퍼티의 조합은 각 사이드 바 아이템에 대한 해싱 값을 결정하고 프로퍼티 값은 변경되지 않으므로 식별자 대신 `SidebarItem` 구조체로 스냅샷 (snapshot) 을 채우는 것이 가능합니다.

```swift
private func createSnapshotOfStandardItems() -> NSDiffableDataSourceSectionSnapshot<SidebarItem> {
    let items = [
        SidebarItem(title: StandardSidebarItem.all.rawValue, type: .standard),
        SidebarItem(title: StandardSidebarItem.favorites.rawValue, type: .standard),
        SidebarItem(title: StandardSidebarItem.recents.rawValue, type: .standard)
    ]
    return createSidebarItemSnapshot(.standardItems, items: items)
}
```

이런 접근방식의 단점은 비교가능한 데이터 소스 (diffable data source) 가 식별자를 추적할 수 없습니다. 기존 아이템의 변경되면 비교가능한 데이터 소스 (diffable data source) 는 이전 아이템 삭제 그리고 새로운 아이템 삽입으로 인지합니다. 그 결과로 콜렉션 뷰는 아이템과의 연결된 상태를 잃게 됩니다. 예를 들어 선택된 아이템의 속성을 변경하면 비교가능한 데이터 소스 (diffable data source) 의 관점에서 앱이 아이템을 삭제하고 새로운 아이템을 추가하므로 선택된 아이템의 선택이 취소됩니다.

또한 `animatingDifferences` 가 `true` 이면 스냅샷 (snapshot) 을 적용할 때 모든 변경은 오래된 셀과 새로운 셀 모두 애니메이션을 적용하는 프로세스를 요구합니다. 이것은 셀 내에서 성능에 안좋으며 UI 상태와 애니메이션 상태를 잃을 수 있습니다.

추가적으로 [`reconfigureItems(_:)`](https://developer.apple.com/documentation/uikit/nsdiffabledatasourcesnapshot/3804468-reconfigureitems) 또는 [`reloadItems(_:)`](https://developer.apple.com/documentation/uikit/nsdiffabledatasourcesnapshot/3375783-reloaditems) 메서드는 적절한 식별자를 사용해야 하므로 적절하지 않습니다. 기존 아이템에 대한 데이터 업데이트를 위한 메카니즘은 비교가능한 데이터 소스 (diffable data source) 가 각 변경된 아이템에 대해 삭제와 삽입을 수행하는 새로운 데이터 구조체를 포함하는 새로운 스냅샷 (snapshot) 을 적용하는 방법 뿐입니다.

데이터 구조체를 비교가능한 데이터 소스 (diffable data source) 와 스냅샷 (snapshot) 에 직접 저장하는 것은  데이터 소스가 식별을 추적할 수 있는 방법을 잃어버리기 때문에 적절한 솔루션이 아닙니다. 이 방식은 샘플에서 사이드 바 아이템과 같이 아이템이 변경되지 않는 간단한 케이스나 아이템의 식별이 중요하지 않을 때 사용해야 합니다. 다른 경우 또는 사용할 접근방식이 명확하지 않을 때는 비교가능한 데이터 소스 (diffable data source) 와 스냅샷 (snapshot) 에 적절한 식별자로 채워야 합니다.