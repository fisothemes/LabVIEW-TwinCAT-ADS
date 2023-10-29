# LabVIEW TwinCAT ADS

[![Image](https://www.vipm.io/package/fisothemes_lib_labview_twincat_ads/badge.svg?metric=installs)](https://www.vipm.io/package/fisothemes_lib_labview_twincat_ads/) [![Image](https://www.vipm.io/package/fisothemes_lib_labview_twincat_ads/badge.svg?metric=stars)](https://www.vipm.io/package/fisothemes_lib_labview_twincat_ads/)

Easy to use Unofficial LabVIEW TwinCAT API for communicating with Beckhoff PLCs via ADS. Features include Invoking Rpc Methods, Reading/Writing of PLC variables (including all Standard types, Pointers, References, Time, Strings, WStrings, Structs composed of the aforementioned, Nested STRUCTs and Multi-dimensional Arrays of the aforementioned datatypes), introspective capabilities (get the symbol and type information), the ability to build your own low-level VIs via Extensions and more. 


# Minimum Requirements
* LabVIEW 2018 (32/64 bit) and above
* .NET Framework 4.6.2 (installed by default with LabVIEW)
* TwinCAT 3.1.4024.10 and above (haven't tested below 4024.10 might work with lower versions)


# Changelog

New Features:
-------------
+ Added VI for adding multiple symbol notifications.
+ Added VI for deleting multiple symbol notifications.
+ Added VI for clearing all symbol notifications.
+ Added VI for getting the names and cycle times for all symbol notifications.
+ Added extension VI for getting notification handle.
+ Added extension VI for deleting notification handle.
+ Added extension VI for getting symbol handle.
+ Added extension VI for removing symbol handle.

Fixes:
------
+ Fixed failure in application build due to Flatten .NET Symbol Information.vi.
+ Fixed nested function blocks not being read properly.
+ Fixed premature closure of cached .NET Symbol Information references by LabVIEW's garbage collector when calling the API's VIs in an Asynchronous VI.
+ Fixed cases where characters after the null character in strings would show.
+ Fixed case where you couldn't use the event registration refnum as a control, indicator or constant.

Improvements:
-------------
+ Read.vim automatically changes to read multiple when connecting an array of symbol names.

# Showcase
VIs included in the API:

![Block Diagram](./assets/images/showcase.png)

# Examples
Invoke Method:
-

![Block Diagram](./assets/images/invoke%20method.vi-block-diagram.png)

![Front Panel](./assets/images/invoke-method.vi-front-panel.png)

![TwinCAT XAE Shell Code](./assets/images/invoke%20method.vi-tcxaeshell-code.png)

![TwinCAT XAE Shell Online](./assets/images/invoke%20method.vi-tcxaeshell-online.png)

Event-Driven Reading:
-

![Block Diagram](./assets/images/event%20read.vi-block-diagram.png)

![Front Panel](./assets/images/event-read.vi-front-panel.png)

Write Symbol:
-

![Block Diagram](./assets/images/write%20to%20symbol.vi-block-diagram.png)

![Front Panel](./assets/images/write%20to%20symbol.vi-front-panel.png)

![TwinCAT XAE Shell](./assets/images/write%20to%20symbol.vi-tcxaeshell.png)

Read Symbol:
-
![Front Panel](./assets/images/read%20from%20symbol.vi-front-panel.png)

![Block Diagram](./assets/images/read%20from%20symbol.vi-block-diagram.png)

![TwinCAT XAE Shell](./assets/images/write%20from%20symbol.vi-tcxaeshell.png)

Pointer Support:
-
![Block Diagram](./assets/images/pointer.vi-block-diagram.png)

![Front Panel](./assets/images/pointer.vi-front-panel.png)

![TwinCAT XAE Shell](./assets/images/pointer.vi-tcxaeshell.png)

High-Speed Reading:
-
![Block Diagram](./assets/images/high-speed%20reading.vi-block-diagram.png)

![Front Panel](./assets/images/high-speed%20reading.vi-front-panel.png)


# Developer Notes

Connection
----------
1. You may notice that the `Connect.vi` is slow in connection. This is normal.
    The VI is simply caching the symbols and types in LabVIEW rather than keeping them in a .NET collection. The reason for this is that lookup times on the .NET Ads-XYZ-Collection are very slow. The further I moved away from making calls to any of Beckhoff's .NET libraries the faster operations became. The penalty for doing this is a slow connection.

2. Because of the caching system, online changes are currently not supported.

3. You do not need to disconnect before disposal. `Dispose.vi` VI will disconnect for you.

Read, Write and Invoke Method
---------------------------------

1. Operations involving `UNION`s and `INTERFACE` types are not yet supported.

2. To read and write to pointers simply add the dereferencing symbol to your symbol name e.g. `pValue^`.

3. First calls to an operation (Reading/Writing a symbol or Invoking a Method) will be slow.

    Typical first call times for my RYZEN 3600X system with 1 isolated core are ~5ms for `ARRAY[0..999] OF LREAL` and ~40ms for `ARRAY[0..999] OF ST_STRUCT` where `ST_STRUCT` is:
    ```pascal
    TYPE ST_STRUCT :
    STRUCT
        bVar  : BOOL;
        dtVar : DT;
        fVar  : LREAL;
    END_STRUCT
    END_TYPE
    ```
    Subsequent calls resulted in read/write/method invoking times of 1-2ms for `ARRAY[0..999] OF LREAL` and ~20ms for `ARRAY[0..999] OF ST_STRUCT`. The task cycle time was 1ms.

4. When sending numbers, arrays of numbers or structs with numbers, enums, etc. The numbers will be coerced to the type defined in the PLC for that variable/parameter. This is by design. LabVIEW is a graphical language so switching between Single Precision Float and Doubles is an absolute pain.

5. LabVIEW Arrays don't need to be the same size as the PLC arrays to write them. Write what you need, the rest will be filled in with defaults.


Events
------

1. Online change is not supported. However, you can update variables on the PLC without needing to disconnect on the LabVIEW side via `Login with Download`, `Restart TwinCAT System` and `Activate TwinCAT Configuration`. All symbols will automatically be reloaded into memory.

    *NOTE: Any reads or writes running during this process will produce errors, just ignore these errors until reconnection is complete. They will stop once that happens.*

2. Don't forget to close the Registration Refnum after you're done. User Events references are automatically closed on disposal.

Other
------
1. When using extensions. Please only close the .NET references with an asterisk (*) on their label at the connector pane. The ones without an * are managed for you.

**Please feel free to contribute to the project or report bugs**
- - - -
