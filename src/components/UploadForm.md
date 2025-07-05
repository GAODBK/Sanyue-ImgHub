你希望将一个 `curl` 上传请求转换为 **Flutter/Dart 中的 Dio 上传风格**，用于 Flutter 开发项目，格式应与你给出的示例一致。

---

你提供的 `curl` 请求：

```bash
curl --progress-bar -X POST "https://ykjtb.xyz/upload?authCode=dfg&serverCompress=true&uploadChannel=telegram" \
     -H "Content-Type: multipart/form-data" \
     -H "Authorization: Basic $(echo -n "dfg:sfgnsgzvzcx" | base64)" \
     -F 'file=@"D:\code\bookstore\src\assets\9.jpg"'
```

---

## ✅ 对应的 Flutter (Dio) 实现代码如下：

```dart
import 'dart:io';
import 'package:dio/dio.dart';
import 'package:path/path.dart';

Future<void> uploadFile(File file, void Function(int)? onProgress) async {
  final dio = Dio();

  // 参数
  const authCode = 'dfg';
  const serverCompress = true;
  const uploadChannel = 'telegram';

  // 授权头部
  final username = 'dfg';
  final password = 'sfgnsgzvzcx';
  final basicAuth =
      'Basic ${base64Encode(utf8.encode('$username:$password'))}';

  // 表单数据
  final formData = FormData.fromMap({
    'file': await MultipartFile.fromFile(
      file.path,
      filename: basename(file.path),
    ),
  });

  try {
    final response = await dio.post(
      'https://ykjtb.xyz/upload',
      queryParameters: {
        'authCode': authCode,
        'serverCompress': serverCompress,
        'uploadChannel': uploadChannel,
      },
      data: formData,
      options: Options(
        headers: {
          'Authorization': basicAuth,
          'Content-Type': 'multipart/form-data',
        },
      ),
      onSendProgress: (int sent, int total) {
        int percentCompleted = ((sent / total) * 100).round();
        onProgress?.call(percentCompleted);
      },
    );

    print('Upload completed: ${response.data}');
  } catch (e) {
    print('Upload failed: $e');
  }
}
```

---

## 📝 使用说明：

* 参数 `authCode`、`serverCompress` 和 `uploadChannel` 会作为 URL 查询参数附加。
* 授权方式使用 HTTP Basic Auth。
* 上传进度通过 `onProgress(int percent)` 回调方式暴露，便于外部使用。
* `MultipartFile.fromFile` 会自动处理文件路径和内容。
* 记得在 `pubspec.yaml` 中添加依赖：

```yaml
dependencies:
  dio: ^5.0.0
  path: ^1.8.0
```

你提供的是一段 JavaScript 中用于上传文件的函数，现在希望将其**迁移为 Flutter 项目使用**。Flutter 中一般使用 `dio` 包来处理上传文件、进度监听等逻辑。下面是一个完整的等效 Dart 版本（假设你使用的是 `dio` 和 `cookie_jar` / `shared_preferences` 等管理登录状态和 cookie）：

---

### ✅ **Flutter 版本的等效代码（使用 `dio` 上传）**

```dart
import 'dart:io';
import 'package:dio/dio.dart';

class FileUploadManager {
  List<FileItem> fileList = [];
  List<FileItem> waitingList = [];
  List<FileItem> exceptionList = [];
  int maxUploading = 3;

  final Dio dio = Dio();

  int get uploadingCount => fileList.where((f) => f.status == 'uploading').length;
  int get waitingCount => waitingList.length;

  bool uploading = false;
  String uploadChannel = "default";

  Future<void> uploadFile(FileItem file) async {
    // 文件是否存在于 fileList 中
    if (!fileList.any((item) => item.uid == file.uid)) {
      return;
    }

    if (uploadingCount >= maxUploading) {
      waitingList.add(file);
      file.status = 'waiting';
      return;
    } else {
      file.status = 'uploading';
    }

    String? authCode = await getAuthCode(); // 从 SharedPreferences 或其它方式获取
    String serverCompress = file.serverCompress.toString();

    FormData formData = FormData.fromMap({
      'file': await MultipartFile.fromFile(file.path, filename: file.filename),
    });

    try {
      await dio.post(
        '/upload',
        queryParameters: {
          'authCode': authCode,
          'serverCompress': serverCompress,
          'uploadChannel': uploadChannel,
        },
        data: formData,
        onSendProgress: (int sent, int total) {
          int percentCompleted = ((sent / total) * 100).round();
          file.onProgress?.call(percentCompleted);
        },
      );

      file.onSuccess?.call(file);
    } on DioException catch (e) {
      if (e.response?.statusCode == 401) {
        waitingList.clear();
        fileList.clear();
        exceptionList.clear();
        showError('认证状态错误！');
        redirectToLogin();
      } else {
        exceptionList.add(file);
        file.onError?.call(e);
      }
    } finally {
      if (uploadingCount + waitingCount == 0) {
        uploading = false;
      }
    }
  }

  Future<String?> getAuthCode() async {
    // 替换为你实际的 authCode 获取逻辑
    // 比如从 SharedPreferences 中读取
    return "your_auth_code_here";
  }

  void showError(String msg) {
    // 弹出提示框或 Toast
    print("Error: $msg");
  }

  void redirectToLogin() {
    // 使用 Navigator 重定向
    // Navigator.of(context).pushNamed('/login');
  }
}

class FileItem {
  String uid;
  String path;
  String filename;
  bool serverCompress;
  String status;

  Function(int percent)? onProgress;
  Function(FileItem file)? onSuccess;
  Function(Object error)? onError;

  FileItem({
    required this.uid,
    required this.path,
    required this.filename,
    this.serverCompress = false,
    this.status = 'pending',
    this.onProgress,
    this.onSuccess,
    this.onError,
  });
}
```

---

### 📌 说明：

| 原 JavaScript              | Flutter Dart 替换                      |
| ------------------------- | ------------------------------------ |
| `axios.post(...)`         | `dio.post(...)`                      |
| `FormData`                | `FormData` (from `dio`)              |
| `file.file`               | `file.path`                          |
| `file.onProgress(...)`    | `file.onProgress(percent)`           |
| `file.onSuccess(...)`     | `file.onSuccess(file)`               |
| `file.onError(...)`       | `file.onError(error)`                |
| `cookies.get('authCode')` | `getAuthCode()` (读取保存的 cookie/token) |

---

### ✅ 使用：

你可以在需要上传的时候这样调用：

```dart
FileItem file = FileItem(
  uid: '123456',
  path: '/path/to/file.jpg',
  filename: 'file.jpg',
  serverCompress: true,
  onProgress: (percent) {
    print("上传进度: $percent%");
  },
  onSuccess: (file) {
    print("上传成功: ${file.filename}");
  },
  onError: (error) {
    print("上传失败: $error");
  },
);

FileUploadManager().uploadFile(file);
```

---

如果你使用的是 `file_picker` 或 `image_picker` 选择文件，请告诉我，我可以再补充前端文件选择逻辑。是否也需要上传队列或多个任务并发控制？欢迎继续细化需求。

