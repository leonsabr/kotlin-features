//TODO 5, 9, 10, 14, 15 (=5)

1. Создание и заполнение коллекций (list и map).
  **Профит:** лаконичность кода.
    ```
    val usersWithSameEmailAndSimilarName = listOf(
            User("john.smith", "John Smith", "js@jetbrains.com"),
            User("j.smith", "j.smith", "js@jetbrains.com"),
            User("smith", "smith", "js@jetbrains.com")
    )
    ```
    ```
    mapOf(
        projectA.name to listOf(developerRole.name, projectAdminRole.name),
        projectB.name to listOf(developerRole.name)
    )
    ```
2. Работа с коллекциями: фильтрация, трансформация, поиск. Если у тестировщиков обычно нет необходимости использовать stream api, то они ограничиваются for или forEach. У kotlin богатая стандартная библиотека методов для коллекций.
  **Профит:** просто удобно и наглядно.
    ```
    val license = licenses.firstOrNull { it.serviceName.text == serviceName }
    ```
    ```
    val lastLicense = licenses.last()
    ```
    ```
    val usersWithLicense = users.filter { it.hasLicense }
    ```
    ```
    val guthubUserDetails = user.details.filterIsInstance<GithubUserDetails>
    ```
    ```
    val links = menuItems.map { it.link.text }
    ```
3. Extension функции как замена статическим методам.
  **Профит:**
      * автокомплит в среде разработки
      * более единообразный код (как методы объекта java)
      * можно чейнить .f1(...).f2(...) вместо f2(f1(...), ...)
    ```
    fun <T : HtmlElement> T.waitForIt(
            message: String? = null,
            exists: Boolean = true,
            isEnabled: Boolean? = null,
            timeout: Period = Period.seconds(10)
    ) = apply {
        ...
    }

    fun <T : HtmlElement> T.attachScreenshot(name: String = "$this") = apply {
        ...
    }

    editLicenseDialog
            .waitForIt()
            .attachScreenshot()
            .setNewLicense(licenseName, licenseKey)
    ```

4. data классы
  **Профит:** лаконичность кода.
    ```
    data class User(val login: String, val username: String, val password: String, val email: String)
    ```
    **Про "apply { } и also { } как замена билдерам" нужно еще подумать**
5. Более простые лямбда-функции. Профит: лаконичность кода, можно писать DSL.
6. Нет checked exceptions.
  **Профит:** очевиден.
    ```
    val driver = RemoteWebDriver(URL(seleniumHub), ...)
    ```
7. String templates.
  **Профит:** нет конкатенации, нет возни с string builder.
    ```
    fun defaults(baseURL: String, listenPort: String) = listOf(
            "--$BASE_URL_PROP=$baseURL",
            "--listen-port=$listenPort",
            "--entropy-check=false"
    )
    ```
8. Дефолтные значения параметров функций вместо перегрузки методов.
  **Профит:** нет перегруженных методов.
    ```
    fun <T : HtmlElement> T.waitForIt(
            message: String? = null,
            exists: Boolean = true,
            isEnabled: Boolean? = null,
            timeout: Period = Period.seconds(10)
    ) = apply {
        ...
    }

    backupFinishedStatus.waitForIt(timeout = seconds(30))

    loader.waitForIt(exists = false)

    createBackupButton.waitForIt(isEnabled = true)
    ```
9. Null safety. Профит: компилятор заставляет добавить обработку потенциального NPE еще при компиляции.
10. Операторы `?.` и `?:`
11. `try/catch`, `if` и `when` являются выражениями (expression), то есть возвращают значения.
    ```
    fun <T : HtmlElement> T.staleSafeExists() = try {
        exists()
    } catch (e: StaleElementReferenceException) {
        false
    }
    ```
    ```
    videoUrl = if (sessionId == null || Config.isIE || Config.isLocalRun) {
        null
    } else {
        "http://${URL(seleniumHub).authority}/video/$sessionId"
    }
    ```
    ```
    val keys = when {
        isMacOS && isChrome -> Keys.chord(Keys.SHIFT, Keys.INSERT)
        isMacOS -> Keys.chord(Keys.COMMAND, "v")
        else -> Keys.chord(Keys.CONTROL, "v")
    }
    ```
12. Иногда `with(...) { }` может сделать код более лаконичным.
    ```
    //before
    createGroupDialog.waitForIt()
    createGroupDialog.groupNameInput.sendKeys(name)
    createGroupDialog.projectSelect.selectItem(project)
    createGroupDialog.attachScreenshot("New group dialog")
    createGroupDialog.confirmButton.click()

    //after
    with(createGroupDialog.waitForIt()) {
        groupNameInput.sendKeys(name)
        projectSelect.selectItem(project)
        attachScreenshot("New group dialog")
        confirmButton.click()
    }
    ```
13. Сложный синглетон в java -> простой object в koltin
    ```
    object Config {
        val hubImage: String? = System.getProperty("hubImage")
        val hubTag: String? = System.getProperty("hubTag")
        ...
    }
    ```
14. Геттер в ленивой инициализацией `by lazy { }`
15. reified type parameters могут сделать более читаемым парсинг json
16. передача функции в функцию через `::funcName`
    ```
    private fun generateOneTimePassword(secretKey: String) = TOTPProviders.googleAuthenticator()
            .createOneTimePassword(TOTPSecretKey.from(TOTPSecretKey.KeyRepresentation.BASE32, secretKey))
            .toString()

    userProfilePage.enableTwoFactorAuthentication(::generateOneTimePassword)

    fun enableTwoFactorAuthentication(codeGenerator: ((String) -> String)? = null) = apply {
       ...
       with(enableEnable2FADialog.waitForIt()) {
           secretKey = secretKeyText.text
           enterCode(codeGenerator(secretKey))
       }
    }
    ```