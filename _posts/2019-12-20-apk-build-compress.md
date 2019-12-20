---
layout: post
title: "压缩、混淆和优化您的应用"
subtitle: ' 压缩、混淆和优化您的应用 '
author: "Song"
header-style: text
tags:
  - gradle
  - build
  - compress
---

## 压缩、混淆和优化您的应用

> 官网链接 https://developer.android.com/studio/build/shrink-code

要尽可能减小应用的大小，您应在发布版本中启用压缩功能来移除未使用的代码和资源。启用压缩功能后，您还会受益于两项功能，一项是混淆功能，该功能会缩短应用的类和成员的名称；另一项是优化功能，该功能会应用更积极的策略来进一步减小应用的大小。本页介绍 R8 如何为您的项目执行这些编译时任务，以及您如何对这些任务进行自定义。

当您使用 Android Gradle 插件 3.4.0 或更高版本编译您的项目时，该插件不再使用 ProGuard 执行编译时代码优化，而是与 R8 编译器一起使用，共同处理以下编译时任务：

- 代码压缩（即摇树优化）：从您的应用及其库依赖项中检测并安全地移除未使用的类、字段、方法和属性（这使其成为一个用来规避 64k 引用限制的非常有用的工具）。例如，如果您仅使用某个库依赖项的少数几个 API，压缩功能可以识别您的应用未使用的库代码，仅从您的应用中移除这部分代码。要了解详情，请转到有关如何压缩代码的部分。
- 资源压缩：从您的封装应用中移除未使用的资源，包括应用的库依赖项中未使用的资源。此功能可与代码压缩结合使用，这样一来，移除未使用的代码后，也可以安全地移除不再引用的任何资源。要了解详情，请转到有关如何压缩资源的部分。
- 混淆：缩短类和成员的名称，从而减小 DEX 文件大小。要了解详情，请转到有关如何对代码进行混淆处理的部分。
- 优化：检查并重写代码，以进一步减小应用 DEX 文件的大小。例如，如果 R8 检测到从未采用过给定 if/else 语句的 else {} 分支，R8 便会移除 else {} 分支的代码。要了解详情，请转到有关代码优化的部分。

在编译应用的发布版本时，默认情况下，R8 会自动为您执行上述编译时任务。不过，您可以停用某些任务或通过 ProGuard 规则文件来自定义 R8 的行为。事实上，R8 支持所有现有 ProGuard 规则文件，因此在更新 Android Gradle 插件以使用 R8 时，不应要求您更改现有规则。

### 启用压缩、混淆和优化功能
当您使用 Android Studio 3.4 或 Android Gradle 插件 3.4.0 及更高版本时，R8 是默认编译器，可将项目的 Java 字节码转换为在 Android 平台上运行的 DEX 格式。不过，当您使用 Android Studio 创建新项目时，默认情况下并不启用压缩、混淆和代码优化功能。这是因为，这些编译时优化功能会增加项目的编译时间，而且如果您没有充分自定义要保留的代码，还可能会引入错误。

因此，最好在编译应用的最终版本（也就是在发布应用之前测试的版本）时启用这些编译时任务。要启用压缩、混淆和优化功能，请在您的项目级 build.gradle 文件中添加以下代码。

```gradle
android {
        buildTypes {
            release {
                // Enables code shrinking, obfuscation, and optimization for only
                // your project's release build type.
                minifyEnabled true

                // Enables resource shrinking, which is performed by the
                // Android Gradle plugin.
                shrinkResources true

                // Includes the default ProGuard rules files that are packaged with
                // the Android Gradle plugin. To learn more, go to the section about
                // R8 configuration files.
                proguardFiles getDefaultProguardFile(
                        'proguard-android-optimize.txt'),
                        'proguard-rules.pro'
            }
        }
        ...
    }
```

### R8 配置文件
R8 使用 ProGuard 规则文件来修改其默认行为并更好地了解应用的结构，如充当应用代码入口点的类。虽然您可以修改其中一些规则文件，但某些规则可能由编译时工具（如 AAPT2）自动生成，或从应用的库依赖项继承而来。下表介绍了 R8 使用的 ProGuard 规则文件的来源。

