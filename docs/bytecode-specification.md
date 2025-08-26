# ARCSIN BYTECODE COMPLETE (.ABC) SPECIFICATION
> VERSION 1α, SUBJECT TO FUTURE CHANGE, UNSTABLE.
## 1. Introduction
ARCSIN (*as in the trigonometric function*, pronounced [ˈark.saɪn] or ARK-sine), a modern programming language, uses a platform-agnostic bytecode format which may be interpreted or compiled into a platform-dependent executable (such as *Windows EXE*, *ELF*, or otherwise). This specification will provide the format by which all programs turn ARCSIN code into an ABC, different from the ARCSIN Object File (.arco), and an ABC interpreted.

## 2α. File Header
.ABC files use a specific header which include a magic number and multiple declarations providing addresses for different segments. The first 12-byte header is as follows:
| Offset| Value | Description |  
|---|---|---|  
| 0x0 | `0x41524353494E414243` | 9-byte Magic Number (`ARCSINABC`) |  
| 0x9 | `0x00FE00` | 3-byte Split Marker, also functions as part of the magic number |

## 2β. Segment Address Descriptors (SADs)
### 2β.1. What even is a Segment Address Descriptor in ARCSIN? 
A Segment Address Descriptor is a descriptor segment within the header providing addresses for different data segments, such as file metadata, checksums, and the actual code.
### 2β.2. Primary Segment Address Descriptors
There are eight primary SADs, with four working descriptors:
- File Segment Address Descriptor (FSAD): Provides file data such as date of creation, checksum, file type (.abc or .arco).
- Metadata Segment Address Descriptor (MSAD): Provides extra metadata for the bytecode interpreter to use.
- Pre-Interpretation Segment Address Descriptor (PISAD): Provides extremely useful, often crucial, data for an interpreter to know before running the program.
- Debug Data Segment Address Descriptor (DDSAD): Provides program debug data for use by the developers and debuggers.
- Execution Segment Address Descriptor (ESAD): Provides the execution segment.
- 3 more **reserved** SADs.
### 2β.3. Segment Address Descriptor Structure
Segment Address Descriptors do not have a fixed length, but rather a fixed structure with markers for its start and end. Immediately after the 12-byte header, a 4-byte marker indicates where the segment address descriptors begin. This may be placed anywhere **AFTER** the 12-byte header, all data in-between are skipped. The 4-byte marker is `0x534144EF`. Immediately after this marker, a 2-byte primary descriptor type (FSAD = `FD` or `0x4644`, MSAD = `MD` or `0x4D44`, PISAD = `PD` or `0x5044`, DDSAD = `DD` or `0x4444`, ESAD = `ED` or `0x4544`) is expected, followed by `0x00EF` and the address (see 2β.4.), and lastly `0xFE00`, where it expects the same structure again until all **working** (FSAD, MSAD, DDSAD, and ESAD) primary descriptors are referenced.

### 2β.4. Address Structure
Segment Address Descriptors do not always use regular absolute or relative addresses. Instead, it implements multiple methods/types that may be independently used for the address. The first byte of the address indicates the method/type the address uses, there are 4 primary methods:
- `00`: Absolute Addressing Mode, references the absolute address offset from the file to the segment.
- `A0`: Relative Addressing Mode, references the relative address offset from the Header-Segment Split Marker (see 2β.5.).
- `CE`: Back Stack Addressing Mode, uses an automatic ordering from end-to-start (relative to Header-Segment Split Marker), can be unstable if incorrect.[†]
- `EC`: Front Stack Addressing Mode, uses an automatic ordering from start-to-end (relative to Header-Segment Split Marker), can be unstable if incorrect.[†]

[†]: Back/Front Stack Addressing Mode explained further in 2β.6. \
Using absolute or relative addressing requires an address immediately after the type, whereas using stack addressing requires the SAD descriptor (2β.3. `0xFE00`) immediately after, as it's automatically calculated.

### 2β.5. Header-Segment Split Marker
The Segment Address Descriptors are the final part of the header. The header is followed by a series of bytes, referred to as the *"Header-Segment Split Marker,"* separating the header from the subsequent data segments. The bytes are as follows: `0xEF2A53554253514853532AFE`.
