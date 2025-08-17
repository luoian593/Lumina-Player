# 🎵 Lumina Player 1.0项目详细技术文档

---

## 📚 项目简介

**Lumina Player1.0** 是一款功能完整、结构清晰、性能稳定的 Android 本地音乐播放器应用。它从零构建，不依赖任何第三方 UI 框架，完全使用 Android 原生组件和 Java 语言开发，适合作为 Android 学习者、初/中级开发者的实战项目参考。

该项目不仅实现了音乐播放的核心功能，还深入优化了用户体验，包括：**快速启动、缓存机制、封面加载、歌词读取、沉浸式播放、无缝跳转、内存安全**等。代码结构高度模块化，注释完整，逻辑清晰，具备极高的可维护性和扩展性。

---

## 🌟 核心特性

| 特性 | 说明 |
|------|------|
| ✅ 极速启动 | 优先加载缓存，秒开音乐列表，提升用户体验 |
| ✅ 内存安全 | 封面按需异步加载，避免 OOM（内存溢出） |
| ✅ 无缝播放 | 全局共享 `MediaPlayer`，跳转不中断、不重复播放 |
| ✅ 沉浸体验 | 播放页自动隐藏 `Toolbar`，全屏播放 |
| ✅ 自动缓存 | 扫描结果持久化为 JSON，减少重复扫描 |
| ✅ 权限兼容 | 支持 Android 11+ 的 `MANAGE_EXTERNAL_STORAGE` |
| ✅ UI 简洁 | 扁平化设计，无冗余元素，操作直观 |

---

## 📂 项目文件目录结构（完整）

```
Lumina Player/
├── app/
│   ├── src/main/
│   │   ├── java/github/lumina/player/
│   │   │   ├── MainActivity.java          # 主界面：音乐列表、底部播放栏、权限管理
│   │   │   ├── PlayActivity.java          # 播放页：封面、歌词、控制、进度条
│   │   │   ├── Music.java                 # 音乐数据模型，实现 Parcelable
│   │   │   ├── MusicScanner.java          # 扫描本地音乐文件（MediaStore）
│   │   │   ├── MusicAdapter.java          # RecyclerView 适配器，按需加载封面
│   │   │   └── MusicPlayerManager.java    # 播放器单例管理（共享 MediaPlayer）
│   │   │
│   │   ├── res/
│   │   │   ├── layout/
│   │   │   │   ├── activity_main.xml      # 主界面布局：RecyclerView + BottomBar + Toolbar
│   │   │   │   ├── activity_play.xml      # 播放页布局：封面 + 歌词 + SeekBar + 控制按钮
│   │   │   │   ├── item_music.xml         # 列表项布局：封面 + 标题 + 艺术家 + 时长
│   │   │   │   └── toolbar_main.xml       # 自定义 Toolbar 布局（独立文件）
│   │   │   │
│   │   │   ├── values/
│   │   │   │   ├── strings.xml            # 字符串资源（如 "Lumina Player"）
│   │   │   │   ├── colors.xml             # 颜色定义（如 #F1F1F1, #F5F5F5）
│   │   │   │   └── themes.xml             # 主题配置（NoActionBar）
│   │   │   │
│   │   │   ├── drawable/
│   │   │   │   ├── ic_music_note.xml      # 默认专辑封面（Vector）
│   │   │   │   ├── ic_play_arrow.xml      # 播放图标
│   │   │   │   ├── ic_pause.xml           # 暂停图标
│   │   │   │   ├── ic_refresh.xml         # 刷新图标
│   │   │   │   ├── ic_skip_next.xml       # 下一首
│   │   │   │   ├── ic_skip_previous.xml   # 上一首
│   │   │   │   ├── ic_shuffle.xml         # 随机播放
│   │   │   │   ├── ic_repeat.xml          # 循环播放
│   │   │   │   ├── ic_volume_up.xml       # 音量
│   │   │   │   ├── ic_queue_music.xml     # 播放列表
│   │   │   │   └── ic_more_vert.xml       # 更多
│   │   │   │
│   │   │   └── mipmap-xxhdpi/
│   │   │       └── ic_launcher.png        # 应用图标
│   │   │
│   │   └── AndroidManifest.xml            # 应用配置：权限、Activity 声明
│   │
│   └── build.gradle.kts                   # 构建脚本（Gradle）
│
└── gradle.properties                      # Gradle 配置（如 org.gradle.jvmargs）
```

