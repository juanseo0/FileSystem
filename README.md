# BFS - a Unix-Like FileSystem

The filesystem comprises 3 layers. From top to bottom, these are:
 - fs – user-level filesystem, with functions like fsOpen, fsRead, fsSeek, fsClose.
 - bfs – functions internal to BFS such as bfsFindFreeBlock, bfsInitFreeList.
 - bio - lowest level block IO functions: bioRead and bioWrite.

## bio (Block IO level)
A raw, unformatted BFS disk consists of a sequence of 100 blocks, each 512 bytes in size. The BFS disk is
implemented on the host filesystem as a file called BFSDISK. Blocks are numbered from 0 up to 99. Block
0 starts at byte offset 0 in BFSDISK. The next block starts at byte offset 512, and so on. We can read an
entire block from disk into memory. And we can write an entire block from memory onto a disk block.

It’s perfectly legal to update an existing disk block by over-writing its contents.
It is not possible to read only part of a block (for example, its middle 100 bytes) from disk. It’s all or
nothing. And the same for writing to a disk block: exactly 512 bytes, no more, no less.

The bio (for “Block Input Output”) interface lets us read and write whole blocks of data on disk, as follows:
- i32 bioRead(i32 dbn, void* buf) – read block number dbn from the disk into the memory array
buf. Return 0 for success. Any failure will abort the program.
- i32 bioWrite(i32 dbn, void* buf) – write the contents of the memory array buf into block number
dbn on disk. Return 0 for success. Any failure will abort the program.

i32 is a typedef for 32-bit signed int. See alias.h for details.

If the program encounters a fatal error, it prints out the file and line number where the error arose,
together with a description of the error. See error.h and error.c for details.

## fs (User-level fileystem)
An interface of raw, fixed-size blocks is too primitive for regular use. BFS presents a more user-friendly
interface: one that lets us create files, where “file” is a collection of zero or more bytes on disk, reached
by a user-supplied name. The bytes that comprise the file may be scattered over the disk’s blocks, rather
than having to occupy a contiguous sequence of block numbers. Files can be grown automatically as a
user writes more and more data into that file

The diagram below shows a sequence of disk blocks, containing 4 files, called Hi.txt, touch, Install.log and
solitaire. Hi.txt is small, and uses up just one disk block (in green). touch is larger: it uses 7 blocks, spread
across the disk. Similarly for the other two files. The uncolored blocks are free – available to be allocated
to future files; or to existing files, should they be extended.

BFS keeps track of those file names, and the blocks their files occupy. It also remembers where all the
free blocks are on the disk.

The filesystem, or fs, interface comprises the following functions. Notice that BFS lets us read or write an
arbitrary number of bytes (not restricted by block size), at any arbitrary byte-offset within the file.

We call the numbers used to describe blocks within a file as File Block Numbers, or FBNs. We call the
numbers used to describe blocks within the BFS disk as Disk Block Numbers, or DBNs. Both FBNs and
DBNs start at 0.

## bfs (raw IO level)
BFS uses certain blocks – which we will call metablocks – at the start of the BFS disk, to remember where
all its files are stored, and to keep track of all free blocks. These metablocks are not visible to the
programmer via the fs interface.
(As explained below, there are 3 metablocks: SuperBlock, Inodes and Directory, occupying the DBNs 0, 1
and 2 of the BFS disk)
The BFS (BFSDISK) disk is 100 blocks long. It can hold, at most, just 8 files. As will be seen later, the biggest
file supported by BFS is 5+256 blocks. So a full disk would require 8 * (5 + 256) = 2,088 blocks. This is
larger that the BFS disk size of 100 block (on purpose); so BFS needs to check against “disk full”.
The metablocks are used to hold 3 kinds of structure, as follows:
SuperBlock
Always DBN 0 in BFSDISK. Holds 4 numbers that characterize the disk:
 - numBlocks: total number of blocks in BFSDISK: always 100
 - numInodes: total number of inodes in BFSDISK: always 8
 - numFree: total number of free block in BFSDISK
 - firstFree: DBN of the first free block in BFSDISK

The freelist is kept as a linked list: the first u16 cell in each free block holds the block number of the
next free block. (u16 is a typedef for a 16-bit unsigned integer. See alias.h for details)
Directory
The Directory is a simple mapping from filename to Inode number (see later). Each filename can be
up to 15 characters long (plus a trailing NUL).


### Files

- bfs.h/c : Functions internal to bfs
- bio.h/c : Lowest level block io functions
- deb.h/c : Functions to help debug
- errrors.h/c : Functions to handle errors
- fs.h/c : User Level File System
- p5test.h/c : Test output of program
- main.c
  
### Files for testing
- BFSDISK

### Testing
- Build the source files - *.c and *.h and run