### 添加其他配置

当您使用 Android Studio 创建新项目或模块时，IDE 会创建一个 <module-dir>/proguard-rules.pro 文件，以便您添加自己的规则。此外，您还可以通过将相应文件添加到模块的 build.gradle 文件中的 proguardFiles 属性，从其他文件添加额外的规则。

例如，您可以通过在相应的 productFlavor 代码块中再添加一个 proguardFiles 属性，添加每个编译变体专用的规则。以下 Gradle 文件向 flavor2 产品特性添加了 flavor2-rules.pro。现在，flavor2 使用全部三个 ProGuard 规则，因为也应用了 release 代码块中的规则。

```
android {
        ...
        buildTypes {
            release {
                minifyEnabled true
                proguardFiles getDefaultProguardFile(
                  'proguard-android-optimize.txt'),
                  // List additional ProGuard rules for the given build type here. By default,
                  // Android Studio creates and includes an empty rules file for you (located
                  // at the root directory of each module).
                  'proguard-rules.pro'
            }
        }
        flavorDimensions "version"
        productFlavors {
            flavor1 {
              ...
            }
            flavor2 {
                proguardFile 'flavor2-rules.pro'
            }
        }
    }
```

### 压缩代码

如果将 minifyEnabled 属性设为 true，默认情况下会启用 R8 代码压缩。

代码压缩（也称为“摇树优化”）是指移除 R8 确定在运行时不需要的代码的过程。例如，如果您的应用包含许多库依赖项，但仅利用它们的一小部分功能，那么此过程可以大大减小应用的大小。

要压缩应用的代码，R8 首先根据组合的配置文件集确定应用代码的所有入口点。这些入口点包括 Android 平台可用来打开应用的 Activity 或服务的所有类。从每个入口点开始，R8 会检查应用的代码来构建一张图，列出您的应用可能会在运行时访问的所有方法、成员变量和其他类。未连接到该图的代码被视为执行不到，可能会从应用中移除。

图 1 显示了一个具有运行时库依赖项的应用。经过检查应用的代码，R8 确定可以从 MainActivity.class 入口点执行到 foo()、faz() 和 bar() 方法。不过，您的应用从未在运行时使用过 OkayApi.class 类或其 baz() 方法，因此 R8 会在压缩应用时移除该代码。

R8 通过项目的 R8 配置文件中的 -keep 规则来确定入口点。也就是说，保留规则指定 R8 在压缩应用时不应舍弃的类，R8 将这些类视为应用的可能入口点。Android Gradle 插件和 AAPT2 会自动为您生成大多数应用项目（如应用的 Activity、视图和服务）所需的保留规则。不过，如果您需要使用其他保留规则来自定义此默认行为，请阅读有关如何自定义要保留的代码的部分。

如果您只想减小应用资源的大小，请跳到有关如何压缩资源的部分。

### 自定义要保留的代码

对于大多数情况，默认的 ProGuard 规则文件 (proguard-android- optimize.txt) 足以满足需要，让 R8 仅移除未使用的代码。不过，在某些情况下，R8 很难做出正确分析，因此可能会移除您的应用实际上需要的代码。下面列举了几个例子，说明了它在什么情况下可能会错误地移除代码：

当您的应用通过 Java 原生接口 (JNI) 调用方法时
当您的应用在运行时查询代码时（如使用反射）
通过测试您的应用应该能够发现因移除代码不当而导致的任何错误，但您也可以通过生成已移除代码的报告来检查移除了哪些代码。

要修复错误并强制 R8 保留某些代码，请在 ProGuard 规则文件中添加 -keep 代码行。例如：

```
-keep public class MyClass
```

或者，您也可以向要保留的代码添加 @Keep 注解。在类上添加 @Keep 会使整个类保持原样。在方法或字段上添加该注解会使方法/字段（及其名称）以及类名称保持不变。请注意，只有在使用 AndroidX 注解库且您添加 Android Gradle 插件随附的 ProGuard 规则文件时，此注解才可用，如有关如何启用压缩的部分中所述。

