# LabVIEW TwinCAT ADS
Easy to use Unofficial LabVIEW TwinCAT API for communicating with Beckhoff PLCs via ADS. Features include Invoking Rpc Methods, Reading/Writing of PLC variables (including all Standard, Time, Strings, WStrings, Structs composed of the aforementioned, Nested STRUCTs and Multi-dimensional Arrays of the aforementioned datatypes), introspective capabilities (get the symbol and type information), the ability to build your own low-level VIs via Extensions and more. 


# Minimum Requirements
* LabVIEW 2018 (32/64 bit) and above
* .NET Framework 4.6.2 (installed by default with LabVIEW)
* TwinCAT 3.1.4024.10 and above (haven't tested below 4024.10 might work with lower versions)

# Showcase
VIs included in the API:

![Block Diagram](./assets/images/showcase.PNG)

# Examples
**Invoke Method**

![Block Diagram](./assets/images/invoke%20method.vi-block-diagram.png)

![Front Panel](./assets/images/invoke-method.vi-front-panel.png)

![TwinCAT XAE Shell Code](./assets/images/invoke%20method.vi-tcxaeshell-code.png)

![TwinCAT XAE Shell Online](./assets/images/invoke%20method.vi-tcxaeshell-online.png)

 **Writing to Symbol:**

![Block Diagram](./assets/images/write%20to%20symbol.vi-block-diagram.png)

![Front Panel](./assets/images/write%20to%20symbol.vi-front-panel.png)

![TwinCAT XAE Shell](./assets/images/write%20to%20symbol.vi-tcxaeshell.png)

 **Reading from Symbol:**

![Front Panel](./assets/images/read%20from%20symbol.vi-front-panel.png)

![Block Diagram](./assets/images/read%20from%20symbol.vi-block-diagram.png)

![TwinCAT XAE Shell](./assets/images/write%20from%20symbol.vi-tcxaeshell.png)


# Developer Notes

* **Connection**
1. You may notice that the `Connect.vi` is slow in connection. This is normal.
    The VI is simply caching the symbols and types in LabVIEW rather than keeping them in a .NET collection. The reason for this is that lookup times on the .NET Ads-XYZ-Collection are very slow. The further I moved away from making calls to any of Beckhoff's .NET libraries the faster operations became. The penalty for doing this is a slow connection.

2. Because of the caching system, online changes are currently not supported. I have a workaround in mind but that currently requires me to implement callbacks and events. I have them working on my private repo but I am currently not happy with the implementation.

* **Read, Write and Invoke Method**

1. Operations involving `REFERENCE TO X`, `UNION`s and `POINTER TO X` types are not yet supported.

2. First calls to an operation (Reading/Writing a symbol or Invoking a Method) will be slow.

    Typical first call times for my RYZEN 3600X system with 1 isolated core are ~5ms for `ARRAY[0..999] OF LREAL` and ~15ms for `ARRAY[0..99] OF ST_STRUCT` where `ST_STRUCT` is:
    ```pascal
    TYPE ST_STRUCT :
    STRUCT
        bVar  : BOOL;
        dtVar : DT;
        fVar  : LREAL;
    END_STRUCT
    END_TYPE
    ``` 

    Subsequent calls resulted in read/write/method invoke times of 1-2ms for `ARRAY[0..999] OF LREAL` and ~6ms for `ARRAY[0..99] OF ST_STRUCT`.

3. Enums must include the type in their definition.
    You include a type to an enum like so:
    ```Pascal
    TYPE E_OPERATION :
    (
        Array_Sum 	:= 0,
        Array_Product
    )UDINT;
    END_TYPE
    ```
    Notice there is a type of `UDINT` end of the definition. This corresponds to `Enum U32` on LabVIEW. Not including a type may result in undefined behavior. LabVIEW uses `Enum U16` by default which is a `UINT`. Do what you may with that knowledge.

4. Besides enums, when sending numbers, arrays of numbers or structs with numbers, etc. The numbers will be coerced to the type defined in the PLC for that variable/parameter. This is by design. LabVIEW is a graphical language so switching between Single Precision Float and Doubles is an absolute pain. There are no aliases in LabVIEW. 


* **Events**

There is already code for events in the API. Currently working on that.

**Please feel free to contribute to the project or report bugs**
- - - -
