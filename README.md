# Module CoreNetwork in ViettelPay App
This is guide setup and usage module corenetwork in app ViettelPay
1. Implementation module core network in build.gradle module feature
```kotlin
implementation(project(BuildModules.Core.NETWORK))
```
2. Create file API Service in module feature
```kotlin
interface ApiService {
  // format for call api with SOAP API(old server mobile service)
  @POST("ServiceAPI")
  suspend fun getListServicesMobileMoneyCoroutine(@Body requestBody: RequestBody): GenericXmlResponse<MMListServicesResponse>

  @POST("ServiceAPI")
  fun getListServicesMMRx(@Body requestBody: RequestBody): Observable<GenericXmlResponse<MMListServicesResponse>>

  // format for call api with REST API(server CDCN)
  @POST("customer/v1/accounts/change-pin")
  fun changePinRx(
    @Body request: ChangePinRequest
  ): Observable<GenericResponse<ChangePinResponse>>

  @POST("customer/v1/accounts/change-pin")
  suspend fun changePinCoroutine(
    @Body request: ChangePinRequest
  ): GenericResponse<ChangePinResponse>

}
```
- ``` suspend fun ``` is keyword of kotlin coroutines so if you want to call api with coroutine then you need using keyword suspend before function.
NOTE: Call API with format request Xml then need using object GenericXmlResponse and the request is JSON then using GenericResponse
- ```MMListServicesResponse ``` is object response once call api successful(call api with format Xml). It needs to extend XmlBaseResponse
- ```ChangePinResponse``` is object response once call api succesful(call api with format Json) and need extend BaseResponse

3. Create file DI or declared dependency for ApiService in your file DI
```kotlin
val networkDependency = module {
  single(named("Xml")) { provideApi<ApiService>(retrofit = get(named("Xml"))) }
  single{ provideApi<ApiService>(retrofit = get())}
}
```
You need to create file DI or add 2 lines above to the file DI
At here definition 2 instances of ApiService. With name is Xml then instance of ApiService will call api with format SOAP XML and default for call api with REST API

4. Load module in activity or fragment runs first
```kotlin
private val loadModuleAPI by lazy { loadKoinModules(networkDependency) }

override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)
    loadModuleAPI
    // init something at here
  }
```
Now, You can create files Repository in your module. 
Declared both instance ApiService REST and SOAP API.
Differences: testApiXml need by inject with named is "Xml"(We have definition it in the DI file). And testApiJson is by inject default 
``` kotlin
class TestRepository : KoinComponent {
  val testApiJson: ApiService by inject()
  val testApiXml: ApiService by inject(named("Xml"))
}
```
In the file Repository, You can call api and return data for layer Use Case following architecture in ViettelPay App
## Code example:
1. Call API with SOAP API(Old Server)
Set data to Request object and convert to RequestBody
``` kotlin
val request = Request(context, "GET_LIST_SERVICE_V2", Data("flag_update_service_client", "0"), Data("is_new", "1"))
val requestBody = Toolbox.convertRequestBodyXml(request)
```
Note: Request and Toolbox import from corenetwork. Dont import with Request and Toolbox in app

Case call api with Kotlin Coroutine. You need execute in suspend function or Coroutine Scope.
```kotlin
val responseXml = testApiXml.getListServicesMobileMoneyCoroutine(requestBody)
    when (responseXml) {
      is NetworkResponse.Success<*> -> {
        /*do something with response success*/
      }
      is NetworkResponse.Unauthenticated<*> -> {
        /*do something at here*/
      }
      is NetworkResponse.Error<*> -> {
        /*do something at here*/
      }
      else -> {
        /*do something at here*/
      }
    }
```
Case call api with RxKotlin
```kotlin
testApiXml.getListServicesMobileMoneyRx(requestBody)
      .subscribeOn(Schedulers.io())
      .observeOn(AndroidSchedulers.mainThread())
      .subscribe { response: GenericXmlResponse<MMListServicesResponse> ->
        when (response) {
          is NetworkResponse.Success<*> -> {
            /*do something with response success*/
          }
          is NetworkResponse.Unauthenticated<*> -> {
            /*do something at here*/
          }
          is NetworkResponse.Error<*> -> {
            /*do something at here*/
          }
          else -> {
            /*do something at here*/
          }
        }
      }
```

2. Call API with REST API(server CDCN)
Declared request object:
```kotlin
val request = ChangePinRequest(
                currentPin = password.value.getText(),
                newPin = firstPassword.value.getText(),
                confirmPin = secondPassword.value.getText()
        )
```
Case call api with coroutine:
```kotlin
val response = testApiJson.changePinCoroutine(request)
    when (response) {
      is NetworkResponse.Success<*> -> {
        /*do something with response success*/
      }
      is NetworkResponse.Unauthenticated<*> -> {
        /*do something at here*/
      }
      is NetworkResponse.Error<*> -> {
        /*do something at here*/
      }
      else -> {
        /*do something at here*/
      }
    }
```
Case call api with Rx
```kotlin
testApiJson.changePinRx(request)
      .subscribeOn(Schedulers.io())
      .observeOn(AndroidSchedulers.mainThread())
      .subscribe { response: GenericResponse<ChangePinResponse> ->
        when (response) {
          is NetworkResponse.Success<*> -> {
            /*do something with response success*/
          }
          is NetworkResponse.Unauthenticated<*> -> {
            /*do something at here*/
          }
          is NetworkResponse.Error<*> -> {
            /*do something at here*/
          }
          else -> {
            /*do something at here*/
          }
        }
      }
```

```xml
 <vn.viettelpay.views.VDSToggles
      android:id="@+id/toggle"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:layout_gravity="center_horizontal" />
```
Kotlin 
```kotlin
viewBinding.toggle.isChecked = true
    viewBinding.toggle.setOnCheckedChangeListener(object : VDSToggles.OnCheckedChangeListener{
      override fun onCheckedChanged(view: VDSToggles?, isChecked: Boolean) {
        Toast.makeText(context,"Status is $isChecked",Toast.LENGTH_SHORT).show()
      }
    })
```

