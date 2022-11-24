```kotlin
    @Test
    fun test01() {
        val testUrl = "https://my-slack-url"
        val message = "test ${LocalDateTime.now()}"

        val service = SlackApiService()
        changeValue(SlackApiService::class.java, service, "url", testUrl)
        service.sendMessage(message)
    }

    private fun <T> changeValue(clazz: Class<T>, obj: Any, name: String, value: Any) {
        val privateField: Field = clazz.getDeclaredField(name)
        privateField.isAccessible = true
        privateField.set(obj, value)
    }
```