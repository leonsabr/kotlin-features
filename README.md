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
5. Более простые лямбда-функции.
  **Профит:** лаконичность кода, можно писать DSL.
    ```
    portMapping.map { PortBinding(bindPortSpec(it.key), tcp(Integer.parseInt(it.value))) }
    ```
    ```
    @Step("{0}")
    fun <T> step(title: String, body: () -> T) = body()

    step("Update user group") {
        api.userGroupDAO.update(groupId, UserGroupJSON().apply { name = newName })
    }
    ```
    ```
    api.userDAO.create(user("User_$runID") {
        groups = listOf(...)
        projectRoles = listOf(...)
    })
    ```
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
9. Null safety.
    **Профит:** компилятор заставляет добавить обработку потенциального NPE еще при компиляции.
    ```
    data class User (
            val username: String,
            val email: String?
    )

    data class Issue(
            val id: String,
            val project: Project,
            val summary: String,
            val description: String,
            val assignee: User?
    )

    val issue = // parse json string with Gson or another lib

    // тут компилятор скажет, что assignee - это nullable type
    issue.assignee.username
    ```

10. Операторы `?.` и `?:`
    ```
    usersTable.rows.firstOrNull { it.userNameLink.text == userName }
        ?.selectUserCheckbox
        ?.click()
        ?: fail("Can't find [$userName] user!")
    ```
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
    ```
    val api by lazy { RemoteAPI.create(accountsClient) }
    ```
    ```
    val browser by lazy { Browser(...) }
    ```
15. reified type parameters могут сделать более читаемым парсинг json
    ```
    // reified type parameter
    interface JSONConvertable {
        fun toJSON(): String = Gson().toJson(this)
    }

    inline fun <reified T: JSONConvertable> String.toObject(): T = Gson().fromJson(this, T::class.java)

    // data class
    data class User(
        val id: String,
        val login: String,
        val username: String) : JSONConvertable

    // usage
    val user = "{ ... }".toObject<User>()
    val json = user.toJSON()
    ```
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