在使用 -keep 选项时，您应该考虑许多因素；如需详细了解如何自定义规则文件，请阅读 ProGuard 手册。问题排查部分大体介绍了去掉代码后您可能会遇到的其他常见问题。

### 压缩资源

只有在与代码压缩相配合使用时，资源压缩才能发挥作用。代码压缩器移除所有未使用的代码后，资源压缩器便可确定应用仍然使用的资源有哪些。当您添加包含资源的代码库时尤其如此 - 您必须移除未使用的库代码，使库资源变为未引用资源，因而可由资源压缩器移除。

要启用资源压缩，请在 build.gradle 文件中将 shrinkResources 属性设为 true（在用于代码压缩的 minifyEnabled 旁边）。例如：

```
android {
        ...
        buildTypes {
            release {
                shrinkResources true
                minifyEnabled true
                proguardFiles getDefaultProguardFile('proguard-android.txt'),
                        'proguard-rules.pro'
            }
        }
    }
```

如果您尚未使用代码压缩用途的 minifyEnabled 编译应用，请先尝试使用它，然后再启用 shrinkResources，因为您可能需要修改 proguard-rules.pro 文件以保留动态创建或调用的类或方法，然后再开始移除资源。

### 自定义要保留的资源

如果您希望保留或舍弃特定资源，请在您的项目中创建一个带有 <resources> 标记的 XML 文件，并在 tools:keep 属性中指定要保留的每个资源，在 tools:discard 属性中指定要舍弃的每个资源。这两个属性都接受逗号分隔的资源名称列表。您可以将星号字符用作通配符。

例如：

	```
	<?xml version="1.0" encoding="utf-8"?>
	<resources xmlns:tools="http://schemas.android.com/tools"
	    tools:keep="@layout/l_used*_c,@layout/l_used_a,@layout/l_used_b*"
	    tools:discard="@layout/unused2" />
	```


将此文件保存在项目资源中，例如，保存在 res/raw/keep.xml。编译系统不会将此文件打包到 APK 中。

指定要舍弃的资源可能看似愚蠢，因为您本可将它们删除，但在使用编译变体时，这样做可能很有用。例如，如果您知道给定的资源好像会在代码中使用（因此不会被压缩器移除），但实际上它不会用于给定的编译变体，那么就可以将所有资源放入通用项目目录，然后为每个编译变体创建一个不同的 keep.xml 文件。编译工具也可能会将某个资源错误地识别为需要的资源，之所以可能发生这种情况是因为，编译器会以内嵌方式添加资源 ID，如果真正引用的资源与代码中的某个整数值恰巧具有相同的值，资源分析器可能不知道这两者之间的差别。

### 启用严格引用检查

通常，资源压缩器可以准确地判断是否使用了某个资源。不过，如果您的代码调用了 Resources.getIdentifier()（或您的任何库进行了这一调用 - AppCompat 库会执行该调用），这就表示您的代码是根据动态生成的字符串查询资源名称。当您执行这一调用时，资源压缩器默认情况下会采取防御性行为，将所有具有匹配名称格式的资源标记为可能已使用，无法移除。

例如，以下代码会使系统将所有带 img_ 前缀的资源标记为已使用。

	```
	String name = String.format("img_%1d", angle + 1);
	res = getResources().getIdentifier(name, "drawable", getPackageName());
	```

资源压缩器还会浏览代码以及各种 res/raw/ 资源中的所有字符串常量，查找格式类似于 file:///android_res/drawable//ic_plus_anim_016.png 的资源网址。如果它找到这样的字符串，或发现一些其他字符串看似可用来构建这样的网址，就不会将它们移除。

这些是默认情况下启用的安全压缩模式的示例。不过，您可以停用这种“安全总比后悔好”的处理方式，并指定资源压缩器只保留其确定已使用的资源。为此，请在 keep.xml 文件中将 shrinkMode 设为 strict，如下所示：

	```
	<?xml version="1.0" encoding="utf-8"?>
	<resources xmlns:tools="http://schemas.android.com/tools"
	    tools:shrinkMode="strict" />
	```

