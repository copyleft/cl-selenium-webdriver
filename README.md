# CL WebDriver Client

[![Quicklisp](http://quickdocs.org/badge/cl-webdriver-client.svg)](http://quickdocs.org/cl-webdriver-client/)

CL WebDriver Client is a client library for WebDriver.

WebDriver is a remote control interface that enables introspection and control of user agents. It provides a platform- and language-neutral wire protocol as a way for out-of-process programs to remotely instruct the behavior of web browsers.

Provided is a set of interfaces to discover and manipulate DOM elements in web documents and to control the behavior of a user agent. It is primarily intended to allow web authors to write tests that automate a user agent from a separate controlling process, but may also be used in such a way as to allow in-browser scripts to control a — possibly separate — browser.

See [W3C WebDriver spec](https://www.w3.org/TR/webdriver).

NOTE: This is a fork of CL Selenium WebDriver, a binding library to the Selenium.

## Usage

```lisp
;; see examples/*.lisp and t/*.lisp
(in-package :cl-user)

(ql:quickload :cl-webdriver-client)

(defpackage go-test
  (:use :cl :webdriver-client))

(in-package :go-test)

(defparameter *code* "
package main
import \"fmt\"

func main() {
    fmt.Print(\"Hello WebDriver!\")
}")

(with-session ()
  (setf (url) "http://play.golang.org/?simple=1")
  (let ((elem (find-element "#code" :by :css-selector)))
    (element-clear elem)
    (element-send-keys elem *code*))
  (let ((btn (find-element "#run")))
    (element-click btn))

  (loop
     with div = (find-element "#output")
     for output = (element-text div)
     while (equal output "Waiting for remote server...")
     do (sleep 0.1)
     finally (print output)))
```

## Installation

Available on Quicklisp:

```
(ql:quickload :cl-webdriver-client)
```

To run, you will need a running instance of Selenium Server version 4.0.0 or later.

[Download](https://www.selenium.dev/downloads/) it and run:
```sh
java -jar selenium-server.jar standalone
```

Alternatively, you can try directly connecting to a standalone WebDriver server implementation, such as [ChromeDriver](https://developer.chrome.com/docs/chromedriver):

```sh
chromedriver --port=4444
```

Firefox and other Gecko-based browsers use their own automation driver, [Marionette](https://firefox-source-docs.mozilla.org/testing/marionette/), but Mozilla maintains [geckodriver](https://firefox-source-docs.mozilla.org/testing/geckodriver/), which acts as a proxy between the Marionette and WebDriver protocols.

``` sh
geckodriver
```

## Documentation

[Read the manual](https://copyleft.github.io/cl-webdriver-client/)

## Utils

There's a `webdriver-client-utils` package which should reduce boilerplate. 

The exported definitions work with an implicit element. The default implicit element is the current active element. So, it is not necessary to pass the element you are working with around most of the time.

For example:

```lisp
(defpackage my-test
  (:use :cl :webdriver-client)
  (:import-from :webdriver-client-utils
                :send-keys
                :click
                :wait-for
                :classlist))

(in-package :my-test)

(with-session ()
  (setf (url) "http://google.com")
  (send-keys "cl-webdriver-client")
  (click "[name=btnK]")
  (wait-for "#resultStats"))

```

### Interactive session

You can just start the session and control it from your repl:

```lisp
(in-package :my-test)

(start-interactive-session)

(setf (url) "http://google.com")
(send-keys "cl-webdriver-client")
(send-keys (key :enter))
(classlist "#slim_appbar") ; prints ("ab_tnav_wrp")

(stop-interactive-session)
```

### Utils API conventions

If utility function needs an element to work on it defaults to `(active-element)`.
```lisp
(click) ; click on the current active element.
```
You can also pass a CSS selector as a last parameter.
```lisp
(print (id "#submit")) ; print id the of matched element

(assert (= (first (classlist "div")) "first-div-ever"))
```

To change default element you can:
```lisp
(setf webdriver-client-utils:*default-element-func* (lambda () (find-element "input[type=submit]"))
```


### Waiting for the reaction

Often you need to wait for some action to be done. For example if you
do a `(click)` on the button to load search results, you need to wait
them to load.
```lisp
(wait-for ".search-result" :timeout 10) ; wait 10 seconds
```
Timeout defaults to 30 seconds. You can globally change it:
```lisp
(setf webdriver-client-utils:*timeout* 3)
```

## Running tests

### REPL
```lisp
(ql:quickload '(:cl-webdriver-client :prove))
(setf prove:*enable-colors* nil)
(prove:run :cl-webdriver-client-test)
```

### Shell
```sh
./test.sh
```

## Copyright

Licensed under the MIT License.
