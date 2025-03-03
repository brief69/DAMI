# dami protocol要件

## 1. システム基盤

### データ管理システム
IPFSを活用したデータチャンク管理システムとして、データを再利用可能な小さなチャンクに分割して保存し、クライアント側でこれらのチャンクを「レシピ」として組み合わせて管理するシステムを提案します。

#### チャンク化の具体例
- よく使用される単語やフレーズ（例：「2024年」「10月1日」「ボブ」など）を個別のチャンクとして保存
- 各チャンクは固有のCID（コンテンツ識別子）を持ち、IPFS上で一意に管理
- 同じデータは再利用され、重複を防止

#### データタイプごとの固定チャンク分割
- 動画: 6秒単位
- 画像: 1画像 = 1チャンク
- JSON: フィールドごと
- その他: データタイプに応じた適切な分割

### 基本メカニズム

1. **チャンク化とアップロード**
   - データの最小単位での分割
   - 各チャンクを個別にIPFSにアップロード
   - アップロード後、CIDを取得
   - クライアントはCIDを組み合わせてデータを再構築

2. **データ整合性**
   - 各チャンクのハッシュと元データ全体のハッシュを保持
   - データの改ざん検出と正確な再構築を保証
   - 分散型タイムスタンプを使用してデータの改ざんを検出
   - 非線形的なデータ構造（従来のブロックチェーンの線形構造ではなく）

### 通信基盤
- libp2pをメインのP2P通信基盤として採用
  - DHT（Distributed Hash Table）による効率的なピア発見
  - PubSubによる効率的なデータ同期
  - マルチストリームによる柔軟な通信制御
  - マルチプロトコル対応による拡張性確保
- STUN/TURNサーバーによる補助
  - NATトラバーサルの確実性向上
  - フォールバック時の接続保証
  - P2P接続確立の補助

## 2. アーキテクチャ設計

### クリーンアーキテクチャの採用

1. **ドメイン層（Core）**
   - エンティティ
     - Chunk（データの最小単位）
     - Recipe（チャンクの組み合わせ）
     - User（AT Protocol準拠のID）
   - バリューオブジェクト
     - ContentHash
     - Timestamp
     - Zeny
   - ドメインサービス
     - ChunkManager
     - RecipeManager

2. **ユースケース層**
   - チャンク管理（作成・取得・削除）
   - レシピ管理（作成・実行）
   - 価値システム（ゼニー計算・XLL連携）

3. **インターフェース層**
   - P2P通信制御
   - データ同期管理
   - NATトラバーサル
   - フォールバック接続

4. **インフラ層**
   - IPFS接続（go-ipfs-http-client）
   - libp2p接続管理
   - 永続化管理
   - キャッシュ制御

## 3. サービス開発フレームワーク

### 認証システム
- AT Protocolベースの分散型ID（DID）システム
- デバイスベースの公開鍵暗号方式
- 中央サーバーに依存しない認証
- ユーザーによる完全なデータ所有権

### インセンティブ設計の哲学
現在のwebサービスにおいてのレコメンドアルゴリズムは、完全な偏見に基づいています。これは、コンピュータの機械学習と人間の生物学的な構造に根本的な互換性がないためです。そのため、damiでは人間主導の推薦システムを採用します。

#### 基本構造
1. **三角形のインセンティブ設計**
   - 3つの正のインセンティブ
   - 3つの逆インセンティブ
   - ユーザー紹介インセンティブ

2. **動的調整システム**
   - ネットワーク状態に応じた調整
   - 問題対応のための強化機能
   - サービス健全性の維持

### 実装例（動画サービスatem）

#### 正のインセンティブ群
1. 動画投稿（コンテンツ提供）
2. 動画視聴（体験共有）
3. 動画おすすめ（価値発見）

#### 逆インセンティブ群
4. 有益性判定（品質管理）
5. カテゴリー判定（整理・分類）
6. おすすめ評価（推薦品質）

#### 基盤インセンティブ
7. ユーザー紹介（コミュニティ成長）

## 4. 価値システム（ゼニー）

### 基本概念
- ユーザーのデータアクセスが価値を生む
- アクセス履歴がトークン（ゼニー）として機能
- IPFSのDHTによる自然なデータ重複排除
- アクセスされないデータの自然な消失

### 価値評価
- XLLシステムとの連携
- 経済市場全体の価値をXLLで表現
- 中央値をゼニーの価値として使用

## 5. 実装上の注意点

1. リアルタイム性の確保
   - PubSubを活用したリアルタイムデータ同期
   - libp2pストリームによる効率的な通信
   - 分散型タイムスタンプによる時系列管理

2. データタイプごとの適切なチャンク分割
3. ユーザー認証の堅牢性
4. XLLシステムとの連携方式
5. libp2pとの効率的な統合
6. STUN/TURNサーバーの適切な配置と管理
7. P2P接続確立の最適化

## 6. 将来の拡張性

### 技術的拡張
- 新データタイプ対応
- クロスプラットフォーム展開
- プロトコル拡張

### エコシステム
- サービス間連携
- 共通認証基盤
- 分散型経済圏の形成

## 7. 要検討・検証ポイント

### チャンク化実装
- 動画の6秒単位分割の具体的な実装方法と最適化
- JSONフィールド分割の深さと範囲の決定基準
- 画像の最適なチャンクサイズとフォーマット
- チャンク分割時のメタデータ管理方法

### インセンティブ計算
- ゼニー発行・計算の具体的なアルゴリズム
- 各アクションに対する報酬量の設定
- 逆インセンティブの重み付けと計算方法
- XLLとの価値連動における為替レート決定方式

### 時刻同期と整合性
- 分散型タイムスタンプの生成アルゴリズム
- ノード間での時刻同期メカニズム
- データ改ざん検出の具体的な実装
- 履歴管理とロールバック方式

### エラー対策
- チャンク欠損時のリカバリー手順
- P2P接続確立失敗時のフォールバックフロー
- データ整合性検証失敗時の回復プロセス
- ネットワーク分断時の動作

### パフォーマンス目標
- 同時接続数の目標値（初期：100-1000ノード）
- チャンク転送の目標レイテンシー（100ms以内）
- リアルタイムデータ同期の更新頻度（1秒以内）
- キャッシュ戦略とストレージ効率

これらの項目は実装と検証を進めながら具体的な数値や方式を決定していく必要がある。

---
