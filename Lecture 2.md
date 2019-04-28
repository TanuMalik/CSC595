---


---

<h1 id="lecture-2">Lecture 2</h1>
<p><mark>These notes are sketchy and must be updated</mark></p>
<h2 id="overview">Overview</h2>
<p>Lets do a 2 minute review of what we covered last time:</p>
<ul>
<li>A process is a currently executing instance of a program.</li>
<li>All programs by default execute in the user mode.</li>
<li>A system call can be defined as a request to the operating system to do something on behalf of the program–an entry point into the kernel.</li>
<li>
<ul>
<li>The kernel, the core of the operating system program in fact has control over everything.</li>
</ul>
</li>
<li>During the execution of a system call, the mode is change from user mode to kernel mode (or system mode) to allow the execution of the system call.</li>
<li>All OS software is trusted and executed without any further verification.</li>
</ul>
<blockquote>
<p>Important concept: While in kernel mode no further interrupts are accepted so the call is finished atomically–without further interrupts. The kernel provides that guarantee.</p>
</blockquote>
<p>Today and for the next lecture we will take a deep dive into the system calls</p>
<h2 id="system-calls-versus-library-routines">System Calls versus Library Routines</h2>
<p>The tricky thing about system calls is that they look very much like a library routine (or a regular function) that you have already been using (for instance,  printf). The only way to tell which is a library routine and which is a system call is to remember which is which.</p>
<p>Another way to obtain information about the system call is to refer to Section 2 of the man pages. For instance, to find more about the “read” system call you could type:</p>
<p>% man -S 2 read</p>
<p>By contrast, on our current version of Linux, if you try:</p>
<p>% man -a read</p>
<p>Section 1 of the library will be displayed (indicated by the 1 in parenthesis). To go the the next section, you can use  q  followed by an enter key.</p>
<p>Section 3 contains library routines. By issuing a:</p>
<p>% man -S 3 fread</p>
<p>you will learn more about the “fread” library routine.</p>
<p>Some library functions have embedded system calls. For instance, the library routines  scanf and  printf  make use of the system calls  read  and  write. The relationship of library functions and system calls is shown in the below diagram (taken from John Shapley Gray’s  <em>Interprocess Communications in UNIX</em>)</p>
<p><img src="http://www.cs.uregina.ca/Links/class-info/330-bkup/SystemCall_IO/System_Calls.gif" alt=""></p>
<p>“The arrows in the diagram indicate possible paths of communication. As shown, executable programs may make use of system calls directly to request the kernel to perform a specific function. Or, the executable program may invoke a library function which in turn may perform system calls.” (page 4 and 5, <em>Interprocess Communications in UNIX</em>).</p>
<h2 id="file-io-system-calls">File I/O System Calls</h2>
<p>We will start with file system related system calls.<br>
These calls are:</p>
<p>creat( ), open( ), close( ) – managing I/O channels read( ), write( ) – handling input and output operations lseek( ) – for random access of files<br>
link( ), unlink( ) – aliasing and removing files<br>
stat( ) – getting file status<br>
access( ), chmod( ), chown( ) – for access control</p>
<p>All these calls must identify a location of the file. How do we identify the location of a file? As a path.<br>
How does the system identify the location? Not as a path.</p>
<p>The system assigns a small nonnegative integer to an open file. This integer is called the file descriptor.</p>
<p>So an open call is important because it assigns an FD.</p>
<h2 id="open">Open</h2>
<p>int open(char* filename, int flags, int mode);</p>
<p>The sys call ‘open’ opens the filename for reading or writing as specified by the access mode and returns an integer descriptor for that file. The integer descriptor is used subsequently to refer to the file.</p>
<p>filename: A string that represents the absolute, relative or filename of the file<br>
flags: An integer code describing the access (see below for details)<br>
mode: The file protection mode usually given by 9 bits indicating rwx permission</p>
<p>The flag codes are given by<br>
O_RDONLY – opens file for read only<br>
O_WRONLY – opens file for write only<br>
O_RDWR – opens file for reading and writing</p>
<p>These are also the access modes. So these flags must be included. These request opening the file read-<br>
only, write-only, or read/write, respectively.</p>
<p>In addition, zero or more file creation flags and file status flags  can be bitwise-<em>or</em>’d in <em>flags</em>. The distinction between these two groups of flags is that the file creation flags affect the semantics of the open operation itself, while the file status flags affect the semantics of subsequent I/O operations.</p>
<p>Some of the <em>file creation flags</em> are<br>
<strong>O_CREAT</strong>,  <strong>O_TMPFILE</strong>, and<br>
<strong>O_TRUNC</strong>.</p>
<p>The <em>file status flags</em> are all of the remaining flags listed below.</p>
<p><strong>O_NDELAY</strong> – prevents possible blocking <strong>O_APPEND</strong> — opens the file for appending<br>
<strong>O_EXCL</strong> – Ensure that this call creates the file: produces an error if the <strong>O_CREAT</strong> bit is on and file exists</p>
<p>Flags can be system specific. For example, to open a file for read and truncates the size to zero we could use,</p>
<p>open(“filename”, O_RDONLY | O_TRUNC, 0);</p>
<p>When open call fails:</p>
<p>If the open call fails, a -1 is returned; otherwise a descriptor is returned.</p>
<p>Situations when failure happens:</p>
<ol>
<li>The file permissions don’t allow the calling process to open the file in the mode specified by flags.</li>
<li>The specified file is on a read-only file system and the caller tried to open it for writing.</li>
<li>The specified file is an executable file (a program) that is currently executing.</li>
<li>The specified file is a directory, and the caller attempted to open it for writing.</li>
</ol>
<h2 id="read">Read</h2>
<p>The read() system call reads data from the open file referred to by the descriptor fd.</p>
<p>ssize_t read(int fd, void *buffer, size_t count);</p>
<ul>
<li>The count argument specifies the maximum number of bytes to read. (The size_t data type is an unsigned integer type.)</li>
<li>The buffer argument supplies the address of the memory buffer into which the input data is to be placed. This buffer must be at least count bytes long.</li>
</ul>
<p>A successful call to read() returns the number of bytes actually read, or 0 if end-of- file is encountered. On error, the usual –1 is returned. The ssize_t data type is a signed integer type used to hold a byte count or a –1 error indication.</p>
<p>When may read return less than the requested number of bytes? That is the value of ssize_t is less than count?</p>
<p>Read reads any sequence of bytes–it doesn’t care if those bytes are text, binary integers, or C data structures in bibary form.</p>
<p>There is no way for read() to tell the difference, and so it can’t attend to the C convention of null terminating character strings.</p>
<h2 id="write">Write</h2>
<p>The write() system call writes data to an open file.</p>
<p>ssize_t write(int fd, void *buffer, size_t count);</p>
<p>Arguments are same as read. Except that now the  buffer is the address of the data to be written; count is the number of bytes to write from buffer;  and fd is a file descriptor referring to the file to which data is to be written.</p>
<p>On success, write() returns the number of bytes actually written; this may be less than count.</p>
<p>Write does not imply data copied to disk.<br>
When performing I/O on a disk file, a successful return from write() doesn’t guarantee that the data has been transferred to disk, because the kernel performs buffering of disk I/O in order to reduce disk activity and expedite write() calls.</p>
<h2 id="close">Close</h2>
<p>The close() system call closes an open file descriptor, freeing it for subsequent reuse by the process. When a process terminates, all of its open file descriptors are automatically closed.</p>
<p>int close(int fd);</p>
<p>What if you want to copy a file from File A to File B.<br>
How would you write that program using these system calls?</p>
<h2 id="lseek">LSeek</h2>
<ul>
<li>For each open file, the kernel records a file offset, sometimes also called the read- write offset or pointer.</li>
<li>This is the location in the file at which the next read() or write() will commence.</li>
<li>The file offset is expressed as an ordinal byte position relative to the start of the file. The first byte of the file is at offset 0.</li>
</ul>
<p>off_t lseek(int fd, off_t offset, int whence);<br>
Returns new file offset if successful, or –1 on error</p>
<p>The offset argument specifies a value in bytes.</p>
<p>SEEK_SET: The file offset is set offset bytes from the beginning of the file.</p>
<p>SEEK_CUR: The file offset is adjusted by offset bytes relative to the current file offset.</p>
<p>SEEK_END: The file offset is set to the size of the file plus offset. In other words, offset is interpreted with respect to the next byte after the last byte of the file.</p>
<p>The return value from a successful lseek() is the new file offset. The following call retrieves the current location of the file offset without changing it:</p>
<p>lseek(fd, 0, SEEK_SET);<br>
lseek(fd, 0, SEEK_END);<br>
lseek(fd, -1, SEEK_END);<br>
lseek(fd, -10, SEEK_CUR);<br>
lseek(fd, 10000, SEEK_END);</p>
<p>/* Start of file <em>/<br>
/</em> Next byte after the end of the file <em>/ /</em> Last byte of file <em>/<br>
/</em> Ten bytes prior to current location <em>/ /</em> 10001 bytes past last byte of file */</p>
<h2 id="relationship-between-fds-and-open-files">Relationship between FDs and Open Files</h2>
<ul>
<li>Up until now, it may have appeared that there is a one-to-one correspondence between a file descriptor and an open file.</li>
<li>However, this is not the case. It is possible— and useful—to have multiple descriptors referring to the same open file.</li>
<li>These file descriptors may be open in the same process or in different processes</li>
</ul>
<h2 id="processes">Processes</h2>
<blockquote>
<p>Example: The copy program</p>
</blockquote>
<p>What is a file descriptor?<br>
A file descriptor is a usually small nonnegative integer assigned to an open file. File descriptors are used to refer to all types of open files, including pipes, FIFOs, sockets, terminals, devices, and regular files. Each process has its own set of file descriptors.</p>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>

