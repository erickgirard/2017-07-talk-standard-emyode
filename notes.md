# Coding Notes

## Tracing

**Nuget Package installation offline**

```powershell
install-package "C:\Users\conta\Documents\Offline Nuget\EnterpriseLibrary.Logging.6.0.1304.0.nupkg"
```

**Logger Interface (ILogger.cs)**

```csharp
using System;

namespace Emyode.CoE.Samples.Logging
{
    public interface ILogger
    {
        void WriteWarning(string message);

        void WriteError(string message);

        void WriteError(Exception ex);

        void WriteCritical(string message);

        void WriteCritical(Exception ex);

        void WriteInformation(string message);

        void WriteVerbose(string message);
    }
}
```

**Event Viewer Logger**

**LogCategory est important ici!**

```csharp
using System;
using System.Diagnostics;
using System.Text;
using Microsoft.Practices.EnterpriseLibrary.Logging;

namespace Emyode.CoE.Samples.Logging
{
    public class GenericLogger : ILogger
    {
        private static readonly string LogCategory = "General";

        static GenericLogger()
        {
            Logger.SetLogWriter(new LogWriterFactory().Create());
        }

        public void WriteVerbose(string message)
        {
            WriteLog(message, TraceEventType.Verbose, Priority.Trace);
        }

        public void WriteInformation(string message)
        {
            WriteLog(message, TraceEventType.Information, Priority.Low);
        }

        public void WriteWarning(string message)
        {
            WriteLog(message, TraceEventType.Warning, Priority.Medium);
        }

        public void WriteError(string message)
        {
            WriteLog(message, TraceEventType.Error, Priority.High);
        }

        public void WriteError(Exception ex)
        {
            WriteLog(FormatErrorExceptions(ex), TraceEventType.Error, Priority.High);
        }

        public void WriteCritical(string message)
        {
            WriteLog(message, TraceEventType.Critical, Priority.Critical);
        }

        public void WriteCritical(Exception ex)
        {
            WriteLog(FormatErrorExceptions(ex), TraceEventType.Critical, Priority.Critical);
        }

        private void WriteLog(string message, TraceEventType severity, Priority priority)
        {
            Logger.Write(new LogEntry
            {
                TimeStamp = DateTime.Now,
                Categories = new[] { LogCategory },
                Severity = severity,
                Priority = (int)priority,
                Message = message
            });

            Logger.FlushContextItems();
        }

        private string FormatErrorExceptions(Exception ex)
        {
            var message = new StringBuilder();
            var execeptionToLog = ex;

            while (execeptionToLog != null)
            {
                message.AppendLine(string.Format("Message: {0}", execeptionToLog.Message));
                message.AppendLine("-----------");
                message.AppendLine(string.Format("StackTrace: {0}", execeptionToLog.StackTrace));
                execeptionToLog = execeptionToLog.InnerException;
            }

            return message.ToString();
        }

        public enum Priority
        {
            Critical = 25,

            Debug = 4,

            High = 20,

            Low = 10,

            Medium = 15,

            /// <summary>
            /// The default value if one is not specified in Enterprise Library Logging is -1. 
            /// </summary>
            None = -1,

            /// <summary>
            /// The Tracer class in Enterprise Library Logging uses a priority of 5.
            /// </summary>
            Trace = 5
        }
    }
}​
```

## Error Handling
### AJOUTER SYSTEM.CONFIGURATION DANS REFS!

**Nuget Package installation offline**

```powershell
install-package "C:\Users\conta\Documents\Offline Nuget\EnterpriseLibrary.ExceptionHandling.6.0.1304.0.nupkg"
install-package "C:\Users\conta\Documents\Offline Nuget\EnterpriseLibrary.ExceptionHandling.Logging.6.0.1304.0.nupkg"
```

** Program.cs **

```csharp
using System;
using Microsoft.Practices.EnterpriseLibrary.Common.Configuration;
using Microsoft.Practices.EnterpriseLibrary.ExceptionHandling;
using Microsoft.Practices.EnterpriseLibrary.Logging;

namespace MyApplication
{
    class Program
    {
        static void Main(string[] args)
        {
            var logger = new GenericLogger();

            try
            {
                logger.WriteInformation("application started");
                Console.WriteLine("app start");

                throw new DivideByZeroException();
            }
            catch (Exception e)
            {                
                // load the configuration in the web.config/app.config file
                var factory = new ExceptionPolicyFactory(ConfigurationSourceFactory.Create());            
                
                // create an exception managed based on the configuration factory
                var exceptionManager = factory.CreateManager();

                // log the exception
                // Note: you should use here the same name as the one you configured in "Error Policy"
                exceptionManager.HandleException(filterContext.Exception, "Policy");
            }

            Console.WriteLine("Application Ended");
            Console.ReadLine();
        }
    }
}
```

## Application web : 

**HandleErrorFilter.cs**

```csharp
using System;
using System.Web.Mvc;
using System.Web.Routing;
using Emyode.CoE.Samples.Logging;

namespace MyApplication.ErrorManagement
{
    public class HandleErrorFilter : HandleErrorAttribute
    {
        public override void OnException(ExceptionContext filterContext)
        {
            // set the logger writer instance
            Logger.SetLogWriter(new LogWriterFactory().Create());

            // load the configuration in the web.config/app.config file
            var factory = new ExceptionPolicyFactory(ConfigurationSourceFactory.Create());            
            // create an exception managed based on the configuration factory
            var exceptionManager = factory.CreateManager();

            // log the exception
            // Note: you should use here the same name as the one you configured in "Error Policy"
            exceptionManager.HandleException(filterContext.Exception, "Exception Policy");

            base.OnException(filterContext);
        }
    }
}
```

** FilterConfig.cs**

```csharp
using System.Web.Mvc;
using MyApplication.ErrorManagement;

namespace MyApplication
{
    public class FilterConfig
    {
        public static void RegisterGlobalFilters(GlobalFilterCollection filters)
        {
            filters.Add(new HandleErrorFilter());
        }
    }
}
```

## 404-500

** Web.config**

```xml
<configuration>
  (...)
  <system.web>
    <customErrors mode="RemoteOnly">
      <error statusCode="404" redirect="~/error404" />
    </customErrors>
  </system.web>
  (...)
</configuration>
```

## Par défaut vas utiliser la page error.cshtml dans shared quand error 500 avec base.OnException
## peut être modifié
## 404 a besoin de créer controleur


## Unit Testing

**Nuget Package installation offline**

```powershell
install-package "C:\Users\conta\Documents\Offline Nuget\Moq.4.7.99.nupkg"
install-package "C:\Users\conta\Documents\Offline Nuget\AutoFixture.3.50.3.nupkg"
```