---

## 🧩 核心文件功能详解

### 1. `MainActivity.java`
- **作用**：主界面控制器，负责音乐列表展示、权限申请、缓存管理、底部播放栏控制。
- **核心功能**：
  - 检查并申请 `MANAGE_EXTERNAL_STORAGE` 权限
  - 优先从 `music_cache.json` 加载缓存音乐
  - 后台扫描 `MediaStore`，更新音乐列表
  - 初始化 `MusicAdapter`，绑定 `RecyclerView`
  - 管理底部播放栏：显示当前歌曲、播放/暂停、点击跳转
  - 设置自定义 `Toolbar`：左文字“Lumina Player”，右刷新按钮

### 2. `PlayActivity.java`
- **作用**：播放页控制器，提供全屏沉浸式播放体验。
- **核心功能**：
  - 接收 `MainActivity` 传递的 `Music` 对象
  - 显示专辑封面（从 `path` 读取内嵌图片）
  - 显示歌词（读取同名 `.lrc` 文件）
  - 播放/暂停、上一首、下一首（功能待扩展）
  - `SeekBar` 进度条：可拖动调整播放位置
  - 实时更新当前时间和总时长
  - 自动隐藏 `Toolbar`，实现沉浸式 UI
  - 使用 `MusicPlayerManager` 共享播放器，避免重复播放

### 3. `Music.java`
- **作用**：音乐数据模型，封装单首音乐的元数据。
- **字段**：
  - `title`：歌曲标题
  - `artist`：艺术家
  - `album`：专辑
  - `duration`：时长（格式化为 MM:SS）
  - `path`：文件绝对路径
- **特性**：实现 `Parcelable` 接口，支持跨 Activity 传递。

### 4. `MusicScanner.java`
- **作用**：扫描设备上的音频文件（`.mp3`, `.flac`, `.m4a` 等）。
- **实现方式**：
  - 使用 `ContentResolver.query()` 查询 `MediaStore.Audio.Media.EXTERNAL_CONTENT_URI`
  - 筛选条件：`IS_MUSIC != 0` 且 `DURATION >= 30000`（30秒以上）
  - 提取：标题、艺术家、专辑、时长、路径
  - **不提取封面**，避免内存爆炸
- **线程安全**：在子线程中执行，通过 `runOnUiThread` 更新 UI

### 5. `MusicAdapter.java`
- **作用**：`RecyclerView` 适配器，管理音乐列表的展示。
- **核心功能**：
  - 绑定 `item_music.xml` 布局
  - 显示：封面（`ImageView`）、标题（`TextView`）、艺术家、时长
  - **按需异步加载封面**：使用 `MediaMetadataRetriever` 从 `path` 读取
  - **压缩 Bitmap**：使用 `inSampleSize` 防止 OOM
  - 点击事件：调用 `OnItemClickListener` 播放音乐
  - 优化：`ViewHolder` 模式，避免重复 `findViewById`

### 6. `MusicPlayerManager.java`
- **作用**：播放器单例管理类，确保全局只有一个 `MediaPlayer` 实例。
- **核心功能**：
  - 单例模式：`private static MusicPlayerManager instance`
  - 提供 `getMediaPlayer()` 获取播放器
  - 管理当前播放的 `Music` 对象
  - 提供 `isPlaying()` 状态查询
  - **不主动释放**，由 `MainActivity` 在 `onDestroy` 中统一管理

