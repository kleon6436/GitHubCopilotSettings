---
name: mfc-ui-guidelines
description: 'MFC アプリケーションの UI 設計・実装ガイドラインを参照・適用する。Document/View アーキテクチャ、メッセージマップ、ダイアログ設計、DDX/DDV、コントロール配置、リボン UI、GDI リソース管理、CMFCVisualManager、高 DPI 対応、アクセシビリティ、UI スレッド管理を確認・適用したいときに使用。Use when: designing MFC application UI, reviewing MFC dialog layout, message handling, Document/View architecture, ribbon UI, GDI resource management, CMFCVisualManager theming, high-DPI support, accessibility in MFC apps.'
argument-hint: '確認・適用したい MFC UI ガイドラインの項目（省略可）'
---

# MFC アプリケーション UI ガイドライン

## 概要

このスキルは MFC（Microsoft Foundation Classes）アプリケーションの UI 設計・実装ガイドラインを定義します。
モダン C++（C++17 以降）を前提とし、MFC 固有の UI パターン・リソース管理・アクセシビリティをカバーします。
一般的な C++ コーディング規約については `skills/cpp-coding-standards/SKILL.md` を参照してください。

> **対象**: Visual Studio 2019 以降 / MFC Feature Pack 搭載環境 / Windows 10 以降

---

## 1. アプリケーション構造

### 1.1 アプリケーション種別の選択基準

| 種別 | 適用ケース | 基底クラス |
|------|-----------|-----------|
| SDI（単一ドキュメント） | 単一ファイル編集アプリ | `CFrameWnd` |
| MDI（複数ドキュメント） | 複数ファイル同時編集アプリ | `CMDIFrameWnd` / `CMDIChildWnd` |
| ダイアログベース | ユーティリティ・設定ツール | `CDialogEx` |
| タブ付き MDI | モダンなタブ切り替え UI | `CMDIFrameWndEx` + `CMDITabInfo` |

### 1.2 Document/View アーキテクチャ

- **CDocument**: データの保持・永続化（シリアライズ）を担当する。UI ロジックを含めない。
- **CView**: 描画とユーザー操作を担当する。データの直接操作は CDocument を介して行う。
- Document と View は `UpdateAllViews()` / `OnUpdate()` で同期する。
- ドキュメントテンプレート（`CSingleDocTemplate` / `CMultiDocTemplate`）で Document・View・Frame を結合する。

```cpp
// ✅ Good — Document はデータ管理のみ
class CMyDocument : public CDocument {
 public:
  const std::vector<Record>& records() const { return records_; }

  void AddRecord(Record record) {
    records_.push_back(std::move(record));
    SetModifiedFlag(TRUE);
    UpdateAllViews(nullptr);
  }

 protected:
  BOOL OnNewDocument() override;
  void Serialize(CArchive& ar) override;

 private:
  std::vector<Record> records_;

  DECLARE_MESSAGE_MAP()
};

// ❌ Bad — Document 内で直接 UI を操作
class CMyDocument : public CDocument {
 public:
  void AddRecord(Record record) {
    records_.push_back(record);
    AfxMessageBox(_T("追加しました"));  // UI ロジックが混入
  }
};
```

---

## 2. メッセージハンドリング

### 2.1 メッセージマップの基本

- メッセージマップマクロ（`BEGIN_MESSAGE_MAP` / `END_MESSAGE_MAP`）を使用する。
- ハンドラ関数の命名は `On` + 動詞 + 対象（例: `OnClickSave`, `OnPaint`）とする。
- `ON_COMMAND` のハンドラは `OnCmd` + コマンド名とする。
- カスタムメッセージは `WM_APP` ベースで定義し、マジックナンバーを避ける。

```cpp
// ✅ Good — 明確な命名とカスタムメッセージ定義
constexpr UINT WM_PROCESSING_COMPLETE = WM_APP + 1;
constexpr UINT WM_DATA_UPDATED        = WM_APP + 2;

BEGIN_MESSAGE_MAP(CMainFrame, CFrameWndEx)
  ON_WM_CREATE()
  ON_WM_CLOSE()
  ON_COMMAND(ID_FILE_SAVE, &CMainFrame::OnCmdFileSave)
  ON_COMMAND(ID_EDIT_UNDO, &CMainFrame::OnCmdEditUndo)
  ON_UPDATE_COMMAND_UI(ID_EDIT_UNDO, &CMainFrame::OnUpdateCmdEditUndo)
  ON_MESSAGE(WM_PROCESSING_COMPLETE, &CMainFrame::OnProcessingComplete)
END_MESSAGE_MAP()

// ❌ Bad — マジックナンバー、曖昧な命名
BEGIN_MESSAGE_MAP(CMainFrame, CFrameWndEx)
  ON_COMMAND(ID_FILE_SAVE, &CMainFrame::OnFunc1)
  ON_MESSAGE(WM_APP + 100, &CMainFrame::Handler2)  // 意味不明な定数
END_MESSAGE_MAP()
```

