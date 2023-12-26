---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Windows Services

[Windows Services](https://learn.microsoft.com/en-us/dotnet/framework/windows-services/introduction-to-windows-service-applications) enables users to create long-running executable applications that run in their own Windows sessions. These services can be automatically started when the computer boots, can be paused and restarted, and do not show any user interface. These features make services ideal for use on a server or whenever a users needs long-running functionality that does not interfere with other users who are working on the same computer. One can also run services in the security context of a specific user account that is different from the logged-on user or the default computer account.

Windows services can be managed by the _Services snap-in_, PowerShell, or the `sc.exe` command line tool. Windows uses the `LocalSystem` (includes the SIDs of `NT AUTHORITY\SYSTEM` and `BUILTIN\Administrators` in its token), `Network Service`, and `Local Service` user accounts to run its own services. Users or programs creating a service can choose either one of those accounts, a domain user, or a local user.

## Service Lifetime

A service goes through several internal states in its lifetime. First, the service is installed onto the system on which it will run. This process executes the installers for the service project and loads the service into the **Services Control Manager** for that computer. The Services Control Manager is the central utility provided by Windows to administer services.

After the service has been loaded, it must be started. Starting the service allows it to begin functioning. One can start a service from the Services Control Manager, from **Server Explorer**, or from code by calling the [Start](https://learn.microsoft.com/en-us/dotnet/api/system.serviceprocess.servicecontroller.start) method. The `Start` method passes processing to the application's [OnStart](https://learn.microsoft.com/en-us/dotnet/api/system.serviceprocess.servicebase.onstart) method and processes any code you have defined there.

A running service can exist in this state indefinitely until it is either stopped or paused or until the computer shuts down. A service can exist in one of three basic states: [Running](https://learn.microsoft.com/en-us/dotnet/api/system.serviceprocess.servicecontrollerstatus#system-serviceprocess-servicecontrollerstatus-running), [Paused](https://learn.microsoft.com/en-us/dotnet/api/system.serviceprocess.servicecontrollerstatus#system-serviceprocess-servicecontrollerstatus-paused), or [Stopped](https://learn.microsoft.com/en-us/dotnet/api/system.serviceprocess.servicecontrollerstatus#system-serviceprocess-servicecontrollerstatus-stopped). The service can also report the state of a pending command: [ContinuePending](https://learn.microsoft.com/en-us/dotnet/api/system.serviceprocess.servicecontrollerstatus#system-serviceprocess-servicecontrollerstatus-continuepending), [PausePending](https://learn.microsoft.com/en-us/dotnet/api/system.serviceprocess.servicecontrollerstatus#system-serviceprocess-servicecontrollerstatus-pausepending), [StartPending](https://learn.microsoft.com/en-us/dotnet/api/system.serviceprocess.servicecontrollerstatus#system-serviceprocess-servicecontrollerstatus-startpending), or [StopPending](https://learn.microsoft.com/en-us/dotnet/api/system.serviceprocess.servicecontrollerstatus#system-serviceprocess-servicecontrollerstatus-stoppending). These statuses indicate that a command has been issued, such as a command to pause a running service, but has not been carried out yet.

## Types of Services

Two types of services can be created in Visual Studio using the .NET Framework.

1. Services that are the only service in a process are assigned the type [Win32OwnProcess](https://learn.microsoft.com/en-us/dotnet/api/system.serviceprocess.servicetype#system-serviceprocess-servicetype-win32ownprocess).&#x20;
2. Services that share a process with another service are assigned the type [Win32ShareProcess](https://learn.microsoft.com/en-us/dotnet/api/system.serviceprocess.servicetype#system-serviceprocess-servicetype-win32shareprocess).&#x20;

The service type can be retrieved by querying the [ServiceType](https://learn.microsoft.com/en-us/dotnet/api/system.serviceprocess.servicecontroller.servicetype) property.

## Service Requirements

* Services must be created in a **Windows Service** application project or another .NET Framework–enabled project that creates an .exe file when built and inherits from the [ServiceBase](https://learn.microsoft.com/en-us/dotnet/api/system.serviceprocess.servicebase) class.
* Projects containing Windows services must have installation components for the project and its services. This can be easily accomplished from the **Properties** window

## Service Enumeration

### Get-CimInstance

#### Running Services

The PowerShell cmdlet [`Get-CimInstance`](https://learn.microsoft.com/en-us/powershell/module/cimcmdlets/get-ciminstance?view=powershell-7.4) can be used to query the running services. This cmdlet gets the [CIM](common-information-model-cim.md) instances of a class from a CIM server. The caller can specify either the class name or a query for this cmdlet. It returns one or more CIM instance objects representing a snapshot of the CIM instances present on the CIM server.

The cmdlet can be used to search services by setting the `-ClassName` parameter to `win32_service`:

{% code overflow="wrap" %}
```powershell
Get-CimInstance -ClassName win32_service
```
{% endcode %}

From here the output can be refined with by piping the output to the [Select-Object](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/select-object?view=powershell-7.4) cmdlet (aliased as `Select`):

{% code overflow="wrap" %}
```powershell
Get-CimInstance -ClassName win32_service | Select Name,State,PathName | Where-Object {$_.State -like 'Running'}
```
{% endcode %}

This will select the `Name`, `State`, and `PathName` properties from the `Get-CimInstance` output. The [Where-Object](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/where-object?view=powershell-7.4) cmdlet then filters it down to only services that are running if desired.

#### Startup Type

A service can have one of 4 S_tartup Types_. These describe how/when the service is started on the machine. The startup types are:

1. **Automatic:** The service starts at system startup.
2. **Automatic (Delayed):** The service starts a short while after the system has finished starting up.
   * This option was introduced in Windows Vista in an attempt to reduce the boot-to-desktop time
   * Not all services support delayed start.
3. **Manual:** The service starts only when explicitly summoned.
4. **Disabled:** The service is disabled. It will not run.

Startup Types can be queried via the [Get-CimInstance](https://learn.microsoft.com/en-us/powershell/module/cimcmdlets/get-ciminstance?view=powershell-7.4) PowerShell cmdlet. The Startup Type is contained in the `StartMode` output column. All service information can be returned by the cmdlet when the `-ClassName` argument is set to `win32_service`:

```powershell
Get-CimInstance -ClassName win32_service
```

If only the Startup Type is needed the output can be further filtered with `Select-Object`:

```powershell
Get-CimInstance -ClassName win32_service | Select-Object Name, StartMode
```

The output can be further refined to only running services with the `Where-Object` cmdlet:

{% code overflow="wrap" %}
```powershell
Get-CimInstance -ClassName win32_service | Select Name, StartMode | Where-Object {$_.Name -like 'SERVICE_NAME'}
```
{% endcode %}

### WMIC

The [WMIC](https://learn.microsoft.com/en-us/windows/win32/wmisdk/wmic), or Windows Management Instrumentation Command-line utility, provides a command-line interface for [Windows Management Instrumentation](https://learn.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page) (WMI). WMIC is compatible with existing shells and utility commands. See [here](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc779482\(v=ws.10\)) for detailed instructions on using WMIC.

For the purposes of service enumeration it can be called with the word `service` and the `get` verb with the `name` and `pathname` arguments:

```powershell
wmic service get name,pathname
```
