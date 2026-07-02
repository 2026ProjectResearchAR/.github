# システムアーキテクチャ図

`docs/requirements.md` および `docs/specifications.md` に基づく、Web ARアプリケーションとIoTセンサーネットワークを統合したシステムのアーキテクチャ図です。

```mermaid
graph TD
    %% ユーザー / フロントエンド
    User((ユーザー))
    
    subgraph Frontend ["フロントエンド Next.js / React"]
        WebApp["Web AR アプリケーション"]
        ARjs["ARエンジン (AR.js)"]
        MapGen["2D→3Dマップ自動生成ロジック"]
        Features["機能群<br/>(避難シミュレーション, スタンプラリー, 混雑度可視化)"]
        
        WebApp --> ARjs
        WebApp --> Features
        MapGen --> ARjs
    end

    %% バックエンド
    subgraph Backend ["バックエンド & インフラ (Supabase)"]
        API["API エンドポイント"]
        DB[/"データベース<br/>(施設情報, , IoTログ)"/]
        Prediction["混雑需要予測ロジック"]
        
        API <--> DB
        DB <--> Prediction
    end

    %% IoT・エッジ
    subgraph IoTEdge ["IoT センサーネットワーク (エッジ)"]
        Microcontroller["組み込みマイコン"]
        Sensors["環境センサー群<br/>(人感, CO2, 環境音等)"]
        
        Sensors -->|プライバシー配慮データ取得| Microcontroller
    end

    %% 物理・外部要素
    subgraph PhysicalCampus ["キャンパス環境 (物理空間)"]
        ARMarkers["ARマーカー"]
        Maps2D["既存2Dキャンパスマップ"]
    end

    %% データフロー
    User <-->|ブラウザアクセス  用アプリ不要 | WebApp
    ARMarkers -.->|カメラによるマーカー認識| ARjs
    Maps2D -.->|データ入力| MapGen
    Features <-->|混雑度・予測データの取得| API
    Microcontroller -->|リアルタイムデータ送信| API
```

## 各コンポーネントの役割

* **フロントエンド (Next.js / React)**:
  * ユーザーはブラウザからアクセスし、専用アプリ不要でAR機能を利用します。
  * `AR.js` を用いて、キャンパス内のARマーカーを認識し、3Dオブジェクトを描画します。
  * 既存の2Dマップから3Dマップを自動生成するロジックにより、空間認識の補助を行います。
* **バックエンド (Supabase)**:
  * IoTネットワークから送信されるセンサーデータを受け取り、蓄積します。
  * 蓄積されたデータに基づき、食堂や自習室などの混雑状況を予測するアルゴリズムを実行します。
  * フロントエンドに対して、リアルタイムの混雑状況や予測データをAPI経由で提供します。
* **IoTセンサーネットワーク**:
  * 学内各所に設置され、プライバシーに配慮したデータ（人感、CO2、音など）を収集します。
  * 組み込みマイコンを通じて、バックエンド（Supabase）へデータを送信します。
