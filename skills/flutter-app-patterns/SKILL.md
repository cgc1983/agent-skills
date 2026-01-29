---
name: "flutter-app-patterns"
description: "GetX controllers、响应式状态、依赖注入、bindings、导航与最佳实践、项目的目录结构、"
version: "1.0.0"
---

# 项目目录结构

```
lib/
├── ui/
│   ├── pages/             # 页面
│   ├── foundation/        # 设计基础（颜色、圆角、间距、字体）
│   ├── dialogs/           # 弹窗
│   ├── controllers/      # 控制器
│   └── components/        # 通用组件
├── common/                # 公共
├── services/              # 服务
├── config/                # 配置
└── utils/                 # 工具
```

> 项目目录中必须创建一个跟目录名字一样的dart文件，内部的代码都需要通过在这个同名文件中进行导出，例如：
如果utils目录下的dart文件有dev_board.dart,你就需要在utils目录下创建一个utils.dart，在utils.dart中添加如下代码来导出dev_board.dart


```dart

export 'dev_board.dart';

```


## ui/foundation 设计基础

`foundation/` 目录存放 App 的设计令牌（Design Tokens），统一颜色、圆角、间距与字体，便于全局复用与主题切换。

| 文件 | 说明 |
|------|------|
| `color.dart` | App 中颜色的定义 |
| `radius.dart` | App 圆角的定义 |
| `spacing.dart` | Column/Row 等组件的行/列间隔大小定义 |
| `text_style.dart` | App 中字体的定义 |


> color.dart文件中要定义一个class AppColorScheme,用来支持Dark mode 和 light mode

# 常用的插件(plugin)



```yaml

  gap: ^3.0.1
  path: ^1.9.0
  path_provider: ^2.1.5
  sqflite: ^2.4.1
  url_launcher: ^6.3.2
  lottie: ^3.3.2
  swiss_knife: ^3.2.3
  get: ^4.7.2
  flame: ^1.33.0
  swiftify: ^1.3.0
  flutter_svg: ^2.2.1
  dartx: ^1.2.0
  styled_widget: ^0.4.1
  modal_bottom_sheet: ^3.0.0
  signals: ^6.2.0
  vibration: ^3.1.3
  wheel_picker: ^0.2.2
  flutter_soloud: ^3.4.1
  ringer_mode: ^1.0.1
  torch_light: ^1.1.0
  flutter_switch: ^0.3.2
  flutter_secure_storage: ^9.2.4
  shared_preferences_wrapper: ^1.0.0
  shared_preferences: ^2.5.3
  package_info_plus: ^8.3.1
  uuid: ^4.4.2
  logger: ^2.6.2
  flutter_screenutil: ^5.9.3
  persistent_bottom_nav_bar: ^6.2.1
  siri_wave: ^2.3.0
  speech_to_text: ^7.3.0
  extended_text: ^15.0.2
  extended_text_field: ^16.0.2
  syncfusion_flutter_datepicker: ^32.1.22
  syncfusion_localizations: ^32.1.22
  slide_switcher: ^1.1.2
  day_picker: ^2.3.0
  flutter_advanced_drawer: ^1.5.0
  flutter_advanced_avatar: ^1.5.2
  syncfusion_flutter_calendar: ^32.1.23
  flutter_card_swiper: ^7.2.0
  super_context_menu: ^0.9.1
  flex_color_picker: ^3.7.2
  jiffy: ^6.4.4
  ulid: ^2.0.1
  popover: ^0.4.0
```

> 如果功能满足的情况下尽量从上面的plugin中选择插件，因为我比较熟悉方便我维护代码，如果不能满足条件，你可以根据功能从flutter pub dev上搜索插件，尽量选择score和download比较多的插件

## 插件功能与使用场景

