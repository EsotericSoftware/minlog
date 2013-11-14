![](https://raw.github.com/wiki/EsotericSoftware/minlog/images/logo.png)

## MinLog

Please use the [MinLog discussion group](http://groups.google.com/group/minlog-users) for support.

## Overview

MinLog is a tiny Java logging library which features:

- **Zero overhead** Logging statements below a given level can be automatically removed by javac at compile time. This means applications can have detailed trace and debug logging without having any impact on the finished product.

- **Extremely lightweight** The entire project consists of a [single Java file](https://github.com/EsotericSoftware/minlog/blob/master/src/com/esotericsoftware/minlog/Log.java) with ~100 non-comment lines of code.

- **Simple and efficient** The API is concise and the code is very efficient at runtime.

Also see [this](https://github.com/jdanbrown/minlog-slf4j) drop-in replacement for minlog which logs to slf4j.

## Usage

Messages are logged using static methods:

```java
    Log.info("Some message.");
    Log.debug("Error reading file: " + file, ex);
```

A static import can be used to make the logging more concise:

```java
    import static com.esotericsoftware.minlog.Log.*;
    // ...
    info("Some message.");
    debug("Error reading file: " + file, ex);
```

While optional, for brevity the rest of this documentation assumes this static import is in place.

If log statements from different libraries or areas of an application need to be differentiated, a category can be specified as the first argument:

```java
    info("some lib", "Some message.");
    debug("some lib", "Error reading file: " + file, ex);
```

## Log level

Setting the level will log that level, as well as all higher levels. There are multiple ways to set the current level:

```java
    Log.set(LEVEL_INFO);
    Log.INFO();
    INFO();
```

The levels are:

- NONE disables all logging.
- ERROR is for critical errors. The application may no longer work correctly.
- WARN is for important warnings. The application will continue to work correctly.
- INFO is for informative messages. Typically used for deployment.
- DEBUG is for debug messages. This level is useful during development.
- TRACE is for trace messages. A lot of information is logged, so this level is usually only needed when debugging a problem.


## Conditional logging

If a logging method below the current level is called, it will return without logging the message. In order to avoid string concatenation, the current log level can be checked before the message is logged:

```java
    if (ERROR) error("Error reading file: " + file, ex);
    if (TRACE) {
    	StringBuilder builder = new StringBuilder();
    	// Do work, append to the builder.
    	trace(builder);
    }
```

## Fixed logging levels

MinLog users can choose from the regular "minlog.jar" or from from a JAR file like "minlog-info.jar" which has a fixed logging level that cannot be changed at runtime. When a fixed level JAR is used, code that changes the level will have no affect. During compilation, any conditional logging statements below the fixed level will be automatically removed by the Java compiler. This means they will have absolutely no impact on the application.

## Output customization

The default logger outputs messages in this format:

```
    time level: [category] message
```

Where "time" is the time elapsed since the application started. For example:

```
    00:00 TRACE: [kryo] Wrote string: moo
    00:00 TRACE: [kryo] Wrote object: NonNullTestClass
    00:01 TRACE: [kryo] Wrote string: this is some data
    00:01 TRACE: [kryo] Compressed to 7.97% using: DeflateCompressor
    00:12 TRACE: [kryo] Decompressed using: DeflateCompressor
    00:12 TRACE: [kryo] Read string: this is some data
```

The output can be customized:

```java
    static public class MyLogger extends Logger {
    	public void log (int level, String category, String message, Throwable ex) {
    		StringBuilder builder = new StringBuilder(256);
    		builder.append(new Date());
    		builder.append(' ');
    		builder.append(level);
    		builder.append('[');
    		builder.append(category);
    		builder.append("] ");
    		builder.append(message);
    		if (ex != null) {
    			StringWriter writer = new StringWriter(256);
    			ex.printStackTrace(new PrintWriter(writer));
    			builder.append('\n');
    			builder.append(writer.toString().trim());
    		}
    		System.out.println(builder);
    	}
    }
    // ...
    Log.setLogger(new MyLogger());
```

Using this mechanism, log messages can be filtered (eg, by category), written to a file, etc.