### 7. `AndroidManifest.xml`
- **作用**：应用配置文件。
- **关键配置**：
  - `<uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE" />`
  - `<application android:theme="@style/AppTheme.NoActionBar">`
  - 声明 `MainActivity`（LAUNCHER）和 `PlayActivity`

### 8. `build.gradle.kts`
- **作用**：Gradle 构建脚本。
- **依赖**：
  - `implementation 'androidx.appcompat:appcompat:1.6.1'`
  - `implementation 'com.google.code.gson:gson:2.10.1'`
  - `implementation 'androidx.recyclerview:recyclerview:1.3.2'`

---

## 🎨 UI 与交互设计

### 1. 主界面 (`activity_main.xml`)
- **顶部**：`Toolbar`，背景 `#F1F1F1`，左 `TextView` 显示 "Lumina Player"，右 `ImageView` 为刷新按钮
- **中部**：`RecyclerView`，垂直列表，每项高度适中
- **底部**：`bottom_player_bar`，固定在底部，显示当前播放歌曲的封面、标题、播放/暂停按钮
- **加载**：`ProgressBar` 居中，扫描完成后隐藏

### 2. 播放页 (`activity_play.xml`)
- **标题区**：歌曲名、艺术家
- **封面区**：大尺寸正方形 `ImageView`，居中，占屏幕 1/3 高度
- **歌词区**：`TextView`，显示 `.lrc` 内容或“暂无歌词” 暂不支持解析时间流，歌词文件不会同步播放显示对应部分只会显示开头一部分 暂无歌词页 目前歌词仅在封面下方显示一两句
- **进度区**：`SeekBar` + 当前时间 + 总时长
- **控制区**：上一首、播放/暂停、下一首（大按钮）
- **功能区**：随机、循环、音量、播放列表、更多（小图标）

---

## 🚀 技术栈与架构

- **语言**：Java
- **UI 框架**：Android 原生（AppCompat）
- **数据存储**：`SharedPreferences`（未来可扩展）+ `File`（JSON 缓存）
- **网络**：无（纯本地应用）
- **播放引擎**：`MediaPlayer`
- **异步处理**：`Thread` + `Handler`
- **序列化**：`Gson`（JSON 缓存）
- **权限管理**：`ActivityCompat.requestPermissions` + `Settings.ACTION_MANAGE_ALL_FILES_ACCESS_PERMISSION`

---

## 🛠️ 开发与构建

- **IDE**：Android Studio / AndroidIDE
- **minSdkVersion**：21
- **targetSdkVersion**：34
- **编译**：`./gradlew build`
- **安装**：`./gradlew installDebug`

---

## 📈 未来扩展方向

1. **歌词同步滚动**：解析 `.lrc` 时间戳，同步高亮当前行，提前音乐文件内嵌的歌词文件
2. **播放队列**：支持上，下一首、随机播放、循环模式
3. **通知栏控制**：锁屏状态下控制播放
4. **深色模式**：跟随系统主题切换
5. **文件夹扫描**：用户自定义扫描路径
6. **均衡器**：音效调节
7. **专辑页**：按专辑分类浏览
8.界面美化 目前界面并不特别精美极简
9.适配横屏，目前不适配
---

# 🎵 Lumina Player 1.1 - 功能增强、体验升级的现代音乐播放器

> **Lumina Player v1.1** 在 v1.0 基础上实现了重大功能突破：**全屏歌词页、内嵌歌词解析、外挂歌词加载、歌词高亮居中滚动、手势跳转、封面点击跳转、Parcelable 跨页通信、RecyclerView 动态刷新**等。  
> 本版本标志着项目从“基础播放器”正式迈向“沉浸式音乐体验”阶段，用户体验大幅提升。

---

## 📚 项目简介