| 插件 | 功能简述 | 典型使用场景 |
|-----|----------|--------------|
| **gap** | 在 Column/Row 等 Flex 布局中插入固定间距的 widget（Gap、MaxGap、SliverGap） | 列表项间距、表单项间距，替代 SizedBox 做统一间距 |
| **path** | Dart 跨平台路径操作（join、split、normalize，支持 Windows/POSIX） | 拼接/解析文件路径，多平台兼容的路径处理 |
| **path_provider** | 获取系统常用目录（临时、文档、缓存、下载、Library 等） | 数据库文件路径、缓存目录、用户文件存储位置 |
| **sqflite** | SQLite 数据库封装，支持 CRUD 与事务 | 本地结构化数据、离线数据、用户数据持久化 |
| **url_launcher** | 打开 URL、电话、短信、邮件等系统能力 | 打开网页、拨号、发短信/邮件、应用商店链接 |
| **lottie** | 播放 Lottie JSON 动画（After Effects 导出） | 加载动画、引导动效、插画动效、空状态动画 |
| **swiss_knife** | Dart 工具集（集合、数学、日期、URI、JSON、正则等） | 通用工具方法、减少重复工具代码 |
| **get** | GetX：状态管理、路由（无 context）、依赖注入 | 全局状态、无 BuildContext 导航、Bindings/DI |
| **flame** | 2D 游戏引擎（组件、碰撞、粒子、音频等） | 小游戏、复杂动效、粒子/动画场景 |
| **swiftify** | SwiftUI 风格链式修饰符（padding、背景、手势等） | 减少嵌套、链式写 UI，接近 SwiftUI 写法 |
| **flutter_svg** | 渲染 SVG 1.1，支持颜色过滤与自定义 ColorMapper | 矢量图标、插图、可缩放图形 |
| **dartx** | Kotlin 风格扩展（slice、sortedBy、distinctBy、DateTime/String 等） | 集合/字符串/日期操作更简洁 |
| **styled_widget** | 链式样式扩展（padding、decorated、alignment 等），类似 CSS/SwiftUI | 快速写 padding、背景、圆角、卡片样式 |
| **modal_bottom_sheet** | 可定制底部弹层（Material/Cupertino/自定义），支持滚动与拖拽 | 选择器、筛选面板、表单项、操作菜单 |
| **signals** | 细粒度响应式状态（类 Preact Signals），按依赖更新 | 局部状态、高性能 UI 更新、与 GetX 互补 |
| **vibration** | 设备振动 API（时长、模式、振幅等） | 触觉反馈、按钮反馈、通知提醒 |
| **wheel_picker** | 基于 ListWheelScrollView 的滚轮选择器 | 时间选择、选项选择、Picker 类 UI |
| **flutter_soloud** | 低延迟游戏向音频（3D 定位、效果、流式、多格式） | 游戏音效、BGM、需要低延迟的音频场景 |
| **ringer_mode** | 铃声模式检测与切换（正常/静音/振动） | 设置页铃声模式、勿扰/静音相关功能 |
| **torch_light** | 手电筒/闪光灯开关（检测可用性、开关控制） | 扫码页补光、夜间照明、相机相关 |
| **flutter_switch** | 可定制开关组件（颜色、尺寸、图标） | 设置项开关、布尔选项、开关样式统一 |
| **flutter_secure_storage** | 加密存储（Keychain/EncryptedSharedPreferences，可选生物识别） | Token、密码、敏感配置、密钥存储 |
| **shared_preferences_wrapper** | 对 shared_preferences 的封装，简化类型与默认值 | 简单键值存储、带默认值的配置读写 |
| **shared_preferences** | 系统键值存储（NSUserDefaults/SharedPreferences） | 用户偏好、简单配置、非敏感设置 |
| **package_info_plus** | 获取应用包名、版本号、build 号、安装来源等 | 关于页、更新检测、埋点、反馈信息 |
| **uuid** | 生成与解析 UUID（v1/v4/v5 等，RFC4122） | 唯一 ID、分布式 ID、实体主键 |
| **logger** | 分级日志（trace/debug/info/warning/error/fatal）、Pretty 输出 | 开发调试、生产日志、问题排查 |
| **flutter_screenutil** | 按设计稿尺寸做宽高/字体/圆角等适配（.w/.h/.sp/.r） | 多机型 UI 适配、设计稿 1:1 标注 |
| **persistent_bottom_nav_bar** | 持久化底部导航栏，多种样式与动画 | 主框架 Tab 导航、底部多 Tab 不重建 |
| **siri_wave** | Siri 风格波形动画（iOS 7/9 风格可配置） | 语音助手 UI、录音中/识别中指示 |
| **speech_to_text** | 语音转文字（设备语音识别） | 语音输入、无障碍、听写功能 |
| **extended_text** | 扩展 Text：富文本、@提及、内联图、自定义溢出与选区 | 社交内容、评论、带特殊格式的展示 |
| **extended_text_field** | 扩展输入框：内联图、@提及、自定义选区与工具栏 | 发帖、评论输入、富文本输入 |
| **syncfusion_flutter_datepicker** | Syncfusion 日期/范围选择器（月/年/十年视图、黑名单等） | 订票、报表、筛选日期、日期范围（需 Syncfusion 许可） |
| **syncfusion_localizations** | Syncfusion 组件本地化资源 | 日期/日历等组件多语言（配合 Syncfusion） |
| **slide_switcher** | 滑动切换 widget，可定制样式 | Tab 切换、二元选择、分段控制 |
| **day_picker** | 选择一周内的多天（SelectWeekDays） | 重复闹钟、营业日、周期设置 |
| **flutter_advanced_drawer** | 高级抽屉（动画、手势、背景、RTL） | 主导航抽屉、侧边菜单 |
| **flutter_advanced_avatar** | 高级头像（首字母、图片、状态点、装饰） | 用户头像、列表头像、在线状态 |
| **syncfusion_flutter_calendar** | Syncfusion 日历与日程视图（日/周/月/时间线等） | 日程、会议、预约（需 Syncfusion 许可） |
| **flutter_card_swiper** | Tinder 式卡片滑动（四向、可撤销、回调） | 推荐流、配对、滑动选择/淘汰 |
| **super_context_menu** | 原生风格上下文菜单（长按/右键） | 长按菜单、桌面端右键菜单 |
| **flex_color_picker** | 颜色选择器（多种样式、色轮、Material 色板、复制色值） | 主题色、背景色、自定义颜色设置 |
| **jiffy** | 日期解析、格式化、相对时间（fromNow）、多语言 | 展示「3 天前」、多格式日期、日程时间显示 |
| **ulid** | 可排序唯一 ID（ULID，128 位、26 字符） | 分布式 ID、按时间排序的 ID、替代部分 UUID 场景 |
| **popover** | 气泡/浮层（符合 Apple HIG，箭头、方向可配置） | 说明浮层、选项菜单、轻量弹层 |

