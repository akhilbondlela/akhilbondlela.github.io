---
layout: page
title: Network Centric Programming
permalink: /Courses/Network_Centric/
---

## Introduction

- This course covers coding network programs in the C programming language.
- Topics that the course covers
  - Files, IO, Network Communication
  - Socket Network programming
  - Network Server Design
    - Multi-threaded programs and synchronization
  - Secure Programming
  - Profiling and Performance Analysis
  - Remote Procedure calls

## Input and Output in C

### File Operations

- Key functions
  - fopen - returns pointer to FILE structure(file pointer)
  - fclose
  - fread
  - fwrite
  - fgetc / fputc
  - fgets / fputs
  - fseek / ftell



### Low-level File I/O Functions

- Unbuffered I/O
- Every read or write call invokes a system call
- Open files are identified by file descriptor(nonnegative integer)
- Shells follow this convention
  - 0 = STDIN_FILENO
  - 1 = STDOUT_FILENO
  - 2 = STDERR_FILENO

- System Calls
  - open
  - read
  - write
  - lseek
  - close

#### fopen/open

- Both of these statements open a file and creates a file pointer to identify the location of the current file position in the file.
- The syntax from the fopen statement is
```C
FILE* fp = fopen(const char* pathname, const char* mode);
```
  - returns a file pointer (to buffer)
  - Mode
    - "r" - read only
    - "w" - write(create new file or overwrite)
    - "a" - write or append if file exists
    - "r+, w+, a+" - different read or write modes
- The syntax from the open statement is
```C
int fd = open(const char* pathname, int oflag, ...)
```
  - Oflags
    - Required: O_RDONLY, O_WRONLY, or O_RDWR
    - Optional: O_APPEND, O_CREAT, O_SYNC, O_EXCL

#### fclose/close

- ```fclose(FILE* stream)```
- ```Close(int filedescriptor)```
  - cleans up kernel data structures
  - Files automatically closed if program ends

### Error Handling

- Most system call library functions return -1(or sometimes 0) when an error occurs
  - Must be checked after every function call
- The global errno variable contains an error number that describes why the function failed
  - Errno is only valid if an error has occurred
- Use strerr() or perror() functions to convert error numbers to meaningful strings
- Sample code for error Handling

```c
#include	<errno.h>
#include	"ourhdr.h"

int main(int argc, char* argv[])
{
     // … some library call here

	fprintf(stderr, "EACCES: %s\n", strerror(EACCES));

	// another example – perror directly prints to stderr
	errno = ENOENT;
	perror(argv[0]);

	exit(0);
}
```

### File Pointers and Seeking

- The system remembers the position in a file through a file pointer(32-bit offset), which automatically advances when you read or write
- These functions can query or modify the position of the pointer:

```C
long ftell(FILE* fp);
int fseek(FILE* fp, long offset, int whence);
void rewind(FILE* fp);
```

- In the fseek function, to access key locations of the file, we can use special codes in the offset section of fseek
  - SEEK_SET - beginning of file
  - SEEK_CUR - current file position
  - SEEK_END - end of file

## Files and Directories

### Getting File Metadata

- ```Int stat(char* filename, struct stat* buf)```
  - Fills in the stat data structures with information about file type, size, permissions, time, etc.

- The stat data structure is

```C
struct	stat
{
  dev_t		st_dev;
  ino_t		st_ino;
  mode_t	st_mode;
  nlink_t	st_nlink;
  uid_t		st_uid;
  gid_t		st_gid;
  dev_t		st_rdev;
  off_t		st_size;
  // SysV/sco doesn't have the rest... But Solaris, eabi does.
#if defined(__svr4__) && !defined(__PPC__) && !defined(__sun__)
  time_t	st_atime;
  time_t	st_mtime;
  time_t	st_ctime;
#else
  time_t	st_atime;
  long		st_spare1;
  time_t	st_mtime;
  long		st_spare2;
  time_t	st_ctime;
  long		st_spare3;
  long		st_blksize;
  long		st_blocks;
  long	st_spare4[2];
#endif
};
```

- lstat
  - Similar to stat, but does not follow links

- fstat
  - Used for already opened files

### File Types(modes)

- Regular file
- Directory file
- Character special file
  - device access, e.g serial port
- Block special file
  - device access, e.g disk
- FIFO
  - pipes
- Socket
  - Network connection
- Symbolic link
  - pointer to another file

- Using MACROS to test file types
  - ```S_ISREG(m)``` - is it a regular file?
  - ```S_ISDIR(m)``` - is it a directory?
  - ```S_ISCHR(m)``` - is it a character device?
  - ```S_ISBLK(m)``` - is it a block device?
  - ```S_ISFIFO(m)``` - is it a fifo?
  - ```S_ISLNK(m)``` - is it a symbolic link?
  - ```S_ISSOCK(m)``` - is it a socket?

- The OS needs to be able to distinguish the type of a file to be able to know what commands/operations can be performed on the file
  - No seek on FIFO or sockets
  - No write on directory
  - Open on symbolic link requires redirection

### Directory Manipulation in UNIX

- mkdir - create a directory
- rmdir - remove a directory
- opendir - open directory for reading
- readdir - read directory entries
- rewinddir - move pointer back to the beginning of the directory
- closedir - close directory after reading is complete
- chdir - set the working directory for the current process
- getcwd - get the working directory for the current process

- A simple code to access a library through system calls. This an emulation of the ls command

```c
#include	<sys/types.h>
#include	<dirent.h>
#include	"ourhdr.h"

int main(int argc, char* argv[])
{
	DIR* dp;
	struct dirent* dirp;

	if (argc != 2)
		err_quit("a single argument (the directory name) is required");

	if ( (dp = opendir(argv[1])) == NULL)
		err_sys("can't open %s", argv[1]);

	while ( (dirp = readdir(dp)) != NULL)
		printf("%s\n", dirp->d_name);

	closedir(dp);
	exit(0);
}
```

### Symbolic links

- Can span over the user's file system, because the user can create links to directories on other parts of their file system
- They are represented by special files
- Loops are easier to remove
- Symlink(actualpath, sympath) - creates a symbolic link to sympath and the link will be located at actualpath
- Readlink(pathname, ...) - reads sympath which is located at pathname
  - These files cannot be read with the open command

### Common file Operations

- access - access control check
- chdir - change directory
- chown - change owner of a directory
- open - open a file
- opendir - open directory
- remove - delete file or directory
  - also unlink(file only), and rmdir(directory only)
- rename - rename directory

### Hard Links

- Every directory entry points to an i-node(which represents a file)
  - Multiple entries can point to the same file(hard links)
- ```Link(existingpath, newpath)``` - creates an additional directory entry to an existing file

### Problem: Loops

- Recursive listing of files does not work
  - This means that one link is pointing to another link, and the other link is pointing back to the original link
- There is no easy way to fix this
  - Unlink does not remove links in a directories
  - rmdir removes links to directories only when they are empty
- Only root users can create hard links to directories

## Socket Programming

### Client-Server Model

![Client-Server Model](/resources/images/net_cent/client_server_model.png)

- Network software consists of multiple programs that run on different machines and communicate over the network
- The client-server model shows how a client and a server interact over the network
  1. Client sends a request for information to the Server
  2. Server processes the request and retrieves requested data or modifies specified data
  3. Server sends response back to the Client
  4. Client processes the response

### Messages and Protocols

- Message: A sequence of bytes(data) which is transmitted over a network
  - The requests and responses are examples of messages
- Protocol: An agreement about ...
  - How messages are to be interpreted
  - What messages are valid
  - In which sequence that messages can be sent

### Layers of Protocols, Packets, and Encapsulation

![Network Encapsulation](/resources/images/net_cent/packet_headers.png)

- The unit of transmission over the network is a packet. A message from the client may be split up into multiple packets, if it does not fit into a single one. This is often hidden from the application.

- Each packet receives various headers to the front of the packet to show where the packet is supposed to be sent across the network. These are put on by the client machine, and removed by the server machine.
  - If these headers were not placed on the packet, then the packet may get lost over the network.
- In the figure above, there would be a separate header for each step of Host A to get the packet on the network. This means that there is a separate header from the client, protocol software, and the LAN 1 adapter. Each of these headers are removed by Host B to get access to the data.

### Network Structures and Elements: A Local Area Network

![LAN Structures](/resources/images/net_cent/LAN_structures.png)

- Host - a computer which is connected to the network
- Network medium - transmits bits as electrical or electromagnetic signals(Standards: IEEE 802.11, IEEE 802.3)
- Star topology - Includes a hub which forwards appropriate packets to the correct hosts based on the headers.
- Bus topology - All of the hosts are connected to the LAN bus and only take packets which belong to them

### A Campus Network

![Campus Network Structure](/resources/images/net_cent/campus_network.png)

- This network configuration has various LANs connected together through a bridge, each device on the network is addressed by its MAC address, which ensures that each computer get all the packets that it needs.
- The messages are not sent to everyone because of reachability of the host, security reasons, and overall network performance

### A Small Internet

![A small Internet](/resources/images/net_cent/small_internet.png)

- The router can find long paths over the network and can translate between different technologies(i.e. different computer makes and different network cards)
- The addressing scheme is through IP addresses to identify various computers over the network

### Naming and Addressing(DNS)

- DNS - Domain Name Service
  - This is a service which converts an ip address into a string which we can read and use to easily identify machines over the network
  - If we did not have this service, we would not urls to go to websites and we would have to remember the ip of each website to access it

### Port Numbers

- Essentially application identifiers, since one host may run many applications
- Any 16-bit value(0-65535), but values below 1000 are reserved by the operating system and other devices

### Sockets API

- General interface for network programming and inter-processes communication
- A socket is a communication endpoint, a tap into the network
- Network protocol is independent, but usually it's used with Internet protocols
  - This allows for a variety of addressing formats

- Types of socket connections
  - Datagram
    - Unordered message-oriented communication
    - Application multiplexing
    - Usually mapped to UDP
  - Stream
    - Application multiplexing
    - Reliable, flow controlled data stream
    - Usually mapped to TCP
  - Raw
    - Direct access to network layer
    - Usually mapped to IP

### Typical Implementation of a Client-Server System

![Server Client Structure](/resources/images/net_cent/server-client.png)

### Creating a Socket

- ```int socket(int family, int type, int protocol);```
  - Returns a socket file descriptor
  - Family: AF_INET, AF_INET6, ...
  - Type: SOCK_STREAM, SOCK_DGRAM, SOCK_SEQPACKET, SOCK_RAW
  - Protocol: 0(selected by default), IPPROTO_TCP, IPPROTO_UDP, IPPROTO_SCTP
  - Not all combinations of the things above are valid

### Useful Network Terminal Commands

- ```netstat -ni```
  - Lists network interfaces
- ```ifconfig <interface>```
  - Show the IP address and configuration of a specified interface

### Server Program

#### Bind

- ```int bind(int sockfd, sockaddr* a, socklen_t l);```
- Assigns a local address to a socket (e.g IP address or port number)
  - IP address must belong to a local interface
  - Calling bind is optional, otherwise ...
    - the system chooses a default port
    - for client, system choose IP of outgoing interface
    - for server, system chooses destination IP of incoming packet(SYN packet)
    - getsockname() returns the chosen address/port
  - Typically, only used for sockets that accept incoming connections
- Only root users can bind to privileged ports(ports < 1024)

#### Listen

- ```int listen(int sockfd, int backlog);````
- This statement configures a socket to accept incoming connections
  - Incomplete and established/completed queue necessary to implement TCP handshake
  - Backlog specifies the buffer size(queue length) for incoming connections
    - Traditional BSD max of 5 is not sufficient for high volume servers
    - May have a fudge factor of 1.5
    - From Linux 2.2 the backlog refers to the size of the established queue(>syn flood)
    - Don't use zero because of portability issues

#### Accept

- ```int accept(int sockfd, sockaddr* a, socklen_t* len);```
- Returns the next completed connection from the established queue
  - The return value is a different sockfd for the connected socket
  - The listen command returns a listening socket and the accept command returns the connected socket
- This statement is blocking
- ```sockaddr* a``` returns the address of the connected client
- ```socklen_t* len``` is a value-result argument(takes the length of the input structure, and returns the number of bytes stored in this structure)

#### Three-Way TCP Handshake

![TCP handshake](/resources/images/net_cent/tcphandshake.png)

- This is the handshake between the server and client which is done when the accept statement is called
- This can also be done while handling multiple clients as well.

#### Code for an Iterative Server

```c
listenfd = Socket(AF_INET, SOCK_STREAM, 0);

bzero(&servaddr, sizeof(servaddr));
servaddr.sin_family      = AF_INET;
servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
servaddr.sin_port        = htons(13);	/* daytime server, can only be run as root */

Bind(listenfd, (SA*) &servaddr, sizeof(servaddr));

Listen(listenfd, LISTENQ);

for ( ; ; ) {
  len = sizeof(cliaddr);
  connfd = Accept(listenfd, (SA*) &cliaddr, &len);
  printf("connection from %s, port %d\n",
       Inet_ntop(AF_INET, &cliaddr.sin_addr, buff, sizeof(buff)),
       ntohs(cliaddr.sin_port));

        ticks = time(NULL);
        snprintf(buff, sizeof(buff), "%.24s\r\n", ctime(&ticks));
        Write(connfd, buff, strlen(buff));

  Close(connfd);
}
```

### Client Programming

#### Connect

- ```int connect(sockfd, servaddr, addrlen);```
- Requires an open socket and destination address
- Possible Errors:
  - ETIMEDOUT – no response received after connection attempt
  - ECONNREFUSED – the server has refused the connection attempt (often wrong IP or port number)
  - EHOSTUNREACH – a router has notified the client that the destination could not be found

#### Address Conversion

- The system should convert from string/presentation representation("192.168.0.1") into a binary representation
- "Presentation" to "Numeric"
  - ```int inet_pton(family, strptr, addrptr);```
- "Numeric" to "Presentation"
  - ```char* inet_ntop(family, addrptr, strptr, len);```

#### Client Code

```c
Int main(int argc, char* argv[]) {
	int		sockfd, n;
	char	recvline[MAXLINE + 1];
	struct sockaddr_in	servaddr;

	if (argc != 2)
		err_quit("usage: a.out <IPaddress>");

	if ( (sockfd = socket(AF_INET, SOCK_STREAM, 0)) < 0)
		err_sys("socket error");

	bzero(&servaddr, sizeof(servaddr));
	servaddr.sin_family = AF_INET;
	servaddr.sin_port   = htons(13);	// daytime server
	if (inet_pton(AF_INET, argv[1], &servaddr.sin_addr) <= 0)
		err_quit("inet_pton error for %s", argv[1]);
	if (connect(sockfd, (SA*) &servaddr, sizeof(servaddr)) < 0)
		err_sys("connect error")

	while ( (n = read(sockfd, recvline, MAXLINE)) > 0) {
		recvline[n] = 0;	// null terminate
		if (fputs(recvline, stdout) == EOF)
			err_sys("fputs error");
	}
	if (n < 0)
		err_sys("read error");
}
```

### Byte Ordering

- Byte ordering may differ between the server and the client
- Network protocols specify a network byte order and each host is responsible for translation to the network byte order if needed
  - Internet protocols are big endian
  - x86 architectures are little endian

## HTTP Protocol

### HTTP

- Hypertext Transfer Protocol
  - It was designed in 1990 by Tim Berners at CERN
  - It applies a stateless request/response protocol

### URL

- URL - Uniform Resource Locator
- Protocol: Servername:port/path/filename?arguments
- It is a string that uniquely identifies web resources
- Examples
  - https://www.google.com/
  - https://www.facebook.com/

### Typical Steps that a web browser takes

- Steps to access https://www.google.com
  1. Check the protocol of the HTTP or HTTPS request
  2. Parse and resolve the hostname - This is found separated by the symbols ```://```, which can be resolved by using the gethostbyname(), which invokes DNS
  3. Parse the port number - allows the browser to connect to the specific port to obtain the desired information
  4. Open stream socket to host - this is opened on the specified port number, otherwise it is opened on the default port number(80 for regular HTTP requests)
  5. Send HTTP requests for path a filename for the different components of the requested page by the user

### Protocol Methods

- GET
  - Retrieves contents of the URL
- POST
  - Post information to the URL
  - This is usually used to send form inputs or files to a sever side script
- HEAD
  - Retrieve only the header information of the site
- PUT
  - Store data under the URL
- DELETE
  - Delete data under the URL

#### An example of a GET request from a browser

```
GET /log HTTP/1.0
Connection: Keep-Alive 
User-Agent: Mozilla/4.03 [en] (X11; I; HP-UX B.10.20 9000/777)
Host: w0:4711
Accept: image/gif, image/x-xbitmap, image/jpeg, image/pjpeg, */* 
Accept-Language: en 

Accept-Charset: iso-8859-1,*,utf-8 
```

### Return codes from web servers

- 200 - OK
- 400 - Malformed Request
- 404 - URL could not be found on that specific server
- 500 - Server Errors
- 501 - Client used an unknown method in the request

### Response Headers

- Date - timestamp
- Server - vendor name
- Content Length - Length of the reply
- Content Type - plain text, html, mp3, etc.
- Last Modified - timestamp of the last modification of the requested resources
- Cache-control - no-cached used to disable web-caching in between server and browser
- Expires - timestamp that caches use to discard cached pages

## Debugging

- What happens if we have a segmentation fault, this means that some portion of our code is trying to access a piece of memory that it does not have access to. Now how do we debug this and find the problem?

- One way to find the error is the use print statements to find which statement doesn't print to show the location of the error. However, this may get the job done, but is very difficult to use on code with a lot of lines and it is very time consuming to find the right place in the code.

- GDB is a tool which allows you to inspect the program state at the moment of failure.
  - Look at variable contents
  - Look at which function was executed
  - Look at register values to see which address caused the segfault

- GDB is used through these shell commands

- ```gcc -g mysource.c``` - compiles the source code with the gdb flag and gdb compatibility with the -g flag

- ```limit -c unlimited``` - enables core dumps when using the gdb debugger

- ```./a.out``` - execute the compiled code until segfault

- ```gdb a.out core``` - allows you to use gdb and look at the current state of the code when it had the segfault

### GDB commands

- Run arg1 arg2
- Break 17/continue
  - Break main
- Next/Step
- Info Registers
- Disass main
- Print a
  - Disp a
- BT
  - Backtrace function calls

- These commands make inspection of the program easier
  - The program requires no recompiling
  - No adding or removing print statements
  - Breakpoints allow partial execution of the program at a time
  - The gdb debugger also allows for step by step execution of the program to find where the error occurs

### Other useful tools

- Network
  - Netstat/Nmap - these tools show the open ports on a specific device on your network
  - Telnet - send/recieve TCP ascii data
    - This can also be used as a client if you are debugging a server program
  - Ping - used to test basic IP configuration and network connectivity
  - Tcpdump/ethereal
  - Valgrind
  - Memcheck
  - DDD
  - Boundschecker
  - lint

## Processes

- A process is a running instance of a program
  - Multiple instances of a program can be running at once at the same time on the same machine
- Operating Systems "multitask" to give the impression that all process run at the same time
  - The process scheduler runs in the background a regulates that all running processes get an equal amount of run time
- The ```ps -a``` command in the terminal outputs the list of active processes on the current machine

### Fork - creating processes via cloning

- Fork is a system call which clones and runs another instance of the current process
- pid_t fork(void)
  - Creates a task structure
  - Copies the page table
- Copy-on-Write for address space
- Returns twice(once for the parent and once for the child)
  -  The fork system calls essentially returns the PID of the child to the calling process. The parent gets the PID of the newly created child and the child process gets 0 as it has no child for itself.
  - This can be taken advantage of in code to allows the parent to execute a certain set of instructions as the child executes another.

### How is the first process created?

- Every process has an identifier, its PID.
- "Init" is the first process to run after booting
  - It receives the PID of 1
  - The kernel scheduler receives a PID of 0
- Every process except Init has a parent
  - The parent process is the one that called fork to create a child
- Init adopts orphan processes that lose their parent

### What does a child process inherit?

- The child process inherits basically everything from the parent process
  - User,group IDs
  - Content of all program variables
  - File descriptor table (parent and child share same file table entries)
  - Controlling terminal
  - Working and root directory
  - Umask
  - Environment
  - Shared memory segments
  - Resource Limits
- However the child does not inherit
  - Process ID
  - Parent Process ID
  - Execution time counters
  - File locks
  - Pending alarms and signals

### Process Termination

- Return, exit()
  - Performs cleanup operations(fclose on stdio streams)
  - Exit status undefined if not explicitly given
  - Calls exit handlers are registered, and are called in reverse order of registration
- \_exit()
  - Returns to kernel immediately
- Abort
- Signal Termination

## Threads

- Threads are "lightweight" processes
  - The represent an execution path through a program
- One process can contain multiple threads, but each thread only belongs to one process
  - If the process terminates, all threads terminate
  - Process resources(file descriptors are shared among threads)
- Creation and context switching other threads is faster than processes
- Threads share the same address spacce, but each thread has a different stack

### Benefits of a threaded program over a sequential program

- Improve application responsiveness
  - Webserver answering requests
  - GUI responding to user input
- Can improve code readability
- More efficient use of resources
  - This is beneficial in multiprocessor machines
  - Known as "Hyperthreading"

### User level & Kernel Level Threads

- Threads can be implemented as a user-level library or in the kernel(accessible through system calls)
- User-level advantages
  - No mode switching for thread context switches, creation, etc.
  - Independent of the OS
- Kernel-level advantages
  - A blocking system call only stops one thread and not the whole process
  - Can take advantage of multiple processors

### Creation and Termination

- ```int pthread_create(tid, attributes, function pointer, arguments)```
  - Tid - Thread ID(also accessible within the thread by pthread_self)
  - Attributes - priority, stack size, etc.
  - Function pointer - a pointer to the function which the thread will execute
  - Arguments - arguments are passed as a void pointer
- Termination
  - When the function returns
  - Explicit call to pthread_exit
  - When the creating process terminates

### Waiting for thread termination

- Joinable thread: exit status for thread is retained until another thread calls join
  - pthread_join(tid, void \**status)
  - Can only wait for specific thread for which id is known
- Detached thread: exit status not retained
  - pthread_detach(tid)
  - Detaches an existing thread

## Race conditions

- Unpredictable interleaved execution can lead to race conditions when accessing shared resources
  - Between signal handlers and main execution path
  - Between processes(e.g. file access)
  - Between threads(global, static variables)
- Solutions
  - Use atomic(uninterruptable) commands(e.g sigsuspend)
  - Use locking to make a series of commands uninterruptible(e.g. sigprocmask)
  - Get rid of shared resources
- Thread libraries provide such mechanisms for threads

## Memory Layout

<img src="/resources/images/net_cent/memorylayout.png" alt="Memory Layout" height="500" width="400">

## Shared Variables

- Threads
  - Global and static variables
  - Dynamic memory if pointer is shared(but not if local pointers are used and malloc is called separately in each thread)
- Processes
  - None(unless specifically allocated in shared memory)

## Locking

### MUTEX

- Acts as a lock for certain lines of code in a tread which makes sure that a thread accesses a shared variable when another thread is not using it, to avoid corruption of data.

- To create a lock, we would execute the following code

```C
//To create a MUTEX lock
pthread_mutex_t childsum_mutex = PTHREAD_MUTEX_INITIALIZER;
```

- To start the lock, place the following piece of code at the beginning of the portion of code which you would like to lock

```C
//Mark the beginning of the portion of code you want to lock
pthread_mutex_lock(&childsum_mutex);
```

### File Locks - "Record Locks"

- Synchronize access from multiple processes from shared memory
- Allows you to specify what portion of a file to lock to improve performance

### Deadlock

- Can occur is threads acquire several locks for one operation

- Atomic instructions - thread safe instructions which already in your operating system
- Thread safe functions - functions which can be used without using a MUTEX lock
  - The only way to identify if a function is thread safe is to read the documentation for the function
  - A list of some functions that are NOT thread safe are listed on the man page for pthreads

## Threads Intercommunication

- How can we allow one thread to notify another thread of a specific activity that has occurred?
  - Condition Variables
- Condition variables are mechanisms which allow you to wait for a condition to become true and then proceed in a thread safe manner
  - Waits and releases lock for the duration of the waiting period
  - ```pthread_cond_wait(pthread_cond_t, pthread_mutex_t);``` - The calling thread unlocks the mutex and then goes to sleep or waits
  - ```pthread_cond_signal(pthread_cond_t);``` - The calling thread sends a conditional signal to another thread


## Signals

### Signal Names

- Signal: Soft Interrupt
- Kill commands sends signals to processes
- Interrupt numbers mapped to names
  - SIGSEGV
  - SIGKILL
  - SIGUSR1
  - SIGUSR2
  - SIGINT
  - SIGTERM
  - SIGALARM
  - SIGCHILD
- More are listed on page 7 of the man page for signal

### Catching a signal

- Signals handlers can be written to take in a signal and interpreting the signal
- SIGKILL, SIGSTOP cannot be ignored
- No library functions, and no static storage should be used

### Creating signals



<hr>

# Midterm Topics

## I/O

- Standard I/O
- Low-level I/O functions
- Indirection and redirection through file descriptors
  - 3 Default file descriptors
    - STDOUT_FILENO = \#1
    - STDIN_FILENO = \#0
    - STDERR_FILENO = \#2
- Handling Errors - ERRNO - perror
- Buffering
- Binary vs. ASCII representation
  - ASCII representation is 1 byte for each character in the string
  - Binary form - ```char a = 17;```
  - ASCII form - ```char* a = "17";```
  - Network clients expect things in ASCII encoding

## Network Sockets

- Focused on TCP("STREAM sockets")
- Initializing
  - Server-side
    - socket()
    - bind() -  Specifies port number
    - listen()
    - accept()
    - read()
    - write()
    - close()
  - Client-side - Connections are started by the client
    - socket()
    - connect()
    - write()
    - read()
    - close()
- DNS names vs. IP addresses
  - gethostbyname
  - inet_pton
  - inet_ntop
- Port numbering
  - Port numbers that we don't want to use is anything less than 1000
    - Numbers less than 1000 are already reserved by the machine

## Concurrent Servers and Programs

- Processes - more overhead than threads & harder to share resources
  - Fork
  - Waiting and termination
  - Shared resources
    - Files
    - [Variables copied at the time of fork]
  - IPC
    - Pipes
    - Sockets
  - File locks - to prevent race conditions
- Threads
  - Shared resources
    - Memory on the heap - malloc
    - Static variables
    - Global variables
    - Anything shared for processes
  - Mutex - to prevent race conditions
    - Should contain the minimal critical region
      - Minimal critical region - smallest region of code that should not be interpreted during execution
  - Deadlock
    - A lock is never released and then thread tries to get another lock which is held by another thread, so then the program then just halts because threads are not releasing locks and cannot get the desired locks
  - Condition Variable
- Basic Signals

<hr>

## I/O multiplexing

### Connected to UDP Socket

- Use same "connect" call as TCP
- Differences
  - All detected errors are returned to the application
  - Cannot specify destination on per packet basis(use write or send instead of sendto)
  - Only received packets from the connected peer are returned
