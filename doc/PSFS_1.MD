# PSFS
## About
* PSFS stands for **P**avahali'**S** **F**ile**S**ystem (or **P**retty **S**hitty **F**ile**S**ystem)

* It's very easy to implement and extend for future features

* Files and directories have no size limits. Only limit is filesystem size which (not always) depends on sector length. Filesystem can be capable handling from 512GB to 32TB of data

## Table of contents

* [Superblock](#superblock)
* [Entry](#entry)
* [Link group](#link-group)
* [How to start](#how-to-start)

---

## Superblock
The Superblock contains all information about the layout of the file system

|Starting byte|Length|Type|
|:-:|:-:|:-|
|0|1|PSFS signature which is equal to 105 (0x69)|
|1|1|Filesystem version. For now, it equals 0x2|
|2|1|[Size of disk sectors](#size-of-disk-sector)|
|3|1|[Length of pointers](#length-of-pointers)|
|4|4|Number of sectors in fs|
|14|32|C-string countaining disk name|
|46|1|[FS flags](#filesystem-flags)|
|46|~|Unused data|

### Size of disk sector

Basically, just `2^(x + 7)`

|Value|Size|
|:-:|:-|
|0|128 bytes|
|1|256 bytes|
|2|512 bytes|
|3|1024 bytes|
|4|2048 bytes|
|5|4096 bytes|
|6|8192 bytes|

### Length of pointers

Pointers are just addresses of specific sectors

|Value|Length|
|:-:|:-|
|0|1 byte|
|1|2 bytes|
|2|4 bytes|

### Filesystem flags

* Fixed entry size - If true, then enty size is same as sector size. If false, then every entry has it's specific size.

|Value|Meaning|
|:-:|:-|
|0x1|Fixed entry size|
|0x2|Reserved/empty|
|0x4|Reserved/empty|
|0x8|Reserved/empty|
|0x10|Reserved/empty|
|0x20|Reserved/empty|
|0x40|Reserved/empty|
|0x80|Reserved/empty|

## Entry

Entry contains information about every file/directory.

If entry is directory they point to other entries. If entry is file, they point to file contents.

|Starting byte|Length|Type|
|:-:|:-:|:-|
|0|1|Properties|
|1|4 (reserved)|Pointer to linked entry|
|5|1|Entry size (same as sector size in superblock)|
|6|3|Unused/Reserved|
|8|4|Last access time (UNIX timestamp)|
|12|4|Last modification time (UNIX timestamp)|
|16|16|Entry name (ended by `\0`)|
|32|\~|Pointers to sectors containing either data or other entries|


### File properties

* Directory flag - if true, pointers point to other entries instead
* "Extended by link group" - last pointer points to link group
* Parent - entry linked to other and is root one
* Link - entry is linked to other
* Lock - all operations are forbidden (can be used on copying)

|Value|Meaning|
|:-:|:-|
|0x01|Read permission|
|0x02|Write permission|
|0x04|Delete permission|
|0x08|Directory flag|
|0x10|"Extended by link group" flag|
|0x20|"Parent file" flag|
|0x40|Link flag|
|0x80|Lock flag|

## Link group

**NOTE: all link groups have same size as entry they belong to**

|Starting byte|Length|Meaning|
|:-:|:-:|:-|
|0|-|Pointers to sectors containing either data or other entries|
|(last pointer)|-|Pointer to other link group|