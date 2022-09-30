---
title: "Asynctask"
date: 2022-09-10T20:23:16+06:00
---

# AsyncTask
> This tutorial on AsyncTask is for the kotlin programming language. It can easily be adapted for the java programming language

A program running on a single thread performs its tasks synchronously - meaning it waits till one task is complete till starting with another. On an android application UI runs on the main thread. The rendering of UI takes place in microsecond range. So for tasks like network calls that take significantly more time (in the millisecond range), the UI thread would be held up while the other task is finished if both were run on the main thread.

This is where AsyncTask is needed. It hands the secondary task to another thread and waits for a callback upon completion of the task.

For this tutorial we shall use AsyncTask to make a “GET” request to a API and see the result

## Declaration
AsyncTask is an abstract generic class that needs to be implemented to use its functionality. To make a class extend AsyncTask, the generic class needs three parameters - 

-   *Parameters* - to pass onto any parameter into class, used as parmeters for the network call
-   *Progress* - To keep track of progress updates
-   *Result* - Format of the result expected

We declare class *ApiCall* to implement our *AsyncTask.* The argument types are taken in order *<Params, Progress, Result>*. We pass a string parameter and expect a string returned from the API. So declare the class like
```kotlin
class ApiCall: AsyncTask<String, Void, String>() {  
  
}
```

AsyncTask is an abstract method. We need to override some of the methods in the class. The only method compulsory to override is *doInBackground.* This method accepts a variable list of “Params” type arguments and returns the result. We passed a String type for the parameters so our *doInBackground* has a variable list of strings. We expect a JSONObject responce from out api, so the *Result* is JSOBObject type.

```kotlin 
class ApiCall: AsyncTask<String, Void, JSONObject>() {  
    override fun doInBackground(vararg param: String?): JSONObject? {  
        return null  
    }
}
```

This method does the heavy lifting of the network call and actions inside it are executed in the background while the UI thread keeps running. There are also some very useful methods that can be implemented inside the class. We shall overriding the following methods for our this tutorial

-   *onPreExecute*: Takes no arguments as parameter and returns nothing. This method is executed immediately before the *doInBackground* that we have overridden. We can use this to perform any task to set up the process.
-   *onPostExecute*: Takes one single argument of type “Result” (String in our case) and returns nothing. It is executed immediately after *doInBackground*. This is used to take actions after receiving the result - like updating the UI.
-   *onProgressUpdate*: This takes one single “Progress” (Void in our case) type argument and returns nothing. It is used to keep tabs on the progress of the task in this thread. Its actions are discussed later.

The skeleton of our *ApiCall* now looks like this

```kotlin
class ApiCall: AsyncTask<String, Void, JSONObject>() {  
  
    override fun onPreExecute() {  
       
    }  
  
    override fun doInBackground(vararg param: String?): JSONObject? {  
  
        return null  
    }  
  
    override fun onProgressUpdate(vararg values: Void?) {  
         
    }  
  
    override fun onPostExecute(result: JSONObject?) {  
  
    }  
  
}
```

## Making a network call 
For this tutorial we use the free online mangadex api. We will make a simple GET request to get a list of manga URLs. Kotlin uses the java.net packages to make simple network calls. 

URL to mangadex - https://api.mangadex.org

**To make a network call your application must have network permissions enabled. Inside the AndroidManifest.xml. Your xml file should look like this**
```xml
<manifest>

<uses-permission android:name="android.permission.INTERNET" />  
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

...

...

...

</manifest>
```

For the API call we store the link to the API in a constant and keep a variable to store the expected result value. Let’s make a request to the “/manga” endpoint of the api. We open the string as an URL and try to open a connection as a HTTPUrlConnection (from java.net package)

```kotlin
	val tag = "API - call"  
    val api_url = "https://api.mangadex.org"
  
    override fun doInBackground(vararg param: String?): String {  
        val endpoint = param[0]  
        // https://api.mangadex.org/manga  
        val url = URL("$api_url/$endpoint")  
  
        val conn = url.openConnection() as HttpURLConnection  
        conn.requestMethod = "GET"  
        conn.connect()  

		val output = StringBuilder()
		
        return output.toString();  
    }
```

If the connection is established successfully the *conn* object produces an *inputStream* that we can read using basic IO

```kotlin
override fun doInBackground(vararg param: String?): JSONObject {  
	  val url = URL("$api_url/${param[0]}")  
	  val conn = url.openConnection() as HttpURLConnection  
	  conn.requestMethod = "GET"  
	  conn.connect()  
	
	  val inputStream = InputStreamReader(conn.inputStream)  
	  val reader = BufferedReader(inputStream)  
	
	  val output = StringBuilder() 
	
	  var line: String?;  
	  while(reader.readLine().also { line = it } != null) {  // reader sends api responce as input stream
	      output.append(line)  
	  }  
	
	  return JSONObject(output.toString())  
}
```

We create an *inputStream* object from the connection and read it using a *BufferedReader.* For every line we read, we append it onto the output object we declared earlier. Finally we return the output as a string since it’s our result. 

To view the result, we can print the result inside the *onPostExecute* method to check if the network call returned our expected value. This method is called immediately after the *doInBackground* method with the returned result.

