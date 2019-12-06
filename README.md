# WebView

This is a Java port of the fantastic, tiny, light-weight [WebView](https://github.com/zserge/webview) by [Serge Zaitsev](https://zserge.com).

It is packaged into an executable Jar file so that you can run it as a CLI self-contained process or as a Java library inside your current process.

## Running in Separate Process


You can run the WebView in a separate process by running:

~~~~
$ java -jar WebView.jar http://www.example.com
~~~~

This will open the web browser in its own window pointing to http://www.example.com.

### Interacting with the Browser Environment

You can interact with the browser environment by typing into the console while the browser is running.  The browser listens on STDIN, for any input, and it will evaluate any input as Javascript in the context of the current page.  E.g. Type "alert('foo')" then `[ENTER]` to open an alert popup.  

If you need to enter a multi-line Javascript command, then begin your input with `<<<SOME_BOUNDARY`, and end it with `SOME_BOUNDARY`.

For example:

~~~~
<<<END
var url = window.location.href;
alert('You are at '+url);
END
~~~~

NOTE:  If you give it an empty boundary, then it will simply use a blank line as your boundary.

### Getting Information From The Browser

There are two ways get the browser to communicate back to the outside world:

1. The onLoad callback.  Whenever the user nagivates to a new page, it will output `loaded [URL]` to STDOUT.  E.g. If you navigate to google.com, then it will output `loaded https://google.com` to STDOUT.
2. Call `window.external.invoke("some message")`.  This will cause the browser print "some message" to STDOUT.  All messages of this kind are wrapped with beginning and ending boundaries to make the output easier to parse, in case you are writing a program to interact with the browser.

Here is an example of a session, where I load google.com, and then get its page title via `window.external.invoke()`:

~~~~
$ java -jar WebView-shaded.jar "https://www.google.com"

loaded https://www.google.com/
window.external.invoke(document.title)
<<<Boundary1575660241187
Google
Boundary1575660241187
~~~~

A few things to notice here:

1. When the page is loaded, it informed us with "loaded https://www.google.com" in STDOUT
2. I typed the "window.external.invoke(document.title)" command.
3. It responded to my command with an open boundary `<<<Boundary1575660241187` followed by the message ("Google"), followed by the closing boundary `Boundary1575660241187`

## Using Java API

If you want to use the webview directly in your Java app, you can do this also. 

A simple usage example:

~~~~
webview = new WebView()
    .size(width, height)
    .title(title)
    .resizable(resizable)
    .fullscreen(fullscreen)
    .url(u)
    .onLoad(()->{
       //.. Do something on page load.
	   // You can get the url of the page via webview.url()
    })
	.javascriptCallback(message->{
		// Handle a message sent via window.external.invoke(message)
		// message is a string.
	})
	.show();
~~~~

NOTE: The `show()` method will start a blocking event loop.

WARNING: Currently the WebView is picky about being started on the main application thread.  On Mac you may need to add the "-XstartOnFirstThread" flag in the JVM.

## Supported Platforms

This should work on Mac, Linux, and Windows.


## Building Sources

~~~
git clone https://github.com/shannah/webviewjar
cd webviewjar
ant shaded-jar
~~~

This will create both dist/WebView.jar and dist/WebView-shaded.jar.  dist/WebView.jar just contains the library.  WebView-Shaded.jar includes all dependencies, and can be run as an executable jar.

## License

MIT

## Credits

1. This library created by [Steve Hannah](https://sjhannah.com)
2. Original webview library by [Serge Zaitsev](https://zserge.com)