### 2.2 コマンド UI 更新

- `ON_UPDATE_COMMAND_UI` でメニュー・ツールバーの有効/無効状態を制御する。
- ハンドラ内で `pCmdUI->Enable()` / `pCmdUI->SetCheck()` のみ呼び出す。重い処理を入れない。

```cpp
// ✅ Good
void CMainFrame::OnUpdateCmdEditUndo(CCmdUI* pCmdUI) {
  pCmdUI->Enable(CanUndo());
}
```

---

## 3. ダイアログ設計

### 3.1 モーダルとモードレスの選択

| 種別 | 使い分け | 生成方法 |
|------|---------|---------|
| モーダル | ユーザーの入力完了を待つ（設定、確認） | `DoModal()` |
| モードレス | メイン操作と並行して使う（検索、ツール） | `Create()` + `ShowWindow()` |

### 3.2 DDX/DDV（Data Exchange / Validation）

- コントロール値とメンバー変数の同期には DDX マクロを使用する。
- バリデーションは DDV マクロまたは `OnOK()` で行う。
- メンバー変数の初期化はコンストラクタで行い、`OnInitDialog()` で UI を補完する。

```cpp
// ✅ Good — DDX/DDV を活用した値の同期と検証
class CSettingsDialog : public CDialogEx {
 public:
  explicit CSettingsDialog(CWnd* pParent = nullptr);

 protected:
  void DoDataExchange(CDataExchange* pDX) override;
  BOOL OnInitDialog() override;

 private:
  CString user_name_;
  int     port_number_ = 8080;
  BOOL    auto_connect_ = FALSE;

  DECLARE_MESSAGE_MAP()
};

void CSettingsDialog::DoDataExchange(CDataExchange* pDX) {
  CDialogEx::DoDataExchange(pDX);
  DDX_Text(pDX, IDC_EDIT_USERNAME, user_name_);
  DDV_MaxChars(pDX, user_name_, 64);
  DDX_Text(pDX, IDC_EDIT_PORT, port_number_);
  DDV_MinMaxInt(pDX, port_number_, 1, 65535);
  DDX_Check(pDX, IDC_CHECK_AUTO_CONNECT, auto_connect_);
}

// ❌ Bad — DDX を使わず手動でコントロール操作
void CSettingsDialog::OnOK() {
  CString text;
  GetDlgItemText(IDC_EDIT_USERNAME, text);  // DDX を使うべき
  user_name_ = text;
  CDialogEx::OnOK();
}
```

### 3.3 コントロール配置指針

- タブ順序はリソースエディタで論理的な順序に設定する（左→右、上→下）。
- グループボックスでコントロールをグループ化し、関連する設定をまとめる。
- OK / キャンセルボタンはダイアログ右下に配置する（Windows 標準配置）。
- コントロール間のマージンは最低 7 DLU（Dialog Unit）を確保する。

---

## 4. コントロールとレイアウト

### 4.1 推奨コントロール選択

| 用途 | 推奨コントロール | 備考 |
|------|----------------|------|
| テキスト入力 | `CEdit` / `CMFCMaskedEdit` | マスク入力にはMaskedEdit |
| 複数行テキスト | `CRichEditCtrl` | 書式付きテキスト向け |
| 選択（少数） | `CComboBox` | 項目数 10 以下 |
| 選択（多数） | `CListCtrl`（レポートビュー） | ソート・フィルタが必要な場合 |
| ツリー表示 | `CTreeCtrl` | 階層データ |
| プロパティ一覧 | `CMFCPropertyGridCtrl` | 設定画面に最適 |
| 日時入力 | `CDateTimeCtrl` | |
| 進捗表示 | `CProgressCtrl` | |

### 4.2 動的レイアウト（高 DPI 対応）

- `CMFCDynamicLayout` を使用してリサイズ時のコントロール配置を定義する。
- リソースエディタの動的レイアウト設定、またはコードで `SetMinSize()` / `EnableDynamicLayout()` を使用する。
- 固定ピクセル値ではなく比率ベースで配置する。

