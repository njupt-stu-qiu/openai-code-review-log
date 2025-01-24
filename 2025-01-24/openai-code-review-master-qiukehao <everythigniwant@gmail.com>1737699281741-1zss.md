# 小傅哥项目： OpenAi 代码评审.
### 😀代码评分：85
#### 😀代码逻辑与目的：
该代码实现了使用 GitHub Actions 在代码提交后自动进行代码审查的功能。它通过 Git diff 获取代码更改，使用 ChatGLM 进行代码审查，并将审查结果记录到 GitHub 的特定仓库中，并通过微信发送通知。

#### 🤔问题点：
1. **安全性问题**：在 `main` 方法中直接从环境变量中读取 GitHub 令牌，这可能导致令牌泄露。
2. **代码结构**：代码结构较为混乱，类和方法之间的依赖关系不明确。
3. **异常处理**：代码中缺少对潜在异常的捕获和处理。
4. **资源管理**：在 `sendPostRequest` 方法中，没有正确关闭 `HttpURLConnection` 和 `Scanner`，可能导致资源泄露。

#### 🎯修改建议：
1. **安全性**：将 GitHub 令牌存储在 GitHub Secrets 中，并在代码中通过环境变量安全地访问。
2. **代码结构**：重构代码，将功能分解为更小的、更易于管理的类和方法。
3. **异常处理**：添加必要的异常处理，确保代码的健壮性。
4. **资源管理**：确保所有资源在使用后都被正确关闭。

#### 💻修改后的代码：
```java
// 修改后的 main 方法
public static void main(String[] args) {
    try {
        String token = getEnv("GITHUB_TOKEN");
        if (null == token || token.isEmpty()) {
            throw new RuntimeException("token is null");
        }

        GitCommand gitCommand = new GitCommand(
                getEnv("GITHUB_REVIEW_LOG_URI"),
                token,
                getEnv("COMMIT_PROJECT"),
                getEnv("COMMIT_BRANCH"),
                getEnv("COMMIT_AUTHOR"),
                getEnv("COMMIT_MESSAGE")
        );

        WeiXin weiXin = new WeiXin(
                getEnv("WEIXIN_APPID"),
                getEnv("WEIXIN_SECRET"),
                getEnv("WEIXIN_TOUSER"),
                getEnv("WEIXIN_TEMPLATE_ID")
        );

        IOpenAI openAI = new ChatGLM(getEnv("CHATGLM_APIHOST"), getEnv("CHATGLM_APIKEYSECRET"));

        OpenAiCodeReviewService openAiCodeReviewService = new OpenAiCodeReviewService(gitCommand, openAI, weiXin);
        openAiCodeReviewService.exec();
    } catch (Exception e) {
        e.printStackTrace();
    }
}

// 修改后的 sendPostRequest 方法
private static void sendPostRequest(String urlString, String jsonBody) {
    HttpURLConnection conn = null;
    try {
        URL url = new URL(urlString);
        conn = (HttpURLConnection) url.openConnection();
        conn.setRequestMethod("POST");
        conn.setRequestProperty("Content-Type", "application/json; utf-8");
        conn.setRequestProperty("Accept", "application/json");
        conn.setDoOutput(true);

        try (OutputStream os = conn.getOutputStream()) {
            byte[] input = jsonBody.getBytes(StandardCharsets.UTF_8);
            os.write(input, 0, input.length);
        }

        try (Scanner scanner = new Scanner(conn.getInputStream(), StandardCharsets.UTF_8.name())) {
            String response = scanner.useDelimiter("\\A").next();
            System.out.println(response);
        }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        if (conn != null) {
            conn.disconnect();
        }
    }
}
```

#### 🌟代码优点：
- 使用了 GitHub Actions 进行自动化代码审查。
- 使用 ChatGLM 进行代码审查，提高了审查的效率和准确性。
- 将代码审查结果记录到 GitHub 仓库中，方便跟踪和查看。
- 通过微信发送通知，及时告知审查结果。