> 部分 Syncfusion 组件需商业许可或社区许可，使用前请确认；表格若有偏差可自行修改。


# 多端适配

## 移动端适配

必须使用 **flutter_screenutil** 进行多端适配，设计分辨率是390*844；读取使用mcp读取figma设计稿的宽和高都要进行转化；

- 1.竖屏app相关宽高的数据都要使用.w

- 2.横屏app相关宽高数据都要使用.h

## 电脑端适配

如果flutter项目的目录下包含web,windows,macos,linux等桌面端的时候，编写组件的时候尽量能够适配桌面端。


# 动画

> 尽量使用flutter_animate这个组件来实现动画；

## Delay, duration, curve

Effects have optional `delay`, `duration`, and `curve` parameters. Effects run
in parallel, but you can use a `delay` to run them sequentially:

``` dart
Text("Hello").animate()
  .fade(duration: 500.ms)
  .scale(delay: 500.ms) // runs after fade.
```

Note that effects are "active" for the duration of the full animation, so for
example, two fade effects on the same target can have unexpected results
(`SwapEffect` detailed below, can help address this).

If not specified (or null), these values are inherited from the previous effect,
or from `Animate.defaultDuration` and `Animate.defaultCurve` if it is the first
effect:

``` dart
Text("Hello World!").animate()
  .fadeIn() // uses `Animate.defaultDuration`
  .scale() // inherits duration from fadeIn
  .move(delay: 300.ms, duration: 600.ms) // runs after the above w/new duration
  .blurXY() // inherits the delay & duration from move
```

`Animate` has its own `delay` parameter, which defines a delay before the
animation begins playing. Unlike the delay on an `Effect`, it is only applied
once if the animation repeats.

``` dart
Text("Hello").animate(
    delay: 1000.ms, // this delay only happens once at the very start
    onPlay: (controller) => controller.repeat(), // loop
  ).fadeIn(delay: 500.ms) // this delay happens at the start of each loop
```

## Other Effect Parameters

