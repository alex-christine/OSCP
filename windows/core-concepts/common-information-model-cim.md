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

# Common Information Model (CIM)

The [CIM](https://learn.microsoft.com/en-us/windows/win32/wmisdk/common-information-model) is an extensible, object-oriented data model that contains information about different parts of an enterprise. The CIM is a cross-platform open standard maintained by the Distributed Management Task Force ([DMTF](https://www.dmtf.org/)). It defines how managed elements in an IT environment are represented as a common set of objects and relationships between them.

The CIM is a language-independent programming model that uses object-oriented techniques to describe an enterprise. Using three levels of parent/child inheritance, the CIM can describe both general and specific aspects of an enterprise. The CIM also uses a technique called "association" to link different parts of the enterprise model together, and uses schemas to distinguish different management environments.

The CIM is designed to present a consistent view of logical and physical objects in a management environment. The CIM represents managed objects using an object-oriented construct called a "class." Like a C++ or COM class, a CIM class can include properties to describe data and methods to describe behavior. Like a set of COM classes, the CIM is not tied to any platform. However, [WMI](common-information-model-cim.md#windows-management-instrumentation) includes an extension to the CIM that describes the Microsoft Windows operating system platforms.

## CIM Schema

The [CIM Schema](https://en.wikipedia.org/wiki/CIM\_Schema) is a conceptual schema which defines the specific set of objects and relationships between them that represent a common base for the managed elements in an IT environment. The CIM Schema covers most of today's elements in an IT environment, for example computer systems, operating systems, networks, middleware, services and storage. Classes can be, for example: `CIM_ComputerSystem`, `CIM_OperatingSystem`, `CIM_Process`, `CIM_DataFile`. The CIM Schema defines a common basis for representing these managed elements. Since most managed elements have product and vendor specific behavior, the CIM Schema is extensible in order to allow the producers of these elements to represent their specific features seamlessly together with the common base functionality defined in the CIM Schema.

### Class Levels

The CIM defines three levels of classes:

1. **Core:** represents managed objects that apply to all areas of management. These classes provide a basic vocabulary for analyzing and describing managed systems.
   * The [**\_\_Parameters**](https://learn.microsoft.com/en-us/windows/win32/wmisdk/--parameters) and [**\_\_SystemSecurity**](https://learn.microsoft.com/en-us/windows/win32/wmisdk/--systemsecurity) classes are examples of core classes
2. **Common:** represents managed objects that apply to specific management areas. However, common classes are independent from a particular implementation or technology. Common classes are an extension of the core classes.&#x20;
   * The [**CIM\_UnitaryComputerSystem**](https://learn.microsoft.com/en-us/windows/desktop/CIMWin32Prov/cim-unitarycomputersystem) class is an example of a common class
3. **Extended:** represents managed objects that are technology-specific additions to the common classes. An extended class typically applies to a specific platform, such as UNIX or the Microsoft Win32 environment.
   * The [**Win32\_ComputerSystem**](https://learn.microsoft.com/en-us/windows/desktop/CIMWin32Prov/win32-computersystem) class is an example of an extended class

### Windows Management Instrumentation

Windows Management Instrumentation ([WMI](https://learn.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page)) consists of a set of extensions to the Windows Driver Model that provides an operating system interface through which instrumented components provide information and notification. **WMI is Microsoft's implementation of the Web-Based Enterprise Management (WBEM) and Common Information Model (CIM) standards** from the Distributed Management Task Force (DMTF).

Through , a developer can use the CIM to create classes that represent hard disk drives, applications, network routers, or even user-defined technologies, such as a networked air conditioner. By viewing and making changes to a CIM class, a manager can control different aspects of the enterprise. For example, a manager could query a CIM class instance representing a desktop workstation. The manager could then run a script to modify the CIM workstation instance. WMI would translate any change to the workstation CIM class instance into a change to the actual workstation.

## WS-Management

[WS-Management](https://en.wikipedia.org/wiki/WS-Management) (Web Services-Management) is a DMTF open standard defining a [SOAP](https://en.wikipedia.org/wiki/SOAP)-based protocol for the management of servers, devices, applications and various Web services. WS-Management provides a common way for systems to access and exchange management information across the IT infrastructure. It is a firewall-friendly protocol for management clients to communicate with CIM servers.

### Windows Remote Management

[Windows Remote Management](https://learn.microsoft.com/en-us/windows/win32/winrm/portal) (WinRM) is the Microsoft implementation of the WS-Managemet protocol. The WS-Management protocol specification provides a common way for systems to access and exchange management information across an IT infrastructure. WinRM and the [Intelligent Platform Management Interface (IPMI)](https://learn.microsoft.com/en-us/windows/win32/winrm/windows-remote-management-glossary#i) standard, along with the [Event Collector service](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc785056\(v=ws.10\)#event-collector) are components of the set of features known as [Hardware management](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc785056\(v=ws.10\)).

The intended audience for Windows Remote Management is IT professionals—who write scripts to automate the management of servers—and independent software vendor (ISV) developers, who want to obtain data for management applications.

To obtain management data from local and remote computers that might have [baseboard management controllers (BMCs)](https://learn.microsoft.com/en-us/windows/win32/winrm/windows-remote-management-glossary), you can use: WinRM scripting objects; the WinRM command-line tool; or the Windows Remote Shell (WinRS) command-line tool. If the computer runs a Windows-based operating system version that includes WinRM, then the management data is supplied by [Windows Management Instrumentation (WMI)](https://learn.microsoft.com/en-us/windows/win32/WmiSdk/wmi-start-page).

You can also obtain hardware and system data from WS-Management protocol implementations running on operating systems other than Windows in your enterprise. WinRM establishes a session with a remote computer through the SOAP-based WS-Management protocol rather than a connection through DCOM, as WMI does. Data returned using the WS-Management protocol is formatted in XML instead of as objects.
