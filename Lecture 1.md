---


---

<h1 id="lecture1">Lecture1</h1>
<h2 id="what-is-a-machine">What is a machine?</h2>
<p>A computer system aka “a machine” is made of hardware and software. Lets take a look at the architecture of a machine.<br>
The figure shows some components of a computer system. In fact it shows the hardware components. Above the hardware is the software.</p>
<p><em>Q: Can you tell what software can these three boxes possibly contain?</em></p>
<blockquote>
<p>Operating system, Libraries, and Application Programs</p>
</blockquote>
<p>So what function do these boxes achieve? Lets say you have a higher level program, the OS will provide the compiler which will compile it into object code; the OS will also provide the linker which will compile it into machine code, which is run by the CPU.</p>
<p>So why can’t we simply write machine code? Because we are humans not machines. :) We like to write code in a human-understandable language and not 1s and 0s. So in some sense these programs introduce a “hierarchy”— the top layer converts the higher level  program which a user understands to low-level machine code to finally bits.</p>
<p><em>Q: What properties do you observe about the design of the computer system as reflected in this diagram?</em>~~</p>
<blockquote>
<p>Layers~~</p>
</blockquote>
<p>Hierarchy introduces abstraction. As a user you don’t have understand machine code but just the higher level language.</p>
<p>We would like to understand the concept of Abstraction</p>
<h2 id="abstraction">Abstraction</h2>
<p>Abstraction means hide the details of the layer below and only expose a logical view</p>
<!-- Coming back to the computer system, as we can see there are 3 important interfaces in the system: the ISA, the Application Binary Interface (which &#34;abstracts&#34; OS details from the application, and the Application Programming Interface (which &#34;abstracts&#34; application details from another application). But we will come to these interfaces and the abstraction they provide a bit later.-->
<p><em>Q: Have you as a programmer ever used the concept of abstraction for any other part of the computer system?</em></p>
<blockquote>
<p>The file is an abstraction of the disk</p>
</blockquote>
<p>So we all understand the advantage that abstraction gives us: abstractions hide lower-level implementation details, thereby reducing complexity.</p>
<p><em>Q: What is a disadvantage of abstraction?</em><br>
<strong>Note:</strong> The disadvantage is a subtle point that is best illustrated with electric switches.</p>
<blockquote>
<p>In a 2-layer system a layer above may not work with a new layer below as the layer below does not understand the interface.</p>
</blockquote>
<p>In other words if the lower layer changes then there is no way the above layer will work. Lets translate this picture in computer systems. Lets say we have a Intel CPU with Linux installed and a AMD CPU with Linux installed on it. I can always take the source code and translate into machine code in each machine, but I cannot take the machine code from one machine and simply run on another machine.</p>
<blockquote>
<p>Read StackOverflow post for a nice discussion about this example: <a href="https://unix.stackexchange.com/questions/298281/are-binaries-portable-across-different-cpu-architectures">https://unix.stackexchange.com/questions/298281/are-binaries-portable-across-different-cpu-architectures</a></p>
</blockquote>
<p>Now the situation is grave. We not only have different CPUs, we do have different OSs, different versions of libraries. In fact the complexity is number of possibilities times the number of “well-defined” interfaces in computer systems.</p>
<p>There are 3 important interfaces in the system: the ISA, the Application Binary Interface (which “abstracts” OS details from the application, and the Application Programming Interface (which “abstracts” application details from another application). <!--*For example:* the interface between hardware and software is known as the Instruction Set Architecture (ISA).--></p>
<p>Program distribution with only  abstraction is a challenge. Abstraction only transforms application programs to compiled binaries, which are tied to a specific ISA and depend on a specific operating system interface. But, we can never take any intermediate representation and run it on another computer because there are interoperability issues.</p>
<p>If you have heard the term “software distribution”–the above is the holy grail of software distribution that a program compiles with some OS, even though distributed may not work on another OS.</p>
<!--Formally, subsystems and components designed to specifications for one interface will not work with those designed for another.-->
<p><em>Q: As a programmer have you ever faced this situation when a layer above did not work for layer below? Look carefully at the computer system layers.</em>~~</p>
<blockquote>
<p>A program implemented for Windows machine does not  work on Linux machine</p>
</blockquote>
<h2 id="virtualization">Virtualization</h2>
<p><strong>Virtualization does not necessarily aim to simplify or hide details.</strong> It just gives you the underlying ugliness but provides a mapping software so that the underlying ugliness that your software understands can be mapped to the available ugliness.</p>
<p>Formally, virtualizing a system or component—such as a processor, memory, or an I/O device—at a given abstraction level <strong>maps its interface  and visible resources onto the interface and resources of an underlying, possibly different, real system.</strong></p>
<p>Thus now I can run Windows programs in Linux and Linux programs on Windows. I can run a program compiles on Intel CPUs on AMDs with the help of a mapping software.</p>
<p>In other words instead of asking you to compile your software from scratch or in case of different OS re-writing your software, virtualization aims to provide the underlying machine on which the software will run.</p>
<p>But the concept of virtualization is more general. It need not only apply to software. Coming back to our disk example, it can also be applied to data: Show figure.</p>
<p>In this course we only consider virtualization with respect to hardware and software, no disk or network virtualization.</p>
<h2 id="virtual-machines">Virtual Machines</h2>
<p>Informally, it is the concept of virtualization applied to hardware. In its simplest form it is a software layer added to  a host hardware to support the target machine code.</p>
<p>There are various types of machines or in other words different levels of virtualization or in words different ways of implementing virtual machines:</p>
<h3 id="system-virtual-machine-or-hardware-virtualization">System Virtual Machine or Hardware Virtualization</h3>
<p>A system virtual virtual machine is in which the target machine consists of the software layers and the ISA, a virtualization layer maps the ISA to the host physical layer.  The virtualization layer is called a “bare” metal hypervisor.</p>
<blockquote>
<p>Diagram</p>
</blockquote>
<p><a href="http://E.gs">E.gs</a>; VMWare, XEN Hypervisor, Oracle VMServer.</p>
<h3 id="process-virtual-machine-or">Process Virtual Machine or</h3>
<p>A process virtual machine is in which the target machine consists of the software layers, a  virtualization layer maps the OS layer to the Host OS. The virtualization layer is called a “hosted” hypervisor.</p>
<blockquote>
<p>Diagram<br>
E.g.s:  VirtualBox, Parallels VMWare Workstation</p>
</blockquote>
<h2 id="software-virtualization">Software Virtualization</h2>
<p>In software the heterogeneity arises due to different versions of the same software.</p>
<h3 id="application-virtual-machine">Application Virtual Machine</h3>
<p>An application virtual machine is in which the target machine consists of the application and library layers, <mark>a virtualization layer maps the library layer to the library of the host computer</mark>.<br>
What I need to virtualize is OS-behavior but not the complete OS.<br>
This mapping is called emulating the OS behavior, but provide the isolation to the versions of software.</p>
<p>E.g.s: Docker</p>
<p>This course is about application virtual machines. The virtualization layer is also called light-weight virtual machines or containers.</p>
<p><strong>To understand how to virtualize OS-behavior, we first need to understand the operating system and some crucial parts of the OS.</strong></p>
<h1 id="the-operating-system">The Operating System</h1>
<p>The term operating system is commonly used with two different meanings:</p>
<ul>
<li>
<p>To denote the entire package consisting of the central software managing a computer’s resources and all of the accompanying standard software tools, such as command-line interpreters, graphical user interfaces, file utilities, and editors.</p>
</li>
<li>
<p>More narrowly, to refer to the central software that manages and allocates computer resources (i.e., the CPU, RAM, and devices). The term kernel is often used as a synonym for the second meaning.</p>
</li>
</ul>
<h2 id="the-kernel">The Kernel</h2>
<p>What does “manages and allocates computer resources” imply?</p>
<p>For this we have to first understand what is a process, especially in the context of a kernel:</p>
<p><strong>Process:</strong>  Instance if a running program. (the live state of the machine code)</p>
<p>What is a kernel? Kernel is a special process whose  job is to</p>
<ul>
<li>load a new user process into memory,</li>
<li>provide it with the resources (e.g., CPU, memory, and access to files) that it needs in order to run.</li>
<li>It also assigns identifies the owner of the process</li>
</ul>
<p>Compute resources are limited but processes can be in hundreds and thousands. So another job of kernel is process scheduling, i.e., which program gets access to CPU.</p>
<p>Memory is also a limited resource. So another job of kernel is allocating, deallocating memory to processes.</p>
<p>Programs may ask for data on disk through the file system, so one job is to provision a file system, and through that access  to  devices</p>
<p>Yet another is to send data across network by interacting with network drivers.</p>
<p>So if the kernel does so many vital functions for a process, then the question is how does a user process interact with the special process the kernel? To understand we have to, given a program, look at the process vs kernel view of the OS</p>
<h2 id="process-vs-kernel-view-of-the-os">Process Vs Kernel View of the OS</h2>
<p>Consider a user program which simply does:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">int</span> i
i <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">;</span>
<span class="token function">printf</span><span class="token punctuation">(</span><span class="token string">"Hello World"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<p>Lets say this source code was compiled and a machine code was generated. The process was run. Now, the process is not entirely the user process. Because the program needs to access a terminal device which will print Hello World. The kernel becomes the owner of the process and on behalf of the user will print this for you. The kernel never relinquishes control of the resources. All user processes must cede control to the kernel and the kernel will use the resources for the user.</p>
<p>The above raises the question: how to make a request to this special process or the kernel to act on behalf of a user program?</p>
<p>The compiler will identify which parts of the program need to use resources such as use the network, disk, or output, and translate those user program commands to calls that the kernel will recognize. These calls to kernel appear in the object code.<br>
These calls are called System Calls</p>
<h2 id="system-calls">System Calls</h2>
<p>A system call is a <strong>common, controlled</strong> entry point into the kernel, allowing a process to request that the kernel perform some action on the process’s behalf. By <strong>common</strong> we imply all processes use this method. By controlled we imply that sharing has to be done safely.</p>
<p>Some general points about system calls:</p>
<ul>
<li>
<p>A system call changes <strong>the processor state</strong> from user mode to kernel mode, so that the CPU can access protected kernel memory.</p>
</li>
<li>
<p>Examples of sys calls: creating a new process, performing I/O, and creating a pipe for interprocess  communication.</p>
</li>
<li>
<p>Set of system calls is fixed. Each system call is identified by a unique number.</p>
</li>
<li>
<p>System calls are written in C.</p>
</li>
</ul>
<blockquote>
<p>Note: there are a total of 329 calls. See here for a listing<br>
<a href="http://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/">http://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/</a></p>
</blockquote>
<h2 id="how-user-mode-is-switched-to-kernel-mode">How user mode is switched to kernel mode</h2>
<p>Two points to note:</p>
<ul>
<li>
<p>All user provided information needs to be copied from process memory to kernel memory</p>
</li>
<li>
<p>The kernel (which is a process) must be aware that the process has copied the memory.</p>
</li>
<li>
<p>The kernel is also a process.</p>
</li>
</ul>
<p>Look at the diagram presented in class.</p>
<p>Switching to Kernel Mode</p>
<ol>
<li>Application program makes a system call by invoking wrapper function in C library</li>
<li>This wrapper functions makes sure that all the system call arguments are available to trap-handling routine</li>
<li>Generally a stack is used to pass these arguments to wrapper function. But the Kernel process wants these arguments in specific registers. Hence the wrapper function also takes care of copying these arguments to specific registers</li>
<li>The wrapper function sets the register by executing <em>trap</em> instruction (int 0x80).</li>
<li>When the processor sees the register is set, it causes the processor to switch from  <em>‘User Mode’</em>  to  <em>‘Kernel Mode’</em></li>
<li>Each system call has a unique call number which is used by kernel to identify which system call is invoked. The wrapper function again copies the system call number into specific CPU registers</li>
<li>Now the wrapper function executes  <em>trap</em> instruction (int 0x80). This instruction causes the processor to switch from  <em>‘User Mode’</em>  to  <em>‘Kernel Mode’</em></li>
</ol>
<blockquote>
<p>Note, most modern machines are using sysenter rather than 0x80 trap instruction)</p>
</blockquote>
<p>Kernel Response</p>
<ol>
<li>In response to trap to location 0x80, kernel invokes system_call() routine which is located in assembler file arch/i386/entry.S (also called handler)</li>
<li>This routine copies register values onto kernel stack and does some validations like verifying system call number etc.</li>
<li>A map of system call number as key and the appropriate system call as value exists. This is called system_call_table. The handler uses this table to invoke appropriate system call service routine. It also validates the arguments if present.</li>
<li>After proper validations, the service routine performs required actions like modify values at addresses specified in arguments or transfer data between user memory and kernel memory. After all these actions, service routine returns status of execution to the system_call routine</li>
<li>Now the handler restores register values from kernel stack and places the system call return value on the stack</li>
<li>Thus routine is returned to wrapper function, simultaneously returning processor to user mode</li>
<li>Just in case if the return value of system call service routine indicated an error, then wrapper function sets ‘errno’ a global variable and then returns to caller providing integer return value that indicates the status of execution</li>
</ol>
<h2 id="demo">Demo</h2>
<p>Helloworld.c</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;fcntl.h&gt;</span></span>

<span class="token keyword">int</span> <span class="token function">main</span><span class="token punctuation">(</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
 <span class="token keyword">int</span> fd <span class="token operator">=</span> <span class="token function">open</span><span class="token punctuation">(</span><span class="token string">"HelloWorld.txt"</span><span class="token punctuation">,</span> <span class="token number">0</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<p><code>gcc -ggdb -o openHelloWorld openHelloWorld.c</code></p>
<p><code>$ strace ./openHelloWorld</code></p>
<p><code>$ gdb openHelloWorld</code><br>
<code>b openHelloWorld.c:4</code><br>
<code>r</code><br>
<code>s</code><br>
<code>disassemble</code></p>
<h2 id="who-makes-system-calls">Who makes System Calls?</h2>
<p>The libraries that you include in the program internally make system calls. Most common library is glibc.</p>
<p>glibc, <a href="http://www.gnu.org/software/libc/">http://www.gnu.org/software/libc</a><a href="http://www.gnu.org/software/libc/">/</a></p>
<p>How to find version  of  glibc on your computer</p>
<p><code>$ ls /lib/x86_64-linux-gnu/libc.so.6</code></p>
<p>From your program determining the location of the library:</p>
<p><code>$ ldd myprog | grep libc</code></p>
<h2 id="what-if-sys-calls-fail">What if sys calls fail?</h2>
<p>All system call wrappers will return a value indicating success/status or an error</p>
<ul>
<li>See the system call’s man page for details about the meaning of the return value</li>
<li>int errno is a global variable that is set by the kernel side of the system call to provide more details about any error that has occurred</li>
</ul>
<pre class=" language-c"><code class="prism  language-c">fd <span class="token operator">=</span> <span class="token function">open</span><span class="token punctuation">(</span>pathname<span class="token punctuation">,</span> flags<span class="token punctuation">,</span> mode<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token keyword">if</span> <span class="token punctuation">(</span>fd <span class="token operator">==</span> <span class="token operator">-</span><span class="token number">1</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
<span class="token comment">/* Code to handle the error */</span>
<span class="token punctuation">}</span> 
<span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token function">close</span><span class="token punctuation">(</span>fd<span class="token punctuation">)</span> <span class="token operator">==</span> <span class="token operator">-</span><span class="token number">1</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
<span class="token comment">/* Code to handle the error */</span>
<span class="token punctuation">}</span>
</code></pre>
<ul>
<li>Many system calls return -1 to indicate an error; then, you can use errno to extract additional meaning
<ul>
<li>getpid(), exit() do not return an error value. No need to check return values from such system calls</li>
</ul>
</li>
</ul>
<p>A variety of available functions will make it easy to translate the error number into a textual description</p>
<ul>
<li>
<p>errExit()</p>
<ul>
<li>prints  and a description of the error encoded by errno to STDERR • and then exits your program</li>
<li>Defined in the TLPI library</li>
</ul>
</li>
<li>
<p>perror()</p>
<ul>
<li>prints  and a description of the error to STDOUT</li>
<li>Defined in stdio</li>
</ul>
</li>
</ul>
<pre class=" language-c"><code class="prism  language-c">fd <span class="token operator">=</span> <span class="token function">open</span><span class="token punctuation">(</span>pathname<span class="token punctuation">,</span> flags<span class="token punctuation">,</span> mode<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token keyword">if</span> <span class="token punctuation">(</span>fd <span class="token operator">==</span> <span class="token operator">-</span><span class="token number">1</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
<span class="token function">perror</span><span class="token punctuation">(</span><span class="token string">"open"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token function">exit</span><span class="token punctuation">(</span>EXIT_FAILURE<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<h2 id="performance-of-system-calls">Performance of system calls</h2>
<p>All the copying implies your program is not executing instructions.<br>
This slows program behavior.</p>
<pre class=" language-c"><code class="prism  language-c"> <span class="token macro property">#<span class="token directive keyword">include</span><span class="token string">&lt;stdio.h&gt;</span></span>
 <span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">&lt;unistd.h&gt;</span></span>
 <span class="token macro property">#<span class="token directive keyword">include</span><span class="token string">&lt;stdlib.h&gt;</span></span>
 
 <span class="token keyword">int</span> i<span class="token punctuation">,</span>j<span class="token punctuation">;</span>
 <span class="token keyword">int</span> <span class="token function">getvalue</span> <span class="token punctuation">(</span><span class="token keyword">int</span> k<span class="token punctuation">)</span> <span class="token punctuation">{</span>
     <span class="token keyword">return</span> k<span class="token operator">+</span><span class="token number">1</span><span class="token punctuation">;</span>
 <span class="token punctuation">}</span>
 
 <span class="token keyword">int</span> <span class="token function">main</span><span class="token punctuation">(</span><span class="token keyword">int</span> argc<span class="token punctuation">,</span> <span class="token keyword">char</span> <span class="token operator">*</span>argv<span class="token punctuation">[</span><span class="token punctuation">]</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
	 <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token function">atoi</span><span class="token punctuation">(</span>argv<span class="token punctuation">[</span><span class="token number">1</span><span class="token punctuation">]</span><span class="token punctuation">)</span> <span class="token operator">==</span> <span class="token number">1</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
	      <span class="token keyword">for</span> <span class="token punctuation">(</span>j <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">;</span> j <span class="token operator">&lt;=</span> <span class="token number">1000000</span><span class="token punctuation">;</span> j<span class="token operator">++</span><span class="token punctuation">)</span>
		      i <span class="token operator">=</span> <span class="token function">getvalue</span><span class="token punctuation">(</span>j<span class="token punctuation">)</span><span class="token punctuation">;</span>
	 <span class="token punctuation">}</span>
	 <span class="token keyword">else</span> <span class="token punctuation">{</span>
	     <span class="token keyword">for</span> <span class="token punctuation">(</span>j <span class="token operator">=</span> <span class="token number">0</span><span class="token punctuation">;</span> j <span class="token operator">&lt;=</span> <span class="token number">1000000</span><span class="token punctuation">;</span> j<span class="token operator">++</span><span class="token punctuation">)</span>
	      	<span class="token function">printf</span><span class="token punctuation">(</span><span class="token string">"%d"</span><span class="token punctuation">,</span> <span class="token function">getppid</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
	 <span class="token punctuation">}</span>
 <span class="token punctuation">}</span>
</code></pre>
<h3 id="how-to-check-time-a-process-spends-in-user-vs-kernel-mode">How to check time a process spends in user Vs kernel mode?</h3>
<ul>
<li>
<p>Real time is measured either from some standard point (calendar time) or from some fixed point, typically the start, in the life of a process (elapsed or wall clock time).</p>
</li>
<li>
<p>Process time, also called CPU time, is the total amount of CPU time that a process has used since starting.</p>
<ul>
<li>system CPU time, the time spent executing code in kernel mode</li>
<li>user CPU time, the time spent executing code in user mode</li>
</ul>
</li>
</ul>
<pre><code> $ time ./a.out 1 
</code></pre>