```cpp
// ✅ Good — 動的レイアウトでリサイズ対応
BOOL CMyDialog::OnInitDialog() {
  CDialogEx::OnInitDialog();

  auto* layout = GetDynamicLayout();
  if (layout != nullptr) {
    // リストコントロール: 移動なし、サイズは親に追従
    layout->AddItem(IDC_LIST_MAIN,
        CMFCDynamicLayout::MoveNone(),
        CMFCDynamicLayout::SizeHorizontalAndVertical(100, 100));
    // OK ボタン: 右下に移動、サイズ固定
    layout->AddItem(IDOK,
        CMFCDynamicLayout::MoveHorizontalAndVertical(100, 100),
        CMFCDynamicLayout::SizeNone());
  }
  return TRUE;
}

// ❌ Bad — OnSize で手動計算（DPI 非対応になりがち）
void CMyDialog::OnSize(UINT nType, int cx, int cy) {
  CDialogEx::OnSize(nType, cx, cy);
  if (auto* pList = GetDlgItem(IDC_LIST_MAIN)) {
    pList->MoveWindow(10, 10, cx - 20, cy - 50);  // 固定マージン
  }
}
```

### 4.3 高 DPI 対応

- アプリケーションマニフェストで `<dpiAwareness>PerMonitorV2</dpiAwareness>` を宣言する。
- アイコン・ビットマップは複数解像度を用意する（16×16, 32×32, 48×48, 256×256）。
- フォントサイズはシステムメトリクスから取得し、ハードコードしない。

---

## 5. リボン / ツールバー / メニュー

### 5.1 リボン UI（MFC Feature Pack）

- 新規アプリケーションにはリボン UI（`CMFCRibbonBar`）を推奨する。
- カテゴリ → パネル → ボタン の階層でコマンドを整理する。
- よく使うコマンドはクイックアクセスツールバーに登録する。

```cpp
// ✅ Good — リボンの構築
void CMainFrame::InitializeRibbon() {
  auto* category = ribbon_bar_.AddCategory(
      _T("ホーム"), IDB_RIBBON_HOME, IDB_RIBBON_HOME_SMALL);

  auto* panel = category->AddPanel(_T("ファイル"), icon_index);
  panel->Add(new CMFCRibbonButton(ID_FILE_NEW, _T("新規"), 0, 0));
  panel->Add(new CMFCRibbonButton(ID_FILE_OPEN, _T("開く"), 1, 1));
  panel->Add(new CMFCRibbonButton(ID_FILE_SAVE, _T("保存"), 2, 2));
}
```

### 5.2 ツールバー

- ツールバーは `CMFCToolBar` を使用する（`CToolBar` ではなく）。
- ツールバーボタンにはツールチップを必ず設定する。
- カスタマイズ可能にする場合は `CMFCToolBar::EnableCustomizeButton()` を使用する。

### 5.3 メニュー設計

- メニュー階層は最大 2 段（サブメニュー 1 段まで）にとどめる。
- アクセラレータキー（`Ctrl+S` 等）はメニュー項目に明示する。
- ニーモニック（`&File` → `Alt+F`）をすべてのメニュー項目に設定する。

---

## 6. GDI リソース管理

### 6.1 RAII ラッパーの活用

- GDI オブジェクト（`CPen`, `CBrush`, `CFont`, `CBitmap`）は RAII で管理する。
- `SelectObject()` の戻り値（旧オブジェクト）は必ず復元する。
- GDI オブジェクトの生成失敗を検査する。

```cpp
// ✅ Good — RAII パターンで GDI オブジェクトを管理
class GdiSelector {
 public:
  GdiSelector(CDC& dc, CGdiObject& obj)
      : dc_(dc), old_obj_(dc.SelectObject(&obj)) {}
  ~GdiSelector() {
    if (old_obj_ != nullptr) {
      dc_.SelectObject(old_obj_);
    }
  }
  GdiSelector(const GdiSelector&) = delete;
  GdiSelector& operator=(const GdiSelector&) = delete;

 private:
  CDC& dc_;
  CGdiObject* old_obj_;
};

void CMyView::OnDraw(CDC* pDC) {
  CPen pen(PS_SOLID, 2, RGB(0, 120, 215));
  GdiSelector pen_guard(*pDC, pen);

  CBrush brush(RGB(230, 230, 230));
  GdiSelector brush_guard(*pDC, brush);

  pDC->Rectangle(rect);
}

// ❌ Bad — SelectObject の復元漏れ
void CMyView::OnDraw(CDC* pDC) {
  CPen pen(PS_SOLID, 2, RGB(0, 120, 215));
  pDC->SelectObject(&pen);  // 旧オブジェクト未保存 → リーク
  pDC->Rectangle(rect);
  // pen が破棄されるが DC にまだ選択されている → 未定義動作
}
```

