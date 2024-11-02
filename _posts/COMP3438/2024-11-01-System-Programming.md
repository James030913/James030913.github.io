---
title: "COMP3438 System Programming 3 - 4"
date: 2024-11-01 20:00 +0800
categories: [PolyU, COMP3438(System Programming)]
tags: [C, OS]
author: James
---

# Lec 3 Unix file System

## Unix File system 
Provide abstrcation of naming, storage and access to a file.  
A file is container of information(data and program).  
Devices like keyboard and printer are also treated as files.

### How to handle devices?
OS provide system call for performing control and IO. And system call handled by devices drvier.  
Unix provides a uniform device interface (called file descriptors)  
Allow uniform access to most devices through file system calls – open,
close, read, write, etc.

### Type of files
***Regular FILE***  ordinary data or program file on disk  
***Special FILE***  representing a device - in /dev directory.  
* ***Block devices:*** transferring info in blocks or chunks, just like disks, CD ROM.
* ***Character devices:*** transfer information in a stream of bytes.
* ***FIFO:*** pipe.  

***Directories*** provided to allow names (not physical locations) of files to be
used.
 > User gives a file name and Unix makes a translation to the location of the physical file –
done via directories.

#### `ls -l` command
![Desktop View](/assets/img/comp3438/ls.png){: w="300" h="300"  }  

```cp /etc/passwd /tmp/garbage```  
Nothing happen  
```cp /etc/passwd /dev/tty```  
Output content to terminal

#### i-node
i-node store information about a file: size, owner id and group id, also the open, 
modify and access time, pointer.  
Store in disk after boot block and superblock

#### Hard Link vs Symbolic Link (硬連結 vs 軟連結)
Hard links must be on the same file system, while symbolic links can cross file systems.  

Hard linked files stay linked even if either one is moved (unless the move crosses file system boundaries,  
triggering copy-and-delete). Symbolic linked files break if the original is moved.  

Hard linked files are co-equal (data is deleted only when all hard links are deleted), while the original is special in symbolic links (deleting the original deletes the data, deleting the symbolic link does not delete the data).  

Symbolic link can point at any target, but most OS/filesystems disallow hardlinking directories to prevent cycles.  
 
##### Hard link:  
`ln Source Dest`  

运行上面这条命令以后，源文件与目标文件的inode号码相同，都指向同一个inode。inode信息中有一项叫做"链接数"，记录指向该inode的文件名总数，这时就会增加1。

反过来，删除一个文件名，就会使得inode节点中的"链接数"减1。当这个值减到0，表明没有文件名指向这个inode，系统就会回收这个inode号码，以及其所对应block区域。

这里顺便说一下目录文件的"链接数"。创建目录时，默认会生成两个目录项："."和".."。前者的inode号码就是当前目录的inode号码，等同于当前目录的"硬链接"；后者的inode号码就是当前目录的父目录的inode号码，等同于父目录的"硬链接"。所以，任何一个目录的"硬链接"总数，总是等于2加上它的子目录总数（含隐藏目录）

##### Symbolic link:  
`ln -s Source Dest`

文件A和文件B的inode号码虽然不一样，但是文件A的内容是文件B的路径。读取文件A时，系统会自动将访问者导向文件B。因此，无论打开哪一个文件，最终读取的都是文件B。这时，文件A就称为文件B的"软链接"（soft link）或者"符号链接（symbolic link）。

这意味着，文件A依赖于文件B而存在，如果删除了文件B，打开文件A就会报错："No such file or directory"。这是软链接与硬链接最大的不同：文件A指向文件B的文件名，而不是文件B的inode号码，文件B的inode"链接数"不会因此发生变化。

#### Current Working Directory  
***`char *getcwd(char *buf, size_t size)`***  
> size specifies maximum length pathname. If longer than the maximum, returns NULL and sets errono to ERANGE.

***Full code:***  
```c
#include <unistd.h>
#include <stdio.h>
#include <errno.h>
int main(){
    char cwd[1024];
    if (getcwd(cwd, sizeof(cwd)) != NULL) 
    printf("Current working dir: %s\n", cwd);
    else perror("getcwd() error");
    return 0;
}
```

### File descriptors
***0 STDIN_FILENO: standard input ***  
***1 STDOUT_FILENO: standard output ***  
***2 STDERR_FILENO: standard error ***  

#### Open a file 
`int open(const char* pathname, int flags)`  
`int open(const char* pathname, int flags, mode_t mode)`  
![Desktop View](/assets/img/comp3438/fd.png){: .normal } 

#### Read file
`bytes = read(fd, buffer, count)`  
***Sample***  
```c
int fd = open("someFile", O_RDONLY);
char buffer[4];
int bytes = read(fd, buffer, 4*sizeof(char));
```

