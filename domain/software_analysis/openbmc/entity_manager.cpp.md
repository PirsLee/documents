# main

## surface model

```mermaid
sequenceDiagram
    participant M as main()
    participant IO as boost::asio::io_context
    participant B as systemBus
    participant EM as EntityManager
    participant OS as object_server
    participant C as Configuration
    participant F as D-Bus filters

    M->>M: Create configuration directories
    M->>M: Create schema directory
    M->>IO: Create io_context
    M->>B: Create D-Bus connection
    M->>B: request_name(emDbusName)

    M->>EM: Construct EntityManager
    EM->>OS: Create object_server(skipManager=true)
    EM->>C: Load configuration
    C-->>EM: Return probeInterfaces
    EM->>OS: add_manager(inventoryPath)
    EM->>OS: add_interface(emDbusPath, emDbusName)
    EM->>OS: register_method("ReScan")
    EM->>OS: initialize EntityManager interface
    EM->>F: initFilters(probeInterfaces)
    F-->>EM: Register D-Bus signal matches
    EM-->>M: Constructor completed

    M->>IO: post(propertiesChangedCallback)
    M->>EM: handleCurrentConfigurationJson()
    EM-->>M: Cache previous configuration if valid

    M->>IO: run()

    IO->>EM: propertiesChangedCallback()
    EM->>IO: Start 500 ms debounce timer
    IO->>EM: propertiesChangedCallbackDebounced()
    EM->>EM: Start PerformScan
    EM->>EM: Update systemConfiguration
    EM->>OS: Publish discovered interfaces

```

## domain model

EntityManager Work Model...

# EntityManager :: EntityManager

## surface model

1. register /xyz/openbmc_project/inventory in object server to manage inventory subtree.

2. regitster entity manager dbus path : /xyz/openbmc_project/EntityManager and its dbus name : xyz.openbmc_project.EntityManager in object server.

3. register ReScan method under the em interface.

4. register ReScan method to D-Bus.

5. register interface filter to trigger ReScan.

## domain model

```mermaid
flowchart TD
    ObjectServer --creates--> server::manage_t --register--> org.freedesktop.DBus.ObjectManager --at--> managedTree["`Inventory Subtree
    /xyz/openbmc_project/inventory`"]

    entitymanager["`EntityManager
    D-Bus Path/Name
    xyz.openbmc.EntityManager
    + ReScan
    + ReScanFilter`"] --update--> managedTree

    entitymanager -- interface register to --> ObjectServer

    dbusstandardintf["`org.freedesktop.DBus
    org.freedesktop.DBus.ObjectManager
    org.freedesktop.DBus.Properties`"]

    entitymanager --filter depends these intf--> dbusstandardintf
```

# EntityManager::propertiesChangedCallback()
# EntityManager::propertiesChangedCallbackDebounced()

## surface model

```mermaid
sequenceDiagram
    participant D as D-Bus event
    participant C as propertiesChangedCallback
    participant T as Timer
    participant S as PerformScan
    participant P as publishNewConfiguration

    D->>C: InterfacesAdded / PropertiesChanged
    C->>C: propertiesChangedInstance++
    C->>T: start or reset 500 ms timer

    D->>C: another event
    C->>T: cancel old wait and reset timer

    T->>C: timer expires
    C->>C: propertiesChangedCallbackDebounced()

    alt scan already running
        C->>C: schedule another debounce
    else no scan running
        C->>S: start PerformScan
        S->>C: scan-complete callback
        C->>P: publish new D-Bus configuration
    end
```

## domain model

Listener Model inside EntityManager...