### 6.2 ダブルバッファリング

- ちらつき防止にはダブルバッファリングを使用する。
- `CMemDC`（MFC Feature Pack）または手動メモリ DC を使用する。

```cpp
// ✅ Good — CMemDC によるダブルバッファリング
void CMyView::OnDraw(CDC* pDC) {
  CMemDC mem_dc(*pDC, this);
  CDC& dc = mem_dc.GetDC();

  // dc に対して描画操作を行う
  dc.FillSolidRect(&client_rect, ::GetSysColor(COLOR_WINDOW));
  DrawContent(&dc);
}
```

### 6.3 リソースリーク防止チェックリスト

- [ ] すべての `SelectObject()` に対して旧オブジェクトの復元がある
- [ ] `CreateCompatibleDC()` / `CreateCompatibleBitmap()` の戻り値を検査している
- [ ] `LoadImage()` / `LoadBitmap()` で取得したハンドルを `DeleteObject()` で解放している
- [ ] `GetDC()` に対して `ReleaseDC()` がペアになっている
- [ ] 描画コード内で GDI オブジェクトをループ生成していない

---

## 7. ビジュアルスタイルとテーマ

### 7.1 Visual Manager

- `CMFCVisualManager` でアプリケーション全体のテーマを統一する。
- プロジェクト要件に応じてプリセットを選択する。

| Visual Manager | 外観 |
|---------------|------|
| `CMFCVisualManagerOffice2007` | Office 2007 風 |
| `CMFCVisualManagerWindows7` | Windows 7 Aero 風 |
| `CMFCVisualManagerVS2008` | Visual Studio 2008 風 |

```cpp
// ✅ Good — アプリケーション初期化で Visual Manager を設定
BOOL CMyApp::InitInstance() {
  CMFCVisualManager::SetDefaultManager(
      RUNTIME_CLASS(CMFCVisualManagerWindows7));

  // 以降の UI 初期化 ...
}
```

### 7.2 システムカラーの使用

- ハードコードした色ではなく `::GetSysColor()` を使用する。
- ハイコントラストモードで正しく表示されることを確認する。

```cpp
// ✅ Good — システムカラーを使用
pDC->SetTextColor(::GetSysColor(COLOR_WINDOWTEXT));
pDC->SetBkColor(::GetSysColor(COLOR_WINDOW));

// ❌ Bad — 色のハードコード
pDC->SetTextColor(RGB(0, 0, 0));
pDC->SetBkColor(RGB(255, 255, 255));
```

### 7.3 Common Controls v6

- アプリケーションマニフェストで Common Controls v6 を有効にする。
- これによりモダンな外観（Visual Styles）が適用される。

```xml
<!-- ✅ Good — マニフェストで Common Controls v6 を有効化 -->
<dependency>
  <dependentAssembly>
    <assemblyIdentity type="win32" name="Microsoft.Windows.Common-Controls"
                      version="6.0.0.0" processorArchitecture="*"
                      publicKeyToken="6595b64144ccf1df" language="*" />
  </dependentAssembly>
</dependency>
```

---

## 8. アクセシビリティ

### 8.1 基本原則

- すべてのコントロールにアクセシブルな名前（ラベル）を設定する。
- スタティックテキストはラベル対象のコントロールの直前（タブ順序）に配置する。
- カスタムコントロールは `IAccessible` を実装するか、UI Automation プロバイダーを提供する。

### 8.2 キーボードナビゲーション

- すべての操作がキーボードのみで完結できることを保証する。
- タブ順序はリソースエディタで論理的な順序に設定する。
- ショートカットキー（アクセラレータ）をよく使う操作に割り当てる。
- フォーカスの移動が視覚的に確認できることを保証する。

```cpp
// ✅ Good — アクセラレータテーブルの活用
BOOL CMainFrame::PreTranslateMessage(MSG* pMsg) {
  if (accel_table_ != nullptr &&
      ::TranslateAccelerator(m_hWnd, accel_table_, pMsg)) {
    return TRUE;
  }
  return CFrameWndEx::PreTranslateMessage(pMsg);
}
```

### 8.3 スクリーンリーダー対応