Most effects include `begin` and `end` parameters, which specify the start/end
values. These are usually "smart" in the sense that if only one is specified
then the other will default to a "neutral" value (ie. no visual effect). If
both are unspecified the effect should use visually pleasing defaults.

``` dart
// an opacity of 1 is "neutral"
Text("Hello").animate().fade() // begin=0, end=1
Text("Hello").animate().fade(begin: 0.5) // end=1
Text("Hello").animate().fade(end: 0.5) // begin=1
```

Many effects have additional parameters that influence their behavior. These
should also use pleasant defaults if unspecified.

``` dart
Text('Hello').animate().tint(color: Colors.purple)
```

## Sequencing with ThenEffect

`ThenEffect` is a special convenience "effect" that makes it easier to sequence
effects. It does this by establishing a new baseline time equal to the previous
effect's end time and its own optional `delay`. All subsequent effect delays are
relative to this new baseline.

In the following example, the slide would run 200ms after the fade ended.

``` dart
Text("Hello").animate()
  .fadeIn(duration: 600.ms)
  .then(delay: 200.ms) // baseline=800ms
  .slide()
```

## Animating lists


The `AnimateList` class offers similar functionality for lists of widgets, with
the option to offset each child's animation by a specified `interval`:

``` dart
Column(children: AnimateList(
  interval: 400.ms,
  effects: [FadeEffect(duration: 300.ms)],
  children: [Text("Hello"), Text("World"),  Text("Goodbye")],
))

// or shorthand:
Column(
  children: [Text("Hello"), Text("World"),  Text("Goodbye")]
    .animate(interval: 400.ms).fade(duration: 300.ms),
)
```

## Shared effects


Because `Effect` instances are immutable, they can be reused. This makes it easy
to create a global collection of effects that are used throughout your app and
updated in one place. This is also useful for design systems.

``` dart
MyGlobalEffects.transitionIn = <Effect>[
  FadeEffect(duration: 100.ms, curve: Curves.easeOut),
  ScaleEffect(begin: 0.8, curve: Curves.easeIn)
]

// then:
Text('Hello').animate(effects: MyGlobalEffects.transitionIn)
```


## Custom effects & builders

It is easy to write new resuable effects by extending `Effect`, but you can also
easily create one-off custom effects by using `CustomEffect`, `ToggleEffect`,
and `SwapEffect`.

## CustomEffect


`CustomEffect` lets you build custom animated effects. Simply specify a
`builder` function that accepts a `context`, `value`, and `child`. The child is
the target of the animation (which may already have been wrapped in other
effects).

For example, this would add a background behind the text and fade it from red to
blue:

``` dart
Text("Hello World").animate().custom(
  duration: 300.ms,
  builder: (context, value, child) => Container(
    color: Color.lerp(Colors.red, Colors.blue, value),
    padding: EdgeInsets.all(8),
    child: child, // child is the Text widget being animated
  )
)
```

By default it provides a `value` from `0-1` (though some curves can generate
values outside this range), based on the current time, duration, and curve. You
can also specify `begin` and `end` values as demonstrated in the example below.

`Animate` can be created without a child, so you use `CustomEffect` as a
simplified builder. For example, this would build text counting down from 10,
and fading out:

``` dart
Animate().custom(
  duration: 10.seconds,
  begin: 10,
  end: 0,
  builder: (_, value, __) => Text(value.round()),
).fadeOut()
```


## ToggleEffect

`ToggleEffect` also provides builder functionality, but instead of a `double`,
it provides a boolean value equal to `true` before the end of the effect and
`false` after (ie. after its duration).

``` dart
Animate().toggle(
  duration: 2.seconds,
  builder: (_, value, __) => Text(value ? "Before" : "After"),
)
```

This can also be used to activate "Animated" widgets, like `AnimatedContainer`,
by toggling their values with a minimal delay:

``` dart
Animate().toggle(
  duration: 1.ms,
  builder: (_, value, __) => AnimatedContainer(
    duration: 1.seconds,
    color: value ? Colors.red : Colors.green,
  ),
)
```

## SwapEffect


`SwapEffect` lets you swap out the whole target widget at a specified time:

``` dart
Text("Before").animate()
  .swap(duration: 900.ms, builder: (_, __) => Text("After"))
```