#### Write file
`bytes = write(fd, buffer, count)`  
***Sample***   
```c
int fd = open("someFile", O_RDONLY);
char buffer[4];
int bytes = write(fd, buffer, 4*sizeof(char));
fsync(fd);
```
#### Close file
`close(fd)`  
Returns 0 on success, -1 on error  

### File pointer
A file pointer points to a data structure FILE, called a file structure in the user area 
of the process.  
```c
#include <stdio.h>
FILE *myfp;
if ((myfp = fopen("/home/ann/my.dat", "w")) == NULL)
    fprintf(stderr, "Could not fopen file\n");
else
    fprintf(myfp, "This is a test");
```  
 > The above code write string to a file.  
 
#### fopen and fclose
```
FILE *file_stream = fopen(path, mode)
Path: char*, absolute or relative path
```
- r – open file for reading
- r+ - open file for reading and writing
- w – overwrite file or create file for writing
- w+ - open for reading and writing; overwrites file
- a – open file for appending (writing at end of file)
- a+ - open file for appending and reading

`fclose(file_stream)`  

#### I/O redirection

`cat test > my.file`  
> redirect stdout to my.file  

  
`int dup(int oldfd)`  
> duplicates the given file descriptor to the
lowest numbered unused file descriptor in the file descriptor table.

```c
#include<stdio.h>
#include<fcntl.h>
#include<unistd.h>
char* cmd[] = {"/bin/ls", "-l", 0};
int main(int argc, char* argv[]){
    int fd = open(argv[1], O_WRONLY | O_CREAT, 0600);
    //fd will be 3; a file will be opened in write mode
    int fd2 = dup(fd); //duplicate the fd-th pointer to entry 4, the lowest available entry
    close(STDOUT_FILENO);
    dup(fd); //duplicate the fd-th pointer into entry 1
    execvp(cmd[0], cmd); //the old process image is replaced by the new process image for ls
    close(fd); //close file descriptor 3 in the parent process.
    return;
}
```

***Communication between parent/child via pipe***  
```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <string.h>
int main(){
    int fd[2];
    pipe(fd); //fd[0] is for read and fd[1] is for write
    pid_t child = fork();
    if (child == 0){ //child process
    close(fd[1]);
    char data[100]; //a large enough buffer to share data
    read(fd[0], data, sizeof(data));
    printf("%s\n", data);
    close(fd[0]);
    }else{ //parent process
    close(fd[0]);
    char* data = "hello world";
    write(fd[1], data, strlen(data)+1);
    close(fd[1]);
    }
    return 0;
}
```
>fd[1] is for writing, fd[0] is for reading  


# Lec 4 Intro to device driver

##  What is device driver?

Device driver is a special kind of library, which can be loaded into OS kernel, and links the user program with IO devices.

## OS infrastructure

#### Related to devices driver:

**Unix system architecture**  
**File subsystem and its relations with char/block device driver tables**  
**Char/block device driver tables**

#### Unix System Architecture 
![Desktop View](/assets/img/comp3438/UnixSystemArch.png){: .normal }

#### File subsystem & char/block device driver tables
![Desktop View](/assets/img/comp3438/cbdevice.png){: .normal }

#### Char/block device driver tables
![Desktop View](/assets/img/comp3438/cbtable.png){: .normal }

#### Advantage to Sperate device driver from OS 

For OS designer:  
Devices may not be available when OS is desiged.  
No need to worry about operate device eg. set up register, check status.  
Focus on OS itself in order to provide interface for device drivers development.  

Device driver designers:  
No need to worry about how I/O managed in OS.  
Focus on implementing functions of devices with device-related commands
following the generic I/O interface.  

#### Devices and Files

use `mknod <file_name> <c or b> <major_number> <minor_number>` to create device file.  

#### Type of device

Two type of device driver look like    
![Desktop View](/assets/img/comp3438/twotype.png){: .normal }  

##### Block Device Driver
Communicate with the OS through a collection of fixed size buffers  
![Desktop View](/assets/img/comp3438/block.png){: .normal }  
Driver is invoked when requested data is not in the cache; when buffers in the cache have been changed and must be written out (write back to the devices).  
By using buffer cache, block
drivers are insulated from
the many details of user
requests; only need to handle
requests from the OS to fill or
empty fixed size buffers.

##### Character Device Driver
![Desktop View](/assets/img/comp3438/char.png){: .normal }  
Character devices can handle I/O requests of arbitrary size (Support any type of device)  
Be used to handle data a byte at a time (e.g. keyboard); or work best with data in chunks smaller or larger
than the standard fixed size buffer used by device driver (e.g. ADC)

##### Major Differences
***block:***  only interacts with buffer cache  
***char:***  directly interacts with user requests from user
processes  
* I/O requests are directly passed (essentially unchanged) to the drivers
from the user processes  
* Character driver is responsible for transferring data directly to/from
the kernel memory space and the user memory space.





