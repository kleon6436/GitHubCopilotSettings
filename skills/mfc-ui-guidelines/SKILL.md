---
name: mfc-ui-guidelines
description: 'Reference and apply UI design and implementation guidelines for MFC applications. Use when reviewing or applying: Document/View architecture, message maps, dialog design, DDX/DDV, control layout, ribbon UI, GDI resource management, CMFCVisualManager, high-DPI support, accessibility, UI thread management. Use when: designing MFC application UI, reviewing MFC dialog layout, message handling, Document/View architecture, ribbon UI, GDI resource management, CMFCVisualManager theming, high-DPI support, accessibility in MFC apps.'
argument-hint: 'MFC UI guideline item to review or apply (optional)'
---

# MFC Application UI Guidelines

## Overview

This skill defines UI design and implementation guidelines for MFC (Microsoft Foundation Classes) applications.
Assumes modern C++ (C++17 or later) and covers MFC-specific UI patterns, resource management, and accessibility.
For general C++ coding conventions, refer to `skills/cpp-coding-standards/SKILL.md`.

> **Target**: Visual Studio 2019 or later / MFC Feature Pack environment / Windows 10 or later

---

## 1. Application Structure

### 1.1 Application Type Selection Criteria

| Type | Use Case | Base Class |
|------|-----------|-----------|
| SDI (Single Document) | Single-file editing app | `CFrameWnd` |
| MDI (Multiple Documents) | Multi-file simultaneous editing app | `CMDIFrameWnd` / `CMDIChildWnd` |
| Dialog-based | Utility / settings tool | `CDialogEx` |
| Tabbed MDI | Modern tab-switching UI | `CMDIFrameWndEx` + `CMDITabInfo` |

### 1.2 Document/View Architecture

- **CDocument**: Responsible for data storage and persistence (serialization). Do not include UI logic.
- **CView**: Responsible for rendering and user interaction. Manipulate data through CDocument only.
- Document and View synchronize via `UpdateAllViews()` / `OnUpdate()`.
- Use document templates (`CSingleDocTemplate` / `CMultiDocTemplate`) to bind Document, View, and Frame together.

```cpp
// ✅ Good — Document handles data management only
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

// ❌ Bad — Directly manipulating UI inside Document
class CMyDocument : public CDocument {
 public:
  void AddRecord(Record record) {
    records_.push_back(record);
    AfxMessageBox(_T("Record added"));  // UI logic mixed in
  }
};
```

---

## 2. Message Handling

### 2.1 Message Map Basics

- Use message map macros (`BEGIN_MESSAGE_MAP` / `END_MESSAGE_MAP`).
- Name handler functions as `On` + verb + target (e.g., `OnClickSave`, `OnPaint`).
- `ON_COMMAND` handlers should be named `OnCmd` + command name.
- Define custom messages using `WM_APP` as a base; avoid magic numbers.

```cpp
// ✅ Good — Clear naming and custom message definitions
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

// ❌ Bad — Magic numbers, ambiguous naming
BEGIN_MESSAGE_MAP(CMainFrame, CFrameWndEx)
  ON_COMMAND(ID_FILE_SAVE, &CMainFrame::OnFunc1)
  ON_MESSAGE(WM_APP + 100, &CMainFrame::Handler2)  // Meaningless constant
END_MESSAGE_MAP()
```

### 2.2 Command UI Update

- Use `ON_UPDATE_COMMAND_UI` to control the enabled/disabled state of menus and toolbars.
- Only call `pCmdUI->Enable()` / `pCmdUI->SetCheck()` inside the handler. Do not include heavy processing.

```cpp
// ✅ Good
void CMainFrame::OnUpdateCmdEditUndo(CCmdUI* pCmdUI) {
  pCmdUI->Enable(CanUndo());
}
```

---

## 3. Dialog Design

### 3.1 Choosing Between Modal and Modeless

| Type | Usage | Creation Method |
|------|---------|------|
| Modal | Wait for user to complete input (settings, confirmation) | `DoModal()` |
| Modeless | Use in parallel with main operation (search, tools) | `Create()` + `ShowWindow()` |

### 3.2 DDX/DDV (Data Exchange / Validation)

- Use DDX macros to synchronize control values with member variables.
- Perform validation using DDV macros or in `OnOK()`.
- Initialize member variables in the constructor; supplement UI initialization in `OnInitDialog()`.

