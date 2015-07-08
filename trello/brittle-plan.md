

##註冊場景
2015/07/06

為了避免在遊戲自定 App 的 cpp 檔中引入所有 scene 標頭檔的繁瑣場面，Brittle 改用 static 變數來註冊 scene。
```
#include "MenuScene.h"

static const SceneRegister s_scereg( "Menu" );
```
不過我覺得這邊應該不是 Register 而是別的用詞吧～


##Brittle App 啟動流程
2015/07/06

1. DidFinishLaunching() 呼叫 OnApplicationLaunch() 。在這裡面，App 層可以建立**不需載入任何資源**的物件。
	App 層必須要指定 initial scene（by name）。如果沒有指定，接下來 Brittle 會顯示一個內建的簡單 scene，畫面上顯示「Error: Initial Scene NOT Specified!」。
	App 層可以指定 splash action（splash 方案一）。
	App 層在此可以讀取 Preferences 來決定音效、音樂、語系等偏好設定。
2. 全域資源釋放。目前不確定對於 Cocos2d-x 3 而言，有沒有可能從 Director 把 scene 完全卸除？如果不行的話，就必須在進入 initial scene 之後清除資源，這表示 initial scene 不能在 Create() 函式中建立內部結構——不過反正 initial scene 本來就不適合在 Create() 之中建立結構，或許沒差？
	App 執行中要重新啟動的時候，是回到這一點來（如果有方便的手法可以重新啟動整個 App 的話，或許沒必要回到這一點？）。
3. 進入 initial scene。為了確保 App 啟動時正確的資源載入流程，initial scene 請勿在 Create() 函式中建立結構，除了下面提到的例外：

	* 指定 loading action。這也可以拿來當作 splash action（splash 方案二）。

	BrittleApp（或 director 之類）在開始運行 initial scene 的時候，會按照下述的順序：
	
	1. 執行 splash action（App 函式）或 loading action（Scene 函式）。
	2. 載入框架所需的語言文字資源。
	3. 開始 initial scene 自己要展現的內容。

###反思
splash action 或 loading action，或許應該用 idiom 的方式，讓開發者自行控制才對？不然 scene 就必須要有辦法得到前置動作結束的事件，會因此增加不必要的複雜度。


##Brittle App 專案結構
2015/06/11

假設是把 Client、和 Resources 放在一起：

```
assets
assets.ios
assets.win32
proj.eclipse
proj.vc
proj.xcode
res.android
src （裡面還有 android、ios、win32 等子目錄）
tools
work（Win32 工作目錄，帶有可執行所需的檔案，給企劃及美術即時測試之用）
```

Server 能放在一起嗎？但上面三個專案其實編出來是同一個東西（只是不同平台），而把 Server 放在一起顯然不太協調，release 流程也不太有關聯。兩邊會共用的只有一部份遊戲設定資料……而且如果不是硬要搞「一個旗標切換到單機版」，其實有很多資料根本就不該放在 client 側～

##WebView 的插件
2015/06/05

Cocos2d-x 3 沒有提供 Win32 的 WebView 實作。
在 Win32 上，Brittle 會提供一個 WebView 的實作，預設是導向一個 Win32 靜態 Window。不過你也可以選擇加上一個插件（必須要有 ActiveX 或 MFC 才能編譯？），就會導向一個真正的 WebView 視窗。

不然就是讓 Cocos2d-x3 的 brittle-v3 支線帶有 Win32 WebView 的實作，並規定必須使用 Visual Studio Community 來編譯。

##非同步載入資源的方式
2015/06/05

1. Scene 先載入 loading 時簡單畫面所需的資源。
2. 將 Build() 工作交給 background 執行緒。
3. Build() 函式一開始要先建立 OpenGL shared context 。這個 context 會在 Build() 結束的時候釋放掉。
4. 整個 Scene 所需的子元件在這個函式裡面建立出來。其他元件的建立流程維持原樣，也就是說，只有 Scene 才需要決定是否要非同步載入。


> Written with [StackEdit](https://stackedit.io/).