**Lumina Player** 是一款功能完整、结构清晰、性能稳定的 Android 本地音乐播放器应用。它从零构建，不依赖任何第三方 UI 框架，完全使用 Android 原生组件和 Java 语言开发，适合作为 Android 学习者、初/中级开发者的实战项目参考。

v1.1 版本在 v1.0 的基础上，**新增了全屏歌词功能模块**，实现了从“能听”到“能看”的跨越。用户可点击播放页封面或左滑手势进入全屏歌词界面，享受**双语歌词、高亮居中、自动同步、流畅滚动**的沉浸式体验。

代码结构高度模块化，注释完整，逻辑清晰，具备极高的可维护性和扩展性。

---

## 🌟 核心特性（v1.1 新增 ✅）

| 特性 | 说明 |
|------|------|
| ✅ 极速启动 | 优先加载缓存，秒开音乐列表，提升用户体验 |
| ✅ 内存安全 | 封面按需异步加载，避免 OOM（内存溢出） |
| ✅ 无缝播放 | 全局共享 `MediaPlayer`，跳转不中断、不重复播放 |
| ✅ 沉浸体验 | 播放页自动隐藏 `Toolbar`，全屏播放 |
| ✅ 自动缓存 | 扫描结果持久化为 JSON，减少重复扫描 |
| ✅ 权限兼容 | 支持 Android 11+ 的 `MANAGE_EXTERNAL_STORAGE` |
| ✅ UI 简洁 | 扁平化设计，无冗余元素，操作直观 |
| ✅ **全屏歌词页** | 点击封面或左滑进入 `LyricsActivity`，沉浸式查看歌词 |
| ✅ **内嵌歌词解析** | 支持从 `.flac`、`.mp3` 文件中提取 `Vorbis Comment` 或 `ID3` 内嵌歌词 |
| ✅ **外挂歌词加载** | 自动查找同名 `.lrc` 文件并加载，支持双语歌词 |
| ✅ **歌词高亮居中** | 当前播放歌词自动放大、加粗，并**居中显示在屏幕中央** |
| ✅ **实时同步滚动** | 与播放进度精确同步，误差小于 100ms |
| ✅ **手势跳转** | 左滑进入歌词页，右滑返回，交互自然流畅 |
| ✅ **Parcelable 通信** | 安全高效地在 `PlayActivity` 与 `LyricsActivity` 间传递 `Music` 对象 |

---

## 📂 项目文件目录结构（v1.1 完整版）