如果您确实启用了严格压缩模式，并且您的代码也通过动态生成的字符串引用资源（如上所示），则您必须使用 tools:keep 属性来手动保留这些资源。

### 移除未使用的备用资源
Gradle 资源压缩器只会移除未由您的应用代码引用的资源，这意味着，它不会移除用于不同设备配置的备用资源。如有必要，您可以使用 Android Gradle 插件的 resConfigs 属性来移除您的应用不需要的备用资源文件。

例如，如果您使用的是包含语言资源的库（如 AppCompat 或 Google Play 服务），则 APK 将包含这些库中消息的所有已翻译语言字符串，无论应用的其余部分是否翻译为同一语言。如果您希望只保留应用正式支持的语言，可以使用 resConfig 属性指定这些语言。系统会移除未指定语言的所有资源。

以下代码段展示了如何将语言资源限定为仅支持英语和法语：

```
android {
        defaultConfig {
            ...
            resConfigs "en", "fr"
        }
    }
```

同样，您也可以通过编译多个 APK，让每个 APK 以不同的设备配置为目标，自定义要包含在 APK 中的屏幕密度或 ABI 资源。

### 合并重复资源
默认情况下，Gradle 还会合并同名的资源，如可能位于不同资源文件夹中的同名可绘制对象。这一行为不受 shrinkResources 属性控制，也无法停用，因为当多个资源与代码查询的名称匹配时，有必要利用这一行为来避免错误。

只有在两个或更多个文件具有完全相同的资源名称、类型和限定符时，才会进行资源合并。Gradle 会在重复项中选择它认为最合适的文件（根据下述优先顺序），并且只将这一个资源传递给 AAPT，以便在 APK 文件中分发。

Gradle 会在以下位置查找重复资源：

  - 与主源集关联的主资源，通常位于 src/main/res/。
  - 变体叠加，来自编译类型和编译特性。
  - 库项目依赖项。
Gradle 会按以下级联优先顺序合并重复资源：

依赖项 → 主资源 → 编译特性 → 编译类型

例如，如果某个重复资源同时出现在主资源和编译特性中，Gradle 会选择编译特性中的资源。

如果完全相同的资源出现在同一源集中，Gradle 无法合并它们，并且会发出资源合并错误。如果您在 build.gradle 文件的 sourceSet 属性中定义了多个源集，则可能会发生这种情况。例如，如果 src/main/res/ 和 src/main/res2/ 包含完全相同的资源，就可能会发生这种情况。

### 对代码进行混淆处理
混淆处理的目的是通过缩短应用的类、方法和字段的名称来减小应用的大小。下面是使用 R8 进行混淆处理的一个示例：

```
androidx.appcompat.app.ActionBarDrawerToggle$DelegateProvider -> a.a.a.b:
    androidx.appcompat.app.AlertController -> androidx.appcompat.app.AlertController:
        android.content.Context mContext -> a
        int mListItemLayout -> O
        int mViewSpacingRight -> l
        android.widget.Button mButtonNeutral -> w
        int mMultiChoiceItemLayout -> M
        boolean mShowTitle -> P
        int mViewSpacingLeft -> j
        int mButtonPanelSideLayout -> K
```

虽然混淆处理不会从应用中移除代码，但如果应用的 DEX 文件将许多类、方法和字段编入索引，那么混淆处理会显著缩减应用的大小。不过，由于混淆处理会对代码的不同部分进行重命名，因此某些任务（如检查堆栈轨迹）需要其他工具来执行。要了解混淆处理后的堆栈轨迹，请阅读有关如何解码混淆过的堆栈轨迹的下一部分。

此外，如果您的代码依赖于应用的方法和类的可预测命名 - 例如，使用反射时，您应该将相应签名视为入口点并为其指定保留规则，如有关如何自定义要保留的代码的部分中所述。这些保留规则会告知 R8 不仅要在应用的最终 DEX 中保留该代码，而且还要保留其原始命名。

