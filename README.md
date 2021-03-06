## This is an add-in for  [Fody](https://github.com/Fody/Fody) 

Simplifies logging through a static class and some IL manipulation

[Introduction to Fody](https://github.com/Fody/Fody/wiki/SampleUsage)

## Supported Logging Libraries

* [NLog](http://nlog-project.org/) 
* [Log4Net](http://logging.apache.org/log4net/) 
* [Serilog](http://serilog.net/)
* [MetroLog](https://github.com/mbrit/MetroLog)

## Nuget

 * NLog package http://nuget.org/packages/Anotar.NLog.Fody 

    PM> Install-Package Anotar.NLog.Fody
 
 * MetroLog package http://nuget.org/packages/Anotar.MetroLog.Fody 

    PM> Install-Package Anotar.MetroLog.Fody
 
 * Log4Net package http://nuget.org/packages/Anotar.Log4Net.Fody 

    PM> Install-Package Anotar.Log4Net.Fody
 
 * Serilog package http://nuget.org/packages/Anotar.Serilog.Fody 

    PM> Install-Package Anotar.Serilog.Fody
 
## What happened to the old "Anotar.Fody" package

There were too many lossy abstractions happening between Anotar and the target logging libraries. So the new packages make better use of the individual libraries features. You should remove the `Anotar.Fody` package and switch to the package that targets your logging framework of choice.

## Explicit Logging

This example is targeting the [NLog](http://nlog-project.org/).

### Your Code

    public class MyClass
    {
        void MyMethod()
        {
            Log.Debug("TheMessage");
        }
    }

### What gets compiled

#### In NLog

    public class MyClass
    {
        static Logger logger = LogManager.GetLogger("MyClass");

        void MyMethod()
        {
            logger.Debug("Method: 'System.Void MyClass::MyMethod()'. Line: ~12. TheMessage");
        }
    }

#### In Log4Net

    public class MyClass
    {
        static ILog logger = LogManager.GetLogger("MyClass");

        void MyMethod()
        {
            logger.Debug("Method: 'System.Void MyClass::MyMethod()'. Line: ~12. TheMessage");
        }
    }

#### In MetroLog

    public class MyClass
    {
        static ILogger logger = LogManagerFactory.DefaultLogManager.GetLogger("MyClass");

        void MyMethod()
        {
            logger.Debug("Method: 'System.Void MyClass::MyMethod()'. Line: ~24. TheMessage");
        }
    }
    
#### In Serilog

    public class MyClass
    {
        static ILogger logger = Log.ForContext<MyClass>();

        void MyMethod()
        {
            logger
                .ForContext("MethodName", "System.Void MyClass::Debug()")
                .ForContext("LineNumber", "8")
                .Debug("TheMessage");
        }
    }

### Other Log Overloads in Explicit Logging

There are also appropriate methods for Warn, Info, Error etc as applicable to each of the logging frameworks. 

Each of these methods has the expected 'message', 'params' and 'exception' overloads. 

## Exception Logging
    
### Your code

    [LogToErrorOnException]
    void MyMethod(string param1, int param2)
    {
        //Do Stuff
    }
    
### What gets compiled

#### In NLog

    void MyMethod(string param1, int param2)
    {
        try
        {
            //Do Stuff
        }
        catch (Exception exception)
        {
            if (logger.IsErrorEnabled)
            {
                var message = string.Format("Exception occurred in SimpleClass.MyMethod. param1 '{0}', param2 '{1}'", param1, param2);
                logger.ErrorException(message, exception);
            }
            throw;
        }
    }

## Nothing to deploy

After compilation the reference to the Anotar assemblies will be removed so you don't need to deploy the assembly.
    
## But why? What purpose does this serve?

### 1. Dont make me think

When I am coding I often want to quickly add a line of logging code. If I dont already have the static `logger` field I have to jump back to the top of the file to add it. This breaks my train of thought. I know this is minor but it is still an annoyance. Static logging methods are much less disruptive to call.

### 2. I want some extra information

Often when I am logging I want to know the method and line number I am logging from. I don't want to manually add this. So using IL I just prefix the message with the method name and line number. Note that the line number is prefixed with '~'. The reason for this is that a single line of code can equate to multiple IL instructions. So I walk back up the instructions until I find one that has a line number and use that. Hence it is an approximation.

## I don't want extra information

If you don't want the method name and line number Anotar adds to log messages then create this class:

    [AttributeUsage(AttributeTargets.Assembly | AttributeTargets.Module, AllowMultiple = false, Inherited = false)]
    public class LogMinimalMessageAttribute : Attribute
    {
    }
    
And add this to AssemblyInfo.cs:

    [assembly: LogMinimalMessage]

## Why not use CallerInfoAttributes

The CallerInfoAttributes consist of  [CallerLineNumberAttribute](http://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.callerlinenumberattribute.aspx),  [CallerFilePathAttribute](http://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.callerfilepathattribute.aspx) and [CallerMemberNameAttribute](http://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.callermembernameattribute.aspx). The allow you to pass information about the caller method to the callee method. 

So some of this could be achieved using these attributes however there are a couple of points that complicate things.

### 1. Only .net 4.5 and up

So this makes it a little difficult to use with other runtimes.

### 2. Cant be used when passing arrays as `params`

Logging APIs all make use of `params` to pass arguments to a `string.Format`. Since you cant use `params` with CallerInfoAttributes most logging APIs choose not to use these attributes.

You can vote for [Compatibility between `params` with CallerInfoAttributes](http://visualstudio.uservoice.com/forums/121579-visual-studio/suggestions/2762025-caller-membername-filepath-linenumber-of-net-4-5-) 

## Who is this targeting?

This is not designed as a logging toolkit abstraction. By that I mean it is not designed to help you avoid a reference to a library or make it easier for you switch logging frameworks. So this means it is targeted at people logging from applications or services. Not for people trying to expose logging functionality from their library.
    
## Icon

Icon courtesy of [The Noun Project](http://thenounproject.com)