```cpp
// ✅ Good — Synchronizing and validating values using DDX/DDV
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

// ❌ Bad — Manually manipulating controls without DDX
void CSettingsDialog::OnOK() {
  CString text;
  GetDlgItemText(IDC_EDIT_USERNAME, text);  // Should use DDX
  user_name_ = text;
  CDialogEx::OnOK();
}
```

### 3.3 Control Layout Guidelines

- Set tab order to a logical sequence in the resource editor (left→right, top→bottom).
- Group related controls using group boxes.
- Place OK / Cancel buttons at the bottom-right of the dialog (Windows standard layout).
- Ensure a minimum margin of 7 DLU (Dialog Units) between controls.

---

## 4. Controls and Layout

### 4.1 Recommended Control Selection

| Purpose | Recommended Control | Notes |
|------|----------------|------|
| Text input | `CEdit` / `CMFCMaskedEdit` | Use MaskedEdit for masked input |
| Multi-line text | `CRichEditCtrl` | For formatted text |
| Selection (few items) | `CComboBox` | Up to 10 items |
| Selection (many items) | `CListCtrl` (report view) | When sorting/filtering is needed |
| Tree display | `CTreeCtrl` | Hierarchical data |
| Property list | `CMFCPropertyGridCtrl` | Ideal for settings screens |
| Date/time input | `CDateTimeCtrl` | |
| Progress display | `CProgressCtrl` | |

### 4.2 Dynamic Layout (High-DPI Support)

- Use `CMFCDynamicLayout` to define control placement on resize.
- Use the resource editor's dynamic layout settings, or use `SetMinSize()` / `EnableDynamicLayout()` in code.
- Use ratio-based placement rather than fixed pixel values.

```cpp
// ✅ Good — Resize-ready with dynamic layout
BOOL CMyDialog::OnInitDialog() {
  CDialogEx::OnInitDialog();

  auto* layout = GetDynamicLayout();
  if (layout != nullptr) {
    // List control: no movement, size follows parent
    layout->AddItem(IDC_LIST_MAIN,
        CMFCDynamicLayout::MoveNone(),
        CMFCDynamicLayout::SizeHorizontalAndVertical(100, 100));
    // OK button: move to bottom-right, fixed size
    layout->AddItem(IDOK,
        CMFCDynamicLayout::MoveHorizontalAndVertical(100, 100),
        CMFCDynamicLayout::SizeNone());
  }
  return TRUE;
}

// ❌ Bad — Manual calculation in OnSize (tends to break DPI support)
void CMyDialog::OnSize(UINT nType, int cx, int cy) {
  CDialogEx::OnSize(nType, cx, cy);
  if (auto* pList = GetDlgItem(IDC_LIST_MAIN)) {
    pList->MoveWindow(10, 10, cx - 20, cy - 50);  // Fixed margin
  }
}
```

### 4.3 High-DPI Support

- Declare `<dpiAwareness>PerMonitorV2</dpiAwareness>` in the application manifest.
- Provide icons and bitmaps in multiple resolutions (16×16, 32×32, 48×48, 256×256).
- Obtain font sizes from system metrics; do not hardcode them.

---

## 5. Ribbon / Toolbar / Menu

### 5.1 Ribbon UI (MFC Feature Pack)

- Ribbon UI (`CMFCRibbonBar`) is recommended for new applications.
- Organize commands in a hierarchy of Category → Panel → Button.
- Register frequently used commands to the Quick Access Toolbar.

```cpp
// ✅ Good — Building the ribbon
void CMainFrame::InitializeRibbon() {
  auto* category = ribbon_bar_.AddCategory(
      _T("Home"), IDB_RIBBON_HOME, IDB_RIBBON_HOME_SMALL);

  auto* panel = category->AddPanel(_T("File"), icon_index);
  panel->Add(new CMFCRibbonButton(ID_FILE_NEW, _T("New"), 0, 0));
  panel->Add(new CMFCRibbonButton(ID_FILE_OPEN, _T("Open"), 1, 1));
  panel->Add(new CMFCRibbonButton(ID_FILE_SAVE, _T("Save"), 2, 2));
}
```

### 5.2 Toolbar

- Use `CMFCToolBar` for toolbars (not `CToolBar`).
- Always set tooltips for toolbar buttons.
- Use `CMFCToolBar::EnableCustomizeButton()` when customization is needed.

### 5.3 Menu Design

- Keep menu hierarchy to a maximum of 2 levels (at most 1 sub-menu level).
- Explicitly show accelerator keys (e.g., `Ctrl+S`) in menu items.
- Set mnemonics (e.g., `&File` → `Alt+F`) for all menu items.

---

## 6. GDI Resource Management

### 6.1 Using RAII Wrappers

- Manage GDI objects (`CPen`, `CBrush`, `CFont`, `CBitmap`) with RAII.
- Always restore the return value of `SelectObject()` (the old object).
- Check for GDI object creation failures.

```cpp
// ✅ Good — Managing GDI objects with RAII pattern
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

// ❌ Bad — Missing restoration of SelectObject
void CMyView::OnDraw(CDC* pDC) {
  CPen pen(PS_SOLID, 2, RGB(0, 120, 215));
  pDC->SelectObject(&pen);  // Old object not saved → leak
  pDC->Rectangle(rect);
  // pen is destroyed but still selected in DC → undefined behavior
}
```

### 6.2 Double Buffering

- Use double buffering to prevent flickering.
- Use `CMemDC` (MFC Feature Pack) or a manual memory DC.

```cpp
// ✅ Good — Double buffering with CMemDC
void CMyView::OnDraw(CDC* pDC) {
  CMemDC mem_dc(*pDC, this);
  CDC& dc = mem_dc.GetDC();

  // Perform drawing operations on dc
  dc.FillSolidRect(&client_rect, ::GetSysColor(COLOR_WINDOW));
  DrawContent(&dc);
}
```

### 6.3 Resource Leak Prevention Checklist

- [ ] Every `SelectObject()` call has a corresponding restoration of the old object
- [ ] Return values of `CreateCompatibleDC()` / `CreateCompatibleBitmap()` are checked
- [ ] Handles obtained via `LoadImage()` / `LoadBitmap()` are released with `DeleteObject()`
- [ ] Every `GetDC()` is paired with `ReleaseDC()`
- [ ] GDI objects are not created in a loop within drawing code

---

## 7. Visual Styles and Themes

### 7.1 Visual Manager

- Use `CMFCVisualManager` to unify the theme across the entire application.
- Select a preset based on project requirements.

| Visual Manager | Appearance |
|---------------|------|
| `CMFCVisualManagerOffice2007` | Office 2007 style |
| `CMFCVisualManagerWindows7` | Windows 7 Aero style |
| `CMFCVisualManagerVS2008` | Visual Studio 2008 style |

```cpp
// ✅ Good — Setting Visual Manager during application initialization
BOOL CMyApp::InitInstance() {
  CMFCVisualManager::SetDefaultManager(
      RUNTIME_CLASS(CMFCVisualManagerWindows7));

  // Subsequent UI initialization ...
}
```

### 7.2 Using System Colors

- Use `::GetSysColor()` instead of hardcoded colors.
- Verify correct display in high contrast mode.

```cpp
// ✅ Good — Using system colors
pDC->SetTextColor(::GetSysColor(COLOR_WINDOWTEXT));
pDC->SetBkColor(::GetSysColor(COLOR_WINDOW));

// ❌ Bad — Hardcoded colors
pDC->SetTextColor(RGB(0, 0, 0));
pDC->SetBkColor(RGB(255, 255, 255));
```

### 7.3 Common Controls v6

- Enable Common Controls v6 in the application manifest.
- This applies a modern appearance (Visual Styles).

```xml
<!-- ✅ Good — Enabling Common Controls v6 in the manifest -->
<dependency>
  <dependentAssembly>
    <assemblyIdentity type="win32" name="Microsoft.Windows.Common-Controls"
                      version="6.0.0.0" processorArchitecture="*"
                      publicKeyToken="6595b64144ccf1df" language="*" />
  </dependentAssembly>
</dependency>
```

---

## 8. Accessibility

### 8.1 Basic Principles

- Set an accessible name (label) for all controls.
- Place static text immediately before the labeled control in tab order.
- Implement `IAccessible` or provide a UI Automation provider for custom controls.

### 8.2 Keyboard Navigation

- Ensure all operations can be completed using keyboard only.
- Set tab order to a logical sequence in the resource editor.
- Assign shortcut keys (accelerators) to frequently used operations.
- Ensure focus movement is visually discernible.