### 解码混淆过的堆栈轨迹

R8 对您的代码进行混淆处理后，理解堆栈轨迹的难度将会极大增加，因为类和方法的名称可能有变化。除了重命名之外，R8 还可能会更改出现在堆栈轨迹中的行号，以便在写入 DEX 文件时进一步缩减大小。幸运的是，R8 每次运行时都会创建一个 mapping.txt 文件，其中列出了混淆过的类、方法和字段名称与原始名称的映射关系。此映射文件还包含用于将行号映射回原始源文件行号的信息。R8 将此文件保存在 <module- name>/build/outputs/mapping/<build-type>/ 目录中。

注意：您每次编译项目时都会覆盖 R8 生成的 mapping.txt 文件，因此您每次发布新版本时都必须小心地保存一个副本。通过为每个发布版本保留一个 mapping.txt 文件副本，如果用户提交了来自旧版应用的混淆过的堆栈轨迹，您将能够调试相关问题。
在 Google Play 上发布您的应用时，您可以上传每个 APK 版本的 mapping.txt 文件。然后，Google Play 将根据用户报告的问题对传入的堆栈轨迹进行反混淆处理，以便您可以在 Google Play 管理中心进行查看。如需了解详情，请参阅有关如何对崩溃的堆栈轨迹进行反混淆处理的帮助中心文章。

要自行将混淆过的堆栈轨迹转换为可读的堆栈轨迹，请使用 ProGuard 随附的 ReTrace 脚本。

### 代码优化

为了进一步压缩您的应用，R8 会在更深层面上检查您的代码以移除更多未使用的代码，或者在可能的情况下重写您的代码以使其更简洁。下面是此类优化的几个示例：

如果您的代码从不采用给定 if/else 语句的 else {} 分支，R8 可能会移除 else {} 分支的代码。
如果您的代码只在一个位置调用某个方法，R8 可能会移除该方法而将其内嵌在这一个调用点。
如果 R8 确定某个类只有一个唯一的子类且该类本身未实例化（例如，一个抽象基类仅由一个具体实现类使用），那么 R8 可以将这两个类组合在一起并从应用中移除一个类。
要了解详情，请阅读 Jake Wharton 撰写的 R8 优化方面的博文。
R8 不允许您停用或启用离散优化，也不允许您修改优化的行为。事实上，R8 会忽略试图修改默认优化的任何 ProGuard 规则，如 -optimizations 和 - optimizationpasses。此限制很重要，因为随着 R8 的不断改进，维护标准的优化行为有助于 Android Studio 团队轻松排查问题并解决您可能会遇到的任何问题。

### 启用更积极的优化

R8 包含一组额外的优化功能，默认情况下未启用这些功能。您可以通过在项目的 gradle.properties 文件中添加以下代码来启用这些额外的优化功能：

```
android.enableR8.fullMode=true
```

由于额外的优化功能使得 R8 的行为与 ProGuard 不同，因此它们可能要求您添加额外的 ProGuard 规则以避免运行时问题。例如，假设您的代码通过 Java Reflection API 引用一个类。默认情况下，R8 假设您打算在运行时检查和操纵该类的对象（即使您的代码实际上并不这样做），因此它会自动保留该类及其静态初始化程序。

不过，在使用“完整模式”时，R8 不会做出这种假设，如果 R8 断言您的代码从不在运行时使用该类，它会从应用的最终 DEX 中移除该类。也就是说，如果要保留该类及其静态初始化程序，您需要在规则文件中添加相应的保留规则。

如果您在使用 R8 的“完整模式”时遇到任何问题，请参阅 R8 常见问题解答页面，以查看可能的解决方案。如果您无法解决相关问题，请报告错误。

### 排查 R8 问题

本部分介绍在使用 R8 启用代码压缩、混淆和优化时排查问题的一些策略。如果您在下面找不到您的问题的解决方案，您还应参阅 R8 常见问题解答页面和 ProGuard 的问题排查指南。

