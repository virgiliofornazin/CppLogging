# CppLogging
C++ Logging Library provides functionality to log different events with a high
throughput in multi-thread environment into different sinks (console, files,
rolling files, syslog, etc.). Logging configuration is very flexible and gives
functionality to build flexible logger hierarchy with combination of logging
processors (sync, async), filters, layouts (binary, text) and appenders.

[CppLogging API reference](http://chronoxor.github.io/CppLogging/index.html)

[![Build status](https://travis-ci.org/chronoxor/CppLogging.svg?branch=master)](https://travis-ci.org/chronoxor/CppLogging)
[![Build status](https://ci.appveyor.com/api/projects/status/91j5tree7gm7awmd?svg=true)](https://ci.appveyor.com/project/chronoxor/CppLogging)

# Contents
  * [Features](#features)
  * [Requirements](#requirements)
  * [How to build?](#how-to-build)
    * [Clone repository with submodules](#clone-repository-with-submodules)
    * [Windows (Visaul Studio 2015)](#windows-visaul-studio-2015)
    * [Windows (MinGW with MSYS)](#windows-mingw-with-msys)
    * [Linux](#linux)
  * [Logging examples](#logging-examples)
    * [Example 1: Default logger](#example-1-default-logger)
    * [Example 2: Format with logger](#example-2-format-with-logger)
    * [Example 3: Configure custom logger with text layout and console appender](#example-3-configure-custom-logger-with-text-layout-and-console-appender)
    * [Example 4: Configure custom logger with text layout and syslog appender](#example-4-configure-custom-logger-with-text-layout-and-syslog-appender)
    * [Example 5: Configure custom logger with binary layout and file appender](#example-5-configure-custom-logger-with-binary-layout-and-file-appender)
    * [Example 6: Configure logger with custom text layout pattern](#example-6-configure-logger-with-custom-text-layout-pattern)
    * [Example 7: Configure rolling file appender with time-based policy](#example-7-configure-rolling-file-appender-with-time-based-policy)
    * [Example 8: Configure rolling file appender with size-based policy](#example-8-configure-rolling-file-appender-with-size-based-policy)
    * [Example 9: Multi-thread logging with synchronous processor](#example-9-multi-thread-logging-with-synchronous-processor)
    * [Example 10: Multi-thread logging with asynchronous processor](#example-10-multi-thread-logging-with-asynchronous-processor)
  * [Logging benchmarks](#logging-benchmarks)
    * [Benchmark 1: Null appender](#benchmark-1-null-appender)
    * [Benchmark 2: File appender](#benchmark-2-file-appender)
  * [Tools](#tools)
    * [Binary log reader](#binary-log-reader)

# Features
* Optimized for performance
* Binary & text layouts
* Synchronous logging
* Asynchronous logging
* Appenders collection (null, memory, console, file, rolling file, ostream, syslog)
* Logging levels (debug, info, warning, error, fatal)
* Logging filters (by level, by logger name, by message pattern)
* Format logging records using [{fmt} library](http://fmtlib.net)
* Flexible configuration and logger processing hierarchy

# Requirements
* Windows 7 / Windows 10
* Linux
* [CMake](http://www.cmake.org)
* [GIT](https://git-scm.com)
* [GCC](https://gcc.gnu.org)

Optional:
* [Visual Studio 2015](https://www.visualstudio.com)
* [Clion](https://www.jetbrains.com/clion)
* [MinGW](http://mingw-w64.org/doku.php)
* [MSYS](http://www.mingw.org/wiki/msys)

#How to build?

## Clone repository with submodules
```
git clone https://github.com/chronoxor/CppLogging.git CppLogging
cd CppLogging
git submodule update --init --recursive --remote
```

## Windows (Visaul Studio 2015)
```
cd scripts
01-generate-VisualStudio.bat
02-build-VisualStudio.bat
03-tests.bat
04-install-VisualStudio.bat
05-doxygen-VisualStudio.bat
```

## Windows (MinGW with MSYS)
```
cd scripts
01-generate-MSYS.bat
02-build-MSYS.bat
03-tests.bat
04-install-MSYS.bat
05-doxygen-MSYS.bat
```

## Linux
```
cd scripts
./01-generate-Unix.sh
./02-build-Unix.sh
./03-tests.sh
./04-install-Unix.sh
./05-doxygen-Unix.sh
```

# Logging examples

## Example 1: Default logger
This is the simple example of using default logger. Just link it with
CppLogging library and you'll get default logger functionality with
text layout and console appender:

```C++
#include "logging/logger.h"

int main(int argc, char** argv)
{
    // Create default logger
    CppLogging::Logger logger;

    // Log some messages with different level
    logger.Debug("Debug message");
    logger.Info("Info message");
    logger.Warn("Warning message");
    logger.Error("Error message");
    logger.Fatal("Fatal message");

    return 0;
}
```

Example will create the following log in console:
![Default report][default]
[default]: https://github.com/chronoxor/CppLogging/raw/master/images/default.png "Default report"

## Example 2: Format with logger
CppLogging library provides powerful logging format API based on the
[{fmt} library](http://fmtlib.net):

```C++
#include "logging/logger.h"

class Date
{
public:
    Date(int year, int month, int day) : _year(year), _month(month), _day(day) {}

    friend std::ostream& operator<<(std::ostream& os, const Date& date)
    { return os << date._year << '-' << date._month << '-' << date._day; }

private:
    int _year, _month, _day;
};

int main(int argc, char** argv)
{
    // Create default logger
    CppLogging::Logger logger;

    // Log some messages with format
    logger.Info("argc: {}, argv: {}", argc, (void*)argv);
    logger.Info("no arguments");
    logger.Info("{0}, {1}, {2}", -1, 0, 1);
    logger.Info("{0}, {1}, {2}", 'a', 'b', 'c');
    logger.Info("{}, {}, {}", 'a', 'b', 'c');
    logger.Info("{2}, {1}, {0}", 'a', 'b', 'c');
    logger.Info("{0}{1}{0}", "abra", "cad");
    logger.Info("{:<30}", "left aligned");
    logger.Info("{:>30}", "right aligned");
    logger.Info("{:^30}", "centered");
    logger.Info("{:*^30}", "centered");
    logger.Info("{:+f}; {:+f}", 3.14, -3.14);
    logger.Info("{: f}; {: f}", 3.14, -3.14);
    logger.Info("{:-f}; {:-f}", 3.14, -3.14);
    logger.Info("int: {0:d};  hex: {0:x};  oct: {0:o}; bin: {0:b}", 42);
    logger.Info("int: {0:d};  hex: {0:#x};  oct: {0:#o};  bin: {0:#b}", 42);
    logger.Info("The date is {}", Date(2012, 12, 9));
    logger.Info("Elapsed time: {s:.2f} seconds", "s"_a = 1.23);
    logger.Info("The answer is {}"_format(42).c_str());

    return 0;
}
```

Example will create the following log:
```
2016-09-28T13:59:19.970Z [0x000083C8] INFO   - argc: 1, argv: 0x1e5a2aa6b0
2016-09-28T13:59:19.971Z [0x000083C8] INFO   - no arguments
2016-09-28T13:59:19.971Z [0x000083C8] INFO   - -1, 0, 1
2016-09-28T13:59:19.971Z [0x000083C8] INFO   - a, b, c
2016-09-28T13:59:19.971Z [0x000083C8] INFO   - a, b, c
2016-09-28T13:59:19.971Z [0x000083C8] INFO   - c, b, a
2016-09-28T13:59:19.971Z [0x000083C8] INFO   - abracadabra
2016-09-28T13:59:19.971Z [0x000083C8] INFO   - left aligned
2016-09-28T13:59:19.971Z [0x000083C8] INFO   -                  right aligned
2016-09-28T13:59:19.971Z [0x000083C8] INFO   -            centered
2016-09-28T13:59:19.971Z [0x000083C8] INFO   - ***********centered***********
2016-09-28T13:59:19.971Z [0x000083C8] INFO   - +3.140000; -3.140000
2016-09-28T13:59:19.971Z [0x000083C8] INFO   -  3.140000; -3.140000
2016-09-28T13:59:19.971Z [0x000083C8] INFO   - 3.140000; -3.140000
2016-09-28T13:59:19.971Z [0x000083C8] INFO   - int: 42;  hex: 2a;  oct: 52; bin: 101010
2016-09-28T13:59:19.971Z [0x000083C8] INFO   - int: 42;  hex: 0x2a;  oct: 052;  bin: 0b101010
2016-09-28T13:59:19.971Z [0x000083C8] INFO   - The date is 2012-12-9
2016-09-28T13:59:19.972Z [0x000083C8] INFO   - Elapsed time: 1.23 seconds
2016-09-28T13:59:19.972Z [0x000083C8] INFO   - The answer is 42
```

## Example 3: Configure custom logger with text layout and console appender
This example shows how to configure a custom logger with a given name to
perform logging with a text layout and console appender sink:

```C++
#include "logging/config.h"
#include "logging/logger.h"

void ConfigureLogger()
{
    // Create default logging sink processor
    auto sink = std::make_shared<CppLogging::Processor>();
    // Add text layout
    sink->layouts().push_back(std::make_shared<CppLogging::TextLayout>());
    // Add console appender
    sink->appenders().push_back(std::make_shared<CppLogging::ConsoleAppender>());

    // Configure example logger
    CppLogging::Config::ConfigLogger("example", sink);
}

int main(int argc, char** argv)
{
    // Configure logger
    ConfigureLogger();

    // Create example logger
    CppLogging::Logger logger("example");

    // Log some messages with different level
    logger.Debug("Debug message");
    logger.Info("Info message");
    logger.Warn("Warning message");
    logger.Error("Error message");
    logger.Fatal("Fatal message");

    return 0;
}
```

## Example 4: Configure custom logger with text layout and syslog appender
*Syslog appender is available only in Unix platforms and does nothing in
Windows!*

This example shows how to configure a custom logger with a given name to
perform logging with a text layout and syslog appender sink:

```C++
#include "logging/config.h"
#include "logging/logger.h"

void ConfigureLogger()
{
    // Create default logging sink processor
    auto sink = std::make_shared<CppLogging::Processor>();
    // Add text layout
    sink->layouts().push_back(std::make_shared<CppLogging::TextLayout>());
    // Add syslog appender
    sink->appenders().push_back(std::make_shared<CppLogging::SyslogAppender>());

    // Configure example logger
    CppLogging::Config::ConfigLogger("example", sink);
}

int main(int argc, char** argv)
{
    // Configure logger
    ConfigureLogger();

    // Create example logger
    CppLogging::Logger logger("example");

    // Log some messages with different level
    logger.Debug("Debug message");
    logger.Info("Info message");
    logger.Warn("Warning message");
    logger.Error("Error message");
    logger.Fatal("Fatal message");

    return 0;
}
```

## Example 5: Configure custom logger with binary layout and file appender
This example shows how to configure a custom logger with a given name to
perform logging with a binary layout and file appender sink:

```C++
#include "logging/config.h"
#include "logging/logger.h"

void ConfigureLogger()
{
    // Create default logging sink processor
    auto sink = std::make_shared<CppLogging::Processor>();
    // Add binary layout
    sink->layouts().push_back(std::make_shared<CppLogging::BinaryLayout>());
    // Add file appender
    sink->appenders().push_back(std::make_shared<CppLogging::FileAppender>(CppCommon::File("file.bin.log")));

    // Configure example logger
    CppLogging::Config::ConfigLogger("example", sink);
}

int main(int argc, char** argv)
{
    // Configure logger
    ConfigureLogger();

    // Create example logger
    CppLogging::Logger logger("example");

    // Log some messages with different level
    logger.Debug("Debug message");
    logger.Info("Info message");
    logger.Warn("Warning message");
    logger.Error("Error message");
    logger.Fatal("Fatal message");

    return 0;
}
```

## Example 6: Configure logger with custom text layout pattern
Text layout message is flexible to customize with layout pattern. Text layout
pattern is a string with a special placeholders provided inside curly brackets
("{}").

Supported placeholders:
- **{UtcDateTime} / {LocalDateTime}** - converted to the UTC/local date & time (e.g. "1997-07-16T19:20:30.123Z" / "1997-07-16T19:20:30.123+01:00")
- **{UtcDate} / {LocalDate}** - converted to the UTC/local date (e.g. "1997-07-16")
- **{Time} / {LocalTime}** - converted to the UTC/local time (e.g. "19:20:30.123Z" / "19:20:30.123+01:00")
- **{UtcYear} / {LocalYear}** - converted to the UTC/local four-digits year (e.g. "1997")
- **{UtcMonth} / {LocalMonth}** - converted to the UTC/local two-digits month (e.g. "07")
- **{UtcDay} / {LocalDay}** - converted to the UTC/local two-digits day (e.g. "16")
- **{UtcHour} / {LocalHour}** - converted to the UTC/local two-digits hour (e.g. "19")
- **{UtcMinute} / {LocalMinute}** - converted to the UTC/local two-digits minute (e.g. "20")
- **{UtcSecond} / {LocalSecond}** - converted to the UTC/local two-digits second (e.g. "30")
- **{UtcTimezone} / {LocalTimezone}** - converted to the UTC/local timezone suffix (e.g. "Z" / "+01:00")
- **{Millisecond}** - converted to the three-digits millisecond (e.g. "123")
- **{Microsecond}** - converted to the three-digits microsecond (e.g. "123")
- **{Nanosecond}** - converted to the three-digits nanosecond (e.g. "789")
- **{Thread}** - converted to the thread Id (e.g. "0x0028F3D8")
- **{Level}** - converted to the logging level
- **{Logger}** - converted to the logger name
- **{Message}** - converted to the log message
- **{EndLine}** - converted to the end line suffix (e.g. Unix "\n" or Windows "\r\n")

```C++
#include "logging/config.h"
#include "logging/logger.h"

void ConfigureLogger()
{
    // Create a custom text layout pattern
    std::string pattern = "{UtcYear}-{UtcMonth}-{UtcDay}T{UtcHour}:{UtcMinute}:{UtcSecond}.{Millisecond}{UtcTimezone} - {Microsecond}.{Nanosecond} - [{Thread}] - {Level} - {Logger} - {Message} - {EndLine}";

    // Create default logging sink processor
    auto sink = std::make_shared<CppLogging::Processor>();
    // Add text layout
    sink->layouts().push_back(std::make_shared<CppLogging::TextLayout>(pattern));
    // Add console appender
    sink->appenders().push_back(std::make_shared<CppLogging::ConsoleAppender>());

    // Configure example logger
    CppLogging::Config::ConfigLogger("example", sink);
}

int main(int argc, char** argv)
{
    // Configure logger
    ConfigureLogger();

    // Create example logger
    CppLogging::Logger logger("example");

    // Log some messages with different level
    logger.Debug("Debug message");
    logger.Info("Info message");
    logger.Warn("Warning message");
    logger.Error("Error message");
    logger.Fatal("Fatal message");

    return 0;
}
```

## Example 7: Configure rolling file appender with time-based policy
Time-based rolling policy will create a new logging file to write into using
a special pattern (contains date & time placeholders).

Time-based policy composes logging filename from the given pattern using the
following placeholders:
- **{UtcDateTime} / {LocalDateTime}** - converted to the UTC/local date & time (e.g. "1997-07-16T192030Z" / "1997-07-16T192030+0100")
- **{UtcDate} / {LocalDate}** - converted to the UTC/local date (e.g. "1997-07-16")
- **{Time} / {LocalTime}** - converted to the UTC/local time (e.g. "192030Z" / "192030+0100")
- **{UtcYear} / {LocalYear}** - converted to the UTC/local four-digits year (e.g. "1997")
- **{UtcMonth} / {LocalMonth}** - converted to the UTC/local two-digits month (e.g. "07")
- **{UtcDay} / {LocalDay}** - converted to the UTC/local two-digits day (e.g. "16")
- **{UtcHour} / {LocalHour}** - converted to the UTC/local two-digits hour (e.g. "19")
- **{UtcMinute} / {LocalMinute}** - converted to the UTC/local two-digits minute (e.g. "20")
- **{UtcSecond} / {LocalSecond}** - converted to the UTC/local two-digits second (e.g. "30")
- **{UtcTimezone} / {LocalTimezone}** - converted to the UTC/local timezone suffix (e.g. "Z" / "+0100")

```C++
#include "logging/config.h"
#include "logging/logger.h"

void ConfigureLogger()
{
    // Create default logging sink processor
    auto sink = std::make_shared<CppLogging::Processor>();
    // Add text layout
    sink->layouts().push_back(std::make_shared<CppLogging::TextLayout>());
    // Add rolling file appender which rolls each second and create log file with a pattern "{UtcDateTime}.log"
    sink->appenders().push_back(std::make_shared<CppLogging::FileAppender>(CppCommon::RollingFileAppender(".", TimeRollingPolicy::SECOND, "{UtcDateTime}.log", true)));

    // Configure example logger
    CppLogging::Config::ConfigLogger("example", sink);
}

int main(int argc, char** argv)
{
    // Configure logger
    ConfigureLogger();

    // Create example logger
    CppLogging::Logger logger("example");

    // Log some messages with different level
    logger.Debug("Debug message");
    logger.Info("Info message");
    logger.Warn("Warning message");
    logger.Error("Error message");
    logger.Fatal("Fatal message");

    return 0;
}
```

## Example 8: Configure rolling file appender with size-based policy
Size-based rolling policy will create a new logging file to write when the
current file size exceeded size limit. Logging backups are indexed and its
count could be limited as well.

Size-based policy for 5 backups works in a following way:
```
example.log   -> example.1.log
example.1.log -> example.2.log
example.2.log -> example.3.log
example.3.log -> example.4.log
example.4.log -> example.5.log
example.5.log -> remove
```

```C++
#include "logging/config.h"
#include "logging/logger.h"

void ConfigureLogger()
{
    // Create default logging sink processor
    auto sink = std::make_shared<CppLogging::Processor>();
    // Add binary layout
    sink->layouts().push_back(std::make_shared<CppLogging::BinaryLayout>());
    // Add rolling file appender which rolls after append 4kb of logs and will keep only 5 recent archives
    sink->appenders().push_back(std::make_shared<CppLogging::FileAppender>(CppCommon::RollingFileAppender(".", "file", "bin.log", 4096, 5, true)));

    // Configure example logger
    CppLogging::Config::ConfigLogger("example", sink);
}

int main(int argc, char** argv)
{
    // Configure logger
    ConfigureLogger();

    // Create example logger
    CppLogging::Logger logger("example");

    // Log some messages with different level
    logger.Debug("Debug message");
    logger.Info("Info message");
    logger.Warn("Warning message");
    logger.Error("Error message");
    logger.Fatal("Fatal message");

    return 0;
}
```

## Example 9: Multi-thread logging with synchronous processor
Synchronous processor uses critical-section lock to avoid multiple
threads from logging at the same time (logging threads are waiting
until critical-section is released).

This example shows how to configure a custom logger with a given name to
use synchronous processor in multi-thread environment:

```C++
#include "logging/config.h"
#include "logging/logger.h"

#include <atomic>
#include <iostream>
#include <thread>
#include <vector>

void ConfigureLogger()
{
    // Create default logging sink processor
    auto sink = std::make_shared<CppLogging::SyncProcessor>();
    // Add binary layout
    sink->layouts().push_back(std::make_shared<CppLogging::BinaryLayout>());
    // Add file appender with size-based rolling policy and archivation
    sink->appenders().push_back(std::make_shared<CppLogging::RollingFileAppender>(".", "rolling", "bin.log", 4096, 9, true));

    // Configure example logger
    CppLogging::Config::ConfigLogger("example", sink);
}

int main(int argc, char** argv)
{
    // Configure logger
    ConfigureLogger();

    std::cout << "Press Enter to stop..." << std::endl;

    int concurrency = 4;

    // Start some threads
    std::atomic<bool> stop(false);
    std::vector<std::thread> threads;
    for (int thread = 0; thread < concurrency; ++thread)
    {
        threads.push_back(std::thread([&stop, thread]()
        {
            // Create example logger
            CppLogging::Logger logger("example");

            while (!stop)
            {
                // Log some messages with different level
                logger.Debug("Debug message");
                logger.Info("Info message");
                logger.Warn("Warning message");
                logger.Error("Error message");
                logger.Fatal("Fatal message");

                // Yield for a while...
                CppCommon::Thread::Yield();
            }
        }));
    }

    // Wait for input
    std::cin.get();

    // Stop threads
    stop = true;

    // Wait for all threads
    for (auto& thread : threads)
        thread.join();

    return 0;
}
```

## Example 10: Multi-thread logging with asynchronous processor
Asynchronous processor uses lock-free queue to collect logging records from
multiple threads at the same time.

This example shows much better performance with less threads contentions in
comparison with the previous one for lots of threads:

```C++
#include "logging/config.h"
#include "logging/logger.h"

#include <atomic>
#include <iostream>
#include <thread>
#include <vector>

void ConfigureLogger()
{
    // Create default logging sink processor
    auto sink = std::make_shared<CppLogging::AsyncProcessor>();
    // Add text layout
    sink->layouts().push_back(std::make_shared<CppLogging::TextLayout>());
    // Add file appender with time-based rolling policy and archivation
    sink->appenders().push_back(std::make_shared<CppLogging::RollingFileAppender>(".", CppLogging::TimeRollingPolicy::SECOND, "{UtcDateTime}.log", true));

    // Configure example logger
    CppLogging::Config::ConfigLogger("example", sink);
}

int main(int argc, char** argv)
{
    // Configure logger
    ConfigureLogger();

    std::cout << "Press Enter to stop..." << std::endl;

    int concurrency = 4;

    // Start some threads
    std::atomic<bool> stop(false);
    std::vector<std::thread> threads;
    for (int thread = 0; thread < concurrency; ++thread)
    {
        threads.push_back(std::thread([&stop, thread]()
        {
            // Create example logger
            CppLogging::Logger logger("example");

            while (!stop)
            {
                // Log some messages with different level
                logger.Debug("Debug message");
                logger.Info("Info message");
                logger.Warn("Warning message");
                logger.Error("Error message");
                logger.Fatal("Fatal message");

                // Yield for a while...
                CppCommon::Thread::Yield();
            }
        }));
    }

    // Wait for input
    std::cin.get();

    // Stop threads
    stop = true;

    // Wait for all threads
    for (auto& thread : threads)
        thread.join();

    return 0;
}
```

# Logging benchmarks

##Benchmark 1: Null appender
Benchmark source file: [appender_null.cpp](https://github.com/chronoxor/CppLogging/blob/master/perf/appender_null.cpp)

Benchmark report is the following:
```
===============================================================================
CppBenchmark report. Version 1.0.0.0
===============================================================================
CPU architecutre: Intel(R) Core(TM) i7-4790K CPU @ 4.00GHz
CPU logical cores: 8
CPU physical cores: 4
CPU clock speed: 3.998 GHz
CPU Hyper-Threading: enabled
RAM total: 31.962 GiB
RAM free: 21.688 GiB
===============================================================================
OS version: Microsoft Windows 8 Enterprise Edition (build 9200), 64-bit
OS bits: 64-bit
Process bits: 64-bit
Process configuaraion: release
Local timestamp: Wed Sep 28 22:08:14 2016
UTC timestamp: Wed Sep 28 19:08:14 2016
===============================================================================
Benchmark: NullAppender-binary
Attempts: 5
Iterations: 10000000
-------------------------------------------------------------------------------
Phase: NullAppender-binary
Average time: 62 ns / iteration
Minimal time: 62 ns / iteration
Maximal time: 63 ns / iteration
Total time: 624.974 ms
Total iterations: 10000000
Iterations throughput: 16000657 / second
===============================================================================
Benchmark: NullAppender-text
Attempts: 5
Iterations: 10000000
-------------------------------------------------------------------------------
Phase: NullAppender-text
Average time: 174 ns / iteration
Minimal time: 174 ns / iteration
Maximal time: 176 ns / iteration
Total time: 1.744 s
Total iterations: 10000000
Iterations throughput: 5733126 / second
===============================================================================
```

##Benchmark 2: File appender
Benchmark source file: [appender_file.cpp](https://github.com/chronoxor/CppLogging/blob/master/perf/appender_file.cpp)

Benchmark report is the following:
```
===============================================================================
CppBenchmark report. Version 1.0.0.0
===============================================================================
CPU architecutre: Intel(R) Core(TM) i7-4790K CPU @ 4.00GHz
CPU logical cores: 8
CPU physical cores: 4
CPU clock speed: 3.998 GHz
CPU Hyper-Threading: enabled
RAM total: 31.962 GiB
RAM free: 21.181 GiB
===============================================================================
OS version: Microsoft Windows 8 Enterprise Edition (build 9200), 64-bit
OS bits: 64-bit
Process bits: 64-bit
Process configuaraion: release
Local timestamp: Wed Sep 28 22:12:24 2016
UTC timestamp: Wed Sep 28 19:12:24 2016
===============================================================================
Benchmark: FileAppender-binary
Attempts: 5
Iterations: 1000000
-------------------------------------------------------------------------------
Phase: FileAppender-binary
Average time: 106 ns / iteration
Minimal time: 106 ns / iteration
Maximal time: 117 ns / iteration
Total time: 106.598 ms
Total iterations: 1000000
Iterations throughput: 9381018 / second
===============================================================================
Benchmark: FileAppender-text
Attempts: 5
Iterations: 1000000
-------------------------------------------------------------------------------
Phase: FileAppender-text
Average time: 229 ns / iteration
Minimal time: 229 ns / iteration
Maximal time: 249 ns / iteration
Total time: 229.126 ms
Total iterations: 1000000
Iterations throughput: 4364404 / second
===============================================================================
```

# Tools

## Binary log reader
Binary log reader is used to convert binary logs ('.bin.log' files) to the
human-readable text format. By default it reads binary stream from 'stdin' and
outputs converted text to 'stdout'. However it is possible to provide explicit
input/output file names using command line arguments:

```
Usage: binlog [options]

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  -h HELP, --help=HELP  Show help
  -i INPUT, --input=INPUT
                        Input file name
  -o OUTPUT, --output=OUTPUT
                        Output file name

```