```
Lumina Player/
├── app/
│   ├── src/main/
│   │   ├── java/github/lumina/player/
│   │   │   ├── MainActivity.java          # 主界面：音乐列表、底部播放栏、权限管理
│   │   │   ├── PlayActivity.java          # 播放页：封面、歌词、控制、进度条
│   │   │   ├── LyricsActivity.java        # 🔥 新增：全屏歌词页，支持高亮、居中、自动滚动
│   │   │   ├── Music.java                 # 音乐数据模型，实现 Parcelable
│   │   │   ├── MusicScanner.java          # 扫描本地音乐文件（MediaStore）
│   │   │   ├── MusicAdapter.java          # RecyclerView 适配器，按需加载封面
│   │   │   └── MusicPlayerManager.java    # 播放器单例管理（共享 MediaPlayer）
│   │   │
│   │   ├── res/
│   │   │   ├── layout/
│   │   │   │   ├── activity_main.xml      # 主界面布局：RecyclerView + BottomBar + Toolbar
│   │   │   │   ├── activity_play.xml      # 播放页布局：封面 + 歌词 + SeekBar + 控制按钮
│   │   │   │   ├── activity_lyrics.xml    # 🔥 新增：全屏歌词页布局，RecyclerView 显示歌词
│   │   │   │   ├── item_music.xml         # 列表项布局：封面 + 标题 + 艺术家 + 时长
│   │   │   │   └── toolbar_main.xml       # 自定义 Toolbar 布局（独立文件）
│   │   │   │
│   │   │   ├── values/
│   │   │   │   ├── strings.xml            # 字符串资源（如 "Lumina Player"）
│   │   │   │   ├── colors.xml             # 颜色定义（如 #F1F1F1, #F5F5F5）
│   │   │   │   └── themes.xml             # 主题配置（NoActionBar）
│   │   │   │
│   │   │   ├── drawable/
│   │   │   │   ├── ic_music_note.xml      # 默认专辑封面（Vector）
│   │   │   │   ├── ic_play_arrow.xml      # 播放图标
│   │   │   │   ├── ic_pause.xml           # 暂停图标
│   │   │   │   ├── ic_refresh.xml         # 刷新图标
│   │   │   │   ├── ic_skip_next.xml       # 下一首
│   │   │   │   ├── ic_skip_previous.xml   # 上一首
│   │   │   │   ├── ic_shuffle.xml         # 随机播放
│   │   │   │   ├── ic_repeat.xml          # 循环播放
│   │   │   │   ├── ic_volume_up.xml       # 音量
│   │   │   │   ├── ic_queue_music.xml     # 播放列表
│   │   │   │   ├── ic_more_vert.xml       # 更多
│   │   │   │   └── ic_back.xml            # 返回箭头（用于歌词页）
│   │   │   │
│   │   │   └── mipmap-xxhdpi/
│   │   │       └── ic_launcher.png        # 应用图标
│   │   │
│   │   └── AndroidManifest.xml            # 应用配置：权限、Activity 声明
│   │
│   └── build.gradle.kts                   # 构建脚本（Gradle）
│
└── gradle.properties                      # Gradle 配置（如 org.gradle.jvmargs）
```

---

## 🧩 核心文件功能详解（v1.1 更新）

### 1. `MainActivity.java`
- **作用**：主界面控制器，负责音乐列表展示、权限申请、缓存管理、底部播放栏控制。
- **核心功能**：
  - 检查并申请 `MANAGE_EXTERNAL_STORAGE` 权限
  - 优先从 `music_cache.json` 加载缓存音乐
  - 后台扫描 `MediaStore`，更新音乐列表
  - 初始化 `MusicAdapter`，绑定 `RecyclerView`
  - 管理底部播放栏：显示当前歌曲、播放/暂停、点击跳转
  - 设置自定义 `Toolbar`：左文字“Lumina Player”，右刷新按钮

### 2. `PlayActivity.java`
- **作用**：播放页控制器，提供全屏沉浸式播放体验。
- **核心功能**：
  - 接收 `MainActivity` 传递的 `Music` 对象
  - 显示专辑封面（从 `path` 读取内嵌图片）
  - 显示歌词预览（读取同名 `.lrc` 文件前两行）
  - 播放/暂停、上一首、下一首（功能待扩展）
  - `SeekBar` 进度条：可拖动调整播放位置
  - 实时更新当前时间和总时长
  - 自动隐藏 `Toolbar`，实现沉浸式 UI
  - 使用 `MusicPlayerManager` 共享播放器，避免重复播放
  - **新增**：封面点击跳转至 `LyricsActivity`
  - **新增**：左滑手势进入 `LyricsActivity`

### 3. `LyricsActivity.java` 🔥（v1.1 新增）
- **作用**：全屏歌词页控制器，提供沉浸式歌词体验。
- **核心功能**：
  - 接收 `PlayActivity` 传递的 `Music` 对象（通过 `Parcelable`）
  - 解析并显示**内嵌歌词**（使用 `jaudiotagger` 提取 `Vorbis Comment` 或 `ID3` 中的 `LYRICS` 字段）
  - 解析并显示**外挂歌词**（查找 `.lrc` 文件）
  - 支持 `[mm:ss.xx]` 时间格式，精确到毫秒
  - 使用 `RecyclerView` 高效渲染歌词行
  - **当前歌词高亮**：字体放大、颜色加深、行距增加
  - **自动居中滚动**：使用 `scrollToPositionWithOffset()` 将当前歌词置于屏幕中央
  - **实时同步**：通过 `Handler` 每 100ms 检查播放进度，更新高亮行
  - **手势返回**：右滑或点击返回键退出
  - **UI 优化**：简洁布局，大字号，适合远距离观看

