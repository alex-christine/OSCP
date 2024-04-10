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

# Component Object Model

The [Component Object Model](https://learn.microsoft.com/en-us/windows/win32/com/component-object-model--com--portal) (COM) is a fundamental architecture in the Microsoft Windows operating system, designed to enable software components to communicate and interact with each other in a seamless and efficient manner. At its core, COM is a binary interface standard that defines how objects in software components should interact with each other regardless of the programming language they are written in.

One of the key aspects of COM is its emphasis on interoperability. By adhering to the COM standard, developers can create software components that can be used by applications written in different programming languages, running on various hardware architectures. This interoperability is crucial in a heterogeneous computing environment like Windows, where a wide array of applications and services need to work together.

COM achieves interoperability through a set of well-defined interfaces, which specify how objects interact with each other. These interfaces provide a contract between the client application and the component, outlining the methods and properties that can be accessed. This abstraction of interfaces allows components to be easily replaced or upgraded without impacting the applications that use them, promoting modularity and scalability in software development.

The COM plays a critical role in the Windows ecosystem by providing a robust framework for building modular, interoperable, and scalable software components.

## Distributed COM

An important aspect of COM is its support for distributed computing. Through technologies like [Distributed COM](https://learn.microsoft.com/en-us/openspecs/windows\_protocols/ms-dcom/4a893f3d-bd29-48cd-9f43-d9777a4415b0) (DCOM), COM enables communication between software components across different machines on a network. This capability is essential for building distributed systems and client-server architectures, allowing applications to leverage remote resources seamlessly.

DCOM allows applications to instantiate and access the properties and methods of COM objects on a remote computer just like objects on the local machine using the [`[MS-RPCE]`](https://learn.microsoft.com/en-us/openspecs/windows\_protocols/ms-rpce/290c38b1-92fe-4229-91e6-4fc376610c15) DCOM remote protocol. Information about the identity, the implementation and the configuration of every COM (and DCOM) object is stored in the registry, and associated with a few important identifiers:

* `CLSID`: **Class Identifier** is a GUID, which acts as a unique identifier for a COM class, and every class registered in Windows is associated with a CLSID
  * The CLSID key in the registry points to the implementation of the class, using the `InProcServer32` subkey in case of a dll-based object, and the `LocalServer32` key in case of an exe
* `ProgID`: **Programmatic Identifier** is an optional identifier, which can be used as a more user-friendly alternative to a CLSID, as it does not have to adhere to the intimidating GUID format of `CLSID`s
  * For example, `System.AppDomainManager` is much easier on the eyes than a GUID
  * `ProgID`s are not guaranteed to be unique, and unlike CLSID, not every class is associated with a ProgID
* `AppID` : **Application Identifier** is used to specify the configuration of one or more COM objects associated with the same executable
  * Includes the permissions given to various groups to instantiate and access the associated classes, both locally and remotely

To make a COM object accessible by DCOM, an `AppID` must be associated with the `CLSID` of the class and appropriate permissions need to be given to the `AppID`. A COM object without an associated `AppID` cannot be directly accessed from a remote machine.

The instantiation of a remote DCOM object behaves as follows:

1. The client machine requests an instantiation of an object denoted by a `CLSID` from a remote machine. If the client uses a `ProgID`, it is first resolved locally to a `CLSID`.
2. The remote machine checks if there is an `AppID` associated with the `CLSID` in question, and verifies the permissions of the client.
3. If all goes well, the `DCOMLaunch` service creates an instance of the requested class, most commonly by running the executable of the `LocalServer32` subkey, or by creating a `DllHost` process to host a dll referenced by the `InProcServer32` subkey.
4. Communication is established between the client application and the server process. In most cases, the new process is created in the session associated with the DCOM communication.
5. The client is then able to access the members and methods of the newly created object.

DCOM has been found to be a valuable source of lateral movement techniques inside Active Directory. Some of those techniques are described [here](../active-directory/lateral-movement/dcom.md).
