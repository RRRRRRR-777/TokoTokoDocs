## アーキテクチャ
* MV(Model-View)
  * 開発規模が大きくなれば、VMの導入やクリーンアーキテクチャへの移行も検討する
## ディレクトリ構造
※ ファイル名は参考
*  📁 Model/
   * アプリのデータ・ロジックを担う部分。
   * Entities/: データ構造（struct/class）。(例： User.swift, Walk.swift)
   * Services/: データ操作の抽象化。(例： ローカル保存やAPI呼び出し)
   * Network/: API通信関連。APIClient, Endpointなど
* 📁 View/
   * UIに関するすべてを配置。
   * Screens/: 各画面ごとのファイルを作成（例： WalkView.swift, HomeView.swift）
   * Shared/: 共通View。(例： ErrorView, LoadingView)
* 📁 Resources/
   * 画像、ローカライズ、カラーパレットなど。
* 📁 App/
   * アプリのエントリーポイント。AppDelegate, SceneDelegate, YourAppApp.swiftなどを置きます。
```
TokoToko/
├── Model/
│   ├── Entities/
│   │   └── Walk.swift
│   ├── Services/
│   │   └── WalkRepository.swift
│   └── Network/
│       └── APIClient.swift
├── View/
│   ├── Screens/
│   │   ├── WalkView.swift
│   └── Shared/
│       └── LoadingView.swift
├── Resources/
│   ├── Assets.xcassets
│   └── Localizable.strings
├── SupportingFiles/
│   └── Info.plist
└── App/
    └── AppDelegate.swift , TokoTokoApp.swift
```
****

## 選定理由
### 概要
* 最初はMVCアーキテクチャを導入していたが、SwiftUIではControllerの責務が弱いことがわかったため、MVに移行した
* MVCを導入した際のメモとMVに移行したときのメモを以下に記載する
### MVCを導入した際のメモ
- Swiftで採用されるアーキテクチャはMVCとMVVCがあるらしい
    - ほかにもVIPER, Clean Architecture, Redux（Unidirectional Architecture）等々あるらしいが今回はこの2つに着目して選定する
- MVC(Model - View - Controller)

    > MVCについてまとめてみた
      https://qiita.com/kazuma___/items/080dced61370f0b55f87
    >
    - ruby on railsと同じアーキテクチャなので学習コストは特に掛からない
- MVVC(Model - View - ViewModel )

    > 【iOS】MVVM初心者入門！説明から実装まで！
    https://qiita.com/Soccerboy_Hamada/items/fc9c06f58594471a93c0
    >
    - ViewModelという概念とControllerの違いについて深く理解する必要がある
- MVCとMVVMの比較表

| 項目 | MVC | MVVM |
| --- | --- | --- |
| ロジックの場所 | ViewControllerに集中しやすい | ViewModelが分離して責任を分担 |
| テストのしやすさ | 難しい（VCに依存） | 容易（ViewModel単体でテスト可能） |
| UIとの結合 | 強い | 弱い（バインディングで柔軟） |
| SwiftUI対応 | あまり向いていない | 非常に相性が良い |
- この比較表よりMVVMのほうが優れていそうだなと思った

> 「SwiftUIでMVVMを採用するのは止めよう」と思い至った理由
https://qiita.com/karamage/items/8a9c76caff187d3eb838
>
- こういった記事があったり、MVCの学習コストがほぼないのに対してMVVCはすこしコストが掛かる点を考慮して一度MVCで実装することにした。
- 仮にMVVC、または他のアーキテクチャのほうが優れていると判断できた場合はアーキテクチャを置換する
    - その際にコストは掛かるが学習としての意義が高いと判断した
    - MVVCを最初に導入するよりも、MVCを置換するほうが学びにつながりそうだと思った
- 上記の理由よりアーキテクチャはMVCとする
- clineで生成したMVCアーキテクチャのディレクトリとファイル
    - **Model層**
        - **Entities**: データモデルを定義
            - `Item.swift`: 基本的なデータ構造
        - **Services**: ビジネスロジックを実装
            - `ItemService.swift`: データ操作のサービス層
        - **Repositories**: データアクセス層
            - `ItemRepository.swift`: データの永続化と取得
    - **View層**
        - **Screens**: 画面単位のビュー
            - `HomeView.swift`: メイン画面
            - `DetailView.swift`: 詳細画面
        - **Components**: 再利用可能なUIコンポーネント
            - `ItemRow.swift`: リスト表示用のアイテム行
    - Controller層
        - **ViewControllers**: ビューとモデルの接続
            - `HomeController.swift`: ホーム画面のロジック
            - `DetailController.swift`: 詳細画面のロジック
        - **Coordinators**: 画面遷移管理
            - `AppCoordinator.swift`: アプリ全体のナビゲーション

    また、`TokoTokoApp.swift`を更新して、新しいMVC構造を使用するようにしました。

    この構造により、責任の分離が明確になり、テストやメンテナンスが容易になります。モデルはデータとビジネスロジック、ビューはUI表示、コントローラーはその間の調整役として機能します。

    ```mermaid
    graph TD
      subgraph Model層
        Entities[Entities<br>データモデルを定義]
        Services[Services<br>ビジネスロジックを実装]
        Repositories[Repositories<br>データアクセス層]
      end

      subgraph View層
        Screens[Screens<br>画面単位のビュー]
        Components[Components<br>再利用可能なUIコンポーネント]
      end

      subgraph Controller層
        ViewControllers[ViewControllers<br>ビューとモデルの接続]
        Coordinators[Coordinators<br>画面遷移管理]
      end

      Screens --> ViewControllers
      Components --> Screens
      ViewControllers --> Services
      Services --> Repositories
      ViewControllers --> Coordinators
      Entities --> Services

    ```

### MVに移行したときのメモ
* Swiftを使用している人複数人に聞いたところMVCは今ではあまり使用されていないということを聞いたので、他のアーキテクチャを検討する
* MVCを辞める理由はSwiftUIを使用するにあたってCの責務が弱いため
  * 導入後の調査で発覚
* プロジェクトが大きくなればVMを導入するのもあり
  * ただ、SwiftUIを使うならVMいらないって意見がかなり多い印象
* 将来的に移行するときの案
  * MV → MVVMにしたいとき：ViewModel/ フォルダを追加し、WalkViewModel.swiftなどを追加するだけ。
  * MV → Clean Architectureにしたいとき：UseCase/, Interactor/, RepositoryInterface/などを追加して段階的に対応可能。

* 参照記事
  * 個人開発の SwiftUI アプリのアーキテクチャを MVVM から MV にした
    > https://maiyama4.hatenablog.com/entry/2023/12/27/142625
  * 「SwiftUIでMVVMを採用するのは止めよう」と思い至った理由 https://qiita.com/karamage/items/8a9c76caff187d3eb838
    > https://qiita.com/karamage/items/8a9c76caff187d3eb838
