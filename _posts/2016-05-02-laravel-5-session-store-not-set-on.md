---
layout: post
title: Laravel 5 - "Session store not set on request" running functional tests
date: '2016-05-02T11:53:00.000+02:00'
author: Torben Hansen
tags:
- Laravel 5
- disable middleware
- functional tests
modified_time: '2016-05-02T11:53:05.276+02:00'
blogger_id: tag:blogger.com,1999:blog-6517038209122183182.post-3298029146974191869
blogger_orig_url: http://www.derhansen.de/2016/05/laravel-5-session-store-not-set-on.html
permalink: /2016/05/laravel-5-session-store-not-set-on.html
---

tl;dr - If your Laravel functional tests fail with **"RuntimeException: Session store not set on request."**, don't use
the **withoutMiddleware trait** in your tests and **selectively disable middleware** components you don't need when
executing tests by using APP\_ENV in your middleware.

In a Laravel 5.2 project I had some initial problems setting up the functional tests. The test code was really simple as
shown below:

{% highlight php %}
/**
 * Just a simple functional test case
 *
 * @test
 */
public function indexTest()
{
    $this->visit('/')
        ->see('some text');
}
{% endhighlight %}

The test did not execute successfully and resulted in a NotFoundHttpException with the error message "_A request to
\[http://localhost/de\] failed. Received status code \[404\]._". Root cause for this problem was my language middleware,
which redirected the user to a predefined default language by adding the de/ URL prefix if a language prefix was not
set.

My first approach was to disable all middleware components by using the **withoutMiddleware trait** in my tests. Again,
the test did not execute successfully and thew the RuntimeException **"Session store not set on request."**. Since I
used the withoutMiddleware trait, I also disabled middleware components which were required by my application (e.g.
VerifyCsrfToken, Authenticate).

After some research on the internet, I
found [this](http://jakzaprogramowac.pl/pytanie/13326,how-to-disable-selected-middleware-in-laravel-tests) helpful
answer and modified my languageMiddleware, so it will be skipped when running tests as shown below:

{% highlight php %}
/**
 * Handle an incoming request.
 *
 * @param  \Illuminate\Http\Request $request
 * @param  \Closure $next
 * @return mixed
 */
public function handle($request, Closure $next)
{
    if (env('APP_ENV') === 'testing') {
        return $next($request);
    }

    // Redirecting to default language if not set. 
    // Code skipped to keep the example simple. 

    return $next($request);
}
{% endhighlight %}

This technique can be applied to any middleware component, which you want to disable when running functional tests.