This can also be useful for creating sequential effects, by swapping the target
widget back in, effectively wiping all previous effects:

``` dart
text.animate().fadeOut(300.ms) // fade out & then...
  // swap in original widget & fade back in via a new Animate:
  .swap(builder: (_, child) => child.animate().fadeIn())
```

## ShaderEffect

`ShaderEffect` makes it easy to apply animated GLSL fragment shaders to widgets.
See the docs for details.

``` dart
myWidget.animate()
  .shader(duration: 2.seconds, shader: myShader)
  .fadeIn(duration: 300.ms) // shader can be combined with other effects
```


## Events & callbacks


`Animate` includes the following callbacks:

- `onInit`: the internal `AnimationController` has been initialized
- `onPlay`: the animation has started playing after any `Animate.delay`
- `onComplete`: the animation has finished

These callbacks return the `AnimationController`, which can be used to
manipulate the animation (ex. repeat, reverse, etc).

``` dart
Text("Horrible Pulsing Text")
  .animate(onPlay: (controller) => controller.repeat(reverse: true))
  .fadeOut(curve: Curves.easeInOut)
```

For more nuanced callbacks, use `CallbackEffect` or `ListenEffect`.

## CallbackEffect


`CallbackEffect` lets you add a callback to an arbitrary postion in your
animations. For example, adding a callback halfway through a fade:

``` dart
Text("Hello").animate().fadeIn(duration: 600.ms)
  .callback(duration: 300.ms, callback: (_) => print('halfway'))
```

As with other effects, it will inherit the delay and duration of prior effects:

``` dart
Text("Hello").animate().scale(delay: 200.ms, duration: 400.ms)
  .callback(callback: (_) => print('scale is done'))
```

## ListenEffect


`ListenEffect` lets you register a callback to receive the animation value (as a
`double`) for a given delay, duration, curve, begin, and end.

``` dart
Text("Hello").animate().fadeIn(curve: Curves.easeOutExpo)
  .listen(callback: (value) => print('current opacity: $value'))
```

The above example works, because the listen effect inherits duration and curve
from the fade, and both use `begin=0, end=1` by default.


## Adapters and Controllers


By default, all animations are driven by an internal `AnimationController`, and
update based on elapsed time. For more control, you can specify your own
external `controller`, or use an `adapter`. You can also set `autoPlay=false` if
you want to start the animation manually.

Adapters synchronize the `AnimationController` to an external source. For
example, the `ScrollAdapter` updates an animation based on a `ScrollController`
so you can run complex animations based on scroll interactions.

You still define animations using durations, but the external source must
provide a `0-1` value.

Flutter Animate ships with a collection of useful adapters. Check them out for
more information.


# GetX 状态管理模式（State Management Patterns）

## 响应式状态管理（Reactive State Management）

### 响应式变量（Reactive Variables）
```dart
class UserController extends GetxController {
  // Simple reactive variable
  final count = 0.obs;
  
  // Complex reactive variable
  final user = Rx<User?>(null);
  
  // Reactive list
  final users = <User>[].obs;
  
  // Reactive map
  final settings = <String, dynamic>{}.obs;
  
  // Getters (recommended)
  int get countValue => count.value;
  User? get currentUser => user.value;
}
```

### 响应式组件（Reactive Widgets）
```dart
// Option 1: Obx (lightweight)
Obx(() => Text(controller.count.toString()))

// Option 2: GetX (with dependency injection)
GetX<UserController>(
  builder: (controller) => Text(controller.user?.name ?? 'Loading...'),
)

// Option 3: GetBuilder (no reactive variables needed)
GetBuilder<UserController>(
  builder: (controller) => Text(controller.userName),
)
```

## 依赖注入（Dependency Injection）

### 注册方式（Registration Methods）
```dart
// Immediate instance
Get.put(UserController());

// Lazy instance (created when first used)
Get.lazyPut(() => UserController());

// Singleton (persists across routes)
Get.putAsync(() => StorageService().init());

// Fenix (recreates when route is accessed again)
Get.lazyPut(() => UserController(), fenix: true);
```