### 4. `Music.java`
- **作用**：音乐数据模型，封装单首音乐的元数据。
- **字段**：
  - `title`：歌曲标题
  - `artist`：艺术家
  - `album`：专辑
  - `duration`：时长（格式化为 MM:SS）
  - `path`：文件绝对路径
- **特性**：实现 `Parcelable` 接口，支持跨 Activity 传递。**不再需要实现 `Serializable`**。

### 5. `MusicScanner.java`
- **作用**：扫描设备上的音频文件（`.mp3`, `.flac`, `.m4a` 等）。
- **实现方式**：
  - 使用 `ContentResolver.query()` 查询 `MediaStore.Audio.Media.EXTERNAL_CONTENT_URI`
  - 筛选条件：`IS_MUSIC != 0` 且 `DURATION >= 30000`（30秒以上）
  - 提取：标题、艺术家、专辑、时长、路径
  - **不提取封面**，避免内存爆炸
- **线程安全**：在子线程中执行，通过 `runOnUiThread` 更新 UI

### 6. `MusicAdapter.java`
- **作用**：`RecyclerView` 适配器，管理音乐列表的展示。
- **核心功能**：
  - 绑定 `item_music.xml` 布局
  - 显示：封面（`ImageView`）、标题（`TextView`）、艺术家、时长
  - **按需异步加载封面**：使用 `MediaMetadataRetriever` 从 `path` 读取
  - **压缩 Bitmap**：使用 `inSampleSize` 防止 OOM
  - 点击事件：调用 `OnItemClickListener` 播放音乐
  - 优化：`ViewHolder` 模式，避免重复 `findViewById`

### 7. `MusicPlayerManager.java`
- **作用**：播放器单例管理类，确保全局只有一个 `MediaPlayer` 实例。
- **核心功能**：
  - 单例模式：`private static MusicPlayerManager instance`
  - 提供 `getMediaPlayer()` 获取播放器
  - 管理当前播放的 `Music` 对象
  - 提供 `isPlaying()` 状态查询
  - **不主动释放**，由 `MainActivity` 在 `onDestroy` 中统一管理

### 8. `AndroidManifest.xml`
- **作用**：应用配置文件。
- **关键配置**：
  - `<uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE" />`
  - `<application android:theme="@style/AppTheme.NoActionBar">`
  - 声明 `MainActivity`（LAUNCHER）、`PlayActivity`、`LyricsActivity`

### 9. `build.gradle.kts`
- **作用**：Gradle 构建脚本。
- **依赖**：
  - `implementation 'androidx.appcompat:appcompat:1.6.1'`
  - `implementation 'com.google.code.gson:gson:2.10.1'`
  - `implementation 'androidx.recyclerview:recyclerview:1.3.2'`
  - `implementation 'org.jaudiotagger:jaudiotagger:2.2.6'` 🔥 新增：用于解析内嵌歌词

---

## 🎨 UI 与交互设计（v1.1 更新）

### 1. 主界面 (`activity_main.xml`)
- **顶部**：`Toolbar`，背景 `#F1F1F1`，左 `TextView` 显示 "Lumina Player"，右 `ImageView` 为刷新按钮
- **中部**：`RecyclerView`，垂直列表，每项高度适中
- **底部**：`bottom_player_bar`，固定在底部，显示当前播放歌曲的封面、标题、播放/暂停按钮
- **加载**：`ProgressBar` 居中，扫描完成后隐藏