生成移除的（或保留的）代码的报告
为了帮助您排查 R8 的某些问题，查看 R8 从您的应用中移除的所有代码的报告可能很有用。对于要为其生成此报告的每个模块，请将 -printusage <output-dir>/usage.txt 添加到自定义规则文件。当您启用 R8 并编译您的应用时，R8 会输出一个包含您指定的路径和文件名的报告。移除的代码的报告与以下输出类似：

```
androidx.drawerlayout.R$attr
    androidx.vectordrawable.R
    androidx.appcompat.app.AppCompatDelegateImpl
        public void setSupportActionBar(androidx.appcompat.widget.Toolbar)
        public boolean hasWindowFeature(int)
        public void setHandleNativeActionModesEnabled(boolean)
        android.view.ViewGroup getSubDecor()
        public void setLocalNightMode(int)
        final androidx.appcompat.app.AppCompatDelegateImpl$AutoNightModeManager getAutoNightModeManager()
        public final androidx.appcompat.app.ActionBarDrawerToggle$Delegate getDrawerToggleDelegate()
        private static final boolean DEBUG
        private static final java.lang.String KEY_LOCAL_NIGHT_MODE
        static final java.lang.String EXCEPTION_HANDLER_MESSAGE_SUFFIX
    ...
```

如果您要查看 R8 根据项目的保留规则确定的入口点的报告，请在自定义规则文件中添加 -printseeds <output-dir>/seeds.txt。当您启用 R8 并编译您的应用时，R8 会输出一个包含您指定的路径和文件名的报告。保留的入口点的报告与以下输出类似：

```
com.example.myapplication.MainActivity
    androidx.appcompat.R$layout: int abc_action_menu_item_layout
    androidx.appcompat.R$attr: int activityChooserViewStyle
    androidx.appcompat.R$styleable: int MenuItem_android_id
    androidx.appcompat.R$styleable: int[] CoordinatorLayout_Layout
    androidx.lifecycle.FullLifecycleObserverAdapter
    ...
```

### 排查资源压缩问题

当您压缩资源时，Build  窗口会显示从 APK 中移除的资源的摘要。（您需要先点击窗口左侧的 Toggle view 图标  以显示 Gradle 的详细文本输出。）例如：

```
:android:shrinkDebugResources
    Removed unused resources: Binary resource data reduced from 2570KB to 1711KB: Removed 33%
    :android:validateDebugSigning
```

Gradle 还会在 <module-name>/build/outputs/mapping/release/（与 ProGuard 的输出文件所在的文件夹相同）中创建一个名为 resources.txt 的诊断文件。此文件包含一些详细信息，如哪些资源引用了其他资源以及使用或移除了哪些资源。

例如，要了解为什么 @drawable/ic_plus_anim_016 仍在您的 APK 中，请打开 resources.txt 文件并搜索相应的文件名。您可能会发现，有其他资源引用了它，如下所示：

```
16:25:48.005 [QUIET] [system.out] &#64;drawable/add_schedule_fab_icon_anim : reachable=true
    16:25:48.009 [QUIET] [system.out]     &#64;drawable/ic_plus_anim_016
```

您现在需要知道为什么可执行到 @drawable/add_schedule_fab_icon_anim - 如果您向上搜索，就会发现“The root reachable resources are:”之下列有该资源。这意味着，存在对 add_schedule_fab_icon_anim 的代码引用（也就是说，在可执行到的代码中找到了它的 R.drawable ID）。

如果您使用的不是严格检查，则存在看似可用于为动态加载的资源构建资源名称的字符串常量时，可将资源 ID 标记为“可执行到”。在这种情况下，如果您在编译输出中搜索资源名称，可能会发现如下消息：

```
10:32:50.590 [QUIET] [system.out] Marking drawable:ic_plus_anim_016:2130837506
        used because it format-string matches string pool constant ic_plus_anim_%1$d.
```

如果您看到一个这样的字符串，并且您能确定该字符串未用于动态加载给定资源，就可以按照有关如何自定义要保留的资源部分中所述，使用 tools:discard 属性通知编译系统将其移除。
