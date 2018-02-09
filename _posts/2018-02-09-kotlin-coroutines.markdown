---
layout: post
title:  "Kotlin : Getting started with coroutines"
date:   2018-02-09 15:16:45 +0530
categories: Kotlin, Android
---
A lot of times we need do asynchronous tasks & we wait for the result. Most famous use case for that would be API calls, some background image processing etc. Traditionally we do it in a callback way. We create a task, we add a callback & wait for a result.
Something like this in a orthodox java way:

{% highlight java %}
public void createUser(User user) {
  // Some code
  mUserPresenter.createUser(user, new OnUserCreatedListener() {
    @Override
    public void onSuccess() {

    }

    @Override
    public void onError() {

    }
  })
}
{% endhighlight %}

This is kid's stuff. But what if we are doing something more complex. More like a nested callbacks. Which we typically end up doing
{% highlight java %}
public void createUser(User user) {
  // Some code
  mUserPresenter.createUser(user, new OnUserCreatedListener() {
    @Override
    public void onSuccess() {
      // Some code
      mUserPresenter.loginUser(post, new OnLoginSuccessListener() {
        @Override
        public void onSuccess() {
          // Some Code
        }

        @Override
        public void onError() {
          // Some code
        } // <--- Notice callback hell
      }) // <--- Notice callback hell
    }

    @Override
    public void onError() {

    }
  })
}
{% endhighlight %}

It is not wrong to do the things with nested approach. But eventually code becomes messy. You can google 'Callback hell' & see what it means.
RxJava solves this nesting problem but not completely. There is still combinators. Also, error handling becomes difficult in that case. Wouldn't it be very helpful & easy if these calls looked like a normal function. For example :
{% highlight java %}
public void createUser(User user) {
  // Some code
  User newUser = mUserPresenter.createUser(user);
  boolean success = mUserPresenter.loginUser(newUser);
  // some code
}
{% endhighlight %}

Kotlin does exactly this. But how? Thanks to coroutines. They provide a way to avoid a blocking of a thread. Everything about concurrency is taken care by complex Kotlin libraries. The code cab be still written sequentially & Kotlin APIs will figure out what to & how to to handle the asynchronous methods.

Okay, how does Kotlin do it? Obviously there is no magic. You are responsible to write the code. To avoid blocking of a method you have to make it "async" or mark it as "suspend".

Suspend :
{% highlight Kotlin %}
suspend fun createUser(): User {
  // ... some code
}

suspend fun login(): Boolean {
  // ... some code
}
{% endhighlight %}

Above method will not block the thread instead it will run concurrently. But there is a catch, you can't call 'suspended' method from a non-suspended method.
So how can you call it? Simple, using Kotlin builders. You can user 'launch(context)' method. Here is how code will look like.
{% highlight Kotlin %}
fun createAndLoginUser(user: User) {
  // Some code
  launch {
    val createdUser = createUser(user)
    val login = login(createdUser)
  }
  // Some more code
}
{% endhighlight %}

If you take a look at definition of 'launch' method, it takes a 'block: suspend CoroutineScope.() -> Unit' & a coroutine context. This will make createUser & login methods run on a common-pool. You can control on which these methods would run on. Like following :
{% highlight Kotlin %}
suspend fun login(): Boolean = run(CommonPool) { // This is why I love Kotlin
  // ... some code
}
{% endhighlight %}

Okay. Now you must be wondering what if I want to do some UI based tasks after user is created, like notifying user. You just need to pass the UI context or to a launch method.
{% highlight Kotlin %}
fun createAndLoginUser(user: User) {
  // Some code
  launch(UI) {  // 'UI' is Android specific context.
    val createdUser = createUser(user)
    notifyUserCreated() // UI specific task
    val login = login(createdUser)
  }
  // Some more code
}
{% endhighlight %}

Launch a coroutine with UI context which guarantees these things will be called on the UI thread but the suspending function that grabs the user from the API call will not block the UI thread. Meaning, these methods will called sequentially on UI thread, only suspended methods will run concurrently without blocking main thread. But control will still wait for these methods to return a result.

This is when Kotlin comes handy. Code looks clean & more readable giving more clarity & freedom to code. Please take a note that coroutines are experimental from Kotlin 1.1+ verion.
There are many more things in coroutines like async-await. We will talk about them in later post.
