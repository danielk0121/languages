``` kotlin
val gson = Gson()
val typeToken = object : TypeToken<SearchApiRes<EsProduct>>() {}.type  // <-- java.lang.reflect.Type
gson.fromJson(resBody, typeToken)  // <-- gson 은 Class<T> 받거나 Type 받도록 2가지 메소드를 제공

data class SearchApiRes<T>(
    var took: Long? = null,
    @JsonProperty("timed_out")  // <-- jackson
    @SerializedName("timed_out")  // <-- gson
    var timedOut: Boolean? = null,
    var hits: HitOuter<T>? = null,
)
```
