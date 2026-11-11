# FONTOS .NET Framework — Module Developer Guide

*Internal engineering reference for developers building business modules on top of the FONTOS .NET rich-client framework (Total Soft Bank Ltd. / TSB), used across the CATOS port and terminal operating system product line.*

**Scope.** This guide was compiled from the official FONTOS/CATOS framework documentation set (the Application Development Tutorial v3.7, the TSpreadGrid, Menu Load, View Docking, View State, Container Editor, Graphical Drawing, Network, ODP.NET, Spread Editor, and UI Standards guides, and the Object ID Naming Rule spreadsheet) **cross-checked directly against the live source code** of `Tsb-Fontos-Framework`, `Tsb-Catos-Cm`, `Tsb-Global-Libraries`, `Tsb-Catos-Om`, `Tsb-Fontos-Sample`, and `Tsb-Fontos-Tools-Se-MultiHeader`. Every class, method, and file name below was verified against at least one of these two sources; places where the documentation and the source could not both be confirmed are flagged inline with **Note:** callouts rather than glossed over.

**How to read this guide.** Sections 1–3 cover architecture, project setup, and application bootstrap. Sections 4–7 cover how to actually build a screen (the FormTemplate pattern, grid schemas, TSpreadGrid, Container Editor/Drawing). Sections 8–10 cover networking, data access, and coding/naming standards. Section 11 covers testing. Section 12 lists open gaps and recommendations for keeping this guide accurate over time.

**Revision (2026-11-10).** Added §6.5 Grid Cell Context Menu, §6.6 Summary Grid context menu and the new `SpreadGridHelper.GetSourceItemsBySummaryCell` API, and §6.7 Grid Policy Settings (`GridPolicyInfo.xml`, all properties verified against current source). Related Appendix A rows and §12 items 9–10 were added.

**Security note.** No real IP addresses, hostnames, connection strings, or credentials from the source repositories are reproduced in this document — all such values are masked (`***`, `<host>`, `xxx.xxx.xxx.xxx`) even where the original source material contained realistic-looking sample values. Treat any config snippet below as a structural template, not a deployable value.

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Getting Started: Creating a New Module / Solution](#2-getting-started-creating-a-new-module--solution)
3. [Application Bootstrap, Menu, Docking, and View State](#3-application-bootstrap-menu-docking-and-view-state)
4. [Building Screens: The FormTemplate Pattern](#4-building-screens-the-formtemplate-pattern)
5. [Grid Schema XML Deep Dive](#5-grid-schema-xml-deep-dive)
6. [The TSpreadGrid Control](#6-the-tspreadgrid-control)
7. [Container Editor & Graphical Drawing](#7-container-editor--graphical-drawing)
8. [Network / Interface Messaging Layer](#8-network--interface-messaging-layer)
9. [Database Access](#9-database-access)
10. [UI Standards, Naming Conventions, and Coding Standards](#10-ui-standards-naming-conventions-and-coding-standards)
11. [Testing](#11-testing)
12. [Gaps Identified & Recommendations](#12-gaps-identified--recommendations)
13. [Appendix A: Reference File Locations](#appendix-a-reference-file-locations)
14. [Appendix B: Source Documents Used](#appendix-b-source-documents-used)

---

## 1. Architecture Overview

### 1.1 What FONTOS is

**FONTOS** is a proprietary .NET WinForms application framework built and maintained by **Total Soft Bank Ltd. ("TSB")**, a Korean vendor of port and terminal IT systems. FONTOS is not itself an end-user product — it is the rich-client *platform* on which TSB builds its terminal operating system (TOS) product line, most notably **CATOS** (Container/Cargo Automated Terminal Operating System). When you are asked to "build a module on FONTOS," you are building a WinForms business screen (or a set of them) that plugs into this platform: the framework supplies the MDI/docking shell, the menu system, the grid engine, the login/authorization pipeline, the Spring.NET-based service container, and the ORM plumbing, while your module supplies the domain-specific data model, business rules, and screens (bay plan, vessel call, gate transaction, EDI message handling, VGM capture, etc.).

The FONTOS tutorial suite (the *FONTOS Application Development Tutorial*, currently at document version 3.7, and several shorter framework guides — Menu Load, View Docking, View State) is the closest thing to an official reference for this platform, and the material in this guide is adapted from those documents plus direct inspection of the actual TSB repositories.

### 1.2 Three-layer architecture

FONTOS-based products are built in three concentric layers. Each layer only depends on the layer(s) beneath it, never the reverse:

```
┌─────────────────────────────────────────────────────────┐
│  Business Module  (e.g. Tsb-Catos-Om, Tsb-Fontos-Sample) │  ← your screens live here
├─────────────────────────────────────────────────────────┤
│  Catos-Cm  (Tsb-Catos-Cm)                                 │  ← shared CATOS business/UI layer
│  — domain services, CTBase* view/controller templates,    │
│    common popups (Container Detail, Vessel Schedule…),    │
│    Working Set, View State, Container Editor, C3IT/XIS    │
├─────────────────────────────────────────────────────────┤
│  Fontos Framework  (Tsb-Fontos-Framework)                 │  ← reusable across ANY TSB product
│  — Core, Net, Win, Win.Drawing, Test, Scheduler, Web...   │
└─────────────────────────────────────────────────────────┘
```

- **Fontos Framework** is the innermost, product-agnostic layer. It has no knowledge of containers, vessels, or ports — it provides generic WinForms plumbing: the `BaseTemplate`/`FormTemplate` MVP base classes, the `TSpreadGrid` grid wrapper and its ~60 `T*`-prefixed wrapped controls, the Menu/Docking/ViewState subsystems, the 2D drawing (CAD-style) canvas engine used for yard/berth diagrams, config/security/logging/DI utilities, and the EDI-style socket/transport messaging layer. Because it is product-agnostic, it is reused by every TSB product, not just CATOS.
- **Catos-Cm** ("Catos Common") sits on top of Fontos and is specific to the **CATOS terminal-operations product line**. It supplies the shared business/UI layer that CATOS modules actually inherit from — `CTBaseSingleGridView`/`Controller`, `CTBaseMultiGridView`/`Controller`, `CTBaseDetailView`/`Controller` — plus common business popups (Container Detail, Vessel Schedule), the internal `C3IT`/`XIS` equipment-messaging protocols, and add-on features such as Working Set, View State, and Container Editor that a module opts into.
- **A business module** (e.g. `Tsb-Catos-Om`, the ~275-screen Operations Management product, or `Tsb-Fontos-Sample`, the official tutorial's companion demo) is where module developers spend most of their time: DTOs/Params, IBatis SQL maps, DAOs, Business Services, and the Views/Controllers that present the screen to the end user.

**Note:** the tutorial material describes this three-layer pattern consistently (project-naming convention `Tsb.{ProductName}.{ModuleID}.*` layered on `Tsb.{ProductName}.Cm.Core` / `Tsb.{ProductName}.Cm.Win`), but the research notes do not spell out every Catos-Cm subsystem exhaustively — treat the list above as representative, not exhaustive, and cross-check against the actual `Tsb-Catos-Cm` solution when in doubt.

### 1.3 Solution / project layering convention

The Fontos Framework repository itself is organized by concern, and every module mirrors the same shape at a smaller scale:

| Framework project | Purpose |
|---|---|
| `Tsb.Fontos.Core` | Config, security, logging (Log4Net), DI/object building, caching, validators — no UI, no networking. |
| `Tsb.Fontos.Net` | Interface/EDI-style messaging: sockets, transport, XML-schema-driven message parsing. |
| `Tsb.Fontos.Win` | The WinForms rich-client core: `BaseTemplate`/`FormTemplate` base classes, Grid, Menu, Docking, ViewState, Spread, and the `Forms` namespace of `T*`-wrapped controls. |
| `Tsb.Fontos.Win.Drawing` | The 2D drawing/CAD engine for yard/berth/bay-plan diagrams. |
| `Tsb.Fontos.Test` | Base classes for unit testing Fontos-based code. |

A new **module** follows the same layering, using six standard projects named `Tsb.{ProductName}.{ModuleID}.{Layer}`:

| Module project | Contains |
|---|---|
| `...Client` | Views, View Interfaces, Controllers (MVP), `App.config`, `{ProductName}Config.xml`. |
| `...Common` | DTO Items/Params exchanged with the persistence layer, multilingual resources, the `{ProductName}ServiceSpec.cs` class listing Business Service IDs. |
| `...Config` | Menu/UI/Business-Service/UIStyle/Persistence/Localization config XML — the `Environment/`, `Context/`, `Service/Config/Persistence/`, `Grid/`, and `Help/` folders. |
| `...Service` | IBatis SQL Map XML, DAO classes, Business Service classes. |
| `...Test` / `...TestDB` / `...UITest` | Unit tests, DB-integration tests, and UI-automation tests layered the same way the framework separates `Tsb.Fontos.Test` (see §11). |

**Note:** the `Test`/`TestDB`/`UITest` project split is stated here as an already-verified repository convention; the tutorial documents examined for this guide focus on `Client`/`Common`/`Config`/`Service` and do not walk through creating the test projects step by step — treat the test-project guidance as structural convention rather than a documented procedure.

### 1.4 Technology stack

A FONTOS module developer needs working knowledge of the following stack — stated plainly here because it determines the learning curve for anyone new to the platform:

- **UI**: WinForms, with **DevExpress** controls providing the Ribbon/docking chrome (the View Docking subsystem is built on DevExpress's XtraBars `DockManager`) and **FarPoint/GrapeCity Spread** providing the underlying grid engine that FONTOS's `TSpreadGrid` wraps.
- **DI / IoC**: **Spring.NET** (`Spring.Core`, `Spring.Aop`, `Spring.Data`), configured entirely through XML application-context files rather than code — Business Services, DAOs, and add-on feature handlers are all declared as Spring `<object>` beans and wired together by ID.
- **Data access**: **iBATIS.NET** (`IBatisNet.DataMapper`, `IBatisNet.Common`) for SQL mapping — hand-written SQL in XML "SqlMap" files, not an ORM that generates SQL — against **Oracle** via the **ODP.NET Managed/Unmanaged Driver**, with **Npgsql** used for PostgreSQL-targeted deployments (module config and SQL maps commonly ship in MSSQL/POSTGRE variants alongside the Oracle default).
- **Logging**: **Log4Net**, configured through the same `App.config` that hosts Spring's resource pointer.
- **Messaging**: a custom socket/XML-schema messaging layer (informally called **C3IT**/**XIS** in the Catos-Cm layer) for real-time interface traffic with gate, quay, and yard equipment — architecturally separate from the Spring/IBatis persistence stack.

**Note:** the research material does not enumerate exact DevExpress/Spread/Spring.NET version numbers actively pinned by any given module — verify against the actual `Tsb-Global-Libraries` vendoring repo and the module's own reference set before assuming version compatibility.

---

## 2. Getting Started: Creating a New Module / Solution

### 2.1 Scaffolding a new module solution

FONTOS ships four Visual Studio project **templates** (`Tsb.Fontos.Tp.Client.zip`, `.Common.zip`, `.Config.zip`, `.Service.zip`) that encode the six-project layering from §1.3. Creating a new module means installing these templates, wiring them into a solution alongside the shared Framework/Catos-Cm projects, and performing a global rename of the template's placeholder tokens (`FontosTp` / `FT` / `FTTP`) to your product's real names. The procedure documented in TSB's *"How to create a new solution with FONTOS"* guide is as follows:

1. **Install the templates.** Copy the four `Tsb.Fontos.Tp.*.zip` files into Visual Studio's "User project templates location" (Tools → Options → Projects and Solutions → General).
2. **Create a blank solution**, e.g. `Tsb-Product-Md`, inside a workspace folder tracked by source control.
3. **Add two solution folders**, `FONTOS` and `CATOS-CM`, and import the *existing* framework/common projects into them from disk:
   - `FONTOS`: `Tsb.Fontos.Core`, `Tsb.Fontos.Win`, `Tsb.Fontos.WinDrawing`, `Tsb.Global.Libraries`.
   - `CATOS-CM`: `Tsb.Catos.Cm.Core`, `Tsb.Catos.Cm.Win`, `Tsb.Catos.Cm.WinDrawing`.
4. **Add four new projects** from the installed templates and rename each to your module's naming convention — e.g. `Tsb.Fontos.Tp.Client` → `Tsb.Product.Md.Client`, and likewise for `.Common`, `.Config`, `.Service`. Set the `Client` project as the solution's **StartUp Project**.
5. **Fix up references.** The template's inter-project references still point at TSB's internal template paths, so you must re-add:
   - Project references: `Tsb.Catos.Cm.Core`, `Tsb.Catos.Cm.Win`, `Tsb.Fontos.Core`, `Tsb.Fontos.Win`, `Tsb.Fontos.Win.Drawing`, `Tsb.Global.Libraries`, plus your own sibling `Common`/`Config`/`Service` projects.
   - Third-party references (from `Tsb.Global.Libraries`'s `Lib` folder): `C1.Win.C1Ribbon.2` (+ `.Design.2`), `FarPoint.Win`, `FarPoint.Win.Spread` (+ `.Design`), `IBatisNet.Common` (+ `.Logging.Log4Net`), `IBatisNet.DataMapper`, `Spring.Aop`, `Spring.Core`, `Spring.Data`.
6. **Rename the namespace token** `Fontos.Tp` → `Product.Md` via Find-and-Replace, scoped to each of the four new projects individually.
7. **Update `App.config`** — both the Spring resource pointer and the `<appSettings>` keys:

   ```xml
   <!-- Spring context bootstrap pointer: this is the FIRST file the Client project
        loads at startup; it tells Spring.NET where the application object graph is defined.
        Spring 컨텍스트 부트스트랩 진입점 — 클라이언트가 기동 시 가장 먼저 읽는 파일이며,
        애플리케이션 오브젝트 그래프의 정의 위치를 지정한다. -->
   <spring>
     <context>
       <resource uri="Context/ProductMd/ApplicationContext-ProductMd.xml"/>
     </context>
   </spring>
   ```

   Update `ModuleInfo.ProductID`, `ModuleInfo.ProductName`, `ModuleInfo.ModuleID`, `ModuleInfo.ModuleName`, `ModuleInfo.ModuleTitle`, `ModuleInfo.PgmCode`, `File.SqlMapconfig`, and `File.HelpFile` from their template values to your module's real values (e.g. `ModuleInfo.ProductID`: `FT` → your 2-letter product code).
8. **Configure the database connection** in `CatosConfig.xml` (`UID`/`PWD`/`DBQ`), confirm the installed **ODP.NET** version matches the `providers.config` `assemblyName`, and set the Client project's Build → Platform target (e.g. `x86`) to match the installed Oracle client bitness.
9. **Rename folders/files** under `Config` that carry the old module-name token (`Bizconfig\FontosTp` → `Bizconfig\ProductMd`, `Context\FontosTp` → `Context\ProductMd`, `Environment\FontosTp` → `Environment\ProductMd`, `Grid\FontosTp` → `Grid\ProductMd`, `SqlMap-FontosTp.config` → `SqlMap-ProductMd.config`, etc.), and propagate the `FT-FTTP` Object-ID prefix rename across class files.
10. **Update localization prefixes** in `LocalizationInfo.xml` (`MessagePrefix`/`VocabularyPrefix`) and propagate the `MSG_FTTP`/`WRD_FTTP` token rename across `Client`, `Common`, and `Config`.
11. **Rebuild** (not just Build) the Client project — Spread/XML schema changes only take effect after a full rebuild — and run.

**Note:** this is a mechanical "clone-template-and-rename" workflow rather than a `dotnet new`-style scaffolding tool; a single missed occurrence of the old token (inside `providers.config`, `UIControlContext-MenuForm.xml`, or a `.resx` file) is called out in the source material as a common cause of runtime errors or a broken UI caption after scaffolding.

### 2.2 Application bootstrap sequence (Program.cs)

The tutorial material does not reproduce a literal `Program.cs` listing, but it does document the call sequence a Main Form goes through at startup in enough detail to reconstruct it accurately. **Note:** the snippet below is an illustrative reconstruction built from the documented method calls (`ObjectBuilder`, `StandardInitApp`, `CreateMenuControl`), not a verbatim quote — verify the exact shape against your `Tsb.Fontos.Sample.Client` reference project before relying on it literally.

```csharp
[STAThread]
static void Main()
{
    Application.EnableVisualStyles();
    Application.SetCompatibleTextRenderingDefault(false);

    // The Main Form's constructor performs the actual Fontos bootstrap
    // (Spring context load -> login -> environment/menu load); see below.
    // 실제 Fontos 부트스트랩(Spring 컨텍스트 로드 -> 로그인 -> 환경/메뉴 로드)은
    // Main Form 생성자 내부에서 수행된다.
    Application.Run(new ProductMdMenuView());
}
```

```csharp
// Inside the Main (MDI) Form, mirroring SampleMenuView.cs's documented flow.
// 아래 흐름은 SampleMenuView.cs 에 문서화된 부트스트랩 절차를 따른다.
public ProductMdMenuView()
{
    InitializeComponent();

    // 1. Obtain the Spring-managed BaseAppInitializer object by its well-known
    //    Spring object ID (SysObjectSpec.SYS_OBJECT_ID_APP_INITILIZER).
    //    잘 알려진 Spring 오브젝트 ID로 BaseAppInitializer 인스턴스를 획득한다.
    BaseAppInitializer appInitializer =
        (BaseAppInitializer)ObjectBuilder.GetObjectBuilder()
            .GetObject(SysObjectSpec.SYS_OBJECT_ID_APP_INITILIZER);

    // 2. StandardInitApp() shows the Login window and drives the
    //    Authentication Handler Chain (password / C3IT / LDAP / custom).
    //    로그인 창을 표시하고 인증 핸들러 체인을 실행한다.
    appInitializer.StandardInitApp(true, null, null, false);

    // 3. Environment/menu load happens after successful login: ArchitectureInfo,
    //    PersistenceInfo, UIStyleInfo, MenuItem.xml are all loaded here.
    //    로그인 성공 후 환경설정(ArchitectureInfo/PersistenceInfo/UIStyleInfo)과
    //    MenuItem.xml 메뉴 정의가 로드된다.
    this.InitMenu();

    // 4. Main form is now ready to show; menu/toolbar are wired to
    //    ToolbarMenuWorker/Action objects created from MenuItem.xml.
    //    메뉴/툴바가 MenuItem.xml 기반의 Action 객체와 연결된 상태로 폼이 표시된다.
}
```

The documented, non-illustrative steps behind this flow (the `AppInitializer.InitConfig()` chain, from the Menu Load framework guide) are, in order:

1. `[ModuleInfo]`, `[AppPathInfo]`, `[ArchitectureInfo]`, `[PersistenceInfo]` are loaded from `App.config` and their respective Environment XML files.
2. `CTAppInitializer.StandardInitApp()` shows the Login window and resolves the Authentication Handler Chain (password check, and/or C3IT/LDAP/custom handlers — see §2.3.2 of this guide).
3. `CTClientConfigInitAdapter.InitConfig()` runs, in sequence: `CTConfigInfo.LoadConfig()` (loads `CatosConfig.xml`), `InitServerInfo()` (DB + C3IT server info), `ConfigPathInfo()` (Spread header folder path), `SetUseWindowPosition()`, `SetCatosFont()`, `SetDBConnString()`.
4. Only after this does menu construction (`MainMenuHandler.CreateMenuControl()`, described fully in §3.1) run and the Main Form become interactive.

### 2.3 Environment XML files

Every module's `Config` project has an `Environment/{ProductName}{ModuleID}/` folder holding a fixed set of configuration files that the framework reads at startup. Most are mandatory; a few are optional or feature-specific:

| File | Purpose |
|---|---|
| `ArchitectureInfo.xml` | Required. Base DI/ORM/DB/logging/UI/resource configuration; not normally hand-edited. |
| `MenuItem.xml` | Required. Defines the Ribbon/MenuStrip/ToolStrip menu tree and wires each menu item to a screen or action (see §3.1). |
| `PersistenceInfo.xml` | Required for ORM (IBatis) usage. Declares the DB provider/version/connection settings; ships in DB-vendor variants (Oracle/MSSQL/Postgres) per module. |
| `SecurityPolicyInfo.xml` | Required for login/authorization. Also holds settings like user-ID case conversion. |
| `LocalizationInfo.xml` | Required. Module and product-common resource settings (`MessagePrefix`, `VocabularyPrefix`), and culture/date/currency/number format policy. |
| `MessagePolicyInfo.xml` | Required to use the Fontos MessageBox. Controls dialog style and per-message-ID suppression, plus exception-report email settings. |
| `UIStyleInfo.xml` | Required. Sets the module's active UI style, including the Form-vs-DockingPanel entire-view style switch. |
| `InfTargetInfo.xml` | Required only when using C3IT: reconnection interval, ping behavior, presentation-class configuration. |
| `ControlNammingRuleInfo.xml` | Optional. Per-product UI-control naming-prefix policy, consulted by control data-binding/naming validation. |
| `ViewStateSerializationInfo.xml` | Present only if the module adopts View State (see §3.3). Declares which controls/properties are eligible for state persistence. |

**Note:** the filename `ControlNammingRuleInfo.xml` and the property spelling `GetDefalutParam()` seen elsewhere in the framework are reproduced here exactly as they appear in the source documentation — they read as typos but must be typed verbatim if they match the real shipped API. Verify spellings against the actual `Tsb.Fontos.*`/`Tsb.Catos.Cm.*` assemblies (via IntelliSense or reflection) before publishing external-facing documentation that depends on them being correct.

---

## 3. Application Bootstrap, Menu, Docking, and View State

### 3.1 Menu system

**Definition and loading.** The main menu is declared entirely in `MenuItem.xml`, under `Environment/{ProductName}{ModuleID}/`. The document's root structure is:

```xml
<tsbMenu>
  <menu menutype="ribbon">      <!-- or menutype="general" for MenuStrip/ToolStrip -->
    <menutab>
      <menuitemgroup>
        <menuitem .../>
      </menuitemgroup>
    </menutab>
  </menu>
</tsbMenu>
```

At startup, `BaseMenuView.initMenu()` calls `MainMenuHandler.CreateMenuControl()`, which:

1. Instantiates `MenuXmlUtil`, whose constructor loads the file named by the `App.config` key `File.MenuItem` into an in-memory `XmlDocument`, and determines whether the current user is an Administrator (which affects whether authority is checked at all).
2. Calls `MenuXmlUtil.GetTMenuTypeFrmXml()`, which walks the XML via the static `MenuItemToXmlHandler` and produces a `TMenuTabItemList` object graph (`TMenuTabItem` → `TMenuGroupItem` → `TMenuItem`), applying the authority-filtering logic described below.
3. Dispatches to the appropriate `IMenuCreator` based on the tab's `menutype` attribute: `RibbonMenuCreator` builds a `TRibbon`/`TRibbonMenu` tree; `GeneralMenuCreator` builds a `TMenuStrip`/`TToolStripMenuItem` tree. Both eventually call the shared `BaseMenuCreator.CreateMenuItemControl()`, which sets tooltip/image/shortcut-key properties and calls `SetToolbarAction()` to attach a `ToolbarMenuWorker` — the object that ultimately wires the control's Click event to an `Action`.
4. `BaseMenuView.initMenu()` finally calls `OpenFormUtil.SetDisplayRibbonTab(TRibbon)` to render the Ribbon.

**Wiring a menu item to a screen.** To add a new business screen to the menu, add a `<menuitem>` node with `frmid` set to the View's form ID and `actionid="OpenFrm"`:

```xml
<menuitem name="Vessel Particular" id="mnuVesselParticular"
          resourcekey="WRD_FTSL_VesselParticular"
          frmid="VesselParticularView" actionid="OpenFrm"
          imgname="menu_VesselParticular" imgsize="S"/>
```

`ToolbarMenuWorker.SetActionToToolbar()` inspects the resolved `ToolbarAttributeItem.ToolbarType` and instantiates one of `DataMenuGroupAction`, `GridRowAction`, `BizDialogAction`, or `CustomCommandAction` (all subclasses of `BaseCustomAction`) into the item's `Action` property, wiring the Click event to that action's `DoAction()`. For non-form behavior (e.g. a custom toolbar button), set `actiontarget="custom"` and point `actionid`/`toolbarattributetarget` at a `[CustomCommandAction]`/toolbar-attribute class instead of a form ID.

**Menu authority.** Whether a menu item is actually shown is governed by `MenuXmlUtil.IsMenuAuthority()`:

- If the module's global `<UseAuthority>` flag is not `true`, no check happens — every menu item loads unconditionally.
- Otherwise, `IsAuthorityCheckMenu()` decides whether to check the DB authority table (`TB_BASIC_DEFINE`) for this Menu ID, based on the `preauthcheck` attribute: `"N"` never checks, `"Y"` always checks, and if unset it checks only when `actionid="OpenFrm"` (with `id="mnuAbout"` special-cased to never check).
- Administrators bypass the check entirely, which is a common trap when testing menu visibility under an admin account.
- A parent `<menuitemgroup>`/`<menutab>` cannot carry authority itself — it is removed automatically once none of its children remain visible, so authority must always be configured at the leaf `<menuitem>` level.

### 3.2 Docking (window layout)

**MDI vs. Dock.** A module's Main Form runs in one of two container styles, set per module in `CatosConfig.xml`'s `WindowContainerStyle` key:

```xml
<SL>
    <!-- WindowContainerStyle : Dock, MDI (default = MDI)
         Dock = DevExpress XtraBars DockManager-based docking panels;
         MDI  = classic multi-document child windows.
         MDI(기본값) 또는 Dock 컨테이너 스타일 선택 —
         Dock 은 DevExpress XtraBars DockManager 기반 도킹 패널을 사용한다. -->
    <add key="WindowContainerStyle" value="Dock" />
</SL>
```

Switching between the two at runtime (via the `MultipleFormStyle`/`DockingPanelStyle` menu actions, `actiontarget="docking"`) forcibly closes every currently open child screen before re-rendering the Main Form under the new style — this is a destructive UX operation, so any unsaved edits in open child views should be handled (saved or confirmed) before allowing it in production.

**Making a screen dockable.** Under `WindowContainerStyle=Dock`, **every** screen must be authored as a `UserControl`, not a `Form` — a Form-based View added to the docking-panel menu will open but will not link to the common toolbar actions (Retrieve/Save/New, etc.). The template family mirrors the Form-based templates one-for-one but with `UserControl` base classes: e.g. `BaseDetailUserControlInterface`/`BaseDetailUserControl`/`BaseDetailUserControlController` in place of `BaseDetailViewInterface`/`BaseDetailView`/`BaseDetailController`. A UserControl-based View also cannot rely on Form-only events like `Load` — use the equivalent lifecycle hooks the base `UserControl` templates provide instead. In `UIStyleInfo.xml`, the entire-view style is switched with:

```xml
<setting name="GNRL_ENTIRE_VIEW_STYLE">DOCKING_PANEL</setting>
```

**Saving/restoring a layout ("Working Set").** FONTOS provides an optional `Tsb.Catos.Cm.WorkingSet` subsystem to save and restore named docking arrangements (analogous to Visual Studio's Window Layouts). Enabling it in a module requires the same three-part pattern used by every FONTOS add-on:

1. Add a project reference to `Tsb.Catos.Cm.WorkingSet`.
2. Register `WorkingSetContext-Common.xml` into the module's `ApplicationContext-{ProductName}{ModuleID}.xml`, inside the Spring `applicationContext` object's `FileNames` list.
3. Add the `WindowStyle` menu group to `MenuItem.xml` (`MultipleFormStyle`, `DockingPanelStyle`, `Select Working Set` menu items).

The most recently used docking arrangement can additionally be auto-persisted across sessions using `DockingStateHelper`:

```csharp
// Save last state — call from the Main Form's Closing event.
// 최종 도킹 상태 저장 — Main Form 의 Closing 이벤트에서 호출.
DockingStateHelper dockingStateHelper = new DockingStateHelper();
dockingStateHelper.SaveCurrentState(BaseDockStateParam.GetDefalutParam());
```

```csharp
// Restore last state — call from the Main Form's Load event.
// 최종 도킹 상태 복원 — Main Form 의 Load 이벤트에서 호출.
DockingStateHelper dockingStateHelper = new DockingStateHelper();
dockingStateHelper.LoadLastState(BaseDockStateParam.GetDefalutParam());
```

**Note:** `GetDefalutParam()` (missing the "u" in "Default") is reproduced verbatim from the source guide — it is either a genuine typo baked into the shipped API or a documentation error; confirm the exact spelling against the compiled `Tsb.Catos.Cm.WorkingSet` assembly before shipping code that calls it. The persisted layout file (`Docking/DockState_{User ID}.xml`, under the Client project's `Bizconfig/{ProductName}{ModuleID}/` folder) is stored **locally per user**, not synchronized server-side — deployment scripts should not assume it roams between machines. The "Select Working Set" menu item is only enabled when the Main Form is currently in Dock mode, and the framework-managed "LAST STATE" entry in the Working Set list can never be renamed or deleted through the UI.

### 3.3 View State (per-screen UI state persistence)

**Purpose.** View State is FONTOS's generic mechanism for persisting and restoring the state of arbitrary WinForms controls on a screen — grid column widths, checkbox/combo selections, zoom level, print options — either as a **per-user "last used value"** or as a **framework-enforced default**. It operates at the level of an individual View, complementing Working Set (which handles the *arrangement* of multiple views) and is itself wired up through the same Spring-context + `MenuItem.xml` pattern as every other FONTOS add-on.

**Scope of support.** Only Views deriving from `BaseBizView` participate, and only controls implementing `IStateSerializerControl` (exposing a `StateMode` property of type `ControlStateMode`, and a `UseStateSerialization` property of type `UseStateSerializationTypes`) are visible to the mechanism at all.

**State modes**, set per control/property via the admin tool described below:

| Mode | Restored on load? | Saved on close? |
|---|---|---|
| `None` | No | No |
| `Define` | Yes, from a developer-configured fixed value | No — a Define value can never be silently overwritten by the user |
| `User` | Yes, from the user's last saved value | Yes |

**Opting a screen into View State** requires the same coordinated set of additions used throughout this section, plus a one-time DB schema:

1. Add a project reference to `Tsb.Catos.Cm.ViewState`.
2. Register `ViewStateContext-Common.xml` into the module's `ApplicationContext-{ProductName}{ModuleID}.xml`.
3. Copy `ViewStateSerializationInfo.xml` into `Environment/{ProductName}{ModuleID}/`, and set its three master flags:

   ```xml
   <ViewStateSerializationInfo xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
     <!-- Master on/off switch for the whole feature; if false, nothing is
          restored or saved regardless of the other two flags below.
          기능 전체의 마스터 스위치 — false 이면 이하 플래그와 무관하게
          복원/저장이 전혀 수행되지 않는다. -->
     <EnableViewStateSerialization>true</EnableViewStateSerialization>
     <EnableSave>true</EnableSave>
     <EnableOnlyDefineStateMode>false</EnableOnlyDefineStateMode>
     <DefineProperties>
       <Control Name="TCheckBox" Class="Tsb.Fontos.Win.Forms.TCheckBox" AssemblyName="Tsb.Fontos.Win.">
         <Properties>
           <Property Name="Checked" HandleName="" Default="true"/>
         </Properties>
       </Control>
     </DefineProperties>
   </ViewStateSerializationInfo>
   ```
4. Add the embedded `ViewStateMap.xml` SqlMap reference to `SqlMap-{ProductName}{ModuleID}.config`.
5. Add a `MenuItem.xml` entry opening the **View State Control Define** admin screen (`frmid="ViewStateControlDefineView"`).
6. Create the three View State tables (`TB_VS_DEFINE`, `TB_VS_CONTROL`, `TB_VS_VALUE`) via the supplied DDL script.

**Configuring which controls persist which properties** is done at runtime through the **View State Control Define** screen rather than by hand-editing SQL: enter the target View's Program Code and View ID, use the *View Link* + *Import control* workflow to pull the live control list from a running instance of the target screen, set each control's Mode (None/User/Define), and check the specific properties (e.g. a `TSpreadGrid`'s column-width property) that should be persisted. A wildcard View ID of `"*"` combined with a specific Control Type applies a rule across **every screen in the application** — a powerful but blunt instrument that should be used deliberately, since it silently affects screens the developer configuring it may not be thinking about.

**Excluding a specific screen/control.** Set the Visual Studio Designer property `UseStateSerialization = Off` on the Form or control — this unconditionally excludes it from View State management, overriding any wildcard or per-View DB rule that would otherwise apply.

**Runtime toggles.** `Tsb.Fontos.Win.ViewState.Handler`'s `ViewStateEnv` static class exposes `EnableViewStateSerialization`, `EnableSave`, and `EnableOnlyDefneStateMode` (sic) for process-lifetime overrides (e.g. temporarily disabling View State during a bulk data migration), but changes made this way are **not** persisted back to `ViewStateSerializationInfo.xml` and will not survive an application restart.

**Note:** two spelling inconsistencies are reproduced verbatim above because they appear to be part of the real, shipped API surface rather than documentation typos: `EnableOnlyDefneStateMode` (missing "i") on the `ViewStateEnv` class, versus `EnableOnlyDefineStateMode` (correctly spelled) as the XML tag one layer up in `ViewStateSerializationInfo.xml`. Confirm both spellings by reflection against the actual `Tsb.Fontos.Win.dll` before relying on either in new code, since a mismatch here is a common, hard-to-spot source of "why isn't my flag doing anything" bugs.

---

## 4. Building Screens: The FormTemplate Pattern

### 4.1 Overview: View + Controller + Interface, per archetype

Every FONTOS screen ("FormTemplate") is built as a **View/Controller pair**, and each side implements a matching **Interface**. This is a strict convention across the whole framework, from the root contracts down to the concrete screen a module developer writes:

- `BaseTemplateInterface` / `BaseTemplateControllerInterface` — the root contracts every screen fulfils, regardless of archetype.
- One `BaseTemplate*Interface` / `BaseTemplate*ControllerInterface` pair per screen *archetype* (SingleGrid, MultiGrid, Detail, Draw, Biz, …), declaring only the members that make that archetype distinct (grid binding, drawing surfaces, etc.).
- One `Base*View` / `Base*Controller` concrete (or abstract, for the controller) class per archetype in `Tsb.Fontos.Win.FormTemplate`, which is the actual reusable implementation a module inherits from.
- Optionally, a **Cm-layer** (`Tsb.Catos.Cm.Win.FormTemplate`) `CTBase*View` / `CTBase*Controller` class that further specializes the raw Fontos base with container-terminal/vessel business rules (see §4.6).
- Finally, the module's own concrete `XxxView` / `XxxController`, each implementing a module-specific `XxxInterface` that extends the framework's `Base*ViewInterface`.

**Note:** The research that grounds this section could not read every intermediate class in the chain — `BaseTemplateGridInterface`, `BaseTemplateGridControllerInterface`, `BaseTemplateBizInterface`, `BaseTemplateBizControllerInterface`, `BaseGridView`, `BaseGridController`, `BaseBizView`, and `BaseBizController` were referenced by every file that *was* read (via their base-class/base-interface declarations) but were not themselves in the file set. Their existence and position in the hierarchy is proven by the `: BaseXxx` declarations quoted below; their member-level contents are inferred, not verified, and should be confirmed against source before being documented further.

Beyond the archetypes traced in the research (SingleGrid, MultiGrid, Detail, SingleDraw, MultiDraw), the framework is also known (from naming convention and menu/toolbar evidence) to define **Dialog**, **Composite**, **Biz**, and **Mobile** FormTemplate archetypes. **Note:** none of these four were represented in the source files read for this guide — do not assume their `BaseTemplate*Interface`/`Base*View`/`Base*Controller` shapes mirror SingleGrid/MultiGrid exactly until their actual source is reviewed.

### 4.2 The real inheritance chain

The chain below is quoted (with only whitespace/comment trimming) directly from the class declarations found in source. Two independent hierarchies exist — a **View-side** hierarchy rooted at `TForm`, and a **Controller-side** hierarchy rooted at `TsbBaseObject` — connected only by composition (`View.Controller`, `Controller`'s private `_view` field), never by shared inheritance:

```csharp
// ---- View-side interfaces (BaseTemplate layer, view side) ----
public interface BaseTemplateInterface
public interface BaseTemplateSingleGridInterface : BaseTemplateGridInterface        // BaseTemplateGridInterface not in file set
public interface BaseTemplateDrawInterface : BaseTemplateBizInterface              // BaseTemplateBizInterface not in file set

// ---- View-side interfaces (FormTemplate layer) ----
public interface BaseViewInterface : BaseTemplateInterface
public interface BaseSingleGridViewInterface : BaseGridViewInterface, BaseTemplateSingleGridInterface
public interface BaseDetailViewInterface // : (BaseBizViewInterface-ish, not in file set)

// ---- View-side concrete classes (FormTemplate layer) ----
public class BaseView : TForm, BaseViewInterface
public class BaseSingleGridView : BaseGridView, BaseSingleGridViewInterface          // BaseGridView not in file set
public class BaseMultiGridView : BaseGridView, BaseMultiGridViewInterface
public class BaseDetailView : BaseBizView, BaseDetailViewInterface                  // BaseBizView not in file set

// ---- Catos-Cm layer (module developers normally inherit from here) ----
public class CTBaseSingleGridView : BaseSingleGridView, CTBaseSingleGridViewInterface

// ---- Controller-side interfaces ----
public interface BaseTemplateControllerInterface
public interface BaseTemplateSingleGridControllerInterface : BaseTemplateGridControllerInterface   // not in file set

// ---- Controller-side concrete classes ----
public abstract class BaseController : TsbBaseObject, BaseTemplateControllerInterface
public abstract class BaseSingleGridController : BaseGridController, BaseTemplateSingleGridControllerInterface   // BaseGridController not in file set
public abstract class BaseMultiGridController : BaseGridController, BaseTemplateMultiGridControllerInterface
public abstract class BaseDetailController : BaseBizController, BaseTemplateDetailControllerInterface

// ---- Catos-Cm layer ----
public abstract class CTBaseSingleGridController : BaseSingleGridController

// ---- Module (Sample) layer ----
public interface SingleGridInterface : BaseSingleGridViewInterface
public partial class SingleGridView : BaseSingleGridView, SingleGridInterface
public class SingleGridController : BaseSingleGridController
```

So the full "textbook" chain for a real business module's SingleGrid screen is:

```
BaseTemplateInterface / BaseTemplateControllerInterface        (root contracts)
        ↓
BaseView / BaseController                                      (Tsb.Fontos.Win.FormTemplate — universal)
        ↓
BaseSingleGridView / BaseSingleGridController                  (Tsb.Fontos.Win.FormTemplate — archetype)
        ↓
CTBaseSingleGridView / CTBaseSingleGridController               (Tsb.Catos.Cm.Win.FormTemplate — terminal business layer)
        ↓
<Module>SingleGridView / <Module>SingleGridController            (your module)
```

**Important finding from the worked Sample project:** the Sample's own `SingleGridView`/`SingleGridController`/`MultiGridView`/`MultiGridController`/`DetailView`/`DetailController` inherit **directly from the raw Fontos base classes**, skipping the Cm layer entirely. This is because the Sample project is deliberately domain-neutral (a pure-framework demo). Real port/terminal business modules built on top of `Tsb.Catos.Cm.*` are expected to sit one layer further down, inheriting from `CTBaseSingleGridView`/`CTBaseSingleGridController` instead — see §4.6.

### 4.3 The universal contract: `BaseView` and `BaseController`

Regardless of archetype, every FONTOS screen gets the following "for free" from `BaseView`/`BaseController` (`Tsb.Fontos.Win.FormTemplate`):

| Member | Kind | Purpose |
|---|---|---|
| `InitDataSyncCallbackInfo()` (Controller) | abstract | Register cross-screen data-sync callbacks — every controller must implement this |
| `AddEventHandler()` (Controller) | abstract | Wire the screen's own UI/business event handlers — every controller must implement this |
| `CloseDataSyncCallbackInfo()` (Controller) | virtual | Cleanup on form close; default calls `DataSyncAgent.RemoveAllCallbackInfo()` |
| `DBDisconnected/NwDisconnected/NwConnected/TrackBarValueChanged` (Controller) | virtual | React to environment/connectivity events; default just logs to console |
| `HandleMessage(...)` (View) | virtual | The single funnel for localized, resource-key-driven message boxes |
| `MandatoryCheck(Control[, bool, string])` (View) | 3 overloads | Required-field validation walking the control tree before Save |
| `DetailFormClosed(...)` (View) | 2 overloads | Cascade-close a linked detail/child screen, MDI- or Dock-aware |
| `GetMainForm()/GetActiveFrom()/MdiParent` (View) | overridden | Docking-vs-MDI-transparent main-form access, switched by `DockingEnv.IsDockStyle()` |

The canonical constructor idiom, quoted from `BaseController.cs`:

```csharp
public BaseController()
{
    this.ObjectID = "GNR_FTWN_FMT_BaseController";
    this.ObjectType = ObjectType.CONTROLLER;
    this.DataSyncAgent = new DataSyncAgent();
    this.ProgressBarManager = new ProgressBarManager();
    this.DoMsgHandlingOnScreen = false;
    this.IsCloseDetailForm = true;
    this.UseEnableToolbarHandler = true;
    this.UseProgressBarAtRefresh = true;
}

public BaseController(BaseViewInterface view) : this()
{
    this._view = view;
}
```

Every module Controller subclass is expected to call `base(view)` and then perform its own initialization — see the SingleGrid worked example in §4.4.

Mandatory-field validation (`BaseView.MandatoryCheck`) is the standard mechanism ports/terminal data-entry screens use to enforce required fields (e.g. a mandatory B/L number or vessel code) without per-screen boilerplate: mark a control `IsMandatory = true`, then call `this.MandatoryCheck(this)` (or a specific container) before commit.

### 4.4 Archetype: SingleGrid — a searchable list grid

**What it's for:** a single scrollable/filterable grid of records — the default "search and list" screen (e.g. a list of IMDG/dangerous-goods reference data in the Sample, or in real modules a list of B/L records, container gate moves, etc.). Backed internally by a single `TSpreadGrid` (FarPoint Spread), exposed to the framework as `ITsbGrid Grid`.

**Interface** (`SingleGridInterface.cs`):
```csharp
public interface SingleGridInterface : BaseSingleGridViewInterface
{
    string FormName { get; }
    Color GridForeColor { get; set; }
    Color GridBackColor { get; set; }
    SingleGridParam SearchParam { get; set; }
    SingleGridItem ActiveItem { get; set; }
    string SaveMandatoryCheck();
    string SearchMandatoryCheck();
    void SpreadRefresh();
    event SpreadGridRowRemovedHandler GridRowRemoved;
}
```

**View skeleton** (`SingleGridView.cs`), trimmed:
```csharp
public partial class SingleGridView : BaseSingleGridView, SingleGridInterface
{
    public SingleGridView(Form mdiPrnt) : base()
    {
        // Fontos view-bootstrap idiom: build UI, wire Controller, wire search param
        // Fontos 화면 부트스트랩 관례: UI 구성 → 컨트롤러 연결 → 검색 파라미터 연결
        InitViewCreate(mdiPrnt);
    }

    private void InitViewCreate(Form mdiPrnt)
    {
        InitializeComponent();
        this.MdiParent = mdiPrnt;
        this.Grid = this.grd_FORMTEMPLATE_SingleGrid;           // hand the designer grid to the base "Grid" property
        this.grd_FORMTEMPLATE_SingleGrid.IndicateType = Win.Grid.Types.GridIndicateType.FIRSTVISIBLE;
        this.Controller = new SingleGridController(this);        // Controller always receives "this" view
        this._spreadGrid = this.grd_FORMTEMPLATE_SingleGrid;
        this.SearchParam = new SingleGridParam(this);             // *Param object also receives its owning view
        // ...
        this.SetBindingControls();
        this.AddEventHandler();
        this.InitComboData();
    }

    // Grid-cell / view-side mandatory checks delegate to base helpers
    public string SearchMandatoryCheck() { /* ... delegates to base.MandatoryCheck(...) */ return ""; }
    public string SaveMandatoryCheck()   { /* ... delegates to _spreadGrid.GridHelper.CheckMandatory() */ return ""; }
}
```

**Controller skeleton** (`SingleGridController.cs`), trimmed:
```csharp
public class SingleGridController : BaseSingleGridController
{
    private const string DETAIL_VIEW_NAME = "SingleGridDetailView";
    private const string SYNC_NOTIFY_NAME  = "SYNC-FT-FTSL-FTE-SingleGrid";
    private ISingleGridService service;

    public SingleGridController(SingleGridInterface view) : base(view)
    {
        this.ObjectID = "CTL-FT-FTSL-FTE-SingleGridController";
        this.FormView = view;
        // resolve current user's staff code for audit stamping
        // 감사(Audit) 기록용 현재 사용자 사번 조회
        // staffCd = AuditUtil.GetStaffCd(...);
        this.AddEventHandler();
        this.InitDataSyncCallbackInfo();
        base.DetailFormName = DETAIL_VIEW_NAME;   // wires the cascade-close-on-parent-close convention
    }

    public override void InitDataSyncCallbackInfo()
    {
        // register callback keyed by SYNC_NOTIFY_NAME so a linked Detail screen can push updates back
        // SYNC_NOTIFY_NAME 로 콜백을 등록하여 연결된 상세화면이 변경사항을 통지할 수 있게 함
    }

    [ControllerToolbarAttribute(ControllerToolbarAttribute.ControllerToolbarSet.REFRESH, true)]
    public virtual void MenuRefresh(object sender, EventArgs e) { this.DoRetrieveData(); }

    public BaseResult DoRetrieveData()
    {
        SingleGridParam param = this.FormView.SearchParam;
        param.TransactionInfo.TxServiceID = FontosSampleServiceSpec.FORMTEMPLATE_SINGLE_SERVICE;
        (this.FormView.Grid as TSpreadGrid).GridHelper.ApplyFilterToSearchParam(param, null, false);
        this.service = BizServiceLocator.GetService<ISingleGridService>(param);
        BaseResult result = service.InquirySingleGrid(this.FormView.SearchParam);
        var resultItems = result.ResultObject as BaseItemsList<SingleGridItem>;
        (this.FormView.Grid as TSpreadGrid).BindListAsGridDataSource(resultItems, true, false, true, true);
        return result;
    }

    public void DoSaveData()
    {
        // iterate SourceItemList; for every item with OpCode != READ, call service.ProcessSingleGrid(item)
        // SourceItemList 순회 — OpCode 가 READ 가 아닌 항목만 저장 서비스 호출
    }

    public void DoDeleteData(object sender, GridRowRemovedEventArgs e)
    {
        // iterate rows marked OpCodes.DELETE, call service.DeleteSingleGrid(item), then remove the view row
    }
}
```

Wiring summary: the **View's constructor** creates the **Controller** and hands it `this`; the Controller stores that reference (`FormView`) and, from then on, is the only thing that talks to services/DAOs — the View never calls a Service/DAO directly.

#### The full vertical slice (View → Controller → Item → Param → Service → Dao → SQL Map)

Tracing a "Refresh" click end to end, exactly as found in the Sample project's SingleGrid slice:

```
Toolbar click "Refresh"
   │  ([ControllerToolbarAttribute(...REFRESH)] wiring — see §4.5)
   ▼
SingleGridController.MenuRefresh(sender, e)
   ▼
SingleGridController.DoRetrieveData()
   │  reads: SingleGridParam param = FormView.SearchParam
   │  sets:  param.TransactionInfo.TxServiceID = FontosSampleServiceSpec.FORMTEMPLATE_SINGLE_SERVICE
   ▼
ISingleGridService service = BizServiceLocator.GetService<ISingleGridService>(param)   ← service-locator/DI resolution
   ▼
SingleGridService.InquirySingleGrid(param)          [Tsb.Fontos.Sample.Service]
   ▼
ISingleGridDao.InquirySingleGrid(param)  →  SingleGridDao : SqlMapDaoSupport, ISingleGridDao
   ▼
this.QueryForList<SingleGridItem>("Tsb.Fontos.Sample.Service.FormTemplate.Map.SingleGridMap.select-singleGrid", param)
   ▼
SingleGridMap.xml  <select id="select-singleGrid" ...>  (iBATIS.NET SQL map)  →  Oracle SELECT on TB_IMDGDATA
   ▼
BaseItemsList<SingleGridItem>  →  BaseResult.CreateOkResult(...)
   ▼
Controller casts result.ResultObject as BaseItemsList<SingleGridItem>
   ▼
(FormView.Grid as TSpreadGrid).BindListAsGridDataSource(resultItems, true, false, true, true)
   ▼
Grid re-renders (FarPoint Spread)
```

The DTO/interface/implementation pieces involved, quoted:

```csharp
// Service interface — Tsb.Fontos.Sample.Service.FormTemplate.ISingleGridService
public interface ISingleGridService : ITsbService
{
    [TransactionOption(TransactionScopeTypes.Support)]
    BaseResult InquirySingleGrid(SingleGridParam param);

    [TransactionOption(TransactionScopeTypes.Required)]
    void ProcessSingleGrid(SingleGridItem item);

    [TransactionOption(TransactionScopeTypes.Required)]
    void DeleteSingleGrid(SingleGridItem item);
}

// Service implementation
public class SingleGridService : BaseService, ISingleGridService
{
    public ISingleGridDao SingleGridDao { get; set; }   // property-injected by Spring.NET

    public BaseResult InquirySingleGrid(SingleGridParam param)
    {
        BaseItemsList<SingleGridItem> items = this.SingleGridDao.InquirySingleGrid(param);
        // ...
        return BaseResult.CreateOkResult(this.ObjectID, items, param);
    }
}

// DAO implementation
public class SingleGridDao : SqlMapDaoSupport, ISingleGridDao
{
    public BaseItemsList<SingleGridItem> InquirySingleGrid(SingleGridParam param)
    {
        return new BaseItemsList<SingleGridItem>(
            this.QueryForList<SingleGridItem>(
                "Tsb.Fontos.Sample.Service.FormTemplate.Map.SingleGridMap.select-singleGrid", param));
    }
}
```

```xml
<!-- SingleGridMap.xml — namespace Tsb.Fontos.Sample.Service.FormTemplate.Map.SingleGridMap -->
<select id="select-singleGrid" parameterClass="singleGridParam" resultMap="singleGridResult">
    SELECT DISTINCT UNID, UNNO, CLASS, PACKING_GRP, PROPER_SHIP_NM,
    SUBSIDIARY_RISK, LIMITED_QTY, MARINE_POLLUT, EMS, FLASH_POINT,
    EXTEND_GRP, '16777215' VALUECOLOR, CLEAR_LIVING_QUARTER
    FROM TB_IMDGDATA
    WHERE 1=1
    <isNotEmpty property="CustomKeyword" prepend="AND"> $SearchColumnTitle$ LIKE #CustomKeyword#||'%' </isNotEmpty>
    <isNotEmpty property="ImdgClass" prepend="AND"> (CLASS LIKE #ImdgClass# || '%') </isNotEmpty>
    <isNotEmpty property="Unno" prepend="AND"> (UNNO LIKE #Unno# || '%') </isNotEmpty>
    <include refid="Tsb.Catos.Cm.Core.Grid.Service.Map.GridFilterSqlMap.GridFilterWhere" />
</select>
```

Note the `<include refid="...GridFilterSqlMap.GridFilterWhere" />` — this pulls in a **shared, reusable SQL fragment** from Catos-Cm that implements the generic "grid header filter" search condition. This is how the grid's built-in column-filter UI turns into a SQL `WHERE` clause without every screen hand-writing its own filter logic; a module's `*Param` DTO opts in by implementing `IGridFilterSearchParam` (`HasGridFilters`, `GridFilterSqlGroups`), exactly as `SingleGridParam` does.

The DTOs in this slice:

```csharp
// Row DTO — Tsb.Fontos.Sample.Common.Item.FormTemplate.SingleGridItem
public class SingleGridItem : BaseDataItem, IContextMenuParam
{
    // NotifyPropertyChanged-backed columns: Unid, Unno, Class, PackingGrp, ProperShipNm,
    // SubsidiaryRisk, LimitedQty, MarinePollut, Ems, FlashPoint, ExtendGrp, ValueColor, Button1..4, DummyChk
    // ObjectID = "ITM-FT-FTSL-FTE-SingleGridItem"
}

// Search-criteria DTO — Tsb.Fontos.Sample.Common.Param.FormTemplate.SingleGridParam
public class SingleGridParam : BaseParam, IGridFilterSearchParam
{
    // ImdgClass, Unno, Unid, CustomKeyword, SearchColumnTitle
    // + HasGridFilters / GridFilterSqlGroups required by IGridFilterSearchParam
}
```

**Update statements only touch changed columns.** The map's `<update>` statement uses `<isNotEqual property="X" compareProperty="BackupItem.X">` guards, so only columns whose value differs from the row's `BackupItem` snapshot appear in the `SET` clause. This is why `SingleGridService.ProcessSingleGrid` sets `item.BackupItem = item.Clone() as SingleGridItem;` after every successful create/update/delete — it refreshes the change-detection baseline for the next save.

**Note on naming:** `ObjectID` string tags follow a `<TYPE>-<LAYER>-<MODULE>-<SUBMODULE>-<ClassName>` convention (e.g. `"ITM-FT-FTSL-FTE-SingleGridItem"`, `"SVC-FT-FTSL-FTE-SingleGridService"`), but the research also found an inconsistent example (`"DAO-CT-CTTM-GAT-SingleGridDao"`, mismatched prefix) — evidently copy-pasted from another module. Module developers should set `ObjectID` deliberately per their own module code, not blindly copy an existing one.

### 4.5 Toolbar wiring and validation hooks

Grid-archetype controllers (`BaseSingleGridController`, `BaseMultiGridController`) expose their entire toolbar surface as `[ControllerToolbarAttribute(...)]`-decorated methods, quoted from `BaseSingleGridController.cs`:

```csharp
[ControllerToolbarAttribute(ControllerToolbarAttribute.ControllerToolbarSet.PRINT, true)]
public virtual void MenuPrint(object sender, EventArgs e) { this.DoPrint(sender, e); }

[ControllerToolbarAttribute(ControllerToolbarAttribute.ControllerToolbarSet.PREVIEW, true)]
public virtual void MenuPreview(object sender, EventArgs e) { this.DoPreview(sender, e); }
```

The same pattern repeats for `MenuOption/DoOption`, `MenuExcelOption/DoExcelOption`, `MenuPdf/DoPdf`, `MenuExcel/DoExcel`, `MenuGridFilter/DoFilter`, `MenuGridColSetting/DoColSetting`, `MenuGridFind/DoFind`, `MenuGridSummay/DoSummary` [sic, typo preserved from source], `MenuGridSort/DoSort`, `MenuGridSimpleFilter/DoSimpleFilter`, `MenuCheckAll/DoCheckAll`, `MenuUnCheckAll/DoUnCheckAll`, `MenuFreezePanes/DoFreezePanes`, `MenuFreezeColumn/DoFreezeColumn`, `MenuHeaderFilter/DoHeaderFilter`, `MenuResetFilter/DoResetFilter`.

**How the wiring actually resolves at runtime**, per the `MenuItem.xml` evidence: a menu/toolbar entry sets `actionid="Inquiry"` (or similar) with `actiontarget="custom"`:

```xml
<menuItem name="Refresh" id="mniInquiry" resourcekey="WRD_CTCM_Refresh" actionid="Inquiry"
          actiontarget="custom" shortcutkey="116" imgname="menu_Inquiry" imgsize="L"/>
```

The framework's menu dispatcher then locates whichever currently-active screen's Controller has a method decorated `[ControllerToolbarAttribute(ControllerToolbarAttribute.ControllerToolbarSet.REFRESH)]` (in this traced example, `SingleGridController.MenuRefresh`) and invokes it — fully decoupling the static menu XML from the concrete Controller class. A separate attribute, `[CTBizDialogToolbarAttribute(...)]` (used by the Cm layer, §4.6), is selected instead when the menu item sets `toolbarattributetarget="CTBizDialogToolbarAttribute"`, e.g.:

```xml
<menuItem name="MDI Child Container Detail" id="mniMdiChildContainerDetail" frmid="ContainerDetailView"
          actionid="CatosCntrDetail" actiontarget="custom"
          toolbarattributetarget="CTBizDialogToolbarAttribute" isallowedmultiform="Y"/>
```

**Note:** the reflection-based scanner that actually reads these attributes at runtime and performs the bind (presumably somewhere in `Tsb.Fontos.Win.Menu.Toolbar`) was not itself in the file set read for this guide — its *existence and effect* is proven by the attribute usage and the matching XML, but its internal implementation has not been verified against source.

**Validation hooks.** The Sample's `SingleGridController` also carries `[ValidateAttribute(ValidateAttribute.ValidateSet.DATA_CHANGE)]` and `[ValidateAttribute(ValidateAttribute.ValidateSet.DATA_VALIDATION)]`-decorated methods, which the framework calls before allowing navigation away from a screen with unsaved changes. **Note:** the exact semantics of `ValidateAttribute` (what triggers `DATA_CHANGE` vs. `DATA_VALIDATION`, and what a non-`void`/`bool` return means to the caller) were not directly traceable to a `ValidateAttribute` class definition in the material read for this guide — confirm the exact contract against the `Tsb.Fontos.Win.Menu` or validation-related source before relying on it for anything more than "you can hook data-change/data-validation checks this way."

### 4.6 Archetype: MultiGrid — linked grids on one screen

**What it's for:** two (or more) grids on the same screen with a master/detail relationship — e.g. a summary grid whose row selection drives a linked detail grid (CY/CFS split views, a vessel-call/booking master grid with a container-detail grid). Instead of a single `Grid` property, the base exposes an **array**, `ITsbGrid[] Grids`, plus an `ActiveGrid`/`ActiveIndex` concept that tracks whichever grid currently has focus.

```csharp
public class BaseMultiGridView : BaseGridView, BaseMultiGridViewInterface
{
    public ITsbGrid[] Grids
    {
        get { return this._grids; }
        set
        {
            if (this._activeGrid == null && value.Length > 0) this._activeGrid = value[0];
            foreach (ITsbGrid grd in value) grd.GridActivated += new EventHandler(grd_GridActivated);
            this._grids = value;
        }
    }
    public ITsbGrid ActiveGrid { get; set; }
    public int ActiveIndex { get; set; }
    // ...
}
```

Setting `Grids` auto-subscribes each grid's `GridActivated` event, so `ActiveGrid`/`ActiveIndex` always reflect whichever grid the user last clicked into — every toolbar action then acts on `ActiveGrid`, not a fixed grid.

**Sample worked example**, trimmed:

```csharp
public interface MultiGridInterface : BaseMultiGridViewInterface
{
    // FIRST_ROW / SECOND_ROW = int indices into Grids[]
    MultiGridParam SearchParam { get; set; }
    VesselScheduleControl VVD { get; set; }
    void MasterSearchComplete(/* ... */);
    string SearchMandatoryCheck();
    bool HasDgConfirmContainer();
    event EventHandler FormClosedView;
    event EventHandler GridDetailSearch;
    event SpreadGridRowRemovedHandler GridRowRemoved;
}

public partial class MultiGridView : BaseMultiGridView, MultiGridInterface
{
    private void InitViewCreate(Form mdiPrnt)
    {
        InitializeComponent();
        // master grid + detail grid registered together as an array
        // 마스터 그리드와 상세 그리드를 배열로 함께 등록
        this.Grids = new TSpreadGrid[] { grd_..._1, grd_..._2 };
        this.MultiColSettingGridIndexes = new int[] { 0, 1 };
        this.Controller = new MultiGridController(this);
        // ...
    }
    // selecting a master row raises GridDetailSearch, which the Controller subscribes to
}

public class MultiGridController : BaseMultiGridController
{
    public void DoRetrieveData()        { /* queries master grid via VVD/vessel-call keys */ }
    public void DoRetrieveDetailData()  { /* queries detail grid by container no/seq */ }
    // both bind into FormView.Grids[0] / FormView.Grids[1] respectively

    // print-family overrides operate on the WHOLE array, not a single grid:
    public override void DoPrint(object sender, EventArgs e)
    {
        // GridPrintManager over (this.View as BaseMultiGridViewInterface).Grids
    }
}
```

`BaseMultiGridController.GetGridDataViewBuilder()` is the one structurally distinct piece versus SingleGrid: it re-points its cached `SpreadGridDataViewBuilder.TsbGrid` to `ActiveGrid` on every call, and honors `AllowColumnHeaderTemplateToAllGrids`/`AllowColumnHeaderTemplateToAllGridsAutomatically` flags that decide whether a saved column-header template applies to just the focused grid or to every grid in the array.

### 4.7 Archetype: Detail — a single-record edit form

**What it's for:** a plain data-entry/edit form bound to one record — no grid, no grid toolbar family. In the Sample, `DetailView` edits vessel/container fields (`VslCd`, `CallYear`, `CallSeq`, `CntrNo`, `CntrSeq`, `BlNo`, `IxCd`, `Delv`, `Wgt`, `GoodOwner`, `EStackDay`, `MStackDay`, `Fe`).

```csharp
public interface DetailInterface : BaseDetailViewInterface
{
    DetailParam SearchParam { get; set; }
    DetailItem DetailItem { get; set; }
    VesselScheduleItem VvdItem { get; set; }
    string SearchMandatoryCheck();
    string SaveMandatoryCheck();
    void SetForeColor(/* ... */);
    void SetBackColor(/* ... */);
}

public partial class DetailView : BaseDetailView, DetailInterface
{
    private DetailItem _detailItem;

    // The defining Detail-archetype idiom: one bound BindingSource drives every field
    // Detail 화면의 핵심 관례: 하나의 BindingSource 가 모든 입력 필드를 바인딩
    public DetailItem DetailItem
    {
        set
        {
            this._detailItem = value;
            this.bdsDetailItem.DataSource = this._detailItem;
            if (this._detailItem.OpCode == OpCodes.CREATE) { this.ClearView(); }
        }
    }
    // SetDetailItemBindingControls() wires each TextBox/Combo via .CreateBinding<DetailItem>(bdsDetailItem)
}

public class DetailController : BaseDetailController
{
    public DetailController(DetailInterface view) : base(view)
    {
        // registers a cross-screen sync callback so this Detail screen reacts to updates
        // broadcast by a related MultiGrid screen elsewhere in the app
        // MultiGrid 화면에서 브로드캐스트되는 갱신을 수신하기 위한 동기화 콜백 등록
        // this.DataSyncAgent.AddCallbackInfo("SYNC_GRD_FTE_MULTIGRID", this.MultiGrid_DataSyncNotified);
    }

    public void DoRetrieveData() { /* IDetailService.InquiryDetail(param) */ }
    public void DoSaveData()     { /* IDetailService.ProcessDetail(item) */ }
    public void DoDeleteData()   { /* IDetailService.ProcessDetail(item) with OpCode = DELETE */ }
}
```

`BaseDetailView`/`BaseDetailController` are themselves extremely thin (little beyond an `ObjectID` assignment) — almost all real Detail-form behavior (Save/Cancel lifecycle, mandatory-check-before-save orchestration, action history) lives one level up in `BaseBizView`/`BaseBizController`. **Note:** those two classes were not in the file set read for this guide; this section should be revisited once their source is available, rather than asserting specific Save/Cancel semantics not directly observed.

### 4.8 Archetypes: SingleDraw / MultiDraw — yard/vessel drawing canvases

**What they're for:** 2-D visual drawing/diagram screens rather than grids — squarely a container-terminal (TOS) concept, used to render yard blocks/bays/rows or vessel-bay layouts (CY block/bay/row visualization). These are not grid-based at all; the base view exposes a drawing surface (`IDrawControl`) instead of a `Grid`/`Grids` property.

**SingleDraw** — one drawing surface:
```csharp
public interface SingleDrawInterface : BaseSingleDrawViewInterface
{
    YardDefineParam SearchParam { get; set; }
    void ZoomIn();
    void ZoomOut();
    void AddBlock(string blockName);
    void AddContainer(List<ContainerBlockItem> items);
    void RemoveBlock(string blockName);
    void Clear();
    event EventHandler BlockNameChanged;
}

public partial class SingleDrawView : BaseSingleDrawView, SingleDrawInterface
{
    private void InitViewCreate()
    {
        this.DrawControl = this.yTopView1;   // a single yard "top view" drawing surface
        // block-name combo populated from YardDefineFactory
    }
}

public class SingleDrawController : BaseSingleDrawController
{
    public void DoRetrieveData()
    {
        // Note: in the Sample, no service/DAO call is made here — GetContainer(blockName)
        // fabricates hard-coded ContainerBlockItems locally for demo purposes.
        // A real module replaces this with a genuine BizServiceLocator-based inquiry.
        this.FormView.AddBlock(blockName);
        this.FormView.AddContainer(this.GetContainer(blockName));
    }
}
```

**Note:** because the Sample's `SingleDrawController.DoRetrieveData` uses mock/hard-coded data rather than a real service call, this archetype's vertical slice (View→Controller→Service→Dao→SQL) could **not** be verified end-to-end from source, unlike SingleGrid. Document the View/Controller wiring pattern with confidence; treat the service-layer plumbing for Draw screens as unverified until a real (non-Sample) Draw screen's source is read.

**MultiDraw** — the same shape, but with **two linked drawing surfaces and bidirectional selection sync**, analogous to how MultiGrid links two grids:
```csharp
public interface MultiDrawInterface : BaseMultiDrawViewInterface { /* ... */ }

public partial class MultiDrawView : BaseMultiDrawView, MultiDrawInterface
{
    private void InitViewCreate()
    {
        this.DrawControls = new IDrawControl[] { yTopView1, yBayView1 };  // top-view AND bay-view together
    }

    // clicking a slot in the top view drives the bay view, and vice versa:
    private void yTopView1_ViewClick(object sender, EventArgs e)   { /* yBayView1.AddBayRow(...) */ }
    private void yBayView1_BayAdded(object sender, EventArgs e)    { /* yTopView1.AddSelectedBayInfo(...) */ }
    private void yBayView1_BayRemoved(object sender, EventArgs e)  { /* yTopView1.RemoveSelectedBayInfo(...) */ }
}

public class MultiDrawController : BaseMultiDrawController { /* structurally identical to SingleDrawController */ }
```

### 4.9 How the Catos-Cm layer extends the raw Fontos base classes

In practice, a real business-module developer at this company should inherit from `Tsb.Catos.Cm.Win.FormTemplate.CTBase*` classes, **not** the raw `Tsb.Fontos.Win.FormTemplate.Base*` classes — the Sample project uses the raw base only because it is deliberately domain-neutral. The declarations confirming the extra layer:

```csharp
namespace Tsb.Catos.Cm.Win.FormTemplate
{
    public abstract class CTBaseSingleGridController : BaseSingleGridController
}

namespace Tsb.Catos.Cm.Win.FormTemplate
{
    public class CTBaseSingleGridView : BaseSingleGridView, CTBaseSingleGridViewInterface
}
```

`CTBaseSingleGridViewInterface` already shows domain-specific concerns baked directly into what would otherwise be a generic grid-view contract:
```csharp
public interface CTBaseSingleGridViewInterface : BaseSingleGridViewInterface, CTBaseGridViewInterface
{
    BaseDataItem ActiveItem { get; set; }
    event EventHandler HazardousInformationOpening;
    event EventHandler HazardousInformationClosed;
    bool OpenHazardousInformation(object dataItem);
}
```

Concretely, `CTBaseSingleGridController` adds on top of `BaseSingleGridController`:

- **Cross-screen "find the container/BL/RoRo-unit number" navigation helpers** — `GetCntrNo`, `GetInvNo`, `GetUnitNo`, each attributed with `[CTBizDialogToolbarAttribute(...)]` (`CatosCntrDetail`, `CatosCustomCntrDetail`, `CatosInvoiceDetail`, `CatosCustomRoroDetail`). Each first tries a well-known property name on the active row DTO (`"CntrNo"`, `"InvNo"`, `"RoroNo"`), and falls back to reflecting over all properties looking for a name containing the target substring — a generic mechanism that lets one menu item ("Container Detail") work against any grid whose row DTO exposes a container-number-like property.
- **Vessel/rail assignment data-sync handlers** (`AssignVessel_DataSyncNotified`/`AssignRail_DataSyncNotified`) — when a vessel-call or rail-schedule assignment changes elsewhere in the app, these fire `spreadGrid.FireGridDataChangeEvent(...)` so every open container-list grid visually refreshes the affected cells.

And `CTBaseSingleGridView` (roughly 3.5x the line count of the raw `BaseSingleGridView`) adds:

- **Pre-wired grid event handling** for seven `TSpreadGrid` events (`GridDataChanged`, `GridMouseRightClick`, `GridCellEnterKeyPressed`, `GridMouseDoubleClick`, `SelectionChanged`, `GridDataSourceBinded`, `GridDoPostProcAfterRowDataBinded`, `GridDataChanging`) that the raw base leaves entirely unwired — a module inheriting from `CTBaseSingleGridView` gets these for free instead of hand-wiring each one (as the Sample's own `SingleGridView` has to).
- **Code-lookup right-click menus**, routed through a `CTGridBizRule` dispatcher.
- **Reefer/weight/class column post-processing** — after every grid data-bind, scans the bound `GridColumnSchema` for well-known DG/reefer/CFS column keys (`PSztp`, `SetTempF`, `WeightGroup`, `EdiTempF`, `PtiTempF`, `CntrClass`) and runs the matching business-rule calculation per row.
- **Hazardous-cargo (IMDG) panel integration** — `OpenHazardousInformation(object dataItem)` diffs old/new `Imdg`/`Unno` values and cascades related field recalculation via `CTDataChangeManager`.
- **Conditional-mandatory validation** — `SetConditionalMandatory(...)`/`CheckConditionalMandatory(...)`, a rule-driven "this field is mandatory only if condition X" layer with no equivalent in the raw base's `[ValidateAttribute]` hook.
- **Incorrect-value highlighting** — cells that fail validation are painted in a fixed "incorrect" color.

**Practical takeaway:** `BaseSingleGridView`/`BaseSingleGridController` give a module grid plumbing and the generic toolbar; `CTBaseSingleGridView`/`CTBaseSingleGridController` add a pre-wired container/vessel/DG-cargo business-rule integration layer that every real container-terminal SingleGrid screen would otherwise have to hand-roll. New business modules should inherit from the Cm-layer classes unless deliberately building something outside the container/vessel/TOS domain.

---

## 5. Grid Schema XML Deep Dive

### 5.1 What a Grid Schema XML file is, and where it lives

A **Grid Schema XML** file is the on-disk, per-grid definition of column layout, binding, and behavior for a `TSpreadGrid` — column order, header text, cell type/style, width, filter/sort eligibility, alignment, and code-lookup binding. It is loaded and deserialized (not hand-parsed) by `GridBaseSchema` (`Tsb.Fontos.Win.Grid.Schema.GridBaseSchema`), a **generic XML load/save helper** parameterized by the CLR type it deserializes into, quoted:

```csharp
public class GridBaseSchema : TsbBaseObject
{
    public virtual T GetGridSchema<T>(string gridID, string filePostFix, bool doNotExistException, string referenceGridXmlPath)
    {
        string filePath = StringUtil.GetEmptyStringIsNull(referenceGridXmlPath).Equals(string.Empty)
            ? PathUtil.Combine(AppPathInfo.PATH_APP_GRID, gridID + filePostFix)
            : PathUtil.Combine(referenceGridXmlPath, gridID + filePostFix);

        if (FileUtil.Exists(filePath))
            return XmlUtil.Deserialize<T>(filePath);
        // else: doNotExistException ? throw : return a default instance via ObjectCreator.CreateObject(typeof(T))
    }

    public virtual void SaveGridSchema<T>(string gridID, string filePostFix, object objectToWrite, string referenceGridXmlPath) { /* mirrors load */ }

    // Embedded-resource variant — for grid schemas shipped baked into a module DLL:
    public virtual T GetGridSchemaByEmbbedSchemaFile<T>(string gridAssembleName, string gridNameSpace, string gridID, string filePostFix, bool doNotExistException) { /* ... */ }
}
```

This resolves the file path as `AppPathInfo.PATH_APP_GRID` (or a caller-supplied override path) + `gridID` + `filePostFix`. In the Sample application this concretely means files like:

```
Grid/FontosSample/grd_FORMTEMPLATE_SingleGrid_Schema.xml
Grid/FontosSample/grd_FORMTEMPLATE_MultiGrid_1_Schema.xml
```

i.e. the naming convention documented in this guide's task framing — `Grid/<Module>/grd_<Name>_Schema.xml` — is confirmed by the actual Sample file names, where `<Module>` is the Environment folder (`FontosSample`) and `<Name>` is the grid ID passed by the calling screen code (e.g. `FORMTEMPLATE_SingleGrid`).

**Note:** `GridBaseSchema` itself does **not** declare the column/header POCO type — it is a generic `<T>` loader. The concrete class it is invoked with (something like a `GridColumnSchema`/`ColumnSchema` pair under `Tsb.Fontos.Win.Grid.Schema`, given the XML root element name observed at runtime) was not itself among the files read for this research pass; document `GridBaseSchema` as "the load/save engine," and treat the exact CLR property names backing each XML tag as inferred from the deserialized XML shape (§5.2), not independently source-verified against that POCO.

### 5.2 Anatomy of a `<ColumnSchema>` element (flat dialect)

The Sample's SingleGrid/MultiGrid schemas (and the MultiHeader tool's newer schema file) share one XML dialect, root element `GridColumnSchema`, namespace `http://www.tsb.co.kr/xsd/GridColumnXmlSchema.xsd`:

```xml
<?xml version="1.0"?>
<GridColumnSchema xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                   xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                   xmlns="http://www.tsb.co.kr/xsd/GridColumnXmlSchema.xsd">
  <GridSchema>
    <FrozenColumnCount>2</FrozenColumnCount>
  </GridSchema>
  <ColumnSchemas>
    <ColumnSchema>
      <Seq>2</Seq>
      <ViewColumnSeq>2</ViewColumnSeq>
      <Key>Fe</Key>
      <LabelResKey>WRD_FTSL_F_E</LabelResKey>
      <DataField>Fe</DataField>
      <CodeType>FLMT</CodeType>
      <ColumnMaxLength>0</ColumnMaxLength>
      <DBField>Fe</DBField>
      <CellType>Text</CellType>
      <CellStyleID>Basic Text</CellStyleID>
      <CellTypeID>Basic Text</CellTypeID>
      <Width>6</Width>
      <Visible>Y</Visible>
      <ForeColor>0</ForeColor>
      <BackColor>12648384</BackColor>
      <Filter>Y</Filter>
      <Sort>F</Sort>
      <SortSeq>0</SortSeq>
      <HorizontalAlignment>L</HorizontalAlignment>
      <RefDBField />
      <RefCodeType />
      <DummyFieldEditable />
      <VerticalAlignment>C</VerticalAlignment>
      <Lock>Y</Lock>
      <Mandatory>N</Mandatory>
      <PrimaryKey>N</PrimaryKey>
      <Preview>N</Preview>
      <Copy>N</Copy>
      <NotAllowHidden>N</NotAllowHidden>
      <CellEditor>N</CellEditor>
      <CellTypeCode>N</CellTypeCode>
      <ColorRuleType>N</ColorRuleType>
      <CellInput>L</CellInput>
      <CellDisplay>C</CellDisplay>
      <ValueDisplay>M</ValueDisplay>
      <Included>Y</Included>
    </ColumnSchema>
    <!-- one <ColumnSchema> per column, in <Seq> order -->
  </ColumnSchemas>
</GridColumnSchema>
```

Key attributes, grouped by concern:

| Group | Element(s) | Meaning |
|---|---|---|
| Identity / order | `Seq`, `ViewColumnSeq`, `Key` | `Seq` is the column's defined order; `ViewColumnSeq` allows the on-screen display order to differ (user-reordered columns) from the definition order; `Key` is the stable column identifier used by filter/sort/column-setting persistence |
| Label / localization | `LabelResKey` | A **resource-manager key**, not a literal caption string — resolved through the framework's resource system (`RESOURCE=Resx` per the app's `ArchitectureInfo.xml`), essential for the multi-language requirement across this company's EU/SEA/LatAm customer base |
| Data binding | `DataField`, `DBField`, `RefDBField` | `DataField` binds the grid column to a property on the row DTO (e.g. `SingleGridItem.Fe`); `DBField` is the underlying database column name; `RefDBField` supports a secondary/reference field for lookup columns |
| Code-lookup integration | `CodeType`, `RefCodeType` | When set, the column's editable/displayable values are sourced from the framework's `CodeManager` (a shared code-list/lookup service) rather than free text |
| Display / style | `CellType`, `CellStyleID`, `CellTypeID`, `Width`, `ForeColor`, `BackColor`, `HorizontalAlignment`, `VerticalAlignment` | Visual rendering — `CellType`/`CellStyleID`/`CellTypeID` select a FarPoint Spread cell-type template (e.g. `"Basic Text"`); colors are packed integers (`BackColor` `12648384` etc.), not hex strings |
| Behavior flags | `Visible`, `Filter`, `Sort`/`SortSeq`, `Lock`, `Mandatory`, `PrimaryKey`, `Preview`, `Copy`, `NotAllowHidden`, `CellEditor`, `CellInput`/`CellDisplay`/`ValueDisplay`, `Included` | `Y`/`N` (or single-letter, e.g. `HorizontalAlignment` `L`/`R`/`C`) string flags controlling filterability, sortability, editability, column-chooser visibility, print-preview inclusion, copy/paste eligibility, column validation inclusion, etc. |
| Column validation | `Included` | `Y`/`N` flag determining whether this column is included in Column Setting Validation. When set to `Y`, the column's configuration is validated as part of the Column Setting validation process. When set to `N`, the column is excluded from validation checks. See [Issue #1004](https://issue.tsb.co.kr/view.php?id=1004#c4850) for details on handling Non-CUSPS cases where Partner Code references need special validation logic. |
| Grid-level (not per-column) | `<GridSchema><FrozenColumnCount>` | The only grid-wide (as opposed to per-column) setting observed — how many leading columns stay frozen/pinned during horizontal scroll |

**Boolean convention:** exactly as with `TMenuItem` in §7 of the framework internals, grid schema booleans are represented as the literal strings `"Y"`/`"N"` in XML, not `true`/`false` — a company-wide XML idiom.

### 5.3 Schema evolution: an older/reduced variant

Comparing the Sample's schema against the MultiHeader tool's `grd_OPR_DgCntrList_1_Schema.xml`: both use the identical `GridColumnSchema` root/namespace and share the core `<ColumnSchema>` tag set (`Seq`, `Key`, `LabelResKey`, `DataField`, `CellType`/`CellStyleID`/`CellTypeID`, `Width`, colors, `Sort`, alignment, `Lock`/`Mandatory`/`PrimaryKey`/`Preview`/`Copy`/`NotAllowHidden`, `CellEditor`/`CellTypeCode`/`ColorRuleType`) — but the older file is **missing** the `<GridSchema><FrozenColumnCount>` wrapper entirely, and per-column lacks `ViewColumnSeq`, `CodeType`, `RefDBField`, `RefCodeType`, `DummyFieldEditable`, `SortSeq`, `CellInput`, `CellDisplay`, `ValueDisplay`.

Because the file is deserialized via a generic `XmlSerializer`-style mechanism (`XmlUtil.Deserialize<T>`), both the older and newer files load fine against the same schema class — missing elements simply default. This is a backward-compatible schema evolution, not a dialect fork: newer fields (view-column reordering, code-lookup binding, and the three-way `CellInput`/`CellDisplay`/`ValueDisplay` cell-mode flags) were added over time without breaking older grid schema files.

### 5.4 A genuinely different, legacy dialect: DataSet-serialized XML

One further file examined, `grd_SGD_BookingList.xml` from the MultiHeader tool, is **not** the `GridColumnSchema` dialect at all — it is raw ADO.NET `DataSet.WriteXml()` output, root element `<NewDataSet>` with an inline `<xs:schema>` describing a `ColumnHeader` row type:

```xml
<?xml version="1.0" standalone="yes"?>
<NewDataSet>
  <xs:schema id="NewDataSet" xmlns="" xmlns:xs="http://www.w3.org/2001/XMLSchema"
             xmlns:msdata="urn:schemas-microsoft-com:xml-msdata">
    <xs:element name="NewDataSet" msdata:IsDataSet="true" msdata:MainDataTable="ColumnHeader" msdata:UseCurrentLocale="true">
      <xs:complexType>
        <xs:choice minOccurs="0" maxOccurs="unbounded">
          <xs:element name="ColumnHeader">
            <xs:complexType>
              <xs:sequence>
                <xs:element name="Seq" type="xs:int" minOccurs="0" />
                <xs:element name="Key" type="xs:string" />
                <xs:element name="Caption" type="xs:string" minOccurs="0" />
                <!-- ... -->
              </xs:sequence>
            </xs:complexType>
          </xs:element>
        </xs:choice>
      </xs:complexType>
    </xs:element>
  </xs:schema>
  <ColumnHeader>
    <Seq>0</Seq>
    <Key>Plan</Key>
    <Caption>Ship Plan</Caption>
    <DBField>Plan</DBField>
    <CellType>Text</CellType>
    <CellTypeID>RowLock</CellTypeID>
    <Width>3</Width>
    <Visible>Y</Visible>
  </ColumnHeader>
  <!-- one <ColumnHeader> per column -->
</NewDataSet>
```

Differences from `GridColumnSchema`: the row element is named `ColumnHeader` (not `ColumnSchema`), the label field is a literal `Caption` string rather than a `LabelResKey` resource lookup (i.e. **no localization indirection** — a red flag for this company's multi-region customer base if such a file were ever used as-is for a new screen), and it lacks `CellStyleID`, `SortSeq`, `RefDBField`/`RefCodeType`, `CodeType`, `Filter`, `DummyFieldEditable`, and the `CellInput`/`CellDisplay`/`ValueDisplay` triad, as well as the `<GridSchema>` wrapper entirely. This is almost certainly output from an older grid-header-designer tool that predates the current `GridColumnSchema` dialect.

### 5.5 On "multi-header" (grouped/nested column headers)

**Note — explicitly flagged gap:** despite the "MultiHeader" tool's name, **none** of the grid schema files examined for this guide (Sample SingleGrid, Sample MultiGrid, and both MultiHeader-tool files above) contain any `ColumnGroup`/`GroupHeader`/`ParentColumn`-style element. A targeted search across all four files' full element inventories found no grouped/nested-header construct in this XML dialect.

This means grouped or merged column headers are **not expressed in the `GridColumnSchema` (or legacy `NewDataSet`) XML as staged for this research pass.** If the FarPoint Spread grid control supports visually merged/spanned header cells (which it does, generally, via APIs such as `ColumnHeader.Cells[...].AddSpanCell(...)`), the evidence available here suggests that capability — if used by this framework at all — is set up in code by the screen/grid-initialization logic rather than declared in the schema XML, or is controlled by a schema file that was not part of this research pass. Do not assert a specific multi-header XML mechanism in the published guide; instead, flag it as an open question for whoever documents the MultiHeader tool directly, and confirm against that tool's own source (not the grid-schema files it produces) before writing further documentation.
## 6. The TSpreadGrid Control

### 6.1 What TSpreadGrid Is

`TSpreadGrid` (namespace `Tsb.Fontos.Win.Grid.Spread`) is the FONTOS framework's grid control. Architecturally, `TSpreadGrid` is a direct subclass of the third-party **FarPoint Spread for Windows Forms** grid (`FarPoint`/`FpSpread`) — every FONTOS grid you place on a screen is a FarPoint Spread control underneath, extended with roughly nine categories of FONTOS-specific functionality.

FONTOS does not extend FarPoint by inheritance alone. It uses a **Bridge pattern**: `TSpreadGrid` (the abstraction) delegates its feature implementations to a helper class, **`SpreadGridHelper`** (the implementation side), which in turn dispatches to a family of specialized handler classes:

- `DefaultCursorSettingHandler`
- `SpreadActiveRowHandler`
- `SpreadGridCellPaintHandler`
- `SpreadColSettingHandler`
- `SpreadRowCountHandler`
- `SpreadSortHandler`
- `SpreadMaskHandler`
- `GridPrintHandler`
- `SpreadCodeColumnHandler`

**The single most important rule for module developers:** if a grid does **not** have a `Tsb.Fontos.Win.Grid.Schema.ColumnSchema` applied, `TSpreadGrid` can only do one thing — Print. Every other feature (sorting, filtering, column setting persistence, validation, code lookups, functional security, summary rows, Excel import, etc.) is gated on a Grid Schema being loaded.

### 6.2 Grid Schema XML and the Spread Editor Tool

The **Grid Schema** is an XML file, conventionally named `<GridControlID>_Schema.xml`, that defines a grid's columns and their behavior. There are two categories of schema file:

| Schema type | Purpose | Typical location |
|---|---|---|
| **Main Schema File** | Column configuration for one specific `TSpreadGrid` instance on a screen | `[Tsb.{TeamName}.{ModuleID}.Config]` project → `Grid/{TeamName}{ModuleID}/{GridControlID}_Schema.xml` |
| **Cell Type Schema** | Shared, reusable behavior definitions for a cell type (Text, ComboBox, CheckBox, DateTime, Button, etc.) referenced by `CellTypeID` | `[Tsb.Catos.Cm.Win]` project → `Grid/Common/CellStyles-FONTOS/` |

The schema is parsed at runtime by `Tsb.Fontos.Win.Grid.GridColumnSchema.GetGridColumnSchema()` into an in-memory `ColumnSchema` object (per column) and a whole-schema `ColumnSchemaItemList`. A parallel, optional `_ColSetting.xml` file (or a DB-backed store) can overlay user- or template-driven column ordering/visibility/width customizations on top of the base schema, tracked via the `TSpreadGrid.AppliedColSettingName` property.

Schema files are authored with the **Spread Editor** — a dedicated FONTOS design-time tool (`Tsb.Fontos.Tools.Se.*` namespace; "Se" = Spread/Schema Editor), built with an MVP (Model-View-Presenter) + Strategy pattern (`SpreadEditorFONTOSView`, `SpreadEditorFONTOSController`, `SpreadEditorStrategyContext`, `ISpreadEditorStrategy`/`SpreadEditorFONTOSStrategy`). It reads a base configuration path from `CatosConfig.xml`, lists the `_Schema.xml` files it finds, lets you open one and edit its columns in a grid UI, and saves changes back via `DoSave()`. This same tool ships a dedicated **MultiHeader editing mode** (`SpreadEditorFONTOSMultiHeadView`/`Controller`/`Strategy`, selected through `SpreadEditorMultiHeadStrategyContext`) with its own grouped-column-header tree UI (`GroupingTreeView` and the `GroupMapping*`/`GroupingTree*` classes) — see §5.5 of this guide for the important caveat that this mode's on-disk XML representation was not verified against the flat `GridColumnSchema` dialect examined for §5.

Two TSB-defined properties must be set correctly on the control itself before a Main Schema file will be honored:

- `SchemaSettingMode` = `XML_MODE`
- `SchemaVersion` = `FONTOS_VERSION_1_0`

### 6.3 Key Features

| Feature | What it does |
|---|---|
| **Cell types / renderers** | Each column's `CellType` (Text, Mask, Number, UnitNumber, Currency, DateTime, CheckBox, ComboBox, Button, etc.) selects a renderer registered under a matching `CellTypeID` in a Cell Type Schema file. Custom renderers implement FarPoint's `IRenderer`/`Grid.BaseRenderer` extension mechanism — e.g. `Tsb.Catos.Cm.Win.Grid.Renderer.CodeRenderer` converts a stored business-code value into its display text at paint time. |
| **Grid Biz Rule** | A validation/lookup/rendering mechanism actually implemented in the **CATOS framework** (`Tsb.Catos.*`) and exposed to `TSpreadGrid` through the `GridBizRule` property (backed by `Tsb.Catos.Cm.Win.Grid.CTGridBizRule`). It drives Data Validation, Row Lock, Row Bold, Row Blink, and Code-lookup ("Code Popup") behavior. Row Lock/Row Bold are convention-driven: a column's `CellTypeID` must literally be `RowLock` or `RowBold`, and the cell's runtime value must equal `'Y'` to trigger the effect. |
| **Lazy Load** | Defers Column Schema loading past the control's normal Load event. Set `IsLazyLoad = true` to suppress automatic loading, then call `TSpreadGrid.LazyLoad()` (optionally with `SpecifiedSchemaFileName` to pick a schema file by name) when you're ready to load it — useful for grids on tabs that aren't immediately visible, or grids whose schema is chosen dynamically. |
| **Functional Security** | Row/column-level permission enforcement. `ColumnSchema.Visible` and `ColumnSchema.Enable` are set according to the logged-in user's authority: `BaseBizView`/`BaseBizController` (when `UseAuthorityControl = true`) call through `BaseMenuView.InquiryAuthorityControls()` into `Tsb.Catos.Cm.Core.Security.Service.CTSecurityService`, which returns per-column Visible/Enable info keyed to the grid's `ColumnSchema`. |
| **Excel import/export** | `ExcelImportDialog` drives an import pipeline (Factory + Builder + Observer patterns: `ExcelImportHandlerFactory`, `ExcelSheetInfoDirector`/`ExcelSheetInfoBuilder`, `ExcelImportObserverExecuter`) that opens an Excel file via FarPoint's own `OpenExcel()`, derives row/column layout, and inserts validated rows into the grid, surfacing results/errors via `ExcelImportResultDialog`. |
| **Detail View (master-detail)** | A master grid's selected row is pushed into a detail panel using an Observer pattern: `Tsb.Fontos.Core.Observer.DataSyncAgent`/`DataSyncManager` register interested objects and notify them (`NotifyToSync()`) when the master selection changes. |
| **Sorting / filtering** | Default Sort and Template (saved) Sort are handled by `SpreadSortHandler`. Filtering comes in two flavors: a **Header Filter** (per-column dropdown filter, `TSpreadGrid.SetHeaderFilter()`) and a **Dialog Filter** (a full filter-builder dialog, `LegacySpreadFilterView`/`SimpleSpreadFilterView`) implemented via a Strategy pattern (DB- vs. File-backed storage) plus a Chain-of-Responsibility filter pipeline (`BaseFilterHandler` → `EqualFilterHandler`, `LikeFilterHandler`, `InFilterHandler`, `GreaterThanFilterHandler`, etc.). |
| **Column setting persistence** | User (or template) column order/width/visibility overrides, saved/loaded via DB or file storage (`LegacySpreadColSettingDBStrategy` / `LegacySpreadColSettingFileStrategy`), applied through `TSpreadGrid.ApplyColumnSetting()`, which clears the Header Filter for performance before reloading the schema and rebinding. |

Two smaller but practically important behaviors: **Frozen columns** are applied via `SpreadGridHelper.SetFrozenColumnWithCustomColSetting()`, and grid-wide styling (fonts, default colors) is pulled from `UIStyleInfo.xml` through `GeneralGridStyle`.

### 6.4 Authoring and Binding a New Grid — Practical Workflow

Based on the documented call sequence and the Spread Editor's stated purpose, the expected developer workflow is:

1. Drop a `TSpreadGrid` control onto the form and set `SchemaSettingMode = XML_MODE`, `SchemaVersion = FONTOS_VERSION_1_0`.
2. Create the Main Schema file (`{GridControlID}_Schema.xml`) under the module's `Grid/{TeamName}{ModuleID}` config folder — normally generated/edited through the **Spread Editor** tool rather than hand-written.
3. For each column, set `CellType` and a matching `CellTypeID` that resolves against an existing Cell Type Schema entry (create new Cell Type Schema entries only when an existing type doesn't fit).
4. Configure per-column concerns as needed: cell color rules (`ColorRuleType` = `CODE_COLOR`, `DEFAULT_COLOR`, `VALUE_COLOR`, `UISTYLE_COLOR`, or `VERIFED_CNTR` [sic in source]), code lookup (`CodeType`/`CellInputType`/`CellDisplayType`/`ValueDisplayType`), sort (`Sort`/`SortSeq`), `PrimaryKey`, `Filter`, and Dummy Field mappings for non-DB-backed columns.
5. Set the grid's `GridBizRuleClassAssembly`/`GridBizRuleClassName` if the screen needs the CATOS Biz Rule engine (validation, row lock/bold, code popups), and `GridDataItemClassAssemblyName`/`GridDataItemClassName` to bind to the row-item type.
6. Bind actual data by setting `TSpreadGrid.DataSource` — this single assignment triggers the full internal cascade (cell type/style application → sorting → post-bind processing → `GridDataSourceBindedEvent` → active-row color).
7. Layer on optional features only as needed: Header/Dialog Filter, Column Setting, Mask Validation, Print preview, Row Count display, Functional Security, Excel Import, Detail View sync.

**Note:** the source architecture document (`TSpreadGrid Structure`, v1.5) is written for framework engineers extending `TSpreadGrid` itself, not as an end-user "how to drop a grid on a form" tutorial — it contains no literal C# or schema-XML code samples. The workflow above is reconstructed from its documented call sequences; module developers should pair it with a working example screen (e.g., a sample grid view in the reference/sample module — see the worked `SingleGrid` example in §4.4) and the Spread Editor Guide for concrete, copyable XML.

Several method names appear with inconsistent spelling across the source documentation (e.g., `GetCurosrWithDefaultCursor()` vs. `GetCursorWithDefaultCursor()`, `GetCustomColSettinSchemaName()`, `ExcuteApplyRegisterViewFilterCondition()` vs. `ExecuteApplyRegisterViewFilterCondition()`, `SetActiveInex()` vs. `SetActiveIndex()`). **Note:** these are reproduced verbatim from the source; verify actual spelling via IntelliSense/reflection against the shipped `Tsb.Fontos.Win.dll` before treating any single one as authoritative in code you write against it.

### 6.5 Grid Cell Context Menu (right-click menu)

**How a grid context menu is actually shown.** `TSpreadGrid` itself never pops up a menu. It only *raises an event*; the screen template (or the screen) decides what to show:

| Step | Where | What happens |
|---|---|---|
| ① | `TSpreadGrid_MouseDown` (`Tsb.Fontos.Win.Grid.Spread.TSpreadGrid`) | On a right-click over a data cell, the cell under the cursor becomes the active cell and the **`GridMouseRightClick`** event is raised with a `GridMouseRightClickEventArgs` (`CurrRowIndex`, `CurrColIndex`, `ColumnKey`, `ColumnSchema`, `CurrCellValue`, `CurrCellData`, `CurrDataItem`, `CurrMouseLocation`, `CurrCellLocation`, `DialogSearchType`). |
| ② | `CTBaseSingleGridView` / `CTBaseMultiGridView` (`Tsb.Catos.Cm.Win.FormTemplate`) | When the screen sets its `Grid`/`Grids`, the template subscribes `GridMouseRightClick` for **those registered grids only** (`SpreadGrid_GridMouseRightClick`). |
| ③ | `SpreadGrid_GridMouseRightClick` | If the cell is an editable Biz-Rule code column, `CTGridBizRule.HandleGridRightClick(...)` opens the code popup instead. Otherwise it sets `ContextMenuHelper.GetInstance().UserDefineInvokers = GetUserDefineContextMenuInvokers()` and calls `ContextMenuHelper.GetInstance().Show(Cursor.Position, true, ctxParam)`, where `ctxParam` is the clicked row item (`IContextMenuParam`). |

Preconditions for ① to fire:

- The grid must have been initialized with a Grid Schema (`InitializeGrid()`); the mouse/key handlers are only attached there (see §6.1 — no schema, no features).
- `DataSource` must be an `IList` with at least one row. As of this revision, the `MouseDown`, `KeyPress`, `CellDoubleClick`, `LeaveCell`, and `Change` handlers also guard against a non-`IList` data source (e.g. a `DataTable`) instead of failing with a `NullReferenceException`.

**Where the menu items come from.**

| Item group | Source | Loaded by |
|---|---|---|
| Common items (Copy, Filter, Column Setting, Summary, …) | The menu XML's context-menu definition | `MainMenuHandler` → `ContextMenuHelper.SetCommonMenuItem(...)` |
| Custom (module-wide) items | `App.config` key `File.ContextMenuItem` (e.g. `ContextMenuItem.xml` under the module's Environment folder) | `MainMenuHandler.GetCustomContextMenuLoad()` → `ContextMenuHelper.SetCustomMenuItem(...)` |
| Screen-specific items | `GetUserDefineContextMenuInvokers()` override on the View | `CTBase*GridView.SpreadGrid_GridMouseRightClick` |

`ContextMenuHelper` is a singleton. `Show(Point point, bool useCommonContextMenu, IContextMenuParam param)` builds and shows the menu; pass `useCommonContextMenu = false` to show only the user-defined items.

**Adding a screen-specific item (recommended pattern for `CTBase*GridView` screens).** Override `GetUserDefineContextMenuInvokers()` and cache the list (pattern taken from `Tsb.Catos.Om.Client.Operation.CustomsContainerListView`):

```csharp
private List<ContextMenuInvoker> _userDefinedContextMenuInvokers = null;

/// <summary>
/// Screen-specific context menu items. / 화면 전용 컨텍스트 메뉴 항목
/// </summary>
protected override List<ContextMenuInvoker> GetUserDefineContextMenuInvokers()
{
    try
    {
        if (this._userDefinedContextMenuInvokers == null)
        {
            this._userDefinedContextMenuInvokers = new List<ContextMenuInvoker>();

            ContextMenuInvoker openDetail = new ContextMenuInvoker()
            {
                Text = ResourceFactory.GetResource().GetLabel("WRD_XXXX_Menu_OpenDetail"),   // resource key / 리소스 키
                Type = ContextMenuType.UserDefine,
                ActionId = MenuActionConstant.MENU_ACTION_ACTION_ID_OPENFRM,
                ImageIcon = ResourceFactory.GetResource().GetImage("menu_xxx")
            };
            openDetail.ContextMenuClick += new ContextMenuInvoker.ClickEventHandler(this.OpenDetail);

            this._userDefinedContextMenuInvokers.Add(openDetail);
        }
    }
    catch (TsbBaseException tsbEx)
    {
        MessageManager.Show(tsbEx);
    }

    return this._userDefinedContextMenuInvokers;
}

private void OpenDetail(object sender, EventArgs e)
{
    // Use this.ActiveItem (row item of the right-clicked cell) / 오른쪽 클릭한 행의 Item 사용
}
```

**Screens that do not inherit `CTBase*GridView`, or grids not registered in `Grids`.** Subscribe the event yourself and call the helper (pattern used by e.g. `PickupTimeTableView`, `TerminalHoldAuthorizationView`):

```csharp
// Show context menu for a grid that is not registered to the template
// 템플릿에 등록되지 않은 그리드의 컨텍스트 메뉴 표시
this.grdSub.GridMouseRightClick += (sender, e) =>
{
    ContextMenuHelper.GetInstance().UserDefineInvokers = this.GetUserDefineContextMenuInvokers();
    ContextMenuHelper.GetInstance().Show(Cursor.Position, true, e.CurrDataItem as IContextMenuParam);
};
```

**Grids that never raise `GridMouseRightClick`** — a grid bound to a `DataTable` or not initialized with a schema (the typical case is a **Summary grid**, §6.6). Attach a standard WinForms `ContextMenuStrip` to the control and handle the right-click active cell yourself; the common grid items (Copy/Filter/…) operate on schema-bound source grids and do not apply there.

**Note:** the row item passed as `ctxParam` must implement `IContextMenuParam` (see `SingleGridItem` in §4.4) for menu actions that read the active row.

### 6.6 Summary Grid: Context Menu and Source Item Lookup (`GetSourceItemsBySummaryCell`)

**Summary grid recap.** A summary (pivot) of a source grid is rendered into another Spread control with `SpreadGridHelper.ApplySummary(...)`:

```csharp
// Render template "Summary_0" of the source grid into grdSummary
// 원본 그리드의 Summary 템플릿 "Summary_0"을 grdSummary에 표시
this.grdSource.GridHelper.ApplySummary(this.grdSource.Name, "Summary_0", this.grdSummary);
```

`ApplySummary` overloads: `ApplySummary()`, `ApplySummary(string summaryName, FpSpread grdPreviewSummary)`, `ApplySummary(string fileName, string summaryName, FpSpread grdPreviewSummary)`, `ApplySummary(string summaryName, FpSpread grdPreviewSummary, GridSummary gridSummary)`. Set `TSpreadGrid.AutoApplySummary = true` (or the `UseGridSummaryAutoApply` policy, §6.7) to refresh the summary automatically whenever the source grid is re-bound.

**Why a Summary grid shows no context menu.** The summary grid is bound to a `DataTable` (not an `IList`), is normally not initialized with a Grid Schema, and is not registered in the screen template's `Grids`. Therefore none of steps ①–③ in §6.5 apply. Attach a dedicated `ContextMenuStrip` to the summary grid (example below).

**Getting the source items of a summary cell — `SpreadGridHelper.GetSourceItemsBySummaryCell`** *(added in this revision; requires the updated `Tsb.Fontos.Win.dll`)*:

```csharp
// Tsb.Fontos.Win.Grid.Spread.SpreadGridHelper (public)
public IList GetSourceItemsBySummaryCell(FpSpread summaryGrid, int row, int column);
public IList GetSourceItemsBySummaryCell(FpSpread summaryGrid, int row, int column,
                                         out List<CodeDataItem> filterInfoList);
```

| Item | Description |
|---|---|
| Call on | The **source** grid's helper: `grdSource.GridHelper` |
| `summaryGrid` | The Spread control passed to `ApplySummary` |
| `row`, `column` | Summary cell index (data area). Sub Total / Grand Total / "Total of …" cells are supported. |
| `filterInfoList` | Conditions applied for the cell — `Code` = field (DB field), `CodeName` = value |
| Return: item list | **Original** items bound to the source grid (`grdSource.DataSource`), in source order |
| Return: empty list | No matching data, or `row`/`column` out of range |
| Return: `null` | `summaryGrid` is **not the last grid `ApplySummary` was applied to** (e.g. the Summary preview popup was opened afterwards) → apply the summary again |

How it works (`SpreadSummaryHandler.GetSourceItemsBySummaryCell`):

1. Builds the filter for the cell from its row/column headers with the same rule as the preview's cell click (`CreateExpressions`).
2. Filters a **copy** of the summary's working list (display-converted clones produced by `GetViewListFromGrid`), applying the same DateTime formatter handling as `SelectRow`.
3. Maps the matched clones back to the **original** source items by `BaseDataItem.GUID`.

It is side-effect free: unlike the internal `SelectRow()` path used by the preview popup, it does **not** change the summary's selection basis (recap), `SummaryResultList`, the source-grid highlight, or the last-clicked cell.

**Relationship to the existing preview ("Detail List") path.**

| Path | API surface | Returns | Notes |
|---|---|---|---|
| Preview popup cell click → `SelectRow()` → `GetSummaryDetailDatasBySelectedCell()` | internal | display-converted clones | Used by `GeneralSpreadSummaryPreviewView` "Detail List" (`UseGridSummaryDataDetail`). |
| `GridHelper.SetIsLockUseGridSummaryContext(false)` before `ApplySummary`, then read `TSpreadGrid.SummaryResultList` after a cell click | public | display-converted clones of the last clicked cell | Only works when `UseGridSummarySelection = true`; also turns on source-row highlight and selection-based re-summary ("recap"). |
| **`GridHelper.GetSourceItemsBySummaryCell(...)`** | public | original source items | Independent of `UseGridSummarySelection`; no state change. Recommended for "open the matching items in another view". |

**Example — Summary grid context menu that opens the matching source items** (applied to `Tsb.Fontos.Sample.Client\Grids\TSpreadGridView.cs`, region *METHOD AREA (Summary Grid Context Menu)*):

```csharp
/// <summary>
/// Attaches a context menu to the summary grid. / Summary Grid에 컨텍스트 메뉴 연결
/// </summary>
private void InitSummaryGridContextMenu()
{
    ContextMenuStrip summaryMenu = new ContextMenuStrip();
    summaryMenu.Items.Add(new ToolStripMenuItem("Detail List", null, this.SummaryDetailMenu_Click)); // TODO: resource key / 리소스 키
    summaryMenu.Opening += this.SummaryMenu_Opening;
    this.grdSummaryGrid.ContextMenuStrip = summaryMenu;

    // Move active cell to the right-clicked cell / 오른쪽 클릭한 셀로 활성 셀 이동
    this.grdSummaryGrid.MouseDown -= this.GrdSummaryGrid_MouseDown;
    this.grdSummaryGrid.MouseDown += this.GrdSummaryGrid_MouseDown;
}

private void GrdSummaryGrid_MouseDown(object sender, MouseEventArgs e)
{
    if (e.Button != MouseButtons.Right) return;

    SheetView sheet = this.grdSummaryGrid.ActiveSheet;
    if (sheet == null || sheet.RowCount == 0) return;

    FarPoint.Win.Spread.Model.CellRange range = this.grdSummaryGrid.GetCellFromPixel(
        this.grdSummaryGrid.GetActiveRowViewportIndex(), this.grdSummaryGrid.GetActiveColumnViewportIndex(), e.X, e.Y);

    // Ignore headers / out of range / 헤더·범위 밖 무시
    if (range == null || range.Row < 0 || range.Column < 0 ||
        range.Row >= sheet.RowCount || range.Column >= sheet.ColumnCount) return;

    sheet.SetActiveCell(range.Row, range.Column);
}

private void SummaryMenu_Opening(object sender, CancelEventArgs e)
{
    // Show only when a data cell is active / 데이터 셀이 선택된 경우에만 표시
    SheetView sheet = this.grdSummaryGrid.ActiveSheet;
    e.Cancel = (sheet == null || sheet.RowCount == 0 || sheet.ActiveRowIndex < 0 || sheet.ActiveColumnIndex < 0);
}

private void SummaryDetailMenu_Click(object sender, EventArgs e)
{
    SheetView sheet = this.grdSummaryGrid.ActiveSheet;
    int row = sheet.ActiveRowIndex;
    int col = sheet.ActiveColumnIndex;
    if (row < 0 || col < 0) return; // validate / 입력 검증

    List<CodeDataItem> filterInfoList;
    IList items = this.grd_GRD_SpTSpreadGridList.GridHelper.GetSourceItemsBySummaryCell(
        this.grdSummaryGrid, row, col, out filterInfoList);

    if (items == null)       { /* summary grid not current → re-apply / Summary 재적용 필요 */ return; }
    if (items.Count == 0)    { /* no matched data / 일치 데이터 없음 */ return; }

    // Pass copies to the detail view so that source items are not modified / 원본 보호를 위해 복사본 전달
    List<object> detailItems = new List<object>();
    foreach (object item in items)
    {
        BaseDataItem dataItem = item as BaseDataItem;
        detailItems.Add(dataItem != null ? dataItem.Clone() : item);
    }

    // Framework detail view (replace with a business view as needed) / Framework 상세 화면 (필요 시 업무 화면으로 대체)
    GeneralSpreadSummayDataDetailView detailView = new GeneralSpreadSummayDataDetailView(this.grd_GRD_SpTSpreadGridList);
    detailView.Text = "[Summary Detail] - " + this.Text;
    detailView.Show();
    detailView.AddDataSourceAtGrid(detailItems, filterInfoList);
}
```

**Cautions**

- The handler remembers only the **last** summary grid applied for a source grid. If the same source grid also feeds the Summary preview popup, re-apply the summary to the screen's grid (the API returns `null` in that case).
- Sub Total / "Total of …" cells are resolved with full fidelity only when `UseGridSummaryContext = true` (§6.7).
- Lookup is evaluated against all rows currently shown in the source grid (same basis as the preview "Detail List"). If the summary was produced from a selected subset ("recap", `UseGridSummarySelection`), the returned count can differ from the number displayed in the cell. Screen-embedded summary grids do not use recap by default.
- `GeneralSpreadSummayDataDetailView` (class name spelled `Summay` in source) is public and reusable; business screens usually open their own view with the returned items instead.
- Status: framework change made in this revision (`SpreadGridHelper.cs`, `SpreadSummaryHandler.cs`, `TSpreadGrid.cs`) — **not yet built/regression-tested**; verify against the released `Tsb.Fontos.Win.dll` before use.

### 6.7 Grid Policy Settings (`GridPolicyInfo.xml`)

`Tsb.Fontos.Core.Environments.GridPolicyInfo` is a singleton (`GridPolicyInfo.GetInstance()`) holding application-wide grid behavior switches. It is loaded **once** on first access, so changes to the XML require an application restart.

**File location and load order.**

1. `App.config` key `File.GridPolicyInfo` (e.g. `<add key="File.GridPolicyInfo" value="GridPolicyInfo.xml" />`) names the file under the module Environment folder (`AppPathInfo.PATH_APP_ENVIRONMENT`, e.g. `Environment\<ModuleName>\GridPolicyInfo.xml`). If the key is set and the file is missing, loading fails with `MSG_FTCO_00121`.
2. If the key is not set, `GridPolicyInfo.xml` in the root Environment folder (`AppPathInfo.PATH_ROOT_ENVIRONMENT`) is used if it exists; otherwise built-in defaults apply.
3. When both a root-level file and a module file exist, the **root file is read first and the module file overrides it**.

**File format** (sections and settings; unspecified settings keep their defaults):

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <section name="SORT_TYPE">
    <setting name="HeaderSort">NullLast</setting>          <!-- Normal | NullLast -->
    <setting name="DefaultSort">true</setting>
  </section>
  <section name="GENERAL">
    <setting name="IsRetainActiveRowSelection">true</setting>
    <setting name="UseAutoFillOnDoubleClick">false</setting>
    <setting name="DialogSearchType">CONTAINS</setting>     <!-- STARTSWITH | CONTAINS -->
  </section>
  <section name="COPY_TYPE">
    <setting name="CellTextCopy">Description</setting>     <!-- Normal | Description -->
  </section>
  <section name="SUMMARY">
    <setting name="UseGridSummarySelection">true</setting>
    <setting name="UseGridSummaryContext">true</setting>
    <setting name="UseGridSummaryComboboxFilter">false</setting>
    <setting name="UseGridSummaryDataDetail">true</setting>
    <setting name="UseGridSummaryAutoApply">false</setting>
  </section>
  <section name="RENDERER">
    <setting name="CellRenderer">Renderer4</setting>        <!-- Renderer4 | Renderer14 -->
  </section>
  <section name="FILTER">
    <setting name="AllowEmptyOperand">false</setting>
    <setting name="ReleaseSimpleFilterOnClose">false</setting>
    <setting name="UseDefaultOperator">false</setting>
    <setting name="ResetOperand">true</setting>
  </section>
  <section name="COLUMN_SETTING">
    <setting name="DefaultAddTemplateLevel">User</setting>  <!-- All | Group | User -->
  </section>
</configuration>
```

**Property reference** (verified against `GridPolicyInfo.cs` and its usages in `Tsb.Fontos.Win`):

| Section / setting | Property (type) | Default | Effect | Used in |
|---|---|---|---|---|
| `SORT_TYPE` / `HeaderSort` | `SortType` (`GridSortTypes`: `Normal`, `NullLast`) | `NullLast` | Ordering of empty values when sorting by column header. | `SpreadSortHandler`, `SpreadGridHelper`, `SpreadGridUtil` |
| `SORT_TYPE` / `DefaultSort` | `UseGridDefaultSort` (bool) | `false` | Enables the grid Default Sort feature (`GridSortSchema` creates `GridSortInfo`; default sort UI). | `GridSortSchema`, `LegacySpreadSortView`, `TSpreadGrid` |
| `GENERAL` / `IsRetainActiveRowSelection` | `IsRetainActiveRowSelection` (bool) | `false` | Keeps the active-row selection on the same item after an automatic sort. | `SpreadSortHandler`, `TSpreadGrid` |
| `GENERAL` / `UseAutoFillOnDoubleClick` | `UseAutoFillOnDoubleClick` (bool) | `false` | Double-clicking the drag-fill handle auto-fills downward (`DoAutoFill`, Shift modifier supported). | `TSpreadGrid` |
| `GENERAL` / `DialogSearchType` | `DialogSearchType` (string: `STARTSWITH`, `CONTAINS`) | not set | Search mode of code popup dialogs; also passed in `GridMouseRightClickEventArgs.DialogSearchType`. | `CodeValueCheckboxDialog`, `TSpreadGrid` |
| `COPY_TYPE` / `CellTextCopy` | `CellTextCopyTypes` (`GridCellTextCopyTypes`: `Normal`, `Description`) | `Description` | `Description`: copies the displayed text through the FONTOS clipboard handler (`ClipboardCopyByCustom`); `Normal`: FarPoint default copy (unless `UseEnhancedClipboardCopy` is set on the grid). | `TSpreadGrid` |
| `SUMMARY` / `UseGridSummarySelection` | `UseGridSummarySelection` (bool) | `false` | Summary preview cell click/drag/keys highlight the matching source rows, fill `TSpreadGrid.SummaryResultList`, and raise `GridSummaryCellClicked`/`DoubleClicked`/`MouseUp`/`KeyUp`. A following re-summary uses the highlighted rows ("recap", intended behavior per Mantis 73064). Also shows the "reset to original data" button in the preview. Set `false` if recap is not wanted. | `SpreadSummaryHandler`, `GeneralSpreadSummaryPreviewView` |
| `SUMMARY` / `UseGridSummaryContext` | `UseGridSummaryContext` (bool) | `false` | Adds sub-menu items for the active grid under the common **Summary** and **Filter** context-menu items; enables selection-based summary range, code colors on summary headers, Sub Total / "Total of …" spans and their cell→condition mapping. | `SpreadSummaryHandler`, `CreateToolStripStrategy` |
| `SUMMARY` / `UseGridSummaryComboboxFilter` | `UseGridSummaryComboboxFilter` (bool) | `false` | Uses checked-combobox field filters in the Summary preview (with "unset filter" button) and hides non-data rows/columns. | `GeneralSpreadSummaryPreviewView`, `SpreadSummaryHandler` |
| `SUMMARY` / `UseGridSummaryDataDetail` | `UseGridSummaryDataDetail` (bool) | `false` | Adds "Detail List" / "Detail List - Always New Screen" context menu to the Summary preview grid. | `GeneralSpreadSummaryPreviewView` |
| `SUMMARY` / `UseGridSummaryAutoApply` | `UseGridSummaryAutoApply` (bool) | `false` | Refreshes the applied summary automatically when the source grid is re-bound (same as `TSpreadGrid.AutoApplySummary = true` for all grids). | `TSpreadGrid` |
| `RENDERER` / `CellRenderer` | `CellGridCellRendererType` (`GridCellRendererTypes`: `Renderer4`, `Renderer14`) | `Renderer4` | Selects the cell renderer generation. `GridPolicyInfo.IsNewRender()` returns `true` for `Renderer14`. | `TSpreadGrid`, `SpreadGridHelper`, `SpreadGridUtil`, `SheetCloneBuilder`, `SimpleSpreadFindView`, `BaseLegacySpreadFilterStrategy` |
| `FILTER` / `AllowEmptyOperand` | `AllowEmptyOperand` (bool) | `false` | Dialog filter: allows rows with an empty operand to pass the mandatory check. | `FilterOperandHandler` |
| `FILTER` / `ReleaseSimpleFilterOnClose` | `ReleaseSImpleFilterOnClose` (bool, *sic*) | `false` | Clears the applied filter when the Simple Filter view is closed. | `SimpleSpreadFilterView` |
| `FILTER` / `UseDefaultOperator` | `UseDefaultOperator` (bool) | `false` | Pre-selects a default operator (`BETWEEN` for DateTime/Number/Currency columns) when no filter is applied yet. | `BaseLegacySpreadFilterStrategy`, `LegacySpreadFilterView` |
| `FILTER` / `ResetOperand` | `ResetOperand` (bool) | `false` | Resets operand values when a saved filter list is bound to the filter grid. | `LegacySpreadFilterStoreTypeContext`, `SimpleSpreadFilterView` |
| `COLUMN_SETTING` / `DefaultAddTemplateLevel` | `ColumnSettingAddTemplateLevelType` (`GridColumnSettingAddTemplateLevelTypes`: `All`, `Group`, `User`) | `All` | Default template level selected when adding a column-setting template (only if that level is enabled for the user). | `LegacySpreadColSettingAddTemplateView` |

**Notes**

- **Default of `HeaderSort`.** The comment in the shipped sample XML says the default is `Normal`, but `GetInstance()` initializes `SortType = NullLast`. The source value (`NullLast`) is what applies when the setting is omitted.
- **Property name typo.** The XML setting is `ReleaseSimpleFilterOnClose`, but the property is `ReleaseSImpleFilterOnClose` (capital "I") — use the property name exactly when reading it in code.
- **Unused/unknown keys.** The constant `XML_ATT_VALUE_SUMMARY_DB` (`UseSummaryDB`) is declared in `GridPolicyInfo` but not read by it at the time of this review. The sample file `Tsb.Fontos.Sample.Config\Environment\FontosSample\GridPolicyInfo.xml` also contains `FILTER/PreserveOnBindListAsGridDataSource` and `FILTER/EditableDateOperand`, which the current `GridPolicyInfo` does **not** read (ignored).
- All properties have private setters; they can only be changed through the XML file.

---

## 7. Container Editor & Graphical Drawing

Every visual "cell" a module developer draws on screen in FONTOS ultimately sits on a small stack of two libraries: a low-level 2D drawing engine (`Tsb.Fontos.Win.Drawing`) that knows how to paint, scroll, zoom, and hit-test shapes on a canvas, and — for the specific case of container/box-style cells such as bay-plan slots or yard-block positions — a higher-level, rule-driven authoring tool called the **Container Editor** that sits on top of it. This section covers both, starting with the higher-level tool a business analyst or module developer is most likely to touch first, then dropping down into the drawing framework it is built on.

### 7.1 Container Editor

#### 7.1.1 Purpose

The Container Editor (`Tsb.Catos.Cm.ContainerEditor`) is a **design-time / configuration-time visual rule-and-layout editor**. It lets a developer — and, once wired into a screen, a non-programmer power user — define how a single graphical container cell (a box in a bay plan, a yard-block slot, a vessel stowage-plan cell, and similar TOS diagram elements) is rendered, driven entirely by business data and business rules, **without recompiling code**.

It exists to solve a very concrete problem the guide calls out explicitly: the predecessor product, **CATOS**, had a fixed, hard-coded container-cell rendering format. Every visual variation — a different border style for oversized cargo, a diamond IMDG hazard overlay, a color change for restowed containers, a different label layout for reefer boxes — required a code change and a release. The Container Editor generalizes this into a configuration problem: a screen's container-rendering behavior is expressed as data (an XML rule tree, a layout definition, a set of data bindings) that the runtime interprets, so that new rendering rules can be added or tuned by editing configuration rather than shipping code.

Architecturally, the Container Editor is a rule-driven authoring layer built *on top of* the Drawing Control described in §7.2 — it does not replace the drawing engine, it generates and manages the inputs (cell geometry, bindings, conditional rules) that the drawing engine ultimately renders. Configuration is scoped per **Menu ID**, so different screens (e.g. a "Yard Plan" screen and a "Vessel Bay Plan" screen) can define entirely independent sets of rules, layouts, and bindings even if they reuse the same underlying data model.

**Note:** the research notes describe the sample data model fields (`CntrNo`, `POD`, `IMDG`, `CargoType`, oversize flags, `Bay`, `BL No`, `YardPosition`, etc.) and screenshots in terms consistent with a bay-plan/yard-plan/vessel-stowage container cell; the guide itself does not enumerate every TOS screen the Container Editor has been used on, so treat "yard block view, vessel bay plan, gate/CFS views" as representative examples rather than an exhaustive list.

#### 7.1.2 Authoring Workflow

Setting up a container cell that end users can configure involves four cooperating artifacts, each with its own editor screen and its own persisted XML/CLOB payload:

1. **Data model** — a plain C# class describing the business fields available to rules and bindings for this cell (e.g. container number, POD, IMDG class, oversize flags).
2. **Rule Set** (`RuleEditorView` / `TB_CE_RULESETS`) — a tree of conditions and actions that decides, at render time, which layout elements are shown, hidden, or restyled.
3. **Cell Layout** (Cell Editor / `TB_CE_CELL_LAYOUT`) — the visual design of the cell: shapes, text, and other drawing elements placed on a fixed-size canvas.
4. **Binding Source Define** (`TB_CE_BS_DEFINE`) and, optionally, **User Define Set** (`TB_CE_USERDEFINE_SET`) — the mapping from data-model fields to the visual properties (color, text, fore/back color) of layout elements, and optional end-user-selectable display option groups.

**Step 1 — Define the container data model.** Create a class deriving from `Tsb.Catos.Cm.ContainerEditor.DataModel.BaseContainerDataModel`, one property per business field the rule and binding editors should be able to see. Attributes on each property control where and how it appears in the editors:

```csharp
public class ReportBayPlanContainerDataModel : BaseContainerDataModel
{
    // Container number: shown as "Container No" everywhere in the UI,
    // sample value used for live preview, exposed to both Rule and Binding editors.
    // 컨테이너 번호: UI 전체에 "Container No"로 표시, 미리보기용 샘플값 지정,
    // 규칙(Rule) 및 바인딩(Binding) 편집기 양쪽에 노출
    [DisplayName("Container No")]
    [CESampleValueAttribute("SUDU7530505")]
    [Browsable(true)]
    [CEBrowsableAttribute(BrowsableTypes.Rule | BrowsableTypes.Binding)]
    [CEBindingCategoryAttribute(BindingCategoryTypes.General | BindingCategoryTypes.Text)]
    public string CntrNo { get; set; }

    // Port of Discharge: also usable as a Color binding source, because POD is
    // expected to resolve to a registered "Code Color" value in the code table.
    // 양하항(POD): Color 바인딩에도 사용 가능 — POD 값이 코드 테이블에 등록된
    // "Code Color" 값으로 해석되어야 함에 유의
    [DisplayName("POD")]
    [CEBindingCategoryAttribute(BindingCategoryTypes.General | BindingCategoryTypes.Text | BindingCategoryTypes.Color)]
    [ContainerEditorCodeAttribute(CTBizConstant.CodeType.PORT_CODE)]
    public string POD { get; set; }

    // Digits-only mask enforced in the Rule Editor's condition Value box.
    // Rule Editor 조건(Value) 입력란에서 숫자만 입력되도록 마스크 적용
    [DisplayName("Over Size F")]
    [CEInputMaskAttribute(Mask.Digit)]
    public int OVF { get; set; }
}
```

The available attributes, all from `Tsb.Catos.Cm.ContainerEditor.DataModel` unless noted:

| Attribute | Effect |
|---|---|
| `DisplayNameAttribute` (`System.ComponentModel`) | Friendly label shown for the member across all Container Editor screens. |
| `CESampleValueAttribute(object)` | Sample value used to render the live Cell Preview. |
| `BrowsableAttribute` (`System.ComponentModel`) | Whether the field appears in the Cell Preview's test-data input list. |
| `CEBrowsableAttribute(BrowsableTypes)` | Whether the member appears in the Rule Editor and/or Binding Source Define screens. `BrowsableTypes` is bit-flaggable: `Rule`, `Binding`, `Hidden` (default). |
| `CEBindingCategoryAttribute(BindingCategoryTypes)` | Which binding formats the member supports: `None`, `General`, `Color`, `Number`, `Currency`, `Percent`, `Text`, `All` (bit-flaggable). Must be restricted to formats the underlying value genuinely supports — e.g. `Color` requires the value to be a registered "Code Color" — or runtime errors result. |
| `CEInputMaskAttribute(Mask)` | Restricts characters accepted in a Rule Editor condition's Value box (e.g. `Mask.Digit`). |
| `ContainerEditorCodeAttribute` | Declares the member as a "code" field, driving a Code Values popup. Two overloads: one for system/custom code categories (`codeCategory`, `ValueDisplayTypes`, `ValueReturnTypes`, `CodeInputMode`, `CodeSearchMode`), one for enum-backed codes (`Type codeOfEnumType`, ...). `CodeSearchMode.System` reads from `Tsb.Fontos.Core.Codes.CodeManager.CodeManager`; `CodeSearchMode.Custom` reads from a developer-supplied `CECodeHandler`. |
| `DescriptionAttribute` (`System.ComponentModel`) | Tooltip/help text in Cell Preview and Rule Editor condition popovers. |
| `CEDisplayNameAttribute` | Applied to individual enum members to set the friendly "Code Name" shown in the Code Values popup. |

For code lists that don't come from the system code table, implement `CECodeHandler` and register it once at startup:

```csharp
public class SampleCustomCodeHandler : CECodeHandler
{
    private IList<CodeGeneralItem> _customCodes;

    public SampleCustomCodeHandler()
    {
        _customCodes = this.CreateCustomCodes();
    }

    public override IList<Catos.Cm.Core.Codes.Item.CodeGeneralItem> GetCodes()
    {
        return _customCodes;
    }

    public override void Dispose()
    {
        _customCodes = null;
    }
}

// Registered handlers are cached for the lifetime of the application unless
// explicitly removed — call RemoveCodeHandler if the handler must be freed.
// 등록된 핸들러는 명시적으로 제거하지 않으면 애플리케이션 수명 동안 캐시됨
CECodeManager.AddCodeHandler("CustomCode", new SampleCustomCodeHandler());
// ...
CECodeManager.RemoveCodeHandler("CustomCode");
```

**Step 2 — Design the Cell Layout.** In the Cell Editor screen, set the cell's fixed `Size` (Width/Height) and design margins, then use the drawing-tool palette (rectangle, circle/ellipse, triangle, diamond, line, cross-line, polygon, freeform, "Over Slot" info element, text) to lay out the visual elements. Each element's property grid exposes Grayscale colors (for black-and-white print), Data bindings (`BackColor`, `ForeColor`, `Text` — populated from Binding Source Define once it exists), Design (`Locked`, `Name`), Layout (`Anchor`, `DisplayArea`, `Location`, `Size`), and Appearance (`BackColor`, `DashStyle`, `Font`, `ForeColor`, `HatchStyle`, `LineThick`, `Opacity`, `Text`, `TextAlign`).

**Step 3 — Define Binding Sources.** From the Cell Editor, open Binding Source Define (only enabled after the cell layout has been saved once) and, for each element property that should vary with data, pick a data-model member flagged `Binding`-browsable, choose a supported category (`General`, `Color`, `Text`), and configure category-specific options (e.g. substring/padding rules for Text, fore/back-color routing for Color).

**Step 4 — Author the Rule Set.** In the Rule Editor, create a rule set bound to one Cell Layout (this binding is one-time — it cannot be changed later, and a Cell Layout cannot be deleted while any rule set references it). Build a tree of `IfThen` / `IfThenElse` / `IfElse` / `Condition` / `Action` / `Container` nodes: Condition nodes compare a `Rule`-browsable data-model member against a value (AND/OR chained — no parenthesized precedence, no `IN` operator); Action nodes toggle the visibility of one Cell-Layout geometry element, or drive one or more User Define Set subsets (Geometry and Subset targets are mutually exclusive per action). A Cell-Preview button renders the rule set live against the model's sample values.

**Step 5 (optional) — User Define Set.** For rendering options an end user should be able to toggle interactively (e.g. "show/hide reefer plug icon"), define Groups (`Optional Type`: `None`/`Check`/`Radio`) containing Subsets, each Subset carrying its own rule tree exactly like §Step 4, plus a `Lock` flag that (when set on a Group) cascades to and locks all child Subsets.

#### 7.1.3 Integrating the Container Editor into a Module

Wiring the Container Editor into a new business module involves the standard FONTOS "reference + register + configure" pattern:

1. Add a project reference from `Tsb.{ProductName}.{ModuleID}.Client` to `Tsb.Catos.Cm.ContainerEditor`.
2. Copy `ContainerEditorContext-Common.xml` into the module's `Config` project under `Context/{ProductName}/Common/`, and reference it from the product's application context:

```xml
<object id="applicationContext" type="Tsb.Fontos.Core.Context.SpringObjectBuilder, Tsb.Fontos.Core">
  <property name="FileNames">
    <list>
      <!-- APPLICATION COMMON (MANDATORY) AREA START-->
      <value>Context/{TeamName}{ModuleID}/Common/ContainerEditorContext-Common.xml</value>
      <!-- APPLICATION COMMON (MANDATORY) AREA END-->
    </list>
  </property>
</object>
```

3. Register the four embedded SqlMap files in `SqlMap-{TeamName}{ModuleID}.config`:

```xml
<!-- ============================= CONTAINER EDITOR MAP START ============================= -->
<sqlMap embedded="Tsb.Catos.Cm.ContainerEditor.Service.Map.RuleEditorMap.xml, Tsb.Catos.Cm.ContainerEditor" />
<sqlMap embedded="Tsb.Catos.Cm.ContainerEditor.Service.Map.CellEditorMap.xml, Tsb.Catos.Cm.ContainerEditor" />
<sqlMap embedded="Tsb.Catos.Cm.ContainerEditor.Service.Map.BSDefineMap.xml, Tsb.Catos.Cm.ContainerEditor" />
<sqlMap embedded="Tsb.Catos.Cm.ContainerEditor.Service.Map.UserDefineSetMap.xml, Tsb.Catos.Cm.ContainerEditor" />
<!-- ============================= CONTAINER EDITOR MAP END ============================= -->
```

4. Add a menu entry and wire it to a controller handler:

```xml
<menuItem name="ContainerEditor" id="mniContainerEditor" resourcekey="WRD_CTCE_Container_Editor"
  actionid="ContainerEditor" actiontarget="custom" imgname="menu_RuleEditor" imgsize="S"/>
```

```csharp
/// <summary>
/// Menu's container editor button click event handler.
/// </summary>
/// <param name="sender">event source</param>
/// <param name="e">Event args containg event data</param>
[ControllerToolbarAttribute(ControllerToolbarAttribute.ControllerToolbarSet.CONTAINER_EDITOR)]
public void MenuContainerEditor(object sender, EventArgs e)
{
    System.Type dataModelType = typeof({DATA_MODEL});
    RuleEditorView view = new RuleEditorView(dataModelType, "{MENU_ID}", categoryTypes);
    view.ShowDialog();
}
```

5. Add label/message resource sections to `LocalizationInfo.xml` (prefixes `WRD_CTCE` / `MSG_CTCE`), and run the attached `ContainerEditor_Table.DDL` script to create the four `TB_CE_*` tables.

Everything the Container Editor persists is stored as a small relational key plus one large XML/CLOB (or BLOB) payload column:

| Table | Purpose | Payload column |
|---|---|---|
| `TB_CE_RULESETS` | Rule Editor rule sets (keyed by `PGM_CODE`, `MENU_ID`, `RULESET_ID`) | `RULESETS` (BLOB, rule tree XML) |
| `TB_CE_CELL_LAYOUT` | Cell Editor layouts | `LAYOUT` (CLOB, cell layout XML) |
| `TB_CE_BS_DEFINE` | Binding Source Define entries | `DEFINE` (CLOB, binding XML) |
| `TB_CE_USERDEFINE_SET` | User Define Set groups/subsets | `SUBSETS` (CLOB, subset XML) |

An optional system config object, `SYS-CT-CE-ControlVisibleConfig`, toggles whether the Lock checkbox is shown in the Rule Editor and User Define Set screens:

```xml
<!-- Config Start -->
<object id="SYS-CT-CE-ControlVisibleConfig" type="Tsb.Catos.Cm.ContainerEditor.Util.ControlVisibleConfig, Tsb.Catos.Cm.ContainerEditor">
  <property name="IsControlVisible" value="true"/>
</object>
<!-- The End of Config -->
```

**Note:** the source guide's own screenshot of this config XML appeared partially garbled/OCR-affected; verify the exact property name (`IsControlVisible`) against the real `Tsb.Catos.Cm.ContainerEditor.Util.ControlVisibleConfig` source before relying on it verbatim.

#### 7.1.4 Practical Cautions

- A Rule Set's Cell Layout binding is fixed at creation — plan the Cell Layout before creating dependent rule sets.
- `DisplayArea` must be explicitly set to `Outer` on any Cell Layout element that is meant to extend beyond the nominal cell boundary (e.g. an IMDG diamond overlay); otherwise the framework clips it.
- Rule Editor conditions are a flat AND/OR chain — there is no parenthesized precedence and no `IN` operator, so complex boolean logic must be restructured into nested `IfThen`/`IfThenElse` nodes instead.
- The Binding Source Define shortcut in Cell Editor (and the Cell-Editor shortcut in User Define Set) only becomes enabled after the parent object has been saved at least once.
- `CECodeHandler` instances registered via `CECodeManager.AddCodeHandler` live for the process lifetime; call `RemoveCodeHandler` explicitly if they must be released.

### 7.2 Graphical Drawing Framework

#### 7.2.1 Purpose

`Tsb.Fontos.Win.Drawing` is the low-level 2D drawing/graphics framework underlying every FONTOS diagram screen — it is precisely the layer the Container Editor (§7.1) is built on top of. Its stated purpose is to keep the implementation of "Graphical Views" consistent across modules and to raise development/maintenance efficiency by centralizing:

- Definition of 2D geometry objects (rectangle, ellipse, line, triangle, polygon, text, picture) via **GDI+**, along with the event handling they need (move, resize, mouse-over, click, drag-and-drop).
- A **`DrawArea` control** that hosts these objects with zoom in/out, auto-pan, drag-scroll, search, drag-and-drop, and print output built in.
- Performance techniques — redrawing only the visible viewport ("View Boundary") and caching rendered content as bitmaps — aimed at screens with hundreds of shapes.
- Resource-management discipline: geometry objects must be explicitly disposed, fonts reused, and unused memory-cache bitmaps released.

The guide's own worked examples are directly port/terminal diagrams: an "Outbound General Plan" bay-plan screen (a grid of colored container slots with a highlighted viewport), and a walkthrough of building a bay screen out of a `Bay` Drawable containing a title Geometry, a close-button Drawable, container-size Draw objects, and a grid of individual `Container` Drawables. In other words, this is the engine module developers use to build vessel bay/stowage-plan diagrams, yard/berth maps, and similar port-operations visualizations — the framework itself is domain-generic, but its design (zoom, pan, drag-drop, search, and an overlay table feature) is clearly purpose-built for exactly these screens.

#### 7.2.2 Building Blocks

**Object model.** FONTOS drawing is a three-tier object hierarchy, from lightest to heaviest:

- **Draw objects** (`DrawText`, `DrawEllipse`, `DrawLine`, `DrawRectangle`, `DrawTriangle`, `DrawPolygon`) inherit `BaseDraw` and implement `IBaseGeometry`, `IDrawable`, `IDisposable`. They represent a bare shape or text with no mouse-event handling, cannot be registered directly to a `DrawArea`, and must be hosted inside a Drawable object. They are the cheapest option memory-wise.
- **Geometry objects** (`GeometryRectangle` — base for `GeometryTriangle`, `GeometryText`, `GeometryEllipse`, `GeometryPicture` — and `GeometryLine` — base for `GeometryPolygon`) inherit `GeometryObject` and implement `IDrawableGeometry`. They support the full range of mouse events (click, double-click, select, resize, move) and **can** be registered directly with a `DrawArea`, or hosted inside a Drawable, at a higher memory cost than Draw objects.
- **Drawable objects** inherit `DrawableObject` (single default layer) or `LayerDrawableObject` (up to 5 render layers, higher number on top), both ultimately deriving from `BaseGeometry`. A Drawable is a *composite* — a named collection of child Draw and/or Geometry objects — and additionally provides: computed minimum bounding rectangle over its children, `Anchor`-based auto layout on resize, optional whole-object bitmap caching, and abstract mouse-event hooks for subclasses to override.

Key `DrawableObject` properties include `IsMemoryCache`, `IsChanged`, `IsForceDraw`, `IsSelected`, `IsOvered`, `EnableResizable`, `EnableMove`, `Anchor`, `EnableMouseOver`, `Location`/`Size`/`CurrentLocation`/`CurrentSize`/`MinimumSize`, `Visible`, `Bounds`, `IsBackGround`, `Degree`/`RotationCenter` (rotation). Key methods split into registration (`AddGeomObject`, `InsertGeomObject`, `RemoveGeomObject`, `ClearGeomObject`), search (`GetGeomObject`, `GetGeomList`, `GetGeomList<T>`, `Contains`, `ContainsAtMBR`, `GetGeometryAll`), z-order (`MoveFirst`, `MoveLast`, `MoveToIndex`), bounds (`UpdateMBR()`, `UpdateSize()`), and the abstract mouse handlers `OnMouseDown`/`OnMouseMove`/`OnMouseUp`/`OnSelected`/`OnMouseHover`/`OnMouseLeave`/`OnMouseClick`/`OnResize` (must-override) plus `OnResizeBegin`/`OnResizeEnd`/`OnMoveBegin`/`OnMoveEnd` (optional overrides).

**The canvas control.** `BaseDrawArea` is the abstract `UserControl` base for `DrawArea` (single layer) and `LayerDrawArea` (up to 5 layers, with a cross-layer link option); every layer-aware method on `LayerDrawArea` takes an extra `DisplayLayer` parameter. The control exposes:

- Drag&drop enablement (`AllowDragAtDrawControl`, `AllowDropAtDrawControl`) and auto-flow layout (`ArrangeDirection`, `ArrangeFixCount`, `ArrangeLeftMargin`, `ArrangeTopMargin` — `ArrangeFixCount = 0` auto-fits to the control's current size; `ArrangeDirection = None` requires the developer to supply absolute coordinates).
- Interaction-mode control via `ViewStatus` (`CONTROL` — full event handling, the default — vs `NAVIGATE` — pan-only, all other events suppressed), toggled at runtime by configurable shortcut keys (default `Ctrl+Shift+C` / `Ctrl+Shift+N`).
- Zoom via `ZoomMaximun`/`ZoomMinimun`/`ZoomLargeChange`, `GetScale(ZoomType.PAGE_SCALE_IN|PAGE_SCALE_OUT)`, `SetPageScale`, `ZoomInOut(...)`.
- Registration/search/removal (`AddDrawableObject`, `RemoveDrawableObject`, `FindFirst(key)`, `Find(x,y)`, `Find(Rectangle)`, `GetGeometryList<T>(bool deepSearch, [Point])`), selection (`SelectAll`, `UnselectAll`, `DeleteSelection`), and z-order (`MoveSelectionToTop/Front/Back`).
- Printing (`Print()`, `Print(PageProperty)`, `PrintDocument(...)` for assembling multi-page output from several `DrawArea` instances, `PrintToPdfByExportHandler`, `PrintToPreviewByPrintHandler`).
- Search (over drawn component names, or over registered business objects implementing `ISearchable` via `SearchKey`/`SearchKey<T>`, returning a `SearchResult` with `NextKey()`/`PreviousKey()` navigation), and Blink (flashing a set of `IBlinkable` items via `BlinkInfo`/`BlinkCountInfo`/`BlinkRunningTimeInfo` timing and a pluggable `IBlinkFilter`: `GrayscaleFilter`, `ColorChangeFilter`, `ColorFilter`, `ContrastFilter`, `GammaFilter`, `ImageFilter`, `InvertFilter`, `BrightnessFilter`).
- Events: `DrawableObjectClick`/`DoubleClick`/`DragSelect`, the six drag-and-drop events (`DrawableObjectDrag`, `DragEnter`/`Leave`/`Over`/`Drop`, `GiveFeedback`, `QueryContinueDrag`), `DrawAreaMouseWheel`, `DrawAreaResize`, `DrawableObjectMove`/`Resize` (+ Begin/End variants), `PageScaleChanged`, `ScrollPositionChanged`.

**Table overlay.** For summary/statistics panels drawn onto the same canvas (e.g. a stowage-type breakdown table alongside a bay plan), `TTable` provides a full grid object with header/row/column/corner sections, span-merging (`AddColumnHeaderSpanCell`, `MergeCell`), per-section styling (`StyleInfo`, `SetBackColor`/`SetForeColor`/`SetFont`, or a fully custom `ICellStyle`/`BaseCellStyle`), and live data binding via `SetDataSource(IList<T>)` where POCO property names must match the `dataField` names declared in a `ColumnHeaderSchema`/`TableSchema` (this data-bound mode requires the table to have no row header).

#### 7.2.3 Adding a New Diagram Screen — Workflow

1. **Reference the DLLs.** `Tsb.Fontos.Core.dll`, `Tsb.Fontos.Win.dll`, `Tsb.Fontos.Win.Drawing.dll` are required; `Tsb.Catos.Cm.Win.Drawing.dll` is optional (domain-specific extensions).
2. **Drop a `DrawArea` (or `LayerDrawArea`) control** onto the Form from the `Tsb.Fontos.Win.Drawing.Components` toolbox category, and configure its Properties-grid categories: drag&drop enablement, arrange direction, view-status shortcut keys, mouse-event enablement flags, and zoom limits.
3. **Populate the canvas.** For simple, static shapes, construct Geometry objects directly and register them:

```csharp
GeometryRectangle rec = new GeometryRectangle("Rectangle");
rec.Location = new Point(10, 10);
rec.Size = new Size(50, 50);
this.drawArea1.AddDrawableObject(rec);
this.drawArea1.Refresh();
```

4. **Implement a custom Drawable for anything reused or composite** (e.g. a `Container` cell, a `Bay` group). Subclass `DrawableObject` and choose one of two construction styles:
   - *Draw-object based* — override `Draw(Graphics g, float pageScale, Rectangle canvasBoundary, bool isMemoryCatch)`, early-return via `ObjectInCanvas(...)` for viewport clipping, and manually construct+`.Draw(g)` each shape. Cheapest, but the object's children cannot later be individually selected or handled for events. `Size` must be set explicitly or errors may occur.
   - *Geometry-object based* — override `Initialize()`, construct child Geometry objects, register each with `this.AddGeomObject(...)`, then call `this.UpdateMBR()` to compute the composite bounding box. More expensive, but children remain individually addressable/interactive.

```csharp
protected override void Initialize()
{
    // Child geometry built once at construction time and registered
    // with the parent Drawable so it participates in hit-testing/anchoring.
    // 부모 Drawable에 자식 지오메트리를 등록하여 히트테스트/앵커링에 참여시킴
    _myRect = new DrawRectangle(this.Name + "_R");
    _myRect.Attribute.LineColor = Color.Red;
    _myRect.Attribute.FillColor = Color.Red;
    _myRect.Size = new Size(_size, _size);
    this.AddGeomObject(_myRect);

    // Compute the composite's minimum bounding rectangle from its children.
    // 등록된 자식들로부터 최소 경계 사각형(MBR)을 계산
    this.UpdateMBR();
}
```

5. **Enable memory caching for expensive, mostly-static Drawables** registered directly to the `DrawArea` (set `base.IsMemoryCache = true` in the constructor); force a full redraw with `base.ClearMemoryCache()`, or mark just one changed child with `IsChanged = true` for a partial redraw (remember to also clear the previously drawn region — the framework does not do this automatically, so stale and fresh renders can visually overlap otherwise).
6. **Wire up mouse interaction** by overriding the abstract `OnMouseDown`/`OnMouseUp`/`OnMouseClick`/`OnSelected`/`OnMouseHover`/`OnMouseLeave`/`OnResize` methods (a typical pattern changes `Attribute.FillColor` between idle/hover/selected states). Hover only fires if the constructor sets `EnableMouseOver = true`; move requires both `IsDrawableObjectSelect` and `IsDrawableObjectMove` to be true at the control level.
7. **Add drag-and-drop** by implementing `IDragable` (exposing a `DragData` payload property) on the draggable class, setting `AllowDragAtDrawControl`/`AllowDropAtDrawControl = true`, and subscribing all six drag events on the `DrawArea`. In the `Drag` handler call `drawArea1.StartDragDrop(e.Item, e.GeometryItems)`; in `DragOver`, inspect `DragDropUtil.GetDragedDrawItems<T>(e.DragEvent)` and the geometry currently under the cursor to decide `e.DragEvent.Effect = DragDropEffects.Move` or `.None`; in `DragDrop`, move or construct the dropped item at the target location.
8. **Add layers if the diagram has independent visual strata** (e.g. base grid, container icons, labels) — use `LayerDrawArea` and pass a `DisplayLayer` value on every `AddDrawableObject`/`RemoveDrawableObject`/search call.
9. **Add zoom, search, and blink behavior** as needed:

```csharp
private void drawArea1_DrawAreaMouseWheel(object sender, MouseEventArgs e)
{
    // Ctrl+wheel drives zoom; plain wheel is left for normal scrolling.
    // Ctrl+휠로 확대/축소를 제어하고, 일반 휠 동작은 스크롤 용도로 남겨둠
    if (System.Windows.Forms.Control.ModifierKeys == Keys.Control)
    {
        if (e.Delta > 0)
            this.drawArea1.ZoomInOut(this.drawArea1.GetScale(ZoomType.PAGE_SCALE_IN), ZoomType.PAGE_SCALE_IN);
        else
            this.drawArea1.ZoomInOut(this.drawArea1.GetScale(ZoomType.PAGE_SCALE_OUT), ZoomType.PAGE_SCALE_OUT);
    }
}
```

```csharp
Expression<Func<SearchableCntrItem, bool>> filter = c => c.ContainerNo.Contains(this.GetSearchKey());
SearchResult findResult = this.drawArea1.SearchKey<SearchableCntrItem>(_drawingCntrItems, filter);
this.layoutView1.SearchAndAutoMove(findResult.NextKey());
```

10. **Dispose correctly.** Override `Dispose()`, call `base.Dispose()`, then explicitly dispose and null every internally-held Geometry field — the framework does not walk and dispose custom fields for you, so skipping this step leaks GDI+ resources.
11. **Print/export**, if the diagram needs hard-copy output: use `Print()`/`Print(PageProperty)` for the current view as-is, or `PrintDocument(...)` when assembling a multi-page document out of several independent `DrawArea` views; `PrintToPdfByExportHandler`/`PrintToPreviewByPrintHandler` cover PDF export and print-preview respectively.

#### 7.2.4 Practical Cautions

- `UpdateMBR()` recomputes bounds assuming an absolute origin of `(0,0)` — calling it on an object that already has a non-zero `Location` can silently discard that location. Prefer setting `Size` directly when it is already known, and reserve `UpdateMBR()`/`UpdateSize()` for cases where it genuinely is not.
- Move, resize, and drag-and-drop are effectively mutually exclusive feature combinations on the control (`IsDrawableObjectSelect`/`Move`/`Resize` vs. `AllowDragAtDrawControl`/`AllowDropAtDrawControl`) — check the required flag combination for the interaction you want before assuming it "should just work."
- `ViewStatus.NAVIGATE` is the correct way to build a read-only, pan-only overview screen without individually locking every Drawable.
- Blink filter changes do not hot-swap; changing `SetDefaultBlinkFilter` requires restarting the blink cycle via `StartBlink`.
- `TTable` cell merging and border operations (`MergeCell`, `AddCellBorder`, etc.) must be called only after `CreateTable()` has materialized the grid; a `SetDataSource`-bound table cannot have a row header.
- Image-filter numeric ranges are bounded: `BrightnessFilter` -255~255, `ContrastFilter` -100~100, `GammaFilter` 0.2~5 per channel — values outside these ranges are not validated as safe.

**Note:** the Development Guide for Graphical Drawing is a pure in-memory/runtime API reference — it defines no database tables or external config file formats of its own (contrast with the Container Editor's four `TB_CE_*` tables in §7.1.3). Table headers and styling are expressed purely in code via `ColumnHeaderSchema`/`RowHeaderSchema`/`TableSchema` objects, not as an external schema file.
## 8. Network / Interface Messaging Layer

### 8.1 Purpose

**FONTOS Network** (`Tsb.Fontos.Net`) is the client-server TCP/UDP communication layer of the framework. It sits between a FONTOS WinForms client and a back-end/interface server, handling connection lifecycle, message serialization, synchronous vs. asynchronous messaging, raw-socket message reassembly/filtering, and callback-based dispatch of responses back to the client. This is the layer you implement whenever a module needs to talk to a TCP server, as distinct from the database access layer covered in Section 9.

### 8.2 Configuration: `InfTargetInfo.xml`

Each network target is declared as an `<InfTarget Name="...">` entry in `InfTargetInfo.xml` (located under `[{Product}.Config].Environment.{ModuleName}`, mapped in code by `Tsb.Fontos.Net.Schema.InfTargetInfo`). A file can declare multiple targets, selected by name at runtime.

| Attribute | Meaning |
|---|---|
| `TransProtocol` | `TCP` or `UDP` (UDP requires the developer to implement `ITransportConnect` manually — no out-of-box implementation exists) |
| `NetProtocol` | `IP`, `ICMP`, or `IGMP` |
| `TransmissionMethod` | `Unicast`, `Broadcast`, or `Multicast` |
| `DoReconnect` | `Y`/`N` — auto-reconnect while the user session is logged in |
| `ReconnectInterval` | Reconnect interval in milliseconds (used when `DoReconnect = Y`) |
| `HandlerClass` / `HandlerClassAssembly` | Class implementing `IInterfaceHandler` (mandatory) |
| `FilterClass` | Class implementing `ITransportFilter` — reassembles raw received bytes/strings into the desired message shape |
| `ApplicationClass` | Class implementing `INetworkApplication` — passes messages into the framework and forwards incoming messages to the Presentation object |
| `PresentationClass` | Class implementing `INetworkPresentation` — business logic applied to incoming messages before they reach the client |
| `BrokerClass` | Class implementing `IMessageBroker` — converts outgoing objects → strings and incoming strings → objects |
| `AssemblyName` | Assembly containing the Handler/Filter/Application/Presentation/Broker classes |
| `MessageSegmentClass` / `MessageSegmentClassAssembly` | Implements `IMessageSeparator`; validates incoming message boundaries via delimiter characters (only needed with `TcpSimpleTransportConnect`/`TcpSimpleSyncTransportConnect`) |
| `TransportConnectClass` / `TransportConnectClassAssembly` | Implements `ITransportConnect`; connects to the server and sends/receives. Optional — if omitted, inferred from `TransProtocol` (defaults to TCP) |

Three built-in `ITransportConnect` implementations ship in `Tsb.Fontos.Net`:

- **`TcpTransportConnect`** — validates that incoming messages end with the Filter class's `MessageEndDelimiter`. If that property returns an empty string, the framework silently falls back to ASCII `(char)27` (ESC) as the delimiter.
- **`TcpSimpleTransportConnect`** — uses a `MessageSegmentClass` (arbitrary start/end delimiters) instead of an end-delimiter check; keeps all received messages in a dictionary (memory grows with unprocessed messages).
- **`TcpSimpleSyncTransportConnect`** — same as above but only retains the most recently received message.

### 8.3 Key Interfaces a Developer Implements

| Interface | Role |
|---|---|
| `IInterfaceHandler` | **Mandatory.** Constructs the Application/Broker/Filter/Presentation objects from `InfTargetInfo`, owns callback registration, and is the object your client code calls to send messages. |
| `INetworkApplication` | Creates a `TransportContext`, forwards outgoing messages to it, and receives inbound messages via `TransportContext.HandleMessage`. |
| `ITransportFilter` | Reassembles/validates raw server strings against a desired pattern before they reach the Broker; exposes `MessageEndDelimiter`. |
| `IMessageBroker` | `SendBroker` (object → string) and `ReceiveBroker` (string → object) conversions. |
| `INetworkPresentation` | Optional; applies business logic to an incoming message before it reaches the client. |
| `IMessageSeparator` | Optional; needed only with the "Simple" transport connect variants; detects message completion via start/end delimiters and raises `MessageCompleted`. |
| `IBaseMsgObjectSend` / `IBaseMsgObjectReceive` | Marker interfaces all outgoing/incoming message DTOs must implement. |
| `ITransportConnect` | Optional; only implement directly for UDP or a fully custom transport. |

All of the above except `IInterfaceHandler` are optional — implement only what a given target actually needs.

### 8.4 Setup and Usage

1. Add a Class Library project, reference `Tsb.Fontos.Core.dll` and `Tsb.Fontos.Net.dll`.
2. Implement whichever of the interfaces above the target requires (Application, Broker, Filter, Separator, Presentation, Handler — Handler is mandatory).
3. Create `InfTargetInfo.xml` under `[{Product}.Config].Environment.{ModuleName}` (Properties → `Copy to Output Directory = Copy if newer`), and configure the target's attributes.
4. Set the server endpoint the client will connect to in the client project's `CatosConfig.xml` — masking the actual deployment address:

```xml
<add key="TIP" value="xxx.xxx.xxx.xxx" />  <!-- server IP -->
<add key="TPT" value="6778" />              <!-- server port -->
```

5. Send a message:

```csharp
// Build the outgoing message
SampleSendMsg sendMsg = new SampleSendMsg();
sendMsg.Head = "Head";
sendMsg.Body = "Body";

// Resolve the handler for a named InfTarget entry
Tsb.Fontos.Net.Handler.IInterfaceHandler handler =
    Tsb.Fontos.Net.Factory.InfHandlerFactory.CreateInfHandler("C3ITApplication");

// Send synchronously and get a typed response
SampleReceiveMsg receiveMsg = handler.SendMessage(
    sendMsg,
    Tsb.Fontos.Net.Types.SyncTypes.Sync,
    typeof(SampleReceiveMsg)) as SampleReceiveMsg;
```

6. Register a callback for asynchronous responses via `IInterfaceHandler.AddRecvMsgProcInfo(Type, MessageProcessHandler)` and a `MsgProcOptions` object (`RespProcStyle` = `Direct` or `Period`, plus an `ExpirePolicy` of `None` or `QueuedOldMsg`).
7. Always call `Close()` on the Handler when done — this is an explicit resource-release step, not something the framework relies on GC/finalizers for.

### 8.5 Cautions

- `TransportConnectClass` is optional in the XML — leaving it out defaults to TCP via `Tsb.Fontos.Net`, which is usually what you want unless you need one of the "Simple" variants or UDP.
- Choose `TcpSimpleTransportConnect` vs. `TcpSimpleSyncTransportConnect` based on whether unprocessed/queued historical messages matter for that target — the former's dictionary can grow unbounded if responses aren't drained.
- `InfTargetInfo.xml` must be set to `Copy to Output Directory = Copy if newer`, or config edits won't be picked up at runtime.
- A single `InfTargetInfo.xml` may declare multiple `<InfTarget>` entries, each independently addressable by name — use this to model multiple concurrent server connections cleanly rather than juggling connection state manually.

---

## 9. Database Access

### 9.1 ODP.NET Managed Driver Setup

FONTOS's Oracle connectivity is configured to use Oracle's **ODP.NET Managed Driver** (`Oracle.ManagedDataAccess.dll`) — a fully managed, platform-independent (AnyCPU) ADO.NET provider that avoids installing Oracle's full unmanaged client. It requires **.NET Framework 4.0 or later**; running without it produces a blocking runtime dialog rather than a silent failure.

**Connection string** (namespace `Oracle.ManagedDataAccess.Client`), either with an explicit host/port/service, or a `tnsnames.ora` alias — mask real hosts/credentials in any committed sample:

```
User Id=<db_user>; Password=****; Data Source=<host>:1521/<service_name>; Pooling=false;
```

or, resolving the Data Source through a `tnsnames.ora` alias:

```
User Id=<db_user>; Password=****; Data Source=<tns_alias>; Pooling=false;
```

If using an alias, `tnsnames.ora` must be copied alongside the application's executable (and, for a FONTOS Config project, its Properties should be set to `Copy to Output Directory = Copy always` — not "Copy if newer" — to guarantee it is always present).

**Wiring the driver into FONTOS's persistence layer** requires three coordinated edits:

1. Register the provider in `[Config project] > Service > Config > Persistence > providers.config`:

```xml
<provider
  name="oracleProvider"
  description="Oracle, oracle provider V4.121.1.0"
  enabled="true"
  default="true"
  assemblyName="Oracle.ManagedDataAccess, Version=4.121.1.0, Culture=neutral, PublicKeyToken=89b483f429c47342"
  connectionClass="Oracle.ManagedDataAccess.Client.OracleConnection"
  commandClass="Oracle.ManagedDataAccess.Client.OracleCommand"
  parameterClass="Oracle.ManagedDataAccess.Client.OracleParameter"
  parameterDbTypeClass="Oracle.ManagedDataAccess.Client.OracleDbType"
  parameterDbTypeProperty="OracleDbType"
  dataAdapterClass="Oracle.ManagedDataAccess.Client.OracleDataAdapter"
  commandBuilderClass="Oracle.ManagedDataAccess.Client.OracleCommandBuilder"
  usePositionalParameters="true"
  useParameterPrefixInSql="true"
  useParameterPrefixInParameter="true"
  parameterPrefix=":"
  useDeriveParameters="false"
  allowMARS="false"
/>
```

2. Add the ODP.NET config section to `App.config`:

```xml
<configSections>
  <section name="oracle.manageddataaccess.client"
           type="OracleInternal.Common.ODPMSectionHandler, Oracle.ManagedDataAccess" />
</configSections>
<oracle.manageddataaccess.client>
  <version number="*">
    <settings>
      <setting name="TNS_ADMIN" value=".\Service\Config\Persistence"/>
    </settings>
  </version>
</oracle.manageddataaccess.client>
<startup>
  <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.0"/>
</startup>
<runtime>
  <assemblyBinding xmlns="urn:schemas-microsoft-com:asm:v1">
    <probing privatePath="Lib\Oracle\4.121.1.0;" />
  </assemblyBinding>
</runtime>
```

3. Copy `tnsnames.ora` from the local Oracle client's `NETWORK/ADMIN` folder into `Service/Config/Persistence`, then rebuild the Config project.

**Gotchas:**

- The `<section name="...">` in `<configSections>` must match the tag name used in `<configuration>` **exactly** — a common source of silent config mismatches.
- The `assemblyName`/`Version`/`PublicKeyToken` in `providers.config` must match the actual installed driver version — cross-check the GAC-registered version under `C:\Windows\assembly` and the project's Build → Platform target (x86/AnyCPU) against the installed client bitness, or DB connectivity will fail at runtime with no obvious error pointing at the mismatch.
- The Managed driver is not unconditionally faster than the legacy unmanaged 11.2 driver — published benchmarks in the source material show the unmanaged driver winning for very large bulk `DataAdapter` operations (especially with error handling), so don't assume "managed" is always the right choice for high-volume batch DAO code without measuring.
- The Managed driver supports Code Access Security, which may be relevant to your deployment/build security policy.

### 9.2 iBATIS.Net SQL-Map Pattern (brief cross-reference)

FONTOS's data-access layer is built on **iBATIS.Net** (`IBatisNet.DataMapper`, `IBatisNet.Common`), following the standard **Dao + Map XML** pattern used throughout the framework: a `*Dao` class issues named statements, and a `*Map.xml` file (registered via `<sqlMap embedded="...">` in a per-module `SqlMap-{TeamName}{ModuleID}.config`) defines the actual SQL, keyed by Statement ID (`{sql statement type}_{description or DAO method name}`, e.g. `select_codeDetail`). Per the Object ID Naming Rule conventions (Section 10), Map file names and Result/Parameter Map IDs follow fixed transformation rules rather than ad hoc naming. The full View→Controller→Item→Param→Service→Dao→Map vertical slice is worked through end-to-end in §4.4.

Consistent with supporting more than one Oracle-only deployment, the framework's provider configuration is DB-vendor-pluggable — expect (and follow) **per-vendor Map file variants** (e.g., MSSQL / POSTGRE / Oracle) alongside a vendor-neutral Dao layer, matching the `providers.config` entry for whichever database the target environment actually runs.

---

## 10. UI Standards, Naming Conventions, and Coding Standards

### 10.1 UI Standards and Guidelines — Key Rules

The UI Standards and Guidelines document defines the visual/UX baseline every FONTOS-based WinForms screen should follow, so that modules built by different teams present one consistent product.

**Windows**
- Every window must have a unique title; a title-bar icon should indicate the business area.
- Toolbars attach to the MDI window, not to individual child windows, wherever possible.
- Most screens are MDI "Document" style; use modal "Dialog" style only when the user must supply specific information to complete an action, and give modal windows a default button (typically OK).
- Maximum window size: **700×500 px** (to remain usable at 800×600 screen resolution) — size windows only as large as necessary; modal windows should be smaller than the non-modal window that invoked them; all non-modal windows should be resizable; a window must be fully visible when first opened.
- Default font: **Tahoma, 9pt, black**. Do **not** hardcode background color — use the system default "Control" color so the UI respects the end user's OS theme (this is an explicit exception to the otherwise prescriptive color rules elsewhere in the document).

**Canvases / Regions**
- A Content Canvas matches the size of its window; Stacked Canvases are sized exactly to their contents.
- Regions (frame or tab groupings) are titled with a noun, default title color **blue (RGB 0,0,192)**, positioned 6×6 px from the canvas edge, with 18px first-line label spacing, 21px first-line textbox spacing, 9px general field spacing, and 6×6 px spacing between adjacent frames.
- Tab order: left-to-right then top-to-bottom within a region, and left-to-right then top-to-bottom between regions.

**Text items / labels / controls**
- Alignment: imprecise numeric values (IDs, phone numbers) and text/date values → left; precision numeric values (quantities, prices) → **right**.
- Field height **22px**; editable fields **light blue (RGB 192,255,255)**; non-editable fields gray background.
- Labels: default font except **black bold** for mandatory items; default alignment **right**; `AutoSize = false`; label width = widest label in its group.
- Radio groups always have a default value; group title color matches Region title (blue).
- Buttons: one row of buttons per window where possible, one default button per window, all iconic buttons need Tooltip text; standard textual button size **Height 23 / Width 75 / Center-aligned**.

**Spread (grid) list rules** — directly ties into TSpreadGrid/Spread Editor conventions in Section 6:
- Header uses the default background color; bold font reserved for mandatory columns only.
- Body: non-editable fields → white background; editable Text-type fields → **light blue (RGB 192,255,255)**; editable Fixed-Code-type fields → **light green (RGB 192,255,192)**.
- Alignment: Text → left (center only if the value is under ~5 characters, e.g. a size/type code); Fixed Code → center; Number → right.
- Input style: Text types via direct in-cell typing; Fixed Code types via combo-style popup.
- Use the **Remark** cell type (Section 6) for columns whose text may overflow the column width or contain carriage returns.

### 10.2 Object ID Naming Rule — Full Reference Table

This is the master naming convention used across every FONTOS/CATOS product and module. Reproduced in full from the Object ID Naming Rule spreadsheet.

**Product ID**

| Prefix | Description |
|---|---|
| CM | TSB COMMON |
| FT | FONTOS |
| CT | CATOS |
| C3 | C3IT |
| CA | CASP |
| MO | Most |
| PL | Plus |

**Module ID** (Product → Product ID → Module → Module Abbreviation → Module ID)

| Product | Product ID | Module | Module Abbr | Module ID |
|---|---|---|---|---|
| FONTOS | FT | Common | CM | FTCM |
| FONTOS | FT | Core | CO | FTCO |
| FONTOS | FT | Win | WN | FTWN |
| FONTOS | FT | Drawing | DW | FTDW |
| FONTOS | FT | Sample | SA | FTSA |
| CATOS COMMON | — | Core | CO | CTCO |
| CATOS COMMON | — | Win | WN | CTWN |
| CATOS COMMON | — | Drawing | DW | CTDW |
| CATOS | CT | Common | CM | CTCM |
| CATOS | CT | Birth Planning | BP | CTBP |
| CATOS | CT | Yard Planning | YP | CTYP |
| CATOS | CT | Ship Planning | SP | CTSP |
| CATOS | CT | Rail Planning | RP | CTRP |
| CATOS | CT | Yard Define | YD | CTYD |
| CATOS | CT | Operation Management | OM | CTOM |
| CATOS | CT | Delivery Reservation | DR | CTDR |
| CATOS | CT | DG Operation | DG | CTDG |
| CATOS | CT | Reefer Operation | RO | CTRO |
| CATOS | CT | Billing | BL | CTBL |
| CATOS | CT | CFS | CF | CTCF |
| CATOS | CT | Statistics | ST | CTST |
| CATOS | CT | Gate | GA | CTGA |
| CATOS | CT | Berth Monitoring | BM | CTBM |
| CATOS | CT | Terminal Monitoring | TM | CTTM |
| CATOS | CT | Tally | TA | CTTA |
| CATOS | CT | YQ | YQ | CTYQ |
| CATOS | CT | EDI | ED | CTED |
| C3IT | C3 | (none defined) | | |
| CASP | CA | (none defined) | | |
| Most | MO | (none defined) | | |
| Plus | PL | (none defined) | | |

**ID Prefix** (generic object-role prefixes used inside every Object ID)

| Prefix | Description |
|---|---|
| MSG | Message Identifier |
| WRD | Word Identifier |
| VIW | UI VIEW Identifier |
| GRD | Grid (Spread) Identifier |
| CTL | UI Controller Identifier |
| ITM | Item object Identifier |
| PAR | Parameter object Identifier |
| RES | Result object Identifier |
| SVC | Service Identifier |
| DAO | DAO Identifier |
| GNR | Framework/Helper Function Identifier |
| UTL | Utility Function Identifier |

**Message ID** — syntax: `{ID Prefix}_{Module ID}_{5-digit number}`

| Example | Result |
|---|---|
| TSB Common "Please contact administrator" | `MSG_CTCM_00001` |
| CATOS Ship Planning "Vessel not found" | `MSG_CTSP_00001` |

**Word ID** — syntax: `{ID Prefix}_{blank-removed word}`

| Example | Result |
|---|---|
| "POD" | `WRD_POD` |
| "Vessel Schedule" | `WRD_VesselSchedule` |

**SQL Map naming**

| Element | Syntax | Example |
|---|---|---|
| Map file name | `{ID Prefix}_{Module ID}` | `Persistence/FontosSample/BasicFlow/CodeDetailMAP.xml` |
| Result Map ID | `{result description}` + `Result` | `codeDeailResult` [sic, verbatim source spelling] |
| Parameter Map ID | camelCase of the parameter class's simple name | `Tsb.Fontos.Sample.Common.Param.BasicFlow.BasicFlowParam` → `basicFlowParam` |
| Statement ID | `{sql statement type}_{sql description / DAO method name}` | `insert_codeMaster`, `select_codeDetail` |

**Object ID — worked example, CATOS DG Operation module**

Object ID Key-word syntax: `{Product ID}-{Module ID}-{Package Key Word}`
Object ID syntax: `{ID Prefix}-{Object ID Key-word}-{Class Name}` — e.g. `SVC-CT-CTDG-COD-BasicFlowService`

| Package | Pkg Key-Word | Object ID Key Word | VIEW | GRID | CONTROLLER | ITEM | PARAM | RESULT | SERVICE | DAO | UTL |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Code | COD | CT-CTDG-COD | VIW-CT-CTDG-COD | GRD-CT-CTDG-COD | CTL-CT-CTDG-COD | ITM-CT-CTDG-COD | PAR-CT-CTDG-COD | RES-CT-CTDG-COD | SVC-CT-CTDG-COD | DAO-CT-CTDG-COD | UTL-CT-CTDG-COD |
| Operation | OPR | CT-CTDG-OPR | VIW-CT-CTDG-OPR | GRD-CT-CTDG-OPR | CTL-CT-CTDG-OPR | ITM-CT-CTDG-OPR | PAR-CT-CTDG-OPR | RES-CT-CTDG-OPR | SVC-CT-CTDG-OPR | DAO-CT-CTDG-OPR | UTL-CT-CTDG-OPR |

Grid IDs append a further suffix: `{ObjectID}-{List Title, spaces removed}` → e.g. `GRD-CT-CTDG-COD-TestList`.

**Object ID — FONTOS Core**

Syntax: `{ID Prefix}-{Object ID Key-word}-{Class Name}` — e.g. `GNR-FTCO-CFG-AppConfigProvider`

| Package | Pkg Key-Word | Object ID Key Word | Framework Prefix |
|---|---|---|---|
| Cache | CCH | FTCO-CCH | GNR-FTCO-CCH |
| Code | COD | FTCO-COD | GNR-FTCO-COD |
| Command | CMD | FTCO-CMD | GNR-FTCO-CMD |
| Configuration | CFG | FTCO-CFG | GNR-FTCO-CFG |
| Context | CTX | FTCO-CTX | GNR-FTCO-CTX |
| Data | DAT | FTCO-DAT | GNR-FTCO-DAT |
| Environments | ENV | FTCO-ENV | GNR-FTCO-ENV |
| Event | EVT | FTCO-EVT | GNR-FTCO-EVT |
| Exceptions | EXP | FTCO-EXP | GNR-FTCO-EXP |
| Item | ITM | FTCO-ITM | GNR-FTCO-ITM |
| Logging | LOG | FTCO-LOG | GNR-FTCO-LOG |
| Message | MSG | FTCO-MSG | GNR-FTCO-MSG |
| Net | NET | FTCO-NET | GNR-FTCO-NET |
| Object | OBJ | FTCO-OBJ | GNR-FTCO-OBJ |
| Param | PAR | FTCO-PAR | GNR-FTCO-PAR |
| Reflection | REF | FTCO-REF | GNR-FTCO-REF |
| Report | REP | FTCO-REP | GNR-FTCO-REP |
| Resources | RSC | FTCO-RSC | GNR-FTCO-RSC |
| Security | SEC | FTCO-SEC | GNR-FTCO-SEC |
| Service | SVC | FTCO-SVC | GNR-FTCO-SVC |
| Thread | THR | FTCO-THR | GNR-FTCO-THR |
| Transaction | TRS | FTCO-TRS | GNR-FTCO-TRS |
| Util | UTL | FTCO-UTL | GNR-FTCO-UTL |
| Validator | VAL | FTCO-VAL | GNR-FTCO-VAL |
| Xml | XML | FTCO-XML | GNR-FTCO-XML |

**Object ID — FONTOS Win**

Documented syntax: `{ID Prefix}-{Object ID Key_word}_{Class Name}` — e.g. `GNR-FTDW-MNU-AppConfigProvider`

**Note:** the source spreadsheet's stated syntax for FONTOS Win mixes hyphens and underscores, inconsistently with FONTOS Core's all-hyphen syntax. The worked example itself (`GNR-FTDW-MNU-AppConfigProvider`) actually uses hyphens throughout, matching Core. Treat **hyphen-separated** as the de facto rule and the underscore wording as a documentation artifact — confirm against the live codebase before enforcing this rule in a linter/codegen tool.

| Package | Pkg Key-Word | Object ID Key Word | Framework Prefix |
|---|---|---|---|
| Custom | CUS | FTDW-CUS | GNR-FTDW-CUS |
| Event | EVT | FTDW-EVT | GNR-FTDW-EVT |
| Forms | FOM | FTDW-FOM | GNR-FTDW-FOM |
| FormTemplate | FMT | FTDW-FMT | GNR-FTDW-FMT |
| Grid | GRD | FTDW-GRD | GNR-FTDW-GRD |
| Layout | LAY | FTDW-LAY | GNR-FTDW-LAY |
| Menu | MNU | FTDW-MNU | GNR-FTDW-MNU |
| Navigation | NAV | FTDW-NAV | GNR-FTDW-NAV |
| Resources | RSC | FTDW-RSC | GNR-FTDW-RSC |
| Spread | SPR | FTDW-SPR | GNR-FTDW-SPR |

**Object ID — CATOS Core** (worked example: "Container")

Syntax: `{ID Prefix}-{Object ID Key-word}-{Class Name}` — e.g. `SVC-CT-CTCO-CNTR-BasicFlowService`

| Package | Pkg Key-Word | Object ID Key Word | VIEW | GRID | CONTROLLER | ITEM | PARAM | RESULT | SERVICE | DAO | UTL |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Container | CNTR | CT-CTCO-CNTR | VIW-CT-CTCO-CNTR | GRD-CT-CTCO-CNTR | CTL-CT-CTCO-CNTR | ITM-CT-CTCO-CNTR | PAR-CT-CTCO-CNTR | RES-CT-CTCO-CNTR | SVC-CT-CTCO-CNTR | DAO-CT-CTCO-CNTR | UTL-CT-CTCO-CNTR |

**Object ID — CATOS Win** (worked example: "ContainerDetail")

Example: `SVC-CT-CTWN-CTDL-BasicFlowService`

| Package | Pkg Key-Word | Object ID Key Word | VIEW | GRID | CONTROLLER | ITEM | PARAM | RESULT | SERVICE | DAO | UTL |
|---|---|---|---|---|---|---|---|---|---|---|---|
| ContainerDetail | CTDL | CT-CTWN-CTDL | VIW-CT-CTWN-CTDL | GRD-CT-CTWN-CTDL | CTL-CT-CTWN-CTDL | ITM-CT-CTWN-CTDL | PAR-CT-CTWN-CTDL | RES-CT-CTWN-CTDL | SVC-CT-CTWN-CTDL | DAO-CT-CTWN-CTDL | UTL-CT-CTWN-CTDL |

**Object ID — CATOS Drawing** (worked example: "ShipSideView")

Example: `SVC-CT-CTDW-SHIP-BasicFlowService`

| Package | Pkg Key-Word | Object ID Key Word | VIEW | GRID | CONTROLLER | ITEM | PARAM | RESULT | SERVICE | DAO | UTL |
|---|---|---|---|---|---|---|---|---|---|---|---|
| ShipSideView | SHIP | CT-CTDW-SHIP | VIW-CT-CTDW-SHIP | GRD-CT-CTDW-SHIP | CTL-CT-CTDW-SHIP | ITM-CT-CTDW-SHIP | PAR-CT-CTDW-SHIP | RES-CT-CTDW-SHIP | SVC-CT-CTDW-SHIP | DAO-CT-CTDW-SHIP | UTL-CT-CTDW-SHIP |

**How to construct any Object ID:**

1. Determine the Product ID (2 letters, e.g. `CT` for CATOS).
2. Determine the Module ID (4 letters = Product ID + Module Abbreviation, e.g. `CTDG` for CATOS DG Operation).
3. Determine the Package Key-Word — a short 3–4 letter abbreviation for the sub-package/feature (e.g. `COD` for Code, `OPR` for Operation, `CNTR` for Container).
4. Concatenate into the Object ID Key-word: `{ProductID}-{ModuleID}-{PackageKeyWord}`.
5. Prefix with the role/layer prefix (`VIW-`, `GRD-`, `CTL-`, `ITM-`, `PAR-`, `RES-`, `SVC-`, `DAO-`, `UTL-`, or `GNR-` for Core/Win framework code) and append `-{ClassName}`. Grid IDs additionally append `-{List Title with spaces removed}`.

**Note:** only Product IDs actually in use as of the spreadsheet's 2010-03 authoring date (CM/FT/CT/C3/CA/MO/PL) have Module ID breakdowns defined; a new module under C3IT/CASP/Most/Plus would need its Module ID defined following the same pattern before use. Also note two verbatim source typos preserved for fidelity: `codeDeailResult` (missing "t" — not "codeDetail") and "Catos - Commn - Win/Drawing" (missing "o" in "Common") in the CATOS Win/Drawing sheet headers — do not silently "fix" these if they already correspond to identifiers baked into legacy code; verify against source before correcting.

### 10.3 General Coding Standard Callouts

Drawn directly from the recurring cautions across the research material — apply these across all FONTOS module development, not just the areas above:

- **Always follow the Object ID Naming Rule table (§10.2)** for class, view, grid, controller, item, param, result, DAO, and service names — this is the mechanism the whole product family relies on for consistent tooling, search, and cross-team code review.
- **Never hardcode secrets or environment-specific connection details.** DB credentials belong in `CatosConfig.xml`/`providers.config`, not in code; when documenting or sharing configuration (including in tickets, chat, or this guide), mask real IPs, hostnames, connection strings, and passwords (e.g., `UID`/`PWD` in `CatosConfig.xml`, `TIP`/`TPT` in network config, Oracle connection strings) — replace with placeholders such as `****` or `xxx.xxx.xxx.xxx`.
- **Validate every external input** reaching a grid, form, or network message: TSpreadGrid's own history shows repeated defects from features (Delete, Print, Row Count, Drag Fill) failing to account for filtered/hidden rows, and from Column Setting / Sort interactions that assumed data was always in an expected state — treat "grid currently has an active Header Filter / Column Setting / Sort" as a condition every row-affecting operation must explicitly check, not an edge case.
- **Don't hardcode UI colors or window backgrounds** outside the documented palette in §10.1 — in particular, never hardcode a window/control background color; rely on the system "Control" color so themes/OS settings are respected.
- **Match exact config-file requirements precisely.** ODP.NET's `<section>` tag name must match its `<configuration>` counterpart exactly; `providers.config`'s assembly version/PublicKeyToken must match the installed driver; `InfTargetInfo.xml` and other config files must be set to the correct "Copy to Output Directory" mode or edits are silently ignored at runtime.
- **Rebuild (not just Build) after schema, resource, or renamed-assembly changes** — Spread schema XML and localization/resource key changes have been documented as not taking effect without a full project Rebuild.
- **Prefer the framework's pluggable extension points over ad hoc code**: implement a new `IEnableStrategy` for custom menu/toolbar enable logic, a new `BaseCustomAction` subclass for new menu behaviors, a new `ISpreadFilterStrategy`/filter-chain handler for new filter operators, and a `BaseSerializerValueStrategy` for new View-State-serializable property types — rather than special-casing framework call sites.
- **Treat CATOS (`Tsb.Catos.*`) and FONTOS (`Tsb.Fontos.*`) as layered, not interchangeable**: several "TSpreadGrid features" (Biz Rule validation, code lookups, code coloring) are actually implemented in CATOS and only exposed through FONTOS integration points — know which layer you're actually modifying before changing behavior.
- **Write test coverage for validation and mask logic** (Direct vs. Indirect input, custom-string vs. regex masks) and for any new `BaseFilterHandler`/`IEnableStrategy`/`BaseSerializerValueStrategy` implementation — these are exactly the kinds of small, high-fan-in extension classes where a regression silently breaks many screens at once.
- **Watch for known typo-vs-real-API ambiguity.** Several identifiers recur across the source documentation with inconsistent spelling (e.g., `GetDefalutParam()`, `EnableOnlyDefneStateMode`, `codeDeailResult`, `GetCurosrWithDefaultCursor()`). Confirm the actual spelling in the shipped DLL via IntelliSense/reflection before depending on any of these literally in new code — and if you find the real API differs from what's written here, correct this guide rather than propagating the error further.

---

## 11. Testing

FONTOS ships a parallel set of test projects alongside the framework and every business module. The convention, verified directly against `Tsb-Fontos-Framework`, `Tsb-Catos-Cm`, and `Tsb-Fontos-Sample`, is to add **up to three test projects per layer**: a unit-test project (`*.Test`), sometimes a DB-focused unit-test project (`*.TestDB`), and a UI/end-to-end test project (`*.UITest`) built on the legacy Visual Studio **Coded UI Test** technology (`Microsoft.VisualStudio.TestTools.UITesting`) plus MSTest (`Microsoft.VisualStudio.TestTools.UnitTesting`).

> **Note:** Coded UI Test was deprecated by Microsoft starting with Visual Studio 2019/2022 and has no supported successor from Microsoft directly (community guidance points to WinAppDriver/Playwright/Appium for new automation). FONTOS's UI test layer is still built on it as of the reviewed source. New module developers should treat the `*.UITest` layer as a **legacy regression-test facility to extend cautiously**, not a pattern to imitate for new automation investments — flag this with your team lead before adding large amounts of new Coded UI coverage.

### 11.1 Layer 1 — Framework-level base classes (`Tsb.Fontos.Test`)

`Tsb-Fontos-Framework\Tsb.Fontos.Test` defines the root abstractions every module's unit tests ultimately derive from:

| Class / File | Role |
|---|---|
| `TestTemplate/BaseTest.cs` | Root `[TestClass]` abstract base (`public abstract class BaseTest : TsbBaseObject`). Owns a static `ConfigInitAdapter` (`IConfigInitAdapter`) and static `InfHandler` (`IInterfaceHandler`); exposes `OutputHandler` for writing structured test output; declares the abstract `InitConfig()` every subclass must implement; provides a shared `[ClassCleanup]` that closes `InfHandler` if it was opened. `ConfigLoad()` resolves `IConfigInitAdapter` from the Spring/Fontos object builder (`ObjectBuilder.GetObjectBuilder().GetObject(SysObjectSpec.SYS_OBJECT_ID_TEST_CONFIG_INIT_ADAPTER)`) and calls `InitConfig()` on it, wrapping any failure in `AssertFailedException`. |
| `TestTemplate/BaseControllerTest.cs` | `BaseControllerTest : BaseTest` — for testing `BaseController`-derived classes. `InitConfig()` seeds a fake logged-in session (`AppEnv.UserInfo` with `TmnlID = "TMNL-TEST"`, `StaffCD = "TEST"`, `IsAdminYN = "Y"`) so controller/business logic that reads the ambient user context has something valid to read during a test run. |
| `TestTemplate/BaseDaoTest.cs` | `BaseDaoTest : BaseTest` — for testing DAO classes directly against a real database connection. `InitConfig()` is currently a no-op override; DB context comes from the ambient config loaded via `ConfigLoad()`. |
| `TestTemplate/BaseServiceTest.cs` | `BaseServiceTest : BaseTest`, also decorated `[TestClass]` — for testing `BaseService`-derived business-logic classes. `InitConfig()` is likewise a no-op override in the base; module test classes override further as needed. |
| `TestTemplate/BaseTestAssembly.cs` | A thin, currently-empty `BaseTestAssembly : BaseTest` marker class (added 2025 per its header) — **Note:** no members were found in the reviewed source; its exact intended use (assembly-level one-time setup/teardown hook point vs. a future extension point) could not be confirmed from source alone and should be confirmed with the framework team before relying on it. |
| `Handler/IOutputHandler.cs`, `Handler/OutputHandler.cs` | Pluggable test-output sink abstraction used by `BaseTest.OutputHandler`. |
| `Output/BaseOutputFactory.cs`, `Output/OutputConsoleFactory.cs` | Factory classes that construct the concrete output sink (console by default). |
| `UITest/Grid/GridHelper.cs`, `UITest/Drawing/DrawingViewHelper.cs`, `UITest/Win/ControlHelper.cs`, `UITest/Win/MotionHelper.cs` | Shared helper classes for driving `TSpreadGrid` cells, `DrawArea` diagrams, and generic WinForms controls from Coded UI tests — these are the low-level building blocks that the higher-level `Tsb.Catos.Cm.UITest` proxies (below) are built on top of. |

All `BaseTest`-derived classes use the object ID prefix `GNR-FTTS-TST-` (Framework / Test module), consistent with the company Object ID Naming Rule (see Section 10.2) — note `FTTS` reads as "**F**on**T**os **T**e**S**t", an abbreviation not spelled out anywhere in the source comments; treat it as the module's registered ID code rather than deriving it algorithmically.

A minimal module-level unit test class typically looks like this (illustrative, not verbatim source):

```csharp
using Microsoft.VisualStudio.TestTools.UnitTesting;
using Tsb.Fontos.Test.TestTemplate;

namespace Tsb.Fontos.Sample.Test.Service
{
    // Unit tests for SingleGridService business logic.
    // SingleGridService의 비즈니스 로직에 대한 단위 테스트입니다.
    [TestClass]
    public class SingleGridServiceTest : BaseServiceTest
    {
        private ISingleGridService _service;

        [TestInitialize]
        public void SetUp()
        {
            // Load Spring.NET context / DB config exactly once per test class.
            // 테스트 클래스당 한 번만 Spring.NET 컨텍스트/DB 설정을 로드합니다.
            BaseServiceTest.ConfigLoad();
            this.InitConfig();

            _service = (ISingleGridService)ObjectBuilder.GetObjectBuilder()
                .GetObject("SVC-OM-SGOP-SingleGridService");
        }

        [TestMethod]
        public void SearchList_ReturnsExpectedRowCount()
        {
            // Arrange: always validate/construct input params explicitly —
            // never trust a hand-built param object to be well-formed.
            // 입력 파라미터는 항상 명시적으로 검증/구성해야 하며,
            // 임의로 만든 파라미터 객체를 그대로 신뢰하지 않습니다.
            var param = new SingleGridParam { TmnlCD = "TMNL-TEST" };

            // Act
            var result = _service.SearchList(param);

            // Assert
            Assert.IsNotNull(result);
        }
    }
}
```

### 11.2 Layer 2 — Catos-Cm shared test utilities (`Tsb.Catos.Cm.Test`, `Tsb.Catos.Cm.UITest`)

`Tsb-Catos-Cm\Tsb.Catos.Cm.Test` extends the framework test layer with CATOS-specific, reusable **schema/data assertions**:

| Class / File | Role |
|---|---|
| `Util/TAssert.cs` | A large (~108 KB) custom assertion helper library, additional to MSTest's own `Assert`. Given its size, treat it as CATOS's de-facto assertion DSL — **grep it for an existing helper before hand-rolling comparison/validation logic in a new test**, rather than reading it end-to-end. |
| `Util/Dao/ITAssertDao.cs`, `Util/Dao/TAssertDao.cs` | DB-schema-introspection DAO backing `TAssert`'s schema-level assertions (see `IUnitTestService` below). |
| `Item/UnitTestItem.cs`, `Param/UnitTestParam.cs` | Item/Param pair (per the standard FONTOS Item/Param convention — see Section 4) used by the schema-validation service. |
| `Service/IUnitTestService.cs` / `Service/UnitTestService.cs` | `UnitTestService : BaseService, IUnitTestService`, object ID `SVC-CM-CMCO-TST-TestService`. Exposes two DB-schema-oriented test helpers: `ExistsDbSchema(UnitTestParam param)` (returns whether a given DB schema/table exists — useful as a test pre-condition guard) and the generic `ValidDbColumnItemType<T>(UnitTestParam param)` (validates that an Item class's mapped properties match the actual DB column types — catches Item/DB drift before it surfaces as a runtime iBATIS mapping error). Both are marked `[TransactionOption(TransactionScopeTypes.Support)]`, i.e. they participate in an ambient transaction if one exists but do not require starting a new one. |
| `Service/Dao/IUnitTestDao.cs` / `UnitTestDao.cs`, `Service/Map/UnitTestMap.xml` | The DAO + iBATIS SQL map backing `UnitTestService`'s schema queries — the same Dao/Map pattern used by every other business Service (Section 9). |

`Tsb-Catos-Cm\Tsb.Catos.Cm.UITest` supplies the CATOS-wide Coded UI Test base and a large library of **screen-element proxy classes** so module UI tests don't re-implement low-level control automation:

| Class / File | Role |
|---|---|
| `TestTemplate/BaseCatosUITest.cs` | `[CodedUITest] public class BaseCatosUITest` — the base every module `*UITest` class derives from. Constructed with a `programCode` identifying which CATOS module screen to drive, and an `autoRunApplication` flag. `[TestInitialize] BaseTestInitialize()` calls the overridable `DoBeforTest()` (launches/attaches to the application via `ApplicationLogin.Start()`, focuses the window, optionally closes stray windows per `TUITestEnvironment`, and starts `UITestMonitor`); `[TestCleanup] BaseTestCleanup()` calls `DoAfterTest()` (records the outcome via `TestContextStatus`, optionally closes windows, and — per `ModuleRestartRule.Check()` — force-kills and resets state if the environment's restart policy requires it for the next test). **Note (naming inconsistency, preserve as-is):** the two overridable lifecycle hooks are spelled `DoBeforTest()` / `DoAfterTest()` in the real shipped source — "Befor" is missing the trailing "e". Do not "fix" this when overriding; override the method exactly as spelled. |
| `Proxy/ApplicationProxy.cs`, `Proxy/MenuProxy.cs`, `Proxy/ContextMenuProxy.cs`, `Proxy/CodePopupProxy.cs`, `Proxy/DateTimePickerProxy.cs`, `Proxy/GridHeadFilterProxy.cs`, `Proxy/SetColorProxy.cs`, `Proxy/SummaryProxy.cs`, `Proxy/UIControlProxy.cs`, `Proxy/VesselScheduleProxy.cs`, `Proxy/MonitorProxy.cs`, `Proxy/Sp.cs` | Screen/control-specific automation proxies — e.g. `ApplicationProxy` starts/kills/focuses the app under test and reports `CanFocus`/version info; `MenuProxy` drives the ribbon/menu; `CodePopupProxy` drives the shared code-lookup popup dialog; `GridHeadFilterProxy` drives the TSpreadGrid column-filter header. **Note (typo, preserve as-is):** `Proxy/PropertyGridPtoxy.cs` — filename and (per the pattern) class name both spell "Proxy" as "**Ptoxy**". This is a real, shipped file/class name; do not silently rename it if you extend it, and expect `using`/reference statements elsewhere in the codebase to reproduce the same typo. |
| `Environments/TUITestEnvironment.cs`, `Environments/Schema/UITestEnvironmentSchema.cs` (+ `.xsd`), `Config/UITestEnvironment.xml` (module level) | Declarative UI-test-run configuration — auto-run behavior, window-close behavior, exception/interference handling groups — loaded from an XML file validated against `UITestEnvironmentSchema.xsd`. |
| `Environments/ModuleRestartRule.cs`, `TestContextStatus.cs`, `TestOutcomeInfo.cs`, `UITestMonitor.cs` | Cross-test state: whether the app under test needs restarting before the next test, the running test's pass/fail history, and a background monitor (used to detect unexpected dialogs/crashes mid-run). |
| `AutoRun/ApplicationLogin.cs`, `AutoRun/AppStartInfoSchema.cs`, `AutoRun/StartInfoItem.cs` | Automates the CATOS login screen so each UI test can start from a known, authenticated state. |
| `UIMaps/<Feature>/UIMap/*.uitest`, `*.Designer.cs`, `*.cs` | Auto-generated Coded UI **UI Map** recordings (per feature area — Application, Login, ContextMenu, CodePopup, Performance, UIControl) produced by the Visual Studio Coded UI Test Builder. These are generated artifacts — hand-edit only the non-`.Designer.cs` partner file, never the `.Designer.cs`/`.uitest` pair, or the next re-record will silently discard your edits. |
| `OrderedTest1.orderedtest`, `Environments/OderedTest/B0001.orderedtest` | MSTest **Ordered Test** definitions, used where UI test cases must run in a fixed sequence (e.g. login → create → verify → delete) rather than MSTest's default arbitrary ordering. **Note:** the folder name `OderedTest` (missing the "r" in "Ordered") is the real, shipped folder name. |

### 11.3 Layer 3 — Module-level test projects (worked example: `Tsb-Fontos-Sample`)

Per the Sample repo's solution structure, a business module contributes three parallel test projects next to its production code:

```
Tsb.Fontos.Sample.Client      <- production WinForms module (Views/Controllers)
Tsb.Fontos.Sample.Common      <- production shared Item/Param/constants
Tsb.Fontos.Sample.Service     <- production Service/Dao/Map (business + data access)
Tsb.Fontos.Sample.Test        <- unit tests for Service/Dao (derives BaseServiceTest / BaseDaoTest)
Tsb.Fontos.Sample.TestDB      <- DB-schema-focused unit tests (derives BaseDaoTest, exercises real DB)
Tsb.Fontos.Sample.UITest      <- Coded UI end-to-end tests (derives BaseCatosUITest)
```

The `Tsb.Fontos.Sample.UITest` project organizes its Coded UI tests **by feature area**, one subfolder and one `*SampleUITest.cs` class per sample screen family (mirrors the module's own screen groupings, not a 1:1 mapping to individual FormTemplate archetypes):

```
Sample/BearthPlanning/BPSampleUITest.cs        <- Note: "Bearth" (not "Berth") is the real, shipped folder/class spelling
Sample/OperationManagement/OMSampleUITest.cs
Sample/Sample/SampleUITest.cs
Sample/ShipPlanning/SPSampleUITest.cs
Sample/TerminalMonitoring/TMSampleUITest.cs
Sample/YardPlanning/YPSampleUITest.cs
Common/Dialog/SLDialogHelper.cs, Common/Menu/SLMenuHelper.cs   <- module-local thin wrappers over the Cm.UITest proxies
Config/Login/ApplicationStartInfo.xml, Config/UITestEnvironment.xml  <- module-local test-run configuration
```

Each `*SampleUITest.cs` class derives from `BaseCatosUITest`, is decorated `[CodedUITest]`, and drives the module's own generated `UIMaps/<Feature>/UIMap/*SampleUIMap.cs` recordings via the shared `Cm.UITest` proxies described above.

### 11.4 Practical testing workflow for a new module

1. Add three projects to your module's solution using the Sample project names as a template: `<Module>.Test`, optionally `<Module>.TestDB`, and `<Module>.UITest`.
2. For business-logic/DAO unit tests: derive from `BaseServiceTest` or `BaseDaoTest`, call `BaseServiceTest.ConfigLoad()` once per test class (typically in `[TestInitialize]` or `[ClassInitialize]`) before resolving any Spring-managed object via `ObjectBuilder`, and resolve the object under test through its registered Object ID rather than `new`-ing it directly, matching production bootstrap.
3. Where a test needs a logged-in ambient user context (e.g. testing controller logic that reads `AppEnv.UserInfo`), derive from `BaseControllerTest` instead, which seeds a fake admin session automatically.
4. Reach for `TAssert` (`Tsb.Catos.Cm.Test.Util.TAssert`) before writing custom comparison/validation assertion code — it is the shared CATOS assertion library and likely already has what you need.
5. Use `UnitTestService.ExistsDbSchema()` / `ValidDbColumnItemType<T>()` as guard assertions in DAO-layer tests that depend on a particular DB schema being present, to fail fast with a clear message instead of a downstream iBATIS mapping exception.
6. For UI regression tests, derive your module's UI test class from `BaseCatosUITest`, construct it with your module's `programCode`, and drive the screen through the shared `Tsb.Catos.Cm.UITest.Proxy.*` classes and your own generated UIMap rather than raw `Microsoft.VisualStudio.TestTools.UITesting` calls, so behavior (login handling, restart policy, exception monitoring) stays consistent with the rest of CATOS. Given the Coded UI deprecation note in 11.0 above, scope new UI test investment carefully and confirm the current team policy before committing significant effort here.
7. Never embed real production connection strings, staff credentials, or customer cargo/vessel data in test fixtures or `TestData/*.csv`/`*.xml` files (e.g. `Sample/Sample/TestData/SampleSetVVDTestData.csv`, `UIMaps/Login/TestData/AccountData.xml`) — use synthetic identifiers (`TMNL-TEST`, `STAFF-TEST`, masked passwords) exactly as the framework's own `BaseControllerTest` does.

---

## 12. Gaps Identified & Recommendations

This guide was built by cross-referencing the official FONTOS/CATOS documentation set against live source in `Tsb-Fontos-Framework`, `Tsb-Catos-Cm`, `Tsb-Global-Libraries`, `Tsb-Catos-Om`, `Tsb-Fontos-Sample`, and `Tsb-Fontos-Tools-Se-MultiHeader`. The process surfaced several genuine open questions that could not be resolved from the material available in this pass. They are consolidated here so a follow-up reviewer knows exactly what to check, rather than re-discovering them from scratch.

1. **"MultiHeader" tool name vs. observed schema capability.** No grouped/multi-row-header column construct was found in any `<GridColumnSchema>` XML examined, including samples produced by `Tsb-Fontos-Tools-Se-MultiHeader` itself. **Recommendation:** read the MultiHeader tool's own source (not just its output schema files) to confirm whether grouped headers are (a) expressed through a schema construct not present in the sample files reviewed, (b) achieved at the `SpreadGridHelper`/handler level in code rather than declaratively in XML, or (c) a planned/aspirational capability. Update Section 5/6 once confirmed.
2. **Unverified intermediate base classes.** Several classes referenced in the documentation and inferred from naming convention — `BaseGridView`, `BaseBizView`, `BaseTemplateGridInterface`, `BaseTemplateBizInterface` — were not present in the staged file set and their exact contracts are therefore not independently source-verified in Sections 4–5. **Recommendation:** stage and review these directly from `Tsb-Fontos-Framework`'s `FormTemplate`/`Interface` folders in a follow-up pass.
3. **`ValidateAttribute` semantics.** Referenced in framework-level discussion of input validation, but its full validation semantics (which rules it enforces, how failures propagate) were not confirmed end-to-end against source. **Recommendation:** verify directly against `Tsb.Fontos.Core`'s attribute implementation before relying on it as the sole input-validation mechanism for new Item/Param classes; combine with explicit manual validation in Service-layer code in the meantime (see Section 9's validation guidance).
4. **`BaseTestAssembly.cs` purpose unconfirmed.** This class (Section 11.1) is empty in the reviewed source with no usages found. **Recommendation:** confirm its intended role (assembly-level test hook vs. incomplete work-in-progress) with the framework team before building on it.
5. **Recurring spelling/naming inconsistencies in shipped APIs.** This guide documents, and deliberately preserves rather than "corrects", a number of real inconsistencies in the shipped codebase: `GetDefalutParam()`, `EnableOnlyDefneStateMode` vs. `EnableOnlyDefineStateMode`, `codeDeailResult`, `GetCurosrWithDefaultCursor()`, `ExcuteApplyRegisterViewFilterCondition()`, `SetActiveInex()`, `ControlNammingRuleInfo.xml`, `MenuGridSummay`, `VERIFED_CNTR`, `DoBeforTest()`/`DoAfterTest()`, `Proxy/PropertyGridPtoxy.cs`, `Sample/BearthPlanning` (vs. "Berth"), and `Environments/OderedTest`. **Recommendation:** treat this list as a starting inventory, not exhaustive; consider either a coordinated rename pass (high risk/low reward this late in the product's life, and would break binary/serialized compatibility for XML-driven lookups) or, more practically, a living internal glossary that documents each one so new developers stop mis-typing the *correct* spelling when calling these members.
6. **ODP.NET driver version pinning.** Section 9 documents the ODP.NET Managed Driver setup pattern (`providers.config`, `App.config`, `tnsnames.ora`) but the exact version constraint pinned in `Tsb-Global-Libraries` was not independently reconfirmed in this pass. **Recommendation:** confirm the pinned `Oracle.ManagedDataAccess` package version in `Tsb-Global-Libraries` before upgrading Oracle client tooling on a new developer machine, to avoid `App.config` binding-redirect mismatches.
7. **Schema dialect coexistence.** Both the current `GridColumnSchema` XML dialect and a legacy `NewDataSet`/`ColumnHeader` DataSet-serialized dialect were found in live schema files (Section 5). **Recommendation:** when creating a *new* module, always author schemas in the current `GridColumnSchema` dialect; treat any `NewDataSet`-style file encountered in an existing module as a candidate for future migration, not a pattern to copy.
8. **Documentation currency.** The core tutorial and TSpreadGrid guides are dated 2016-04 and 2014-04 respectively; other guides range from 2010 to 2016. Given the multi-year gap to the current date, **this guide's source-code cross-checks should be treated as more current than the PDF text wherever the two disagree**, and the PDF set itself is a candidate for a refresh cycle.
9. **`GridPolicyInfo.xml` drift.** Shipped sample policy files contain keys the current `GridPolicyInfo` does not read (`PreserveOnBindListAsGridDataSource`, `EditableDateOperand`), the declared constant `UseSummaryDB` is not loaded, and the sample's comment on the `HeaderSort` default (`Normal`) disagrees with the code (`NullLast`). **Recommendation:** confirm with the framework team whether these keys were removed or moved elsewhere, and align the sample/policy files with §6.7.
10. **New Summary API not yet released.** `SpreadGridHelper.GetSourceItemsBySummaryCell` (§6.6) and the non-`IList` `DataSource` guard in `TSpreadGrid` mouse/key handlers were added in this revision but have not been built or regression-tested at the time of writing. **Recommendation:** verify against the released `Tsb.Fontos.Win.dll` and complete internal review before modules depend on them.

**General recommendation:** re-run the source cross-check in this guide against a newer snapshot of all six repositories roughly once per major FONTOS/CATOS release cycle, since this document reflects a single point-in-time review and the codebase — as the typo inventory above shows — evolves without always updating the corresponding documentation.

---

## Appendix A: Reference File Locations

Quick-reference index of the key files this guide draws on, by repository. Paths are relative to each repository's root as connected in this review (`Tsb-Fontos-Framework`, `Tsb-Catos-Cm`, `Tsb-Global-Libraries`, `Tsb-Catos-Om`, `Tsb-Fontos-Sample`, `Tsb-Fontos-Tools-Se-MultiHeader`).

| Area | Repository | Path |
|---|---|---|
| FormTemplate base interfaces | Tsb-Fontos-Framework | `Tsb.Fontos.Win\FormTemplate\Interface\Base*TemplateInterface.cs` |
| FormTemplate base View/Controller classes | Tsb-Fontos-Framework | `Tsb.Fontos.Win\FormTemplate\Base*View.cs`, `Base*Controller.cs` |
| Grid Schema engine | Tsb-Fontos-Framework | `Tsb.Fontos.Win\Grid\Schema\GridBaseSchema.cs` |
| Grid policy settings | Tsb-Fontos-Framework | `Tsb.Fontos.Core\Environments\GridPolicyInfo.cs` (sample file: Tsb-Fontos-Sample `Tsb.Fontos.Sample.Config\Environment\FontosSample\GridPolicyInfo.xml`) |
| Grid context menu | Tsb-Fontos-Framework / Tsb-Catos-Cm | `Tsb.Fontos.Win\Menu\ContextMenu\ContextMenuHelper.cs`, `Command\ContextMenuInvoker.cs`; `Tsb.Catos.Cm.Win\FormTemplate\CTBaseSingleGridView.cs`, `CTBaseMultiGridView.cs` (`SpreadGrid_GridMouseRightClick`) |
| Summary engine / source-item lookup | Tsb-Fontos-Framework | `Tsb.Fontos.Win\Grid\Spread\SpreadGridHelper.cs` (`ApplySummary`, `GetSourceItemsBySummaryCell`), `SpreadSummaryHandler.cs`, `Summary\GeneralSpreadSummaryPreviewView.cs`, `Summary\GeneralSpreadSummayDataDetailView.cs` |
| Summary grid context menu example | Tsb-Fontos-Sample | `Tsb.Fontos.Sample.Client\Grids\TSpreadGridView.cs` (region *METHOD AREA (Summary Grid Context Menu)*) |
| Menu system core | Tsb-Fontos-Framework | `Tsb.Fontos.Win\Menu\MainMenuHandler.cs`, `Menu\Item\TMenuItem.cs`, `Menu\Schema\MenuBaseSchema.cs` |
| Docking core | Tsb-Fontos-Framework | `Tsb.Fontos.Win\Docking\DockStateManager.cs`, `TDockContainer.cs` |
| ViewState core | Tsb-Fontos-Framework | `Tsb.Fontos.Win\ViewState\ViewStateManager.cs`, `ViewStateDefineHandler.cs` |
| Graphical Drawing engine | Tsb-Fontos-Framework | `Tsb.Fontos.Win.Drawing\` (Draw/Geometry/Drawable hierarchy, `DrawArea`, `LayerDrawArea`) |
| Network / messaging layer | Tsb-Fontos-Framework | `Tsb.Fontos.Net\` (`IInterfaceHandler`, `INetworkApplication`, `ITransportFilter`, etc.) |
| Framework-level test base classes | Tsb-Fontos-Framework | `Tsb.Fontos.Test\TestTemplate\Base*.cs`, `Handler\`, `Output\`, `UITest\` |
| CATOS shared FormTemplate extensions | Tsb-Catos-Cm | `Tsb.Catos.Cm.Core\...\CTBaseSingleGridView.cs`, `CTBaseSingleGridController.cs`, matching `*ViewInterface.cs` |
| Container Editor | Tsb-Catos-Cm | `Tsb.Catos.Cm.ContainerEditor\` (`BaseContainerDataModel`, `CECodeHandler`, rule set editors) |
| ViewState admin / DB-backed config | Tsb-Catos-Cm | `Tsb.Catos.Cm.ViewState\` |
| Working Set (docking layout persistence) | Tsb-Catos-Cm | `Tsb.Catos.Cm.WorkingSet\` (`DockingStateHelper`) |
| CATOS shared unit-test utilities | Tsb-Catos-Cm | `Tsb.Catos.Cm.Test\Util\TAssert.cs`, `Service\UnitTestService.cs` |
| CATOS shared Coded UI test base + proxies | Tsb-Catos-Cm | `Tsb.Catos.Cm.UITest\TestTemplate\BaseCatosUITest.cs`, `Proxy\*.cs` |
| Third-party library references (Spread/FarPoint, Spring.NET, iBATIS.NET, ODP.NET, DevExpress) | Tsb-Global-Libraries | repository root — see each library's own subfolder for the pinned version |
| Worked business-module example | Tsb-Fontos-Sample | `Tsb.Fontos.Sample.Client\` (SingleGrid/MultiGrid/Detail/SingleDraw/MultiDraw screens), `Tsb.Fontos.Sample.Service\` (Item/Param/Service/Dao/Map) |
| Module bootstrap entry point | Tsb-Fontos-Sample | `Tsb.Fontos.Sample.Client\Program.cs`, `App.config`, `CatosConfig.xml` |
| Module-level test projects (worked example) | Tsb-Fontos-Sample | `Tsb.Fontos.Sample.Test\`, `Tsb.Fontos.Sample.TestDB\`, `Tsb.Fontos.Sample.UITest\` |
| Grid Schema XML samples | Tsb-Fontos-Sample / Tsb-Fontos-Tools-Se-MultiHeader | `*\Environments\**\*.xml` (Sample), tool-generated schema output (MultiHeader tool) |
| Grid Schema authoring tool | Tsb-Fontos-Tools-Se-MultiHeader | repository root (Spread Editor / Grid Schema authoring tool project) |
| CATOS business module (production-scale reference) | Tsb-Catos-Om | full CATOS Operation Management module — largest real-world example of every pattern in this guide applied at scale |
| Object ID Naming Rule source | *(attached document)* | `FONTOS1.0STSPObject_ID_Naming_RuleCATOS_DG_v1.1_Eng_201003.xls` — see Section 10.2 for the reproduced table |

**Note:** exact subfolder names vary slightly by module generation; where this table gives a folder rather than a single file, use `device_list_dir`/your IDE's solution explorer to locate the current file within it, since minor reorganizations happen between releases without documentation updates (see Section 12, item 8).

---

## Appendix B: Source Documents Used

All documents below were supplied as attachments and read in full (paginated) as part of compiling this guide. Version/date strings are taken verbatim from each document's own filename and title page.

| # | Document | Version | Date |
|---|---|---|---|
| 1 | FONTOS Application Development Tutorial (FONTOS-1.0-DVGD) | v3.7 | 2016-04 |
| 2 | FONTOS Framework — TSpreadGrid | v1.5 | 2014-04 |
| 3 | FONTOS Framework — MenuLoad | v1.1 | 2014-11-07 |
| 4 | FONTOS Framework — ViewDocking | v1.0 | 2016-11-08 |
| 5 | FONTOS-1.0-DVGD — View State Guide | v1.0 | 2013-08 |
| 6 | FONTOS-1.0-DVGD — Container Editor Guide | v1.2 | 2015-01-15 |
| 7 | FONTOS-1.0-DVGD — Development Guide for Graphical Drawing | v0.4 | 2012-01-12 |
| 8 | FONTOS-1.0-DVGD — FONTOS Network | v1.0 | 2016-04-26 |
| 9 | FONTOS-1.0-DVGD — How to Create a New Solution with FONTOS | v1.1 | 2016-04 |
| 10 | FONTOS-1.0-DVGD — How to Use ODP.NET Managed Driver | v1.0 | 2016-04 |
| 11 | FONTOS-1.0-DVGD — Spread Editor Guide | v1.3 | 2014-04 |
| 12 | FONTOS-1.0-STSP — Object ID Naming Rule (CATOS DG) | v1.1 | 2010-03 |
| 13 | FONTOS-1.0-STSP — UI Standards and Guidelines | v1.1 | 2010-12 |

Cross-referenced against live source in the following repositories (as connected during this review): `Tsb-Fontos-Framework`, `Tsb-Catos-Cm`, `Tsb-Global-Libraries`, `Tsb-Catos-Om`, `Tsb-Fontos-Sample`, `Tsb-Fontos-Tools-Se-MultiHeader`.