```cpp
// ✅ Good — Using accelerator tables
BOOL CMainFrame::PreTranslateMessage(MSG* pMsg) {
  if (accel_table_ != nullptr &&
      ::TranslateAccelerator(m_hWnd, accel_table_, pMsg)) {
    return TRUE;
  }
  return CFrameWndEx::PreTranslateMessage(pMsg);
}
```

### 8.3 Screen Reader Support

- Respond to the `WM_GETOBJECT` message to provide MSAA / UI Automation information.
- Implement `accName`, `accRole`, and `accState` for custom-drawn controls.
- Fire accessibility events using `NotifyWinEvent()` when state changes occur.

---

## 9. Threading and Asynchronous Processing

### 9.1 UI Thread Principles

- Always update the UI on the main thread (UI thread).
- Use `PostMessage()` when updating the UI from a worker thread (`SendMessage()` risks deadlock).
- Delegate long-running tasks to worker threads; do not block the UI thread.

```cpp
// ✅ Good — Processing in worker thread, notifying completion via PostMessage
constexpr UINT WM_TASK_COMPLETE = WM_APP + 10;

// Worker thread function
UINT WorkerThreadProc(LPVOID pParam) {
  auto* context = static_cast<TaskContext*>(pParam);
  // Execute heavy processing ...
  context->frame_wnd->PostMessage(WM_TASK_COMPLETE, 0, 0);
  return 0;
}

// Launch thread from main frame
void CMainFrame::OnCmdStartProcessing() {
  AfxBeginThread(WorkerThreadProc, &task_context_);
  status_bar_.SetPaneText(0, _T("Processing..."));
}

// Completion notification handler (runs on UI thread)
LRESULT CMainFrame::OnTaskComplete(WPARAM, LPARAM) {
  status_bar_.SetPaneText(0, _T("Done"));
  UpdateDataView();
  return 0;
}

// ❌ Bad — Directly manipulating UI from worker thread
UINT WorkerThreadProc(LPVOID pParam) {
  auto* dlg = static_cast<CMyDialog*>(pParam);
  // Heavy processing ...
  dlg->SetDlgItemText(IDC_STATUS, _T("Done"));  // UI manipulation from another thread → undefined behavior
  return 0;
}
```

### 9.2 Thread-Safe Progress Updates

- Update `CProgressCtrl` from the main thread via `PostMessage`.
- Use `std::atomic` for sharing progress values.

---

## 10. Strings and Internationalization

### 10.1 Basic String Policy

- Projects assume **Unicode build** (`_UNICODE` / `UNICODE`). Do not use MBCS.
- Manage all strings displayed in the UI in the resource string table (`.rc`).
- Do not hardcode UI text in source code.

```cpp
// ✅ Good — Using resource strings
CString message;
message.LoadString(IDS_FILE_SAVE_SUCCESS);
AfxMessageBox(message);

// ❌ Bad — Hardcoded UI text
AfxMessageBox(_T("File saved successfully."));
```

### 10.2 CString Usage Guidelines

- Use `CString` for interfaces with MFC APIs.
- `std::wstring` / `std::wstring_view` may also be used in internal logic.
- Format using `CString::Format()`; avoid manual string concatenation.

```cpp
// ✅ Good
CString status;
status.Format(IDS_ITEM_COUNT_FORMAT, item_count);  // resource string: "Item count: %d"

// ❌ Bad — Manual concatenation + hardcode
CString status = _T("Item count: ") + std::to_wstring(item_count).c_str();
```

### 10.3 Internationalization Considerations for Layout

- Allow sufficient margin in controls for text expansion (30–50% more than English text).
- Use `WS_EX_LAYOUTRTL` when supporting right-to-left (RTL) languages.

---

## 11. Project-Specific Rules

<!-- Add entries specific to your project as needed -->

- {Add project-specific rules here}

---

## Related Skills

- `skills/cpp-coding-standards/SKILL.md` — C++ coding conventions (naming, formatting, error handling)
- `skills/windows-ui-guidelines/SKILL.md` — Windows UI guidelines (WinUI 3 / Fluent Design)
- `skills/ui-accessibility/SKILL.md` — Accessibility common principles
- `skills/ui-review-checklist/SKILL.md` — UI review checklist
- `skills/security-practices/SKILL.md` — Security practices
- `skills/i18n-localization/SKILL.md` — Internationalization and localization
- `skills/performance-optimization/SKILL.md` — Performance optimization