### Bindings 模式（Bindings Pattern）
```dart
class UserBinding extends Bindings {
  @override
  void dependencies() {
    // Data sources
    Get.lazyPut(() => UserProvider(Get.find()));
    Get.lazyPut(() => UserLocalSource(Get.find()));
    
    // Repository
    Get.lazyPut<UserRepository>(
      () => UserRepositoryImpl(Get.find(), Get.find()),
    );
    
    // Use cases
    Get.lazyPut(() => GetUser(Get.find()));
    
    // Controller
    Get.lazyPut(() => UserController(getUserUseCase: Get.find()));
  }
}
```

## 路由导航（Navigation）

### 基础导航（Basic Navigation）
```dart
// Navigate to route
Get.to(() => ProfilePage());

// Navigate with name
Get.toNamed('/profile');

// Navigate and remove previous
Get.off(() => LoginPage());

// Navigate and remove all previous
Get.offAll(() => HomePage());

// Go back
Get.back();

// Go back with result
Get.back(result: user);
```

### 携带参数的导航（Navigation with Arguments）
```dart
// Send arguments
Get.toNamed('/profile', arguments: {'id': '123'});

// Receive arguments
final args = Get.arguments as Map<String, dynamic>;
final id = args['id'];
```

### 路由配置（Route Configuration）
```dart
class AppRoutes {
  static const initial = '/splash';
  
  static final routes = [
    GetPage(
      name: '/splash',
      page: () => SplashPage(),
      binding: SplashBinding(),
    ),
    GetPage(
      name: '/login',
      page: () => LoginPage(),
      binding: AuthBinding(),
      transition: Transition.fadeIn,
    ),
    GetPage(
      name: '/home',
      page: () => HomePage(),
      binding: HomeBinding(),
      middlewares: [AuthMiddleware()],
    ),
  ];
}
```

## Controller 生命周期（Controller Lifecycle）

```dart
class UserController extends GetxController {
  @override
  void onInit() {
    super.onInit();
    // Called immediately after controller is allocated
    loadUser();
  }
  
  @override
  void onReady() {
    super.onReady();
    // Called after widget is rendered
    analytics.logScreenView();
  }
  
  @override
  void onClose() {
    // Clean up resources
    _subscription.cancel();
    super.onClose();
  }
}
```

## Snackbar 与 Dialog（Snackbars & Dialogs）

```dart
// Snackbar
Get.snackbar(
  'Success',
  'User created successfully',
  snackPosition: SnackPosition.BOTTOM,
  backgroundColor: Colors.green,
);

// Dialog
Get.dialog(
  AlertDialog(
    title: Text('Confirm'),
    content: Text('Are you sure?'),
    actions: [
      TextButton(
        onPressed: () => Get.back(),
        child: Text('Cancel'),
      ),
      TextButton(
        onPressed: () {
          deleteUser();
          Get.back();
        },
        child: Text('Delete'),
      ),
    ],
  ),
);

// Bottom sheet
Get.bottomSheet(
  Container(
    child: Column(
      children: [
        ListTile(
          title: Text('Option 1'),
          onTap: () => Get.back(result: 1),
        ),
      ],
    ),
  ),
);
```

## 最佳实践（Best Practices）

### ✅ 推荐实践（DO）
- 使用 bindings 进行依赖注入（dependency injection）
- 使用私有的响应式变量，并通过公共 getter 暴露给 UI
- 在 `onClose()` 中清理资源（如 stream、subscription 等）
- 使用语义清晰的 controller 命名（例如 `UserController`，而不是 `Controller1`）
- 将业务逻辑放在 use case 中，而不是直接写在 controllers 里

### ❌ 不推荐做法（DON'T）
- 不要在 widget 中直接创建 controllers
- 不要把复杂业务逻辑堆在 controllers 中
- 不要忘记释放（dispose）streams 和 subscriptions
- 不要过度依赖全局状态（global state）
- 不要制造循环依赖（circular dependencies）


### Dart 3.x Features

### Pattern Matching
```dart
String describeUser(User user) {
  return switch (user) {
    User(role: 'admin', isActive: true) => 'Active administrator',
    User(role: 'user', isActive: true) => 'Active user',
    User(isActive: false) => 'Inactive account',
    _ => 'Unknown status',
  };
}
```

### Records
```dart
(int, String) getUserInfo() => (123, 'John Doe');

final (id, name) = getUserInfo();
```