- `WM_GETOBJECT` メッセージに応答して MSAA / UI Automation 情報を提供する。
- カスタム描画コントロールには `accName`、`accRole`、`accState` を実装する。
- 状態変化時に `NotifyWinEvent()` でアクセシビリティイベントを発火する。

---

## 9. スレッドと非同期処理

### 9.1 UI スレッドの原則

- UI の更新は必ずメインスレッド（UI スレッド）で行う。
- ワーカースレッドから UI を更新する場合は `PostMessage()` を使用する（`SendMessage()` はデッドロックリスク）。
- 長時間処理はワーカースレッドに委譲し、UI スレッドをブロックしない。

```cpp
// ✅ Good — ワーカースレッドで処理し、完了を PostMessage で通知
constexpr UINT WM_TASK_COMPLETE = WM_APP + 10;

// ワーカースレッド関数
UINT WorkerThreadProc(LPVOID pParam) {
  auto* context = static_cast<TaskContext*>(pParam);
  // 重い処理を実行 ...
  context->frame_wnd->PostMessage(WM_TASK_COMPLETE, 0, 0);
  return 0;
}

// メインフレームでスレッド起動
void CMainFrame::OnCmdStartProcessing() {
  AfxBeginThread(WorkerThreadProc, &task_context_);
  status_bar_.SetPaneText(0, _T("処理中..."));
}

// 完了通知ハンドラ（UI スレッドで実行される）
LRESULT CMainFrame::OnTaskComplete(WPARAM, LPARAM) {
  status_bar_.SetPaneText(0, _T("完了"));
  UpdateDataView();
  return 0;
}

// ❌ Bad — ワーカースレッドから直接 UI を操作
UINT WorkerThreadProc(LPVOID pParam) {
  auto* dlg = static_cast<CMyDialog*>(pParam);
  // 重い処理 ...
  dlg->SetDlgItemText(IDC_STATUS, _T("完了"));  // 別スレッドから UI 操作 → 未定義動作
  return 0;
}
```

### 9.2 スレッド安全な進捗更新

- `CProgressCtrl` の更新はメインスレッドから `PostMessage` 経由で行う。
- 進捗値の共有には `std::atomic` を使用する。

---

## 10. 文字列と国際化

### 10.1 文字列の基本方針

- プロジェクトは **Unicode ビルド**（`_UNICODE` / `UNICODE`）を前提とする。MBCS は使用しない。
- UI に表示する文字列はすべてリソース文字列テーブル（`.rc`）で管理する。
- ソースコード中に UI テキストをハードコードしない。

```cpp
// ✅ Good — リソース文字列を使用
CString message;
message.LoadString(IDS_FILE_SAVE_SUCCESS);
AfxMessageBox(message);

// ❌ Bad — ハードコードされた UI テキスト
AfxMessageBox(_T("ファイルを保存しました。"));
```

### 10.2 CString の使用指針

- MFC API とのインターフェースには `CString` を使用する。
- 内部ロジックでは `std::wstring` / `std::wstring_view` も併用可。
- `CString::Format()` で書式化し、手動の文字列連結を避ける。

```cpp
// ✅ Good
CString status;
status.Format(IDS_ITEM_COUNT_FORMAT, item_count);  // リソース: "項目数: %d"

// ❌ Bad — 手動連結 + ハードコード
CString status = _T("項目数: ") + std::to_wstring(item_count).c_str();
```

### 10.3 レイアウトの国際化考慮

- テキストの伸縮を考慮し、コントロールに十分な余白を確保する（英語比 30〜50% 増）。
- 右から左（RTL）言語をサポートする場合は `WS_EX_LAYOUTRTL` を使用する。

---

## 11. プロジェクト固有のルール

<!-- プロジェクトに応じて追記してください -->

- {プロジェクト固有のルールをここに記載}

---

## 関連スキル

- `skills/cpp-coding-standards/SKILL.md` — C++ コーディング規約（命名・フォーマット・エラーハンドリング）
- `skills/windows-ui-guidelines/SKILL.md` — Windows UI ガイドライン（WinUI 3 / Fluent Design）
- `skills/ui-accessibility/SKILL.md` — アクセシビリティ共通原則
- `skills/ui-review-checklist/SKILL.md` — UI レビューチェックリスト
- `skills/security-practices/SKILL.md` — セキュリティプラクティス
- `skills/i18n-localization/SKILL.md` — 国際化・ローカリゼーション
- `skills/performance-optimization/SKILL.md` — パフォーマンス最適化
