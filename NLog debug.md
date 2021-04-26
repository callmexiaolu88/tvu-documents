NLog debug

set properies in the `nlog` element of `NLog.config`

- throwConfigExceptions="true"

- throwExceptions="true"

- internalLogLevel="Trace"

- internalLogFile="D:\TestTemp\NlogInternal.log"

- internalLogToConsole="true"

- internalLogToConsoleError="true"

- internalLogToTrace="true"

```xml
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      autoReload="true" throwConfigExceptions="true"
      throwExceptions="true"
      internalLogLevel="Trace"
      internalLogFile="D:\TestTemp\NlogInternal.log"
      internalLogToConsole="true"
      internalLogToConsoleError="true"
      internalLogToTrace="true">
 </nlog>
```