### Sealed Classes
```dart
sealed class Result<T> {}
class Success<T> extends Result<T> {
  final T data;
  Success(this.data);
}
class Error<T> extends Result<T> {
  final String message;
  Error(this.message);
}
```


## 需要避免的 Anti-Patterns

```dart
// ❌ BAD: Business logic in controller
class UserController extends GetxController {
  Future<void> createUser(String name, String email) async {
    // Direct API call in controller
    final response = await http.post(...);
    // Business logic in controller
    if (response.statusCode == 201) {
      user.value = User.fromJson(response.body);
    }
  }
}

// ✅ GOOD: Delegate to use case
class UserController extends GetxController {
  final CreateUser createUserUseCase;
  
  Future<void> createUser(String name, String email) async {
    final result = await createUserUseCase(name, email);
    result.fold(
      (failure) => _handleError(failure),
      (user) => user.value = user,
    );
  }
}
```

## 文件命名规范（File Naming Conventions）

- **Files**: `snake_case.dart`
- **Classes**: `PascalCase`
- **Variables/Functions**: `camelCase`
- **Constants**: `lowerCamelCase` or `SCREAMING_SNAKE_CASE` for compile-time constants

```dart
// user_controller.dart
class UserController extends GetxController {
  static const int maxRetries = 3;
  static const String BASE_URL = 'https://api.example.com';
  
  final userName = 'John'.obs;
  
  void fetchUserData() {
    // ...
  }
}
```

## Null Safety

```dart
// Use late for non-nullable fields initialized later
class MyController extends GetxController {
  late final UserRepository repository;
  
  @override
  void onInit() {
    super.onInit();
    repository = Get.find();
  }
}

// Use ? for nullable types
String? userName;

// Use ! only when absolutely certain
final name = userName!; // Use sparingly

// Prefer ?? for defaults
final displayName = userName ?? 'Guest';
```

## Async/Await Best Practices

```dart
// Use async/await for asynchronous operations
Future<User> fetchUser(String id) async {
  try {
    final response = await client.get(Uri.parse('/users/$id'));
    return User.fromJson(jsonDecode(response.body));
  } on SocketException {
    throw NetworkException();
  } catch (e) {
    throw UnknownException(e.toString());
  }
}

// Use Future.wait for parallel operations
Future<void> loadAllData() async {
  final results = await Future.wait([
    fetchUsers(),
    fetchSettings(),
    fetchPreferences(),
  ]);
}

// Use unawaited for fire-and-forget
unawaited(analytics.logEvent('page_view'));
```

## Code Organization Within Files

```dart
class MyClass {
  // 1. Constants
  static const int maxRetries = 3;
  
  // 2. Static fields
  static final instance = MyClass._();
  
  // 3. Instance fields
  final String id;
  final _isLoading = false.obs;
  
  // 4. Constructors
  MyClass(this.id);
  MyClass._();
  
  // 5. Getters/Setters
  bool get isLoading => _isLoading.value;
  
  // 6. Lifecycle methods
  @override
  void onInit() {}
  
  // 7. Public methods
  void publicMethod() {}
  
  // 8. Private methods
  void _privateMethod() {}
}
```

## Widget Best Practices

```dart
// Prefer const constructors
class MyWidget extends StatelessWidget {
  const MyWidget({Key? key}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return const Text('Hello');
  }
}

// Extract widgets for reusability
class UserCard extends StatelessWidget {
  final User user;
  
  const UserCard({Key? key, required this.user}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return Card(
      child: _buildContent(),
    );
  }
  
  Widget _buildContent() {
    return Column(
      children: [
        Text(user.name),
        Text(user.email),
      ],
    );
  }
}
```



## 注意事项

1. **关于文档输出**：默认不要主动生成文档内容，除非用户明确提出需求。项目模块说明以及 `README.md` 文档属于例外场景，在相关上下文中可以按需输出。
2. **关于测试代码**：默认不要编写或输出测试代码（如单元测试、集成测试等）。只有在用户明确要求提供测试代码时，才输出，并且需在回复中**显式说明**「下面的是测试代码」或含义等价的提示。
3. **代码注释**：逻辑简单的代码片段，为必要不要添加注释，但是代码逻辑复杂涉及到公式计算等的场景下，可以酌情添加注释；函数功能描述的注释和参数的注释，有必要可以添加
