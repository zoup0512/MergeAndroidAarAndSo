# AAR + SO 一键合并工具（merge.bat）

本目录提供一个 **单文件** Windows 工具 `merge.bat`，用于把多个 `*.aar` 与 `*.so` 合并为一个新的 `*.aar`。

## 1. 运行方式（无需参数）

在 `merge.bat` 同级目录执行：

```powershell
.\merge.bat
```

默认输出：

- `merged.aar`（生成在 `merge.bat` 同级目录）

## 2. 目录约定（自动模式）

`merge.bat` 默认采用 **自动模式**：不需要显式指定输入文件。

请按以下固定结构放置输入：

- `merge\libs\`：放置 **AAR/JAR**
  - `merge\libs\*.aar`
  - `merge\libs\*.jar`
- `merge\jniLibs\`：放置 **SO**（建议按 ABI 分目录）
  - `merge\jniLibs\arm64-v8a\*.so`
  - `merge\jniLibs\armeabi-v7a\*.so`
  - `merge\jniLibs\x86\*.so`
  - `merge\jniLibs\x86_64\*.so`

说明：

- `merge\libs\` 下的 `*.aar` 会被全部合并。
- `merge\libs\` 下的 `*.jar` 会被复制进输出 AAR 的 `libs/` 目录。
- `merge\jniLibs\` 下的 `*.so` 会被打进输出 AAR 的 `jni/<abi>/`。

## 3. 手动模式（可选）

如果你不想用自动目录结构，也可以显式传参：

```bat
merge.bat [-o out.aar] [-m AndroidManifest.xml] [-abi arm64-v8a,armeabi-v7a] [-so <soDirOrSoFile>]... <in1.aar> <in2.aar> ...
```

示例：

```powershell
.\merge.bat -o out.aar libs\A.aar libs\B.aar -so jniLibs
```

## 4. 合并策略（实现细节）

- **AndroidManifest.xml**
  - 若指定 `-m`：使用你指定的 manifest
  - 否则：使用第一个输入 AAR 中的 `AndroidManifest.xml`

- **classes.jar**
  - 会把所有输入 AAR 的 `classes.jar` 解包后合并，再重新打成一个 `classes.jar`

- **res/**、**assets/**
  - 目录级合并
  - 同名文件冲突：后合并的文件会覆盖先前文件

- **jni/<abi>/*.so**
  - 从输入 AAR 自带的 `jni/` 合并
  - 额外从 `-so`（目录或单个 so 文件）合并
  - 自动模式下默认从 `merge\jniLibs\` 读取

- **其他文件**
  - `R.txt` / `public.txt` / `proguard.txt` / `consumer-rules.pro`：若存在则拷贝一份（先到先得）

## 5. 依赖要求

- Windows PowerShell（系统自带即可）
- JDK 的 `jar.exe`
  - 建议确保 `jar.exe` 在 `PATH` 中
  - 或设置 `JAVA_HOME`，脚本会尝试使用 `JAVA_HOME\bin\jar.exe`

## 6. 常见问题排查

- **报错：`cannot find jar.exe`**
  - 安装 JDK 并确保 `jar.exe` 可用
  - 或设置 `JAVA_HOME` 指向 JDK 根目录

- **报错：`no input aars specified`**
  - 自动模式下请确认：`merge\libs\` 下至少有一个 `*.aar`

- **输出 AAR 里没有 SO**
  - 确认 `merge\jniLibs\<abi>\` 下有对应 `*.so`
  - 或使用 `-so` 显式指定目录/文件
