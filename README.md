# BÀI 2: BIÊN DỊCH ĐÓNG GÓI TỆP ỨNG DỤNG .JAR VỚI GRADLE VÀ LƯU TRỮ ARTIFACTS

- **Môn học:** DevOps / CI-CD
- **Bài thực hành:** Bài 2 - Session 11
- **Mục tiêu:** 
  - Thực thi câu lệnh biên dịch và đóng gói ứng dụng Spring Boot thành tệp thực thi `.jar` bằng Gradle (`./gradlew bootJar`).
  - Lưu trữ thành phẩm biên dịch (`.jar`) từ thư mục `build/libs/` lên GitHub Actions bằng `actions/upload-artifact@v4`.
- **Đường dẫn nộp bài:** `homework/session_11/ex2/`

---

## 1. Cấu trúc thư mục dự án

```text
PTIT_CNTT1_IT209_Session11_Bai02/
├── .github/
│   └── workflows/
│       └── ci.yml                         # Kịch bản GitHub Actions Workflow (đã thêm bootJar & upload-artifact)
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar             # Gradle wrapper executable JAR
│       └── gradle-wrapper.properties      # Cấu hình Gradle distribution (8.8)
├── src/
│   ├── main/java/com/example/demo/
│   │   └── DemoApplication.java          # Spring Boot Application entrypoint
│   └── test/java/com/example/demo/
│       └── DemoApplicationTests.java      # JUnit 5 Unit Tests
├── .gitignore                             # Loại trừ build, .gradle, IDE files
├── build.gradle                           # Cấu hình Spring Boot 3.3.4, Java 17, JUnit
├── gradlew                                # Script chạy trên Linux/macOS
├── gradlew.bat                            # Script chạy trên Windows
├── settings.gradle                        # Định danh root project
└── README.md                              # Tài liệu hướng dẫn chi tiết
```

---

## 2. Chi tiết kịch bản GitHub Actions Workflow (`.github/workflows/ci.yml`)

Workflow được bổ sung thêm 2 bước: đóng gói tệp JAR và đính kèm Artifact:

```yaml
name: Java Spring Boot CI (Gradle)

on:
  push:
    branches: [ "main" ]

jobs:
  test:
    name: Run Unit Tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v5

      - name: Thiết lập môi trường Java JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Cấp quyền cho Gradle Wrapper
        run: chmod +x gradlew

      - name: Thực thi JUnit Test với Gradle
        run: ./gradlew test

      - name: Đóng gói tệp JAR với Gradle
        run: ./gradlew bootJar

      - name: Lưu trữ thành phẩm JAR Artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: build/libs/*.jar
```

### Phân tích các bước mới được bổ sung:
1. **Bước `Đóng gói tệp JAR với Gradle` (`run: ./gradlew bootJar`)**:
   - Biên dịch toàn bộ mã nguồn của dự án Spring Boot.
   - Đóng gói toàn bộ dependencies và mã nguồn thành một tệp thực thi `.jar` hoàn chỉnh (fat/uber jar) nằm trong thư mục `build/libs/`.
2. **Bước `Lưu trữ thành phẩm JAR Artifact` (`uses: actions/upload-artifact@v4`)**:
   - Sử dụng action chính thức của GitHub để lưu trữ file kết quả của workflow.
   - `name: app-jar`: Định danh tên của Artifact xuất hiện trên trang Summary của Workflow Run.
   - `path: build/libs/*.jar`: Đường dẫn tệp tin cần tải lên lưu trữ.

---

## 3. Kiểm thử và xác nhận hoạt động

### 3.1. Chạy đóng gói cục bộ
- Lệnh thực thi: `./gradlew bootJar`
- Kết quả: Sinh ra tệp `build/libs/demo-spring-boot-ci-0.0.1-SNAPSHOT.jar` sẵn sàng chạy bằng lệnh `java -jar`.

### 3.2. Kiểm tra trên GitHub Actions (sau khi push)
1. Đẩy mã nguồn lên nhánh `main` của GitHub Repository.
2. Truy cập tab **Actions** trên GitHub.
3. Chờ workflow **Java Spring Boot CI (Gradle)** chạy hoàn tất (tích xanh).
4. Vào trang chi tiết lần chạy (**Workflow Run Summary**), cuộn xuống mục **Artifacts**.
5. Xác nhận hiển thị artifact **`app-jar`** và có thể click để tải về máy tính.