### 2. 播放页 (`activity_play.xml`)
- **标题区**：歌曲名、艺术家
- **封面区**：大尺寸正方形 `ImageView`，居中，占屏幕 1/3 高度，**可点击**
- **歌词区**：`TextView`，显示 `.lrc` 前两行或“暂无歌词”
- **进度区**：`SeekBar` + 当前时间 + 总时长
- **控制区**：上一首、播放/暂停、下一首（大按钮）
- **功能区**：随机、循环、音量、播放列表、更多（小图标）
- **交互**：左滑进入 `LyricsActivity`

### 3. 全屏歌词页 (`activity_lyrics.xml`) 🔥 新增
- **标题区**：歌曲名 + 艺术家，左对齐，小字号
- **歌词区**：`RecyclerView`，垂直居中，每行居中对齐
- **当前行样式**：20sp、黑色、加粗、行距增加
- **非当前行**：16sp、灰色、正常
- **返回方式**：左上角返回按钮 或 **右滑手势**
- **背景**：纯白，极简风格，专注歌词

---

## 🚀 技术栈与架构（v1.1 更新）

- **语言**：Java
- **UI 框架**：Android 原生（AppCompat）
- **数据存储**：`SharedPreferences` + `File`（JSON 缓存）
- **网络**：无（纯本地应用）
- **播放引擎**：`MediaPlayer`
- **异步处理**：`Thread` + `Handler` + `try-with-resources`
- **序列化**：`Gson`（JSON 缓存） + `Parcelable`（跨页通信）
- **歌词解析**：`jaudiotagger`（内嵌歌词）、`Scanner`（外挂歌词）
- **手势识别**：`GestureDetector`
- **动画**：`overridePendingTransition`（页面跳转）

---

## 🛠️ 开发与构建

- **IDE**：Android Studio / AndroidIDE
- **minSdkVersion**：21
- **targetSdkVersion**：34
- **编译**：`./gradlew build`
- **安装**：`./gradlew installDebug`

---

## 📈 未来扩展方向（v1.2+）

1. ✅ **歌词同步滚动**：已实现居中高亮，后续可优化平滑度
2. 🔜 **播放队列**：支持上/下一首、随机播放、循环模式
3. 🔜 **通知栏控制**：锁屏状态下控制播放
4. 🔜 **深色模式**：跟随系统主题切换
5. 🔜 **文件夹扫描**：用户自定义扫描路径
6. 🔜 **均衡器**：音效调节
7. 🔜 **专辑页**：按专辑分类浏览
8. 🎨 **界面美化**：添加渐变、阴影、动画，提升视觉质感
9. 📺 **横屏适配**：支持横屏播放，歌词显示更宽
10. 📝 **歌词编辑**：支持用户修改并保存歌词
11. ☁️ **云歌词匹配**：自动从网络下载缺失歌词

---

## 📄 开源协议

本项目采用 **MIT License** 开源协议，允许个人与商业项目自由使用、修改、分发，无需授权，但需保留原作者信息。

---

## 🤝 贡献指南

欢迎提交 Issue 或 Pull Request！  
请遵循：功能完整 → 测试通过 → 代码整洁 → 文档更新

---

## 🙏 致谢

感谢所有使用者与开源社区的支持！  
愿 Lumina Player 成为你聆听音乐时最安静的陪伴。

---

> 🎵 **Lumina Player v1.1** —— 让音乐，被听见，也被看见。  
> ✅ **当前版本：v1.1.0** | 字数：约 2200+ 字

```text
   _   _   _   _   _   _   _  
  / \ / \ / \ / \ / \ / \ / \ 
 ( L | u | m | i | n | a | 1.1 )
  \_/ \_/ \_/ \_/ \_/ \_/ \_/ 
```

--- 

> ✅ 此文档已根据你当前开发进度 **完整更新至 v1.1**，可直接替换原 `README.md`。  
> 如需生成 `.md` 文件、添加 Badge、图标或发布到 GitHub，我也可以继续帮你！继续加油！🚀