```kotlin
override fun onPostExecute(result: JSONObject?) {  
  	Log.i(tag, result!!.toString())  
}
```

Expected Log can be seen on this link: https://api.mangadex.org/manga

During the *doInBackground* method we could keep track of the progress of the network call. Say we pass a list of endpoints and are making calls to each one of them. We can keep updating the progress by calling the *publishProgress(Progress)* method.

```kotlin 
override fun doInBackground(vararg param: String?): JSONObject {  
	  val url = URL("$api_url/${param[0]}")  
	  val conn = url.openConnection() as HttpURLConnection  
	  conn.requestMethod = "GET"  
	  conn.connect()  
	
	  val inputStream = InputStreamReader(conn.inputStream)  
	  val reader = BufferedReader(inputStream)  
	
	  val output = StringBuilder()  
	
	  var line: String?;  
	  while(reader.readLine().also { line = it } != null) {  
	      publishProgress() // PROGRESS UPDATE  
	      output.append(line)  
	  }  
	  Log.i(tag, output.toString())  
	  return output.toString();  
}
```

Every time we call the *publishProgress()* method, the task calls the *onProgressUpdate* method. To every time we receive a new line we can update the user on the progress

```kotlin 
override fun onProgressUpdate(vararg values: Void?) {  
  	Log.i(tag, "New Line Received")  
}
```


The task can also be canceled at any point in time by calling the cancel(boolean) method. This will execute the *onCancelled()* methode. It is always a good idea to override this methode

```kotlin
override fun onCancelled () {
	Log.e(tag, "API call was cancelled")
}
```

We can call make this task execute from the UI thread by using an object of this class and calling it’s *execute()* method. To make a network call on the press of a button we could do the following

```kotlin
val fetchButton: Button = findViewById(R.id.mangadexfetch)  
fetchButton.setOnClickListener {  
  	ApiCall().execute("manga")  
}
```

This passes the argument “manga” into the AsyncTask and keeps it functioning on another thread.

## Communicatiung with other threads

> This part of the tutorial is helpful only if you need the AsyncTask responce elsewhere.

After or during the network call, the thread from which it is called might need to communicate with the AsyncTask thread. Case in point - we want our users to see the data that we fetched from the api in the main thread. To allow the asyncTask to communitate with the main thread, we pass the parent object into the AsyncTask object. We need an interface to help with that.

Create a generic interface *NetworkCalled* contains the methodes *onNetworkCallSuccess(Result)* and *onNetworkCallFail()*.

```kotlin
public interface NetworkCaller<T> {
    void onNetworkCallSuccess(T t);
    void onNetworkCallFail();
}
```

The caller object must override this interface and pass itself onto the AsyncTask object that is creates to allow the AsyncTask to execute these methodes when they need. Modify the caller activity to override the *NetworkCaller* interface. Now we can pass the caller object into the AsyncTask.

```kotlin
class MainActivity : AppCompatActivity(), NetworkCaller<JSONObject> { // implenets the NetworkCaller interface with the expected result type *JSONObject*

    private val tag = "Main";

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val apicallbutton = findViewById<Button>(R.id.presstoinvoteapi);
        apicallbutton.setOnClickListener {
            ApiCall(this).execute("manga") // passing *this* into ApiCall object
        }
    }

	// Overrides NetworkCaller methodes
    override fun onNetworkCallSuccess(result: JSONObject?) {
        Log.i(tag, result.toString())
    }

    override fun onNetworkCallFail() {
        Log.e(tag, "Network Call Failed")
    }
}
```

The ApiCall initialization now looks like `class ApiCall(val parent: NetworkCaller<JSONObject>): AsyncTask<String, Void, JSONObject>()` - Accepting a parent object that implements NetworkCaller.  We can now handle post_execute and network cancellations differently

```kotlin
// inside the ApiCall class
override fun onPostExecute(result: JSONObject?) {
	parent.onNetworkCallSuccess(result)
}

override fun onCancelled () {
	parent.onNetworkCallFail()
}
```

## Cautions

1. **The methods *onPreExecute(), onPostExecute(Result), doInBackground(Params...) and onProgressUpdate(Progress...)* SHOULD NEVER BE CALLED MANUALLY.  Use only the *execute()* method to start the task.** 

2. **Tasks can be created and executed on the UI thread only.**

3. **A Task can be executed only once.**

## Errors to Expect
If the network connection cannot be established, a runtime exception  is thrown  
```bash
AndroidRuntime: FATAL EXCEPTION: AsyncTask #1
Process: com.tsunderead.kotlin_api, PID: 8732
```
> This can be due to a wrong api URL or a weak internet
> Verify the URL and check internet connection

Any critical error calls the onCancelled() method. Overriding this method is a good practice to debug network call errors.

## Referenes
1. [AsyncTask theory](https://youtu.be/zHGgSd1wvxY)
2. [AsyncTask implementation](https://youtu.be/EThkglxLxSM)
3. [AsyncTask documentation](https://developer.android.com/reference/kotlin/android/os/AsyncTask)
4. [HttpURLConnection documentation](https://developer.android.com/reference/kotlin/java/net/HttpURLConnection)

## Git Repository 
https://github.com/master-da/kotlin-api
