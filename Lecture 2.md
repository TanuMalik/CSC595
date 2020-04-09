---


---

<h1 id="lecture-2">Lecture 2</h1>
<p><mark>These notes are sketchy and must be updated</mark></p>
<h2 id="overview">Overview</h2>
<p>Lets do a 2 minute review of what we covered last time:</p>
<ul>
<li>
<p>A process is a currently executing instance of a program.</p>
</li>
<li>
<p>A system call is a request to the operating system to do something on behalf of the program–an entry point into the kernel.</p>
</li>
<li>
<p>During the execution of a system call, the mode is change from user mode to kernel mode (or system mode) to allow the execution of the system call.</p>
</li>
<li>
<p>All programs by default execute in the user mode.</p>
<ul>
<li>Restricted capabilities<br>
– Cannot execute privileged instructions<br>
– Cannot directly reference code or data in OS address space</li>
</ul>
</li>
<li>
<p>The kernel, the core of the operating system program in fact has control over all resources.<br>
–Can execute any instructions in the instruction set–Including halting the processor, changing mode bit, initiating I/O<br>
–Can access any memory location in the system–Including code and data in the OS address space</p>
</li>
</ul>
<h2 id="implementing-a-system-call">Implementing a System Call</h2>
<p>05-syscalls_sol.ppt</p>
<ul>
<li>
<p>A system call is implemented by a <code>software interrupt'' that transfers control to kernel code; in Linux/i386 this is</code>interrupt 0x80’’. The specific system call being invoked is stored in the EAX register, and its arguments are held in the other processor registers.</p>
</li>
<li>
<p>All OS software is trusted and executed without any further verification.</p>
</li>
</ul>
<blockquote>
<p>Important concept: While in kernel mode no further interrupts are accepted so the call is finished atomically–without further interrupts. The kernel provides that guarantee.</p>
</blockquote>
<p>Today and for the next lecture we will take a deep dive into the system calls</p>
<h2 id="system-calls-versus-library-routines">System Calls versus Library Routines</h2>
<p>A SYSTEM CALL INTERFACE MAY CHANGE and therefore it is advisable to place system calls in subroutines so subroutines can be adjusted in the event of a system call interface change.</p>
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
<h2 id="system-calls-and-errors">System Calls and Errors</h2>
<ul>
<li>
<p>errno values can be found in &lt;errno.h&gt;. Many library functions follow the same procedure for reporting errors.</p>
</li>
<li>
<p>Note that errno is undefined after a successful library call: this call may well change this variable, even though it succeeds, for example because it internally used some other library function that failed. Thus, if a failing call is not immediately followed by a call to perror(), the value of errno should be saved.</p>
</li>
<li>
<p>EINTR 4</p>
</li>
</ul>
<p>Many system calls will report the  <code>EINTR</code>  error code if a signal occurred while the system call was in progress. No error actually occurred, it’s just reported that way because the system isn’t able to resume the system call automatically. This coding pattern simply retries the system call when this happens, to ignore the interrupt.</p>
<p>For instance, this might happen if the program makes use of  <code>alarm()</code>  to run some code asynchronously when a timer runs out. If the timeout occurs while the program is calling  <code>write()</code>, we just want to retry the system call (aka read/write, etc).</p>
<h2 id="unix-philosophy">Unix Philosophy</h2>
<p>Universality of I/O is achieved by ensuring that each file system and device driver implements the same set of I/O system calls. Because details specific to the file sys- tem or device are handled within the kernel, we can generally ignore device-specific factors when writing application programs.<br>
In an interactive shell, these three file descriptors normally refer to the terminal under which the shell is running. If I/O redirections are specified on a command line, then the shell ensures that the file descriptors are suitably modified before starting the program.</p>
<h3 id="file-descriptors">File descriptors</h3>
<p>What is a file descriptor?<br>
A file descriptor is a usually small nonnegative integer assigned to an open file. File descriptors are used to refer to all types of open files, including pipes, FIFOs, sockets, terminals, devices, and regular files. Each process has its own set of file descriptors.</p>
<h2 id="file-io-system-calls">File I/O System Calls</h2>
<p>We will start with file system related system calls.<br>
These calls are:</p>
<p>open( ), close( ) – managing I/O channels read( ), write( ) – handling input and output operations lseek( ) – for random access of files<br>
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
<p>As previously pointed out, sockets and pipes also get passed to close. But probably the original reason is that open needs flags like O_RDONLY that are defined in fcntl.h, so you might as well put the prototype in that file.<br>
What if you want to copy a file from File A to File B.<br>
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
<p>For each process, the kernel maintains <strong>a table of open file descriptors</strong>. Each entry in this table records information about a single file descriptor, including:<br>
(i) a set of flags controlling the operation of the file descriptor<br>
(ii) a reference to the open file description.</p>
<p>The kernel maintains a system-wide table of all open file descriptions.</p>
<blockquote>
<p>This table is sometimes referred to as the open file table, and its entries are sometimes called open file handles.</p>
</blockquote>
<p>An open file description stores all information relating to an open file, including:<br>
(1) the current file offset (as updated by read() and write(), or explicitly modified using lseek());<br>
(2) status flags specified when opening the file (i.e., the flags argument to open());<br>
(3) the file access mode (read-only, write-only, or read-write, as specified in open());<br>
(4) a reference to the i-node object for this file.</p>
<p>Each file system has a table of i-nodes for all files residing in the file system.  For now, we note that the i-node for each file includes the following information:<br>
(1)  file type (e.g., regular file, socket, or FIFO) and permissions;<br>
(2)  a pointer to a list of locks held on this file; and<br>
(3) various properties of the file, including its size and timestamps relating to different types of file operations.</p>
<p>Here, we are overlooking the distinction between on-disk and in-memory representations of an i-node. The on-disk i-node records the persistent attributes of a file, such as its type, permissions, and timestamps. When a file is accessed, an in-memory copy of the i-node is created, and this version of the i-node records a count of the open file descriptions referring to the i-node and the major and minor IDs of the device from which the i-node was copied. The in-memory i-node also records various ephemeral attributes that are associated with a file while it is open, such as file locks.</p>
<h2 id="dup">Dup</h2>
<h3 id="dup-1">dup()</h3>
<p>So, look at the following program:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token function">main</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token keyword">int</span> fd1<span class="token punctuation">,</span> fd2<span class="token punctuation">;</span>

  fd1 <span class="token operator">=</span> <span class="token function">open</span><span class="token punctuation">(</span><span class="token string">"file1"</span><span class="token punctuation">,</span> O_WRONLY <span class="token operator">|</span> O_CREAT <span class="token operator">|</span> O_TRUNC<span class="token punctuation">,</span> <span class="token number">0644</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  fd2 <span class="token operator">=</span> <span class="token function">open</span><span class="token punctuation">(</span><span class="token string">"file1"</span><span class="token punctuation">,</span> O_WRONLY<span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token punctuation">}</span>
</code></pre>
<p>Now, what has happened? The OS has created two file table entries, one for each <strong>open()</strong> call and two file descriptors, but only one vnode. This is because there is only one file. Both file table entries point to the same vnode, but they each have different seek pointers.</p>
<p>Now if we write into these files:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token function">main</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token keyword">int</span> fd1<span class="token punctuation">,</span> fd2<span class="token punctuation">;</span>

  fd1 <span class="token operator">=</span> <span class="token function">open</span><span class="token punctuation">(</span><span class="token string">"file1"</span><span class="token punctuation">,</span> O_WRONLY <span class="token operator">|</span> O_CREAT <span class="token operator">|</span> O_TRUNC<span class="token punctuation">,</span> <span class="token number">0644</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  fd2 <span class="token operator">=</span> <span class="token function">open</span><span class="token punctuation">(</span><span class="token string">"file1"</span><span class="token punctuation">,</span> O_WRONLY<span class="token punctuation">)</span><span class="token punctuation">;</span>

  <span class="token function">write</span><span class="token punctuation">(</span>fd1<span class="token punctuation">,</span> <span class="token string">"The Brown Dog\n"</span><span class="token punctuation">,</span> <span class="token function">strlen</span><span class="token punctuation">(</span><span class="token string">"The Brown Dog\n"</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token function">write</span><span class="token punctuation">(</span>fd2<span class="token punctuation">,</span> <span class="token string">"Jumped over the moon\n"</span><span class="token punctuation">,</span> <span class="token function">strlen</span><span class="token punctuation">(</span><span class="token string">"Jumped over the moon\n"</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

  <span class="token function">close</span><span class="token punctuation">(</span>fd1<span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token function">close</span><span class="token punctuation">(</span>fd2<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<p>Then what will happen? Well, the first <strong>write()</strong> call will write the string <strong>“The Brown Dog\n”</strong> into <strong>file1</strong>. Then the second <strong>write()</strong> call will overwrite it with <strong>“Jumped over the moon\n”</strong>. This is because each fd points to its own file table entry, which has its own lseek pointer, and thus the first <strong>write()</strong> does not update the lseek pointer of the <strong>fd2</strong>.</p>
<p>The system call <strong>dup(int fd)</strong> duplicates a file descriptor <strong>fd</strong>. It returns a second file descriptor that points to the same file table entry as <strong>fd</strong> does. So now you can treat the two file descriptors as identical.</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;fcntl.h&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;stdio.h&gt;</span></span>
<span class="token function">main</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token keyword">int</span> fd1<span class="token punctuation">,</span> fd2<span class="token punctuation">;</span>

  fd1 <span class="token operator">=</span> <span class="token function">open</span><span class="token punctuation">(</span><span class="token string">"file2"</span><span class="token punctuation">,</span> O_WRONLY <span class="token operator">|</span> O_CREAT <span class="token operator">|</span> O_TRUNC<span class="token punctuation">,</span> <span class="token number">0644</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  fd2 <span class="token operator">=</span> <span class="token function">dup</span><span class="token punctuation">(</span>fd1<span class="token punctuation">)</span><span class="token punctuation">;</span>
  
  <span class="token function">write</span><span class="token punctuation">(</span>fd1<span class="token punctuation">,</span> <span class="token string">"The Brown Dog\n"</span><span class="token punctuation">,</span> <span class="token function">strlen</span><span class="token punctuation">(</span><span class="token string">"The Brown Dog\n"</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token function">write</span><span class="token punctuation">(</span>fd2<span class="token punctuation">,</span> <span class="token string">"Jumped over the moon\n"</span><span class="token punctuation">,</span> <span class="token function">strlen</span><span class="token punctuation">(</span><span class="token string">"Jumped over the moon\n"</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

  <span class="token function">close</span><span class="token punctuation">(</span>fd1<span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token function">close</span><span class="token punctuation">(</span>fd2<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<p>Instead of calling <strong>open()</strong> to initialize <strong>fd2</strong>, it calls <strong>dup(fd1)</strong>. Thus, after the first write, the lseek pointer of <strong>fd2</strong> has been updated to reflect the write to <strong>fd1</strong> – this is because the two file descriptors point to the same file table entry.</p>
<h2 id="processes">Processes</h2>
<p>When a process starts, it gets its own address space.<br>
Each process gets the following.</p>
<ul>
<li>
<p>The <strong>stack</strong> is the place where automatic variable and function call return addresses are stored.</p>
<ul>
<li>Every time a new variable is declared, the program moves the stack pointer down to reserve space for the variable.</li>
<li>This segment of the stack is Writable but not executable.</li>
<li>If the stack grows too far – meaning that it either grows beyond a preset boundary or intersects the heap – you will get a stackoverflow most likely resulting in a SEGFAULT.</li>
</ul>
<blockquote>
<p>Note: Stack overflow is [a] cause, segmentation fault is the result.<br>
When the stack is exhausted, the memory outside of the reserved area will be accessed. But the app did not ask the kernel for this memory, thus a SegFault will be generated for memory protection.</p>
</blockquote>
</li>
<li>
<p>The <strong>heap</strong> is a contiguous, expanding region of memory. If you want to allocate a large object, it goes here.</p>
<ul>
<li>The heap starts at the top of the text segment and grows upward, meaning sometimes when you call <code>[malloc](https://linux.die.net/man/3/malloc)</code> that it asks the operating system to push the heap boundary upward.</li>
<li>This area is also writable but not executable.</li>
</ul>
</li>
<li>
<p><strong>A Data Segment</strong> or as we will tried to refer to it as the Initialized Segement.</p>
<ul>
<li>This contains all of your globals and any other static duration variables.</li>
<li>This section starts at the end of the text segment and is static in size because the amount of globals is known at compile time.</li>
<li>But not those that are default initialized, that is the next segment.</li>
</ul>
</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">int</span>  global  <span class="token operator">=</span>  <span class="token number">1</span><span class="token punctuation">;</span>
</code></pre>
<ul>
<li><strong>A BSS Segment</strong> or the Basic Service Segment. This contains all of your globals and any other static duration variables that are implicitly zeroed out like.</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">int</span>  assumed_to_be_zero<span class="token punctuation">;</span>
</code></pre>
<ul>
<li><strong>A Text Segment</strong>. This is where all your code is stored – all the 1’s and 0’s. The program counter moves through this segment executing instructions and moving down the next instruction. It is important to note that this is the only executable section of the code created by default.</li>
</ul>
<p><mark>Review programs a.c-a4.c on D2L.</mark></p>
<h2 id="know-your-process">Know your process</h2>
<p>To keep track of all these processes, your operating system gives each process a number and that process is called the PID, process ID. Processes also have a  <code>ppid</code>  which is short for parent process id.</p>
<ul>
<li>The getpid() system call returns the process ID of the calling process. The getppid() system call lets a process find out the process ID of its parent.</li>
<li>The Linux kernel limits process IDs to being less than or equal to 32,767 . When a new process is created, it is assigned the next sequentially available process ID.</li>
<li>Each time the limit of 32,767 is reached, the kernel resets its process ID counter so that process IDs are assigned starting from low integer values.</li>
<li>On 32-bit platforms, the maximum value for this file is 32,768, but on 64-bit platforms, it can be adjusted to any value up to 222 (approximately 4 million), making it possible to accommodate very large numbers of processes.<br>
Can a parent know the PID of its child?</li>
</ul>
<blockquote>
<p>Only through a fork()</p>
</blockquote>
<h2 id="process-table-and-pcb">Process Table and PCB</h2>
<p>Relationship between Process Table and Process Control Block. See diagram in slides.</p>
<h2 id="start-process">Start process</h2>
<p>Every process has exactly one parent, that parent could be  <code>init.d</code>.</p>
<ul>
<li>The kernel creates the first process <code>init.d</code>. Init.d boots up things like your GUI, terminals etc. What is important is the operating system only really creates one process by default, all other processes are <a href="https://linux.die.net/man/3/fork">fork</a> and <a href="https://linux.die.net/man/3/exec">exec</a>’ed from that single process.</li>
<li>With the exception of a few system processes such as init (process ID 1), there is no fixed relationship between a program and the process ID of the process that is created to run that program.</li>
</ul>
<h2 id="know-all-processes">Know all processes</h2>
<ul>
<li>If I want to visualize all processes, which data structure will the represent: a tree or a graph?</li>
<li>If a child process becomes orphaned because its “birth” parent terminates, then the child is adopted by the init process, and subsequent calls to getppid() in the child return 1</li>
</ul>
<p><mark>Review program b.c on D2L</mark><br>
To see process tree:</p>
<blockquote>
<p>pstree<br>
ps -aef --forest</p>
</blockquote>
<h2 id="querying-process-information-proc-file-system">Querying process information: /proc file system</h2>
<ul>
<li>/proc virtual file system. This file system resides under the /proc directory and contains various files that expose kernel information, allowing processes to conveniently read that information, and change it in some cases, using normal file I/O system calls.</li>
<li>The /proc file system is said to be virtual because the files and subdirectories that it contains don’t reside on a disk. Instead, the kernel creates them “on the fly” as processes access them.</li>
</ul>
<ul>
<li>Go to /proc/PID do<br>
<code>cat /proc/1/status</code></li>
</ul>
<blockquote>
<p>The parent of any process can be found by looking at the PPID field provided in the Linux-specific /proc/PID/status file.</p>
</blockquote>
<ul>
<li>See diagram of /proc organization on Slides</li>
<li>Accessing /proc
<ul>
<li>Files under /proc are often accessed using shell scripts</li>
<li>For example, we can modify and view the contents of a /proc file using shell commands as follows (you would need root permissions):</li>
</ul>
</li>
</ul>
<pre><code>$ echo 100000 &gt; /proc/sys/kernel/pid_max
$ cat /proc/sys/kernel/pid_max
</code></pre>
<p><mark>Review program n.c on D2L</mark></p>
<ul>
<li>Information in /proc is ephemereal</li>
</ul>
<blockquote>
<p>The /proc/PID directories are volatile. Each of these directories comes into existence when a process with the corresponding process ID is created and disappears when that process terminates. This means that if we determine that a particular /proc/PID directory exists, then we need to cleanly handle the possibility that the process might have been terminated, and the corresponding /proc/PID directory might have been deleted, by the time we try to read a file in that directory.</p>
</blockquote>
<h3 id="information-about-the-kernel-itself">Information about the Kernel itself</h3>
<p>uname -a<br>
/proc/sys/kernel<br>
<mark>Review program d.c on D2L</mark></p>
<h2 id="fork">Fork</h2>
<p>The <a href="https://linux.die.net/man/3/fork">fork</a> system call clones the current process to create a new process. It creates a new process called the child process by duplicating the state of the existing process.</p>
<ul>
<li>The two processes are executing the same program text, but they have separate copies of the stack, data, and heap segments.</li>
<li>Each process can modify the variables in its stack, data, and heap segments without affecting the other process.</li>
<li>The child process does not start from main. Instead it executes the next line after the <code>fork()</code> just as the parent process does.</li>
</ul>
<p><mark>See programs e.c and e1.c on D2L.</mark></p>
<pre class=" language-c"><code class="prism  language-c"><span class="token function">printf</span><span class="token punctuation">(</span><span class="token string">"I'm printed once!\n"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
<span class="token function">fork</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
<span class="token function">printf</span><span class="token punctuation">(</span><span class="token string">"I'm printed twice!\n"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<p>Here is a simple example of this address space cloning. The following program may print out 42 twice - but the  <code>fork()</code>  is after the  <code>[printf]</code>!? Why?</p>
<pre class=" language-c"><code class="prism  language-c">  <span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;unistd.h&gt;</span> </span><span class="token comment">/*fork declared here*/</span>
  <span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;stdio.h&gt;</span> </span><span class="token comment">/* printf declared here*/</span>
  <span class="token keyword">int</span> <span class="token function">main</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token keyword">int</span> answer <span class="token operator">=</span> <span class="token number">42</span><span class="token punctuation">;</span>
    <span class="token function">printf</span><span class="token punctuation">(</span><span class="token string">"Answer: %d"</span><span class="token punctuation">,</span> answer<span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token function">fork</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token keyword">return</span> <span class="token number">0</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
</code></pre>
<blockquote>
<p>Q: why \n to flush buffers?<br>
Sending each character to the terminal will be expensive so sending them in bulk would be more efficient. The end of line is a user-defined way of indicating that these many characters be sent to the terminal now.</p>
</blockquote>
<p>We will see the interplay between fork and file descriptors and memory shortly.</p>
<h2 id="determining-pids">Determining PIDs</h2>
<p>A parent can only determine the PID of the child through a fork(). A child can always determine the PID through getppid() call.</p>
<h3 id="two-things-about-fork">Two things about fork:</h3>
<ul>
<li>The implementation of fork is not standard across kernels. Sometimes the child is scheduled first, and sometimes the parent. This can leads to race conditions if the program logic depends upon the order in which programs are run.
<ul>
<li>But despite a policy taken by the kernel as to which program to run first, it is important to realize that after a fork(), it is indeterminate which of the two processes is next scheduled to use the CPU. It could very well be that the parent’s CPU time slice ran out before it had time to print its message.<br>
<mark>See program f.c on D2L. Run it with arguments close to 1M and 2M. Parse the output to check if the child is printed first.</mark></li>
</ul>
</li>
<li>Fork bombs.</li>
</ul>
<h2 id="fork-bombs">Fork Bombs</h2>
<p>A ‘fork bomb’ is when you attempt to create an infinite number of processes. This will often bring a system to a near-standstill as it attempts to allocate CPU time and memory to a very large number of processes that are ready to run. Simple example</p>
<pre><code>while  (1)  fork();
</code></pre>
<p>On your AWS machines there is no sysadmin. Limit the number of processes by<br>
<code>ulimit -u 40</code></p>
<p>If you fork bombed then using <a href="https://linux.die.net/man/1/killall">killall</a> does not help. Why?</p>
<blockquote>
<p>It requires your shell to <code>fork()</code> and you have reached the limit.</p>
</blockquote>
<p>One solution is to spawn another shell instance as another user (for example root) before hand and kill processes from there.</p>
<p>==See program g.c for setting limits. It uses a library function set limits. ==</p>
<h2 id="fork-and-dup">Fork and Dup</h2>
<p>When fork() is called, <strong>all file descriptors are duplicated as if dup() is called</strong>.</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;fcntl.h&gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;stdio.h&gt;</span></span>
<span class="token function">main</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
<span class="token punctuation">{</span>
  <span class="token keyword">char</span> s<span class="token punctuation">[</span><span class="token number">1000</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
  <span class="token keyword">int</span> i<span class="token punctuation">,</span> fd<span class="token punctuation">;</span>

  fd <span class="token operator">=</span> <span class="token function">open</span><span class="token punctuation">(</span><span class="token string">"file3"</span><span class="token punctuation">,</span> O_WRONLY <span class="token operator">|</span> O_CREAT <span class="token operator">|</span> O_TRUNC<span class="token punctuation">,</span> <span class="token number">0644</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

  i <span class="token operator">=</span> <span class="token function">fork</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token function">sprintf</span><span class="token punctuation">(</span>s<span class="token punctuation">,</span> <span class="token string">"fork() = %d.\n"</span><span class="token punctuation">,</span> i<span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token function">write</span><span class="token punctuation">(</span>fd<span class="token punctuation">,</span> s<span class="token punctuation">,</span> <span class="token function">strlen</span><span class="token punctuation">(</span>s<span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token function">close</span> <span class="token punctuation">(</span>fd<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<p>The file descriptor <strong>fd</strong> is duplicated across <strong>fork()</strong> calls. If not duplicated, but instead re-opened, then one <strong>write()</strong> would overwrite the other.</p>
<h2 id="fork-and-memory">Fork and memory</h2>
<p>Conceptually, we can consider fork() as creating copies of the parent’s text, data, heap, and stack segments. However, actually performing a simple copy of the parent’s virtual memory pages into the new child process would be wasteful for a number of reasons—one being that a fork() is often followed by an immediate exec(), which replaces the process’s text with a new program and reinitializes the process’s data, heap, and stack segments.</p>
<p>Most modern UNIX implementa- tions, including Linux, use two techniques to avoid such wasteful copying:</p>
<ul>
<li>
<p>􏰂 The kernel marks the text segment of each process as read-only, so that a pro- cess can’t modify its own code. This means that the parent and child can share the same text segment. The fork() system call creates a text segment for the child by building a set of per-process page-table entries that refer to the same virtual memory page frames already used by the parent.</p>
</li>
<li>
<p>􏰂 For the pages in the data, heap, and stack segments of the parent process, the kernel employs a technique known as copy-on-write. (The implementation of copy-on-write is described in [Bach, 1986] and [Bovet &amp; Cesati, 2005].) Initially, the kernel sets things up so that the page-table entries for these segments refer to the same physical memory pages as the corresponding page-table entries in the parent, and the pages themselves are marked read-only. After the fork(), the kernel traps any attempts by either the parent or the child to modify one of these pages, and makes a duplicate copy of the about-to-be-modified page. This new page copy is assigned to the faulting process, and the corresponding page- table entry for the child is adjusted appropriately. From this point on, the parent and child can each modify their private copies of the page, without the changes being visible to the other process. Figure 24-3 illustrates the copy-on-write technique.</p>
</li>
</ul>
<ol>
<li><a href="https://courses.engr.illinois.edu/cs241/sp2014/lecture/05-syscalls_sol.pdf">https://courses.engr.illinois.edu/cs241/sp2014/lecture/05-syscalls_sol.pdf</a></li>
<li><a href="https://www.cs.princeton.edu/courses/archive/spring09/cos217/lectures/21SystemCalls.pdf">https://www.cs.princeton.edu/courses/archive/spring09/cos217/lectures/21SystemCalls.pdf</a></li>
<li><a href="https://www.cs.cmu.edu/~guna/15-123S11/Lectures/">https://www.cs.cmu.edu/~guna/15-123S11/Lectures/</a></li>
</ol>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>

