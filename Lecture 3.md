---


---

<h2 id="process-lifecycle">Process Lifecycle</h2>
<p>Last time we learned how to fork processes.<br>
How many processes will this program fork:</p>
<pre><code>fork()
fork()
fork()
</code></pre>
<blockquote>
<p>8 processes.</p>
</blockquote>
<p>In general processes are isolated. That means that by default, no process can communicate with another process. Processes need IPC to communicate. Parents can communicate with a child through IPC and SIGNALS . More on SIGNALS later.</p>
<h2 id="process-diagram">Process Diagram</h2>
<p>Lets review the Process diagram in Slide 1. You must have seen the diagram in CSC407.</p>
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
<h2 id="process-table-and-pcb">Process Table and PCB</h2>
<p>Relationship between Process Table and Process Control Block. See diagram in slides.</p>
<p>To keep track of all these processes, your operating system gives each process a number and that process is called the PID, process ID. Processes also have a  <code>ppid</code>  which is short for parent process id.</p>
<ul>
<li>The getpid() system call returns the process ID of the calling process. The getppid() system call lets a process find out the process ID of its parent.</li>
<li>The Linux kernel limits process IDs to being less than or equal to 32,767 . When a new process is created, it is assigned the next sequentially available process ID.</li>
<li>Each time the limit of 32,767 is reached, the kernel resets its process ID counter so that process IDs are assigned starting from low integer values.</li>
<li>On 32-bit platforms, the maximum value for this file is 32,768, but on 64-bit platforms, it can be adjusted to any value up to 222 (approximately 4 million), making it possible to accommodate very large numbers of processes.</li>
</ul>
<p>Can a parent know the PID of its child?</p>
<blockquote>
<p>Only through a fork()</p>
</blockquote>
<p>Every process has exactly one parent, that parent could be  <code>init.d</code>.</p>
<ul>
<li>The kernel creates the first process <code>init.d</code>. Init.d boots up things like your GUI, terminals etc. What is important is the operating system only really creates one process by default, all other processes are <a href="https://linux.die.net/man/3/fork">fork</a> and <a href="https://linux.die.net/man/3/exec">exec</a>’ed from that single process.</li>
<li>With the exception of a few system processes such as init (process ID 1), there is no fixed relationship between a program and the process ID of the process that is created to run that program.</li>
<li>If I want to visualize all processes, which data structure will the represent: a tree or a graph?</li>
<li>If a child process becomes orphaned because its “birth” parent terminates, then the child is adopted by the init process, and subsequent calls to getppid() in the child return 1</li>
</ul>
<p><mark>Review program b.c on D2L</mark><br>
To see process tree:</p>
<blockquote>
<p>pstree<br>
ps -aef --forest</p>
</blockquote>
<hr>
<p>There are other statuses about processes:</p>
<ul>
<li>Running State - Whether a process is getting ready, running, stopped, terminated, suspended.</li>
</ul>
<blockquote>
<p>Review slide on process states.</p>
</blockquote>
<ul>
<li>
<p>File Descriptors - List of mappings from integers to real devices (files, usb sticks, sockets)</p>
</li>
<li>
<p>Permissions - To which  <a href="https://linux.die.net/man/3/user">user</a>  and  <code>group</code>  the process belongs to. We will see process permissions in next lecture.</p>
</li>
<li>
<p>Arguments - a list of strings that tell your program what parameters to run under.</p>
</li>
<li>
<p>Environment List - a list of strings in the form  <code>NAME=VALUE</code>  that one can modify.</p>
</li>
</ul>
<blockquote>
<p>See figure for process environment list data structures.<br>
Code to print process environment:<br>
<mark>Review program c.c on D2L</mark></p>
</blockquote>
<pre class=" language-c"><code class="prism  language-c"><span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">"tlpi_hdr.h"</span></span>
<span class="token keyword">extern</span> <span class="token keyword">char</span> <span class="token operator">*</span><span class="token operator">*</span>environ<span class="token punctuation">;</span> 
<span class="token keyword">int</span> <span class="token function">main</span><span class="token punctuation">(</span><span class="token keyword">int</span> argc<span class="token punctuation">,</span> <span class="token keyword">char</span> <span class="token operator">*</span>argv<span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">)</span> <span class="token punctuation">{</span> 
<span class="token keyword">char</span> <span class="token operator">*</span><span class="token operator">*</span>ep<span class="token punctuation">;</span>
 <span class="token keyword">for</span> <span class="token punctuation">(</span>ep <span class="token operator">=</span> environ<span class="token punctuation">;</span> <span class="token operator">*</span>ep <span class="token operator">!=</span> <span class="token constant">NULL</span><span class="token punctuation">;</span> ep<span class="token operator">++</span><span class="token punctuation">)</span>
        <span class="token function">puts</span><span class="token punctuation">(</span><span class="token operator">*</span>ep<span class="token punctuation">)</span><span class="token punctuation">;</span> 
 <span class="token function">exit</span><span class="token punctuation">(</span>EXIT_SUCCESS<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
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
<h2 id="process-state-duplication-in-fork">Process state duplication in Fork</h2>
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
<h2 id="process-lifecycle-1">Process Lifecycle</h2>
<p>The process lifecycle: is fork()-&gt; exit(status)-&gt;wait(status)-&gt;-   execve(pathname, argv, envp)</p>
<p><mark>See program h.c for a parent waiting for a child to exit, which the child changes the program it is running.</mark></p>
<h2 id="exitstatus">Exit(status)</h2>
<p>A process may terminate abnormally from an external signal or normally using the _<em>exit(status)</em> system call.</p>
<ul>
<li>_<em>exit(status)</em> terminates a process, making all resources (memory, open file descriptors, and so on) used by the process available for subsequent reallocation by the kernel.</li>
<li>The status argument is an integer that determines the termination status for the process.</li>
<li>If the process exited normally and everything was successful, then a zero should be returned (convention). Beyond that, there isn’t too many conventions except the ones that you place on yourself. The only restriction is that a process can only have 256 return values.</li>
</ul>
<p>The exit() library function is layered on top of the _exit() system call</p>
<ul>
<li>Programs generally don’t call _exit() directly, but instead call the exit() library function, which performs various actions before calling _exit().
<ul>
<li>Exit handlers (functions registered with atexit() and on_exit()) are called</li>
<li>The stdio stream buffers are flushed.</li>
<li>The _exit() system call is invoked, using the value supplied in status.<br>
-Can you do exit(-1)?</li>
</ul>
</li>
</ul>
<p><mark>See example i.c on D2L for registering an exit handler.</mark></p>
<p>The GNU C library provides two ways of registering exit handlers.</p>
<ul>
<li>The atexit() function adds func to a list of functions that are called when the process terminates.
<ul>
<li>The function func should be defined to take no arguments and return no value, thus having the following general form:</li>
</ul>
</li>
</ul>
<pre class=" language-c"><code class="prism  language-c"> <span class="token keyword">void</span> <span class="token function">func</span><span class="token punctuation">(</span><span class="token keyword">void</span><span class="token punctuation">)</span> <span class="token punctuation">{</span> 
 <span class="token comment">/* Perform some actions */</span>
 <span class="token punctuation">}</span>
</code></pre>
<ul>
<li>
<p>Note that atexit() returns a nonzero value (not necessarily –1) on error.</p>
</li>
<li>
<p>A child process created via fork() inherits a copy of its parent’s exit handler registrations. When a process performs an exec(), all exit handler registrations are removed. (This is necessarily so, since an exec() replaces the code of the exit handlers along with the rest of the existing program code.)</p>
</li>
<li>
<p>We can’t deregister an exit handler that has been registered with atexit() (or on_exit(), described below). However, we can have the exit handler check whether a global flag is set before it performs its actions, and disable the exit handler by clearing the flag.</p>
</li>
</ul>
<p>Exit handlers registered with atexit() suffer a couple of limitations. The first is that when called, an exit handler doesn’t know what status was passed to exit(). Occasionally, knowing the status could be useful; for example, we may like to perform different actions depending on whether the process is exiting successfully or unsuccessfully. The second limitation is that we can’t specify an argument to the exit handler when it is called. Such a facility could be useful to define an exit handler that performs different actions depending on its argument, or to register a function multiple times, each time with a different argument.</p>
<p>To address these limitations, glibc provides a (nonstandard) alternative method of registering exit handlers: on_exit().</p>
<p>==See example i.c for exit handlers. ==</p>
<h2 id="waitstatus">Wait(status)</h2>
<p>A parent process needs to know when one of its child processes changes state—when the child terminates or is stopped by a signal. Lead to the wait() system call (and its variants) and the use of the SIGCHLD signal.</p>
<h3 id="waitstatus-1">wait(status)</h3>
<ul>
<li>Wait suspends execution and returns</li>
<li>The process waits until one of the child process terminates.</li>
<li>Acquires termination status from child.</li>
<li>You will only be concerned with two status: WIFEXITED and WEXITSTATUS</li>
</ul>
<p>==See slide for differences between waitpid() and wait()<br>
See program j.c on D2L for checking wait status. ==</p>
<h3 id="zombie-and-orphans">Zombie and Orphans</h3>
<p>Orphan is a child process whose parent has died.</p>
<ul>
<li>The orphaned child is adopted by init, the ancestor of all processes, whose process ID is 1.</li>
</ul>
<p>Zombie is a child process that had never been waited for by the parent.</p>
<ul>
<li>The child has finished its work, the parent should still be permitted to perform a wait() at some later time to determine how the child terminated. The kernel deals with this situation by turning the child into a zombie. This means that most of the resources held by the child are released back to the system to be reused by other processes. The only part of the process that remains is an entry in the kernel’s process table recording (among other things) the child’s process ID, termination status, and resource usage statistics</li>
<li>When the parent does perform a wait(), the kernel removes the zombie, since the last remaining information about the child is no longer required. On the other hand, if the parent terminates without doing a wait(), then the init process adopts the child and automatically performs a wait(), thus removing the zombie process from the system.</li>
</ul>
<p><mark>See program k.c on D2L for viewing zombies.</mark></p>
<blockquote>
<p>The program uses system() function that allows the calling program to execute an arbitrary shell command.<br>
In the program you will see that the parent sends SIGKILL to zombie but you see the zombie still in the system(cmd) function.<br>
The reason is that a zombie process only has entry in the PCB, it cannot respond to signals. (More on this in the next lecture.)</p>
</blockquote>
<h2 id="execvepathname-argv-envp">Execve(pathname, argv, envp)</h2>
<p>Is a system call that loads a new program (pathname, with argument list argv, and environment list envp) into a process’s memory.</p>
<ul>
<li>The existing program text is discarded, and the stack, data, and heap segments are freshly created for the new program.</li>
<li>Some other operating systems combine the functionality of fork() and exec() into a single operation—a so-called spawn—that creates a new process that then executes a specified program.</li>
</ul>
<p>The  <a href="https://linux.die.net/man/3/exec">exec</a>  set of functions replaces the process image with the the process image of what is being called. This means that any lines of code after the  <a href="https://linux.die.net/man/3/exec">exec</a>  call are replaced. Any other work you want the child process to do should be done before the  <a href="https://linux.die.net/man/3/exec">exec</a>  call. The naming schemes can be shortened mnemonically.</p>
<ol>
<li>
<p>e – An array of pointers to environment variables is explicitly passed to the new process image.</p>
</li>
<li>
<p>l – Command-line arguments are passed individually (a list) to the function.</p>
</li>
<li>
<p>p – Uses the PATH environment variable to find the file named in the file argument to be executed.</p>
</li>
<li>
<p>v – Command-line arguments are passed to the function as an array (vector) of pointers.<br>
<mark>See examples k.c and l.c on D2L</mark></p>
</li>
</ol>
<pre class=" language-c"><code class="prism  language-c"><span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;unistd.h&gt;</span> </span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;sys/types.h&gt;</span> </span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;sys/wait.h&gt;</span> </span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;stdlib.h&gt;</span> </span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;stdio.h&gt;</span> int  </span>
<span class="token function">main</span><span class="token punctuation">(</span><span class="token keyword">int</span>  argc<span class="token punctuation">,</span>  <span class="token keyword">char</span><span class="token operator">*</span><span class="token operator">*</span>argv<span class="token punctuation">)</span>  <span class="token punctuation">{</span>  
pid_t  child  <span class="token operator">=</span>  <span class="token function">fork</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  

<span class="token keyword">if</span>  <span class="token punctuation">(</span>child  <span class="token operator">==</span>  <span class="token operator">-</span><span class="token number">1</span><span class="token punctuation">)</span>  
<span class="token keyword">return</span>  EXIT_FAILURE<span class="token punctuation">;</span>  

<span class="token keyword">if</span>  <span class="token punctuation">(</span>child<span class="token punctuation">)</span>  <span class="token punctuation">{</span>  <span class="token comment">/* I am the parent */</span>  
	<span class="token keyword">int</span>  status<span class="token punctuation">;</span>  
	<span class="token function">waitpid</span><span class="token punctuation">(</span>child  <span class="token punctuation">,</span>  <span class="token operator">&amp;</span>status  <span class="token punctuation">,</span><span class="token number">0</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
	<span class="token keyword">return</span>  EXIT_SUCCESS<span class="token punctuation">;</span>  
<span class="token punctuation">}</span>  <span class="token keyword">else</span>  <span class="token punctuation">{</span>  <span class="token comment">/* I am the child */</span>  
	<span class="token comment">// Remember first arg is the program name  </span>
	<span class="token comment">// Last arg must be a char pointer to NULL</span>
	<span class="token function">execl</span><span class="token punctuation">(</span><span class="token string">"/bin/ls"</span><span class="token punctuation">,</span>  <span class="token string">"/bin/ls"</span><span class="token punctuation">,</span>  <span class="token string">"-alh"</span><span class="token punctuation">,</span>  <span class="token punctuation">(</span><span class="token keyword">char</span>  <span class="token operator">*</span><span class="token punctuation">)</span>  <span class="token constant">NULL</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
	<span class="token comment">// Something went wrong!</span>
	<span class="token function">perror</span><span class="token punctuation">(</span><span class="token string">"exec failed!"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
	<span class="token punctuation">}</span>  
<span class="token punctuation">}</span>
</code></pre>
<p>Another example:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;unistd.h&gt;</span> </span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;fcntl.h&gt;</span> </span>
<span class="token comment">// O_CREAT, O_APPEND etc. defined here </span>
<span class="token keyword">int</span>  <span class="token function">main</span><span class="token punctuation">(</span><span class="token punctuation">)</span>  <span class="token punctuation">{</span>  
<span class="token function">close</span><span class="token punctuation">(</span><span class="token number">1</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  <span class="token comment">// close standard out  </span>
<span class="token function">open</span><span class="token punctuation">(</span><span class="token string">"log.txt"</span><span class="token punctuation">,</span>  O_RDWR  <span class="token operator">|</span>  O_CREAT  <span class="token operator">|</span>  O_APPEND<span class="token punctuation">,</span>  S_IRUSR  <span class="token operator">|</span>  S_IWUSR<span class="token punctuation">)</span><span class="token punctuation">;</span>  
<span class="token function">puts</span><span class="token punctuation">(</span><span class="token string">"The Brown Dog Jumped over the Moon"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
<span class="token function">chdir</span><span class="token punctuation">(</span><span class="token string">"/usr/include"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  
<span class="token comment">// execl( executable, arguments for executable including program name and NULL at the end)  </span>
<span class="token function">execl</span><span class="token punctuation">(</span><span class="token string">"/bin/ls"</span><span class="token punctuation">,</span>  <span class="token comment">/* Remaining items sent to ls*/</span>  <span class="token string">"/bin/ls"</span><span class="token punctuation">,</span>  <span class="token string">"."</span><span class="token punctuation">,</span>  <span class="token punctuation">(</span><span class="token keyword">char</span>  <span class="token operator">*</span><span class="token punctuation">)</span>  <span class="token constant">NULL</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  <span class="token comment">// "ls ."  perror("exec failed");  </span>
<span class="token keyword">return</span>  <span class="token number">0</span><span class="token punctuation">;</span>  
<span class="token punctuation">}</span>
</code></pre>
<h2 id="fork-exec-wait-pattern">FORK-EXEC-WAIT Pattern</h2>
<h2 id="file-system-calls">File System Calls</h2>
<h3 id="stat">stat()</h3>
<p>#include &lt;sys/stat.h&gt;<br>
int stat(const char* name, struct stat* buf);</p>
<p>Get information about a file<br>
Returns 0 on success and -1 on error<br>
Parameters: Path to file you want to use (absolute or relative), and buf: statistics structure</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">struct</span> stat <span class="token punctuation">{</span>
          mode_t   st_mode<span class="token punctuation">;</span>     <span class="token comment">/* File mode (see mknod(2)) */</span>
          ino_t    st_ino<span class="token punctuation">;</span>      <span class="token comment">/* Inode number */</span>
          dev_t    st_dev<span class="token punctuation">;</span>      <span class="token comment">/* ID of device containing */</span>
                                <span class="token comment">/* a directory entry for this file */</span>
          dev_t    st_rdev<span class="token punctuation">;</span>     <span class="token comment">/* ID of device */</span>
                                <span class="token comment">/* This entry is defined only for */</span>
                                <span class="token comment">/* char special or block special files */</span>
          nlink_t  st_nlink<span class="token punctuation">;</span>    <span class="token comment">/* Number of links */</span>
          uid_t    st_uid<span class="token punctuation">;</span>      <span class="token comment">/* User ID of the file's owner */</span>
          gid_t    st_gid<span class="token punctuation">;</span>      <span class="token comment">/* Group ID of the file's group */</span>
          off_t    st_size<span class="token punctuation">;</span>     <span class="token comment">/* File size in bytes */</span>
          time_t   st_atime<span class="token punctuation">;</span>    <span class="token comment">/* Time of last access */</span>
          time_t   st_mtime<span class="token punctuation">;</span>    <span class="token comment">/* Time of last data modification */</span>
          time_t   st_ctime<span class="token punctuation">;</span>    <span class="token comment">/* Time of last file status change */</span>
                                <span class="token comment">/* Times measured in seconds since */</span>
          <span class="token keyword">long</span>     st_blksize<span class="token punctuation">;</span>  <span class="token comment">/* Preferred I/O block size */</span>
          <span class="token keyword">long</span>     st_blocks<span class="token punctuation">;</span>   <span class="token comment">/* Number of 512 byte blocks allocated*/</span>
<span class="token punctuation">}</span><span class="token punctuation">;</span>
</code></pre>
<pre class=" language-c"><code class="prism  language-c"><span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt; stdio.h &gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt; sys/types.h &gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt; sys/stat.h &gt;</span></span>

<span class="token function">main</span><span class="token punctuation">(</span><span class="token keyword">int</span> argc<span class="token punctuation">,</span> <span class="token keyword">char</span> <span class="token operator">*</span><span class="token operator">*</span>argv<span class="token punctuation">)</span>
<span class="token punctuation">{</span>
  <span class="token keyword">int</span> i<span class="token punctuation">;</span>
  <span class="token keyword">struct</span> stat buf<span class="token punctuation">;</span>
  <span class="token keyword">int</span> exists<span class="token punctuation">;</span>

  <span class="token keyword">for</span> <span class="token punctuation">(</span>i <span class="token operator">=</span> <span class="token number">1</span><span class="token punctuation">;</span> i <span class="token operator">&lt;</span> argc<span class="token punctuation">;</span> i<span class="token operator">++</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    exists <span class="token operator">=</span> <span class="token function">stat</span><span class="token punctuation">(</span>argv<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token punctuation">,</span> <span class="token operator">&amp;</span>buf<span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token keyword">if</span> <span class="token punctuation">(</span>exists <span class="token operator">&lt;</span> <span class="token number">0</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token function">fprintf</span><span class="token punctuation">(</span><span class="token constant">stderr</span><span class="token punctuation">,</span> <span class="token string">"%s not found\n"</span><span class="token punctuation">,</span> argv<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span> <span class="token keyword">else</span> <span class="token punctuation">{</span>
      <span class="token function">printf</span><span class="token punctuation">(</span><span class="token string">"%10ld %s\n"</span><span class="token punctuation">,</span> buf<span class="token punctuation">.</span>st_size<span class="token punctuation">,</span> argv<span class="token punctuation">[</span>i<span class="token punctuation">]</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre>
<p>An example of reading each directory to do a stat()</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt; stdio.h &gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt; sys/types.h &gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt; sys/stat.h &gt;</span></span>
<span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt; dirent.h &gt;</span></span>

<span class="token function">main</span><span class="token punctuation">(</span><span class="token punctuation">)</span>
<span class="token punctuation">{</span>
  <span class="token keyword">struct</span> stat buf<span class="token punctuation">;</span>
  <span class="token keyword">int</span> exists<span class="token punctuation">;</span>
  DIR <span class="token operator">*</span>d<span class="token punctuation">;</span>
  <span class="token keyword">struct</span> dirent <span class="token operator">*</span>de<span class="token punctuation">;</span>

  d <span class="token operator">=</span> <span class="token function">opendir</span><span class="token punctuation">(</span><span class="token string">"."</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token keyword">if</span> <span class="token punctuation">(</span>d <span class="token operator">==</span> <span class="token constant">NULL</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token function">fprintf</span><span class="token punctuation">(</span><span class="token constant">stderr</span><span class="token punctuation">,</span> <span class="token string">"Couldn't open \".\"\n"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token function">exit</span><span class="token punctuation">(</span><span class="token number">1</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>

  <span class="token keyword">for</span> <span class="token punctuation">(</span>de <span class="token operator">=</span> <span class="token function">readdir</span><span class="token punctuation">(</span>d<span class="token punctuation">)</span><span class="token punctuation">;</span> de <span class="token operator">!=</span> <span class="token constant">NULL</span><span class="token punctuation">;</span> de <span class="token operator">=</span> <span class="token function">readdir</span><span class="token punctuation">(</span>d<span class="token punctuation">)</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
    exists <span class="token operator">=</span> <span class="token function">stat</span><span class="token punctuation">(</span>de<span class="token operator">-&gt;</span>d_name<span class="token punctuation">,</span> <span class="token operator">&amp;</span>buf<span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token keyword">if</span> <span class="token punctuation">(</span>exists <span class="token operator">&lt;</span> <span class="token number">0</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
      <span class="token function">fprintf</span><span class="token punctuation">(</span><span class="token constant">stderr</span><span class="token punctuation">,</span> <span class="token string">"%s not found\n"</span><span class="token punctuation">,</span> de<span class="token operator">-&gt;</span>d_name<span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span> <span class="token keyword">else</span> <span class="token punctuation">{</span>
      <span class="token function">printf</span><span class="token punctuation">(</span><span class="token string">"%s %ld\n"</span><span class="token punctuation">,</span> de<span class="token operator">-&gt;</span>d_name<span class="token punctuation">,</span> buf<span class="token punctuation">.</span>st_size<span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token punctuation">}</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre>
<h3 id="dup">dup()</h3>
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
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>

