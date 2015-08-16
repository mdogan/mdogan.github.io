---
layout: post
title: GO&#58; Call me maybe, Java!
published: true
tags: java jni go golang cgo jnr ffi
comments: true
---

When you need to pass boundaries of managed java environment, you find yourself playing (or fighting :) with C/C++ code (headers, compiler flags, linker flags ...).
With version 1.5, Go is coming to save us, C/C++ is not your only option anymore...

<!--excerpt-->

One of the great features introduced in Go 1.5 is, being able to build a C compatible library. Using `-buildmode={c-archive,c-shared}` flag, you can now build a C archive `go build -buildmode=c-archive` or a C shared library `go build -buildmode=c-shared`. _(For more info: [Go Execution Modes](https://docs.google.com/document/d/1nr-TQHw_er6GOQRsF6T43GGhFDelrAP0NqSS_00RgZQ/))_

Assume we want to create a native math library... And we need a method to multiply two `long` numbers.

```java
public static long multiply(long x, long y)
```

#### JNI with C/C++
To be able to implement a method in C, first we need to define a `native` java method.

```java
package io.dogan.whiteboard.jni;

public class JniMath {
    public static native long multiply(long x, long y);
}
```

Then using `javah` tool, we'll generate the C header file:

```
javah io.dogan.whiteboard.jni.JniMath
```

It will generate the header `io_dogan_whiteboard_jni_JniMath.h`:

```C
#include <jni.h>
#ifndef _Included_io_dogan_whiteboard_jni_JniMath
#define _Included_io_dogan_whiteboard_jni_JniMath
#ifdef __cplusplus
extern "C" {
#endif

JNIEXPORT jlong JNICALL Java_io_dogan_whiteboard_jni_JniMath_multiply
  (JNIEnv *, jclass, jlong, jlong);

#ifdef __cplusplus
}
#endif
#endif
```

This is our contract between java JNI and native code. Implementation of `io_dogan_whiteboard_jni_JniMath.c` will be look like this:

```C
#include "io_dogan_whiteboard_jni_JniMath.h"

JNIEXPORT jlong JNICALL Java_io_dogan_whiteboard_jni_JniMath_multiply
  (JNIEnv * env, jclass clazz, jlong argX, jlong argY) {

  return argX * argY;
}
```

And now, we can call our native multiplication method after building the shared library:

```
gcc -shared -fPIC -I$JAVA_HOME/include -I$JAVA_HOME/include/darwin io_dogan_whiteboard_jni_NativeActions.c -o libmath.dylib
```

```java
System.out.println(JniMath.multiply(12345, 67890));
// output: 838102050
```

This is old school JNI, let's get to the topic...

#### JNI with Go
We'll use the same `JniMath` class and the same contract between java and native code. So, our Go function will use the same signature as C function and we'll export our Go function as a C function.

_Assuming you already know how to export a C function in Go. If you don't then just take a look at [CGO](http://golang.org/cmd/cgo/#hdr-C_references_to_Go). Basic idea is, it's one by adding a special export comment over function:_
`//export Java_io_dogan_whiteboard_jni_JniMath_multiply`

Here is our Go implementation `math.go`:

```go
//math.go

package main

// #cgo CFLAGS: -I/Library/Java/JavaVirtualMachines/jdk1.8.0_51.jdk/Contents/Home/include
// #cgo CFLAGS: -I/Library/Java/JavaVirtualMachines/jdk1.8.0_51.jdk/Contents/Home/include/darwin
// #include <jni.h>
import "C"

//export Java_io_dogan_whiteboard_jni_JniMath_multiply
func Java_io_dogan_whiteboard_jni_JniMath_multiply(env *C.JNIEnv, clazz C.jclass, x C.jlong, y C.jlong) C.jlong {
    return x * y
}

// main function is required, don't know why!
func main() {} // a dummy function
```
_Note that C import statement and #include and #cgo instructions over the import. Alternatively, you can set CFLAGS parameter as an environment variable._

Now, let's build and run:

```
go build -buildmode=c-shared -o libmath.dylib math.go
```

```java
System.out.println(JniMath.multiply(12345, 67890));
// output: 838102050
```

Done? No! Why do we need to import `jni.h`, use `JNIEnv`, `jlong` etc. in our parameter list and call our simple `multiply` method with a such strange and complex name: `Java_io_dogan_whiteboard_jni_JniMath_multiply`? Can it be simpler and easier?

Yes! We have a better option: [JNR (jnr-ffi)](https://github.com/jnr/jnr-ffi).

#### JNR with Go
Just forget about `io_dogan_whiteboard_jni_JniMath.h` header, `Java_io_dogan_whiteboard_jni_JniMath_multiply` function, `#cgo`, `#include <jni.h>` directives...

Let's implement our multiplication as it's supposed to be `math.go`:

```go
// math.go

package main

//export Multiply
func Multiply(x int64, y int64) int64 {
    return x * y
}

// main function is required, don't know why!
func main() {} // a dummy function
```

We also need an java interface matching the signature of Go function `MathLib`:

```java
package io.dogan.whiteboard.jnr;

public interface MathLib {
    long Multiply(long x, long y);
}
```

We have a Go function, we have a java interface... Let's bind them to each other...

_Build go library:_

```
go build -buildmode=c-shared -o libmath.dylib math.go
```

_Load the library using JNR-FFI and call the method:_

```java
package io.dogan.whiteboard.jnr;

import jnr.ffi.LibraryLoader;

public class JnrMath {

    private static final MathLib MATH_LIB;

    static {
        MATH_LIB = LibraryLoader.create(MathLib.class).load("math");
    }

    public static void main(String[] args) {
        System.out.println(MATH_LIB.Multiply(12345, 67890));
        // output: 838102050
    }
}
```



*PS1: Go 1.5 is not released yet by the time this post is written. To be able to test things here, you need to build/install Go 1.5 from source. For more info: [Installing Go from source](https://golang.org/doc/install/source#install).*

*PS2: `libmath.dylib` should be placed in a library directory which is known by JVM; either in a system-wide lib directory or in a path denoted by `LD_LIBRARY_PATH`.*

**Caution: Because both Java and Go are garbage collected languages, watch out for possible reference tracking issues between two. For more info: [Binding Go and Java](https://golang.org/s/gobind)**
