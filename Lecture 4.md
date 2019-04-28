---


---

<p>Today we will cover process previlidge, process tracing, and signals.<br>
These are the last topics needed to start making headway into the container domain.</p>
<h2 id="user-and-special-accounts">User and Special Accounts</h2>
<ul>
<li>whoami: name associated with current uid
<ul>
<li>Each user has a unique UID</li>
<li>root is designated superuser with uid = 0</li>
</ul>
</li>
</ul>
<p>Check out the diagram on board in recording.</p>
<ul>
<li>groups: groups of which current uid is a member
<ul>
<li>Users belong to multiple groups</li>
</ul>
</li>
<li>See users and groups together with the id(1) program</li>
<li>Each file has 12 permission bits
<ul>
<li>read/write/execute for user, group, others</li>
</ul>
</li>
</ul>
<h2 id="permissions">Permissions</h2>
<p>Lets first understand the concept of unix permissions on files.</p>
<h3 id="basic-file-permissions">Basic file permissions</h3>
<p>Everything in Unix is a file i.e. all resources are represented in terms of files. If users want to control resources, the first thing is that Unix must have a way to assign permissions to files.</p>
<p>File permissions determine who can read, write and/or execute a given unix file.</p>
<p>This gives 3 types of domains and 3 types of permissions on a given resource.</p>
<p>How to allocate? Use bits.<br>
3 bits for each domain representing each type of permisison leading to 9 bits.</p>
<h3 id="from-user-to-owner.">From user to owner.</h3>
<p>To understand a file’s permissions, one must first understand the concept of owner, group, and other.</p>
<ul>
<li>The owner of a file is usually the person who created it (although it may be changed).</li>
<li>The group ownership of a file is a collection of users who  group ownyour file.</li>
<li>Finally, other is any other user on the system.</li>
</ul>
<p>The reason unix has all three is because you may wish to give only the owner (generally yourself) the ability to read and write your file, while allowing the group to only read it, and the rest of the computer’s user community, no access to the file at all. So, with a few commands you can protect your files reading, writing, and execution policies between three different domains (owner, group, and other).</p>
<h3 id="viewing-permissions">Viewing permissions</h3>
<p>When you view the output from a  ls -l, you will notice on the far left, there will be dashes and letters. This is the 9-bit access control field of a file’s status. The first three bits (starting from the left) represent the owner, the next three (middle) represent the group, and the last three (on the right) represent all others. Moreover, the  <code>r' stands for read,</code>w’  for write, and  `x’  for execute. So as you can see from doing a  ls -lg  on foo, produces:</p>
<p>-rw-r–r--  1 tot    staff        2437 Sep  8 15:59 foo</p>
<p>Here, tot (the owner), has read and write privileges, staff (the group) has read privileges, and all others also have read privileges. Consider the following output:</p>
<p>-rwxr-x–x  1 tot    staff        2437 Sep  8 15:59 foo</p>
<p>In this case, tot (the owner), has read, write, and execute privileges. The group staff, has read and execute privileges, and all others can only execute it.</p>
<blockquote>
<p>Reading Permissions<br>
<code>----------</code> 0000 	no permissions</p>
</blockquote>
<p><code>-rwx------</code> 0700</p>
<p><code>-rwxrwx---</code> 0770 read, write, &amp; execute for owner and group</p>
<p><code>-rwxrwxrwx</code>   0777   read, write, &amp; execute for owner, group and others</p>
<p><code>---x--x--x</code>   0111  execute</p>
<p><code>--w--w--w-</code>   0222  write</p>
<p><code>--wx-wx-wx</code> 0333 write &amp; execute</p>
<p><code>-r--r--r--</code> 0444 read</p>
<p><code>-r-xr-xr-x</code> 0555 read &amp; execute</p>
<p><code>-rw-rw-rw-</code> 0666 read &amp; write</p>
<p><code>-rwxr-----</code></p>
<h3 id="changing-permissions">Changing permissions</h3>
<p>Describe notation and changing</p>
<p>There are two ways to change a file’s permissions. The first way is to set the permissions with a text string. Say we wish to make a file owner=read, group=read, and other=read. To do this we would do</p>
<p>chmod ugo=r foo</p>
<p>where u=owner, g=group, and o=other, and foo is the filename. Now doing an  ls -l, will produce the following output:</p>
<p>-r–r--r--	1 tot       2437 Sep  8 15:59 foo</p>
<p>Alternatively, (from  `foo’'s current state), we could do:</p>
<p>chmod u+wx foo</p>
<p>where the  `+’  means add to the current permissions, produces:</p>
<p>-rwxr–r--  1 tot        2437 Sep  8 15:59 foo</p>
<p>This method should be obvious by now. The second method uses three sets of binary numbers, converted to decimal. So, if you wish to give owner read, write, group permission and all others read permission, enter:</p>
<p>chmod 644 foo</p>
<p>where the first digit is for the owner, second is for the group, and third is for all other, would produce:</p>
<p>-rw-r–r--  1 tot        2437 Sep  8 15:59 foo</p>
<p>Notice the 644 is easily extracted from  `-rw-r–r--’.</p>
<pre><code>                    6        4       4
                    |        |       |
                  4 + 2      |       |
                   \  |      |       |
                    \ |      |       |
                     421     421     421
                     rw-     r--     r--
</code></pre>
<p>All you have to do is look and see which bits are turned on, add them up and, poof, you have your number to call  chmod  with. Say you wish read, write, and execute (rwx) on owner, read and execute on group and all others (r-x), enter:</p>
<p>chmod 755 foo</p>
<p>-rwxr-xr-x  1 tot        2437 Sep  8 15:59 foo</p>
<p>Figure shows how you can obtain 755 from  `-rwxr-xr-x’.</p>
<pre><code>                      7       5       5
                      |       |       |
                  4 + 2 + 1  4+1     4+1
                   \  |  /   | |     | |
                    \ |/     | |     | |
                     421     421     421
                     rwx     r-x     r-x
</code></pre>
<p>You may take your pick of whichever method you prefer. Directory permissions work the same way, except what it means to  execute  <strong>a directory is basically permissions to list the contents of it.</strong> Hence, a directory of:</p>
<p>drwxr-xr-x  2 tot         512 Sep  8 17:46 foo</p>
<p>(notice the  `d’  on the far left, that means it’s a directory)</p>
<p>this means the owner tot, may read, write &amp; execute it (cd  into it and list it’s contents). It is uncommon to only give a user read privilege to a directory and not execute, although you may. This would allow one to read from your directory but not list it’s contents.</p>
<blockquote>
<p>One more thing, if you wish all files to be created with some certain<br>
permissions, use  umask. For example, if you issue  umask 022  you<br>
will mask all your upcoming created files with 022, thus producing a<br>
newly created file with permissions (644):<br>
-rw-r–r--  1 tot        2437 Sep  8 18:12 foo<br>
If you do a man on  <code>umask</code>  (man umask) it will explain in more<br>
detail about file masking.</p>
</blockquote>
<h2 id="privilege">Privilege</h2>
<p><strong>Privilege</strong> is defined as the delegation of authority to perform security-relevant functions on a computer system. A privilege allows a user to perform an action with security consequences.</p>
<p>This means that if someone acquires privilege they can become quite powerful.</p>
<p>Root is special, but sometimes you may want to assume the identity of another account user.</p>
<h2 id="who-acquires-privileges">Who acquires privileges?</h2>
<p>Proceeses acquire privileges. How?</p>
<p>Each process has its own set of “identities”/tokens to show what are its privileges. We will give these identities and tokens a more fancy term later. Lets write now understand the types of identities/tokens:</p>
<p>Every process has a set of associated numeric user identifiers (UIDs) and group identifiers (GIDs). These identifiers are as follows:</p>
<ul>
<li>real user ID and group ID;</li>
<li>effective user ID and group ID;</li>
<li>saved set-user-ID and saved set-group-ID;</li>
</ul>
<p>Process has 3 user IDs; ruid, euid, suid and 3 groupids: rgid, egid, sgid</p>
<p>When a process is created by fork it inherits all 3 userIDS from its parent process</p>
<p>When a process executes a file by exec it keeps its 3 userids unless the set user id bit on the file is set in which case the effective uid and saved uid are assigned the user id of the owner of the file</p>
<h2 id="when-do-we-acquire-privileges-or-need-to-make-effective-uid-of-a-process-different-from-real-uid">When do we acquire privileges or need to make effective UID of a process different from real UID?</h2>
<h3 id="example">Example</h3>
<p>A good example of setuid would be for the ping command. Ping required root so it can open a socket in raw mode.</p>
<pre><code>icmp_sock = socket(AF_INET, SOCK_RAW, IPPROTO_ICMP);
</code></pre>
<p>However, for ping to be useful, you really want the tool to be available for all users to execute. To achieve this, setuid is assigned, which allows an object to execute with User ID 0 (root). You can view the setuid bit in the permissions of the ping binary.</p>
<pre><code>$ ls -l /bin/ping
 -rwsr-xr-x 1 root root 44168 Jan  2 13:43 /bin/ping
</code></pre>
<p>The ‘s’ denotes the setuid bit.<br>
To show what happens, when ping has no setuid, let’s revoke the setuid…</p>
<pre><code>$ sudo chmod u-s /bin/ping
$ ls -l /bin/ping
 -rwxr-xr-x 1 root root 44168 Jan&amp;nbsp; 2 13:43 /bin/ping
</code></pre>
<p>Now when we try to use ping, without setuid providing us sufficient permissions to open a unix socket, we get permission denied</p>
<pre><code>$ ping localhost
 ping: icmp open socket: Operation not permitted
</code></pre>
<p>Examples of commonly used set-user-ID programs on Linux include: passwd(1), which changes a user’s password; mount(8) and umount(8), which mount and unmount file systems; and su(1), which allows a user to run a shell under a different user ID. An example of a set-group-ID program is wall(1), which writes a message to all terminals owned by the tty group (normally, every terminal is owned by this group).</p>
<h3 id="setuid-and-setgid-bit-on-files-difference-between-s-and-s">Setuid and Setgid Bit on Files (Difference between ‘s’ and ‘S’)</h3>
<p>The concept of  setuid  files means that if you have the  setuid bit  turned on on a file, anybody <strong>executing</strong> that command (file) will inherit the permissions of the owner of the file. So if you have the following setuid file:</p>
<p>-rwsr-xr-x  1 ubuntu ubuntu      2437 Sep  8 18:12 foo</p>
<p>This means, that when any user executes this file  `foo’, he will inherit tot’s uid (which means he inherits all their file access permissions whether you’re tot or not).  <strong>Note: this can be quite dangerous.</strong>  If you have a setuid shell owned by yourself, and I execute it, I essentially inherit your file permissions, hence have the ability to remove all your files.</p>
<p>In the above output, the reason for the small  s  means there’s an  x  (execute) under it that’s hidden. If it were a large  S  as in:</p>
<p>-rwSrw-rw-  1 ubuntu ubuntu        2437 Sep  8 18:12 foo</p>
<p>This means there’s no  x  under the  S. In order to make a file setuid, you prepend the three digits given  chown  with a  4, or use the  s  option. For example to get the first output:</p>
<p>-rwsr-xr-x  1 ubuntu ubuntu       2437 Sep  8 18:12 foo</p>
<p>I did a  chmod 4755. The same is true for group, except use a  2  instead. If you want both, add them, to get  6. Hence a file with  chmod 6755  would look like:</p>
<p>-rwsr-sr-x  1 ubuntu ubuntu        2437 Sep  8 18:12 foo</p>
<p>Again, notice the small  s. What does that mean?</p>
<h3 id="the-case-of-the-password-file">The case of the password file</h3>
<p>What should be the permissions of a password file?</p>
<p>Historically, passwords are maintained in /etc/password. This file has several other information user login information also.</p>
<p>It should definitely be ‘r’ for others so all programs can read other login information. But it also imposes a security problem.</p>
<p>The other programs are unprivileged—they belong to different user accounts. This opens the door for password-cracking programs, which try encrypting large lists of likely passwords (e.g., standard dictionary words or people’s names) to see if they match the encrypted password of a user. The shadow password file, /etc/shadow, was devised as a method of preventing such attacks. The idea is that all of the nonsensitive user information resides in the publicly readable password file, while encrypted passwords are maintained in the shadow password file, which is readable only by privileged programs.</p>
<h2 id="process-credentials">Process Credentials</h2>
<p>The previlidge of a program determined by its credentials.</p>
<p><strong>Need for ruid/rgid</strong>:<br>
The real user ID and group ID identify the user and group to which the process <mark>belongs</mark>. It identifies the login process who is the parent of the current process.</p>
<p><strong>Need for euid/egid:</strong> To help determine credentials for system call invocation.</p>
<p>The distinction between a real and an effective user id is made because you may have the need to temporarily take another user’s identity (most of the time, that would be  <code>root</code>, but it could be any user). If you only had one user iIf euid = 0, the process has all of the privileges of the superuser. Such a process is referred to as a privileged process. Certain system calls can be executed only by privileged processes.</p>
<p>Normally, the effective user and group IDs have the same values as the corresponding real IDs, but there are two ways in which the effective IDs can assume different values. One way is through the use of system calls . The second way is through the execution of set-user-ID and set-group-ID programs.</p>
<h4 id="difference-between-su-and-sudo">Difference between su and sudo</h4>
<ul>
<li>Changing identity
<ul>
<li>su username : “become” username</li>
<li>su - username: “become” username, using initialization files</li>
<li>su: “become” root (su = superuser)</li>
<li>sudo command: Execute command as root (if youre in /etc/sudoers and you give your password.)</li>
</ul>
</li>
</ul>
<p>Review password program on D2L needed to be run from a root login so that int case you are  <code>root</code>, using  <code>root</code>'s privileges to ould access the /etc/shadow file. We could make this program runnable by any user by making it a set-user-ID-root program (Check on D2L)</p>
<p><strong>Need for suid/sgid</strong></p>
<ul>
<li>SUID, SGID - Saved UID and GID; used to support switch    permissions "on and off’’.</li>
</ul>
<p>In the previous ping program: When a set-user-ID program is run (i.e., loaded into a process’s memory by an exec()), the kernel sets the effective user ID of the process to be the same as the user ID of the executable file.</p>
<p>In other words, the executable file was owned by root (superuser) and had the set-user-ID permission bit enabled,  then there would be no way of changing back to your original user id afterwards (other than taking your word for granted, and user process gains superuser privileges when that program is run.</p>
<p>To switch permissions on/off the following twwo steps are taken:</p>
<ol>
<li>If the set-user-ID (set-group-ID) permission bit is enabled on the executable, then the effective user (group) ID of the process is made the same as the owner of the executable.</li>
<li>The values for the saved set-user-ID and saved set-group-ID are copied from the corresponding effective IDs. This copying occurs regardless of whether the set-user-ID or set-group-ID bit is set on the file being executed.</li>
</ol>
<p>As an example of the effect of the above steps, suppose that a process whose real user ID, effective user ID, and saved set-user-ID are all 1000 execs a set-user-ID program owned by root (user ID 0). After the exec, the user IDs of the process will be changed as follows:</p>
<p>real=1000 effective=0 saved=0</p>
<p>Various system calls allow a set-user-ID program to switch its effective user ID between the values of the real user ID and the saved set-user-ID. Analogous system calls allow a set-group-ID program to modify its effective group ID. In this manner, the program can temporarily drop and regain whatever privileges are associated with the user (group) ID of the execed file. (In other words, the program can move between the states of potentially being privileged and actually operating with privilege.) It is secure programming practice for set- user-ID and set-group-ID programs to operate under the unprivileged (i.e., real) ID whenever the program doesn’t actually need to perform any operations associated with the privileged (i.e., saved set) ID.</p>
<h3 id="comment-on-sticky-bits">Comment on Sticky Bits</h3>
<p>Create a directory and provide all the users read-write-execute access to it :</p>
<pre><code># mkdir allAccess

# chmod 777 allAccess/

# ls -ld allAccess/
</code></pre>
<p>Now, create multiple files in this directory (with different users) such that all users have read-write-execute access to them.</p>
<p>In order to avoid users delething (moseach other’s files, sticky bit can be set ofn the time, there are some exceptions).</p>
<p>If euid = 0, the process has all of the privileges of the superuser. Such a process is referred to as a privileged process. Certain system calls can be executed only by privileged processes.</p>
<p>When you log in, the login shell sets both the real and effective user id to the same value (youdirectory allAccess.</p>
<p>Now, turn ON the sticky bit on the directory by using +t flag of chmod command.</p>
<pre><code># chmod +t allAccess/

# ls -ld allAccess/
</code></pre>
<p>As can be observed, a permission bit ‘t’ is introduced in the permission bits of the directory.</p>
<h2 id="principle-of-least-previlidge">Principle of Least Previlidge</h2>
<p>The common feature of these tools is that they know their real identity is of a non-privileged user, but have the ability to assume a privileged identity when required. (Note that “privileged” doesn’t necessarily mean root; it merely means some other identity that has the power to do what the real user can’t.) Such executables are collectively referred as “setuid programs”, because (1) they must be explicitly associated with a “setuid bit” (through the chmod command), and (2) they pull off the identity juggling trick through the use of set∗id system calls (setuid(2), setreuid(2), and all their friends).</p>
<p>There’s another, often overlooked, type of programs that do identity juggling but do not have an associated setuid bit. These start off as root processes and use set∗id system calls to change their identity to that of an ordinary non-privileged user. Examples include the login program, the cron dæmon (which runs user tasks at a specified time), dæmons providing service to remote users by assuming their identity (sshd, telnetd, nfs, etc.), and various mail server components.</p>
<p>Both types of programs share a similar philosophy: in order to reduce the chances of their extra powers being abused, they attempt to obey the principle of least privilege, which states that “every program and 1 every user of the system should operate using the least set of privileges necessary to complete the job” [16].</p>
<p>For setuid programs this translates to</p>
<ol>
<li>minimizing the number and duration of the time periods at which the program temporarily assumes the privileged identity, in order to reduce the negative effect that programming mistakes might have (e.g., mistakenly removing a file as root can have far greal user id) as supplied by the password file.</li>
</ol>
<p>There are two wayter negative implications than doing it when the non-privileged identity is in effect), and<br>
2. permanently giving up the ability to assume the privileged identity as soon as it’s no longer needed, so that if an attacker gains control (e.g., through a buffer overflow vulnerability), he can’t exploit those privileges.</p>
<p>The principle of least privilege is a simple and sensible rule.</p>
<p>But when it comes to identity-changing programs (in which the effective IDs can assume different values. One way is through the immortal words of The Essex [7] or anybody who ever tried to lose weight [14]) it’s easier said than done. Here are a few quotes that may explain why it’s at least as hard as doing a diet: Chen et al. said that “for historical reasons, the uid-setting system calls are poorly designed, insufficiently documented, and widely misunderstood” and that the associated manuals “are often incomplete or even wrong” [2]. Dean and Hu observed that “the usetuid family of system calls . The second way is through the execution of set-user-ID and set-group-ID programs.</p>
<p><strong>Need for suid/sgid</strong><br>
Now, it also happens that you execute a setuid program, and besides running as another user (e.g.  <code>root</code>) the setuid program is  <em>also</em>  supposed to do something on your behalf. How does this work?<br>
After executing the setuid program, it will have your real id (since you’re the process owner) and the effective user id of the file owner (for example  <code>root</code>) since it is setuid.</p>
<p>The program does whatever magic it needs to do with superuser privileges and then wants to do something on your behalf. That means, attempting to do something that you shouldn’t be able to do  <em>should fail</em>. How does it do that? Well, obviously by changing its effective user id to the real user id!</p>
<p>Now that setuid program has no way of switching back since all the kernel knows is your id and…  <em>your id</em>. Bang, you’re dead.</p>
<p>This is what the saved set-user id is for.<br>
Some operations are not modeled as files and require uid = 0<br>
halthing the system<br>
bind/listen on previldged ports<br>
System integrity requires more than controlling who can write but also how it is written</p>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>
<p>is its own rats nest; on different Unix and Unix-like systems, system calls of the same name and arguments can have different semantics, including the possibility of silent failures” [3]. Torek and Dik concluded that “many years after the inception of setuid programs, how to write them is still not well understood by the majority of people who write them” [17]. All these deficiencies have made the setuid mechanism the source of many security vulnerabilities.</p>
<h2 id="capabilities-vs--posix-capabilities">Capabilities Vs  POSIX Capabilities</h2>
<p><a href="https://mirrors.edge.kernel.org/pub/linux/libs/security/linux-privs/kernel-2.2/capfaq-0.2.txt">https://mirrors.edge.kernel.org/pub/linux/libs/security/linux-privs/kernel-2.2/capfaq-0.2.txt</a></p>
<h2 id="posix-capabilities">POSIX Capabilities</h2>
<p>In operating systems, there are many privileged operations that can only be conducted by privileged users. Examples of privilegd operations include configuring network interface card, backing up all the user files, shutting down the computers, etc. Without capabilities, these operations can only be carried out by superusers, who often have many more privileges than what are needed for the intended tasks. Therefore, letting superusers to conduct these privileged operations is a violation of the Least-Privilege Principle.</p>
<p>Privileged operations are very necessary in operating systems. All Set-UID programs invole privileged operations that cannot be performed by normal users. To allow normal users to run these programs, Set-UID programs turn normal users into powerful users (e.g. root) temporarily, even though that the involved privileged operations do not need all the power. This is dangerous: if the program is compromised, adversaries might get the root privilege.</p>
<p>Capabilities divide the powerful root privilege into a set of less powerful privileges. Each of these privileges is called a capability. With capabilities, we do not need to be a superuser to conduct privileged operations. All we need is to have the capabilities that are needed for the privileged operations. Therefore, even if a privileged program is compromised, adversaries can only get limited power. This way, risk of privileged program can be lowered quite significantly.</p>
<p>We will use an example to show how capabilities can be used to remove unnecessary power assigned to certain privileged programs. First, let us login as a normal user, and use the ping command to ping an eos machine:</p>
<p><code>% ping eos24.cis.gvsu.edu</code></p>
<p>The program should run successfully. If you look at the file attribute of the program /bin/ping, you will find out that ping is actually a Set-UID program with the owner being root, i.e., when you execute ping, your effective user id becomes root, and the running process is very powerful. If there are vulnerabilities in ping, the entire system can be compromised. The question is whether we can remove these privileged from ping.</p>
<p>Let us turn /bin/ping into a non-Set-UID program. This can be done via the following command (you need to login as the root):</p>
<p><code># chmod u-s /bin/ping</code></p>
<p>Now, run ping again, and see what happens. Interestingly, the command will not work. This is because ping needs to open RAW socket, which is a privileged operation that can only be conducted by root (before capabilities are implemented). That is why ping has to be a Set-UID program. With capability, we do not need to give too much power to ping. Assign the cap net raw capability to ping with the following command:</p>
<p><code># setcap cap_net_raw=ep /bin/ping</code></p>
<p>Now, the ping command should work once again, verify that it does.</p>
<h2 id="types-of-posix-capablities">Types of POSIX Capablities</h2>
<p>A process has three sets of bitmaps called the inheritable(I),<br>
permitted§, and effective(E) capabilities.  Each capability is<br>
implemented as a bit in each of these bitmaps which is either set or unset.  When a process tries to do a privileged operation, the operating system will check the appropriate bit in the effective set of the process (instead of checking whether the effective uid of the process is 0 as is normally done).  For example, when a process tries to set the clock, the Linux kernel will check that the process has the CAP_SYS_TIME bit (which is currently bit 25) set in its effective set.</p>
<p>The <em>effective</em> capability set indicates what capabilities are effective. When a process tries to do a privileged operation, the operating system will check the appropriate bit in the effective set of the process (instead of checking whether the effective uid of the process i 0 as is normally done). For example, when a process tries to set the clock, the Linux kernel will check that the process has the CAP SYS TIME bit (which is currently bit 25) set in its effective set.</p>
<p>The <em>permitted</em> capability set indicates what capabilities the process can use. The process can have capabilities set in the permitted set that are not in the effective set. This indicates that the process has temporarily disabled this capability. A process is allowed to set a bit in its effective set only if it is available in the permitted set. The distinction between effective and permitted makes it possible for a process to disable, enable and drop privileges.</p>
<p>The <em>inheritable</em> capability set indicates what capabilities of the current process should be inherited by the program executed by the current process. When a process executes a new program (using exec()), its new capability sets are calculated according to the following formula:<br>
pI_new = pI pP_new = (X &amp; fP) | (fI &amp; pI)<br>
pE_new = pP_new if fE == true<br>
pE_new = empty if fE == false A</p>
<h2 id="signals">Signals</h2>
<p>The concept of signals is simple to understand but there are details.</p>
<p><a href="https://www.youtube.com/watch?v=u15BUoioCZw">https://www.youtube.com/watch?v=u15BUoioCZw</a></p>
<p>Check slides after this.</p>
<p>You must know the following:</p>
<ul>
<li>
<p>the various different signals and their purposes;</p>
</li>
<li>
<p>the circumstances in which the kernel may generate a signal for a process</p>
</li>
<li>
<p>how a process responds to a signal by default, and the means by which a process can change its response to a signal, in particular, through the use of a signal handler, a programmer-defined function that is automatically invoked on receipt of a signal;</p>
</li>
<li>
<p>the use of a process signal mask to block signals, and the associated notion of pending signals; and</p>
</li>
<li>
<p>how a process can suspend execution and wait for the delivery of a signal.</p>
</li>
</ul>
<blockquote>
<p>Written with <a href="https://stackedit.io/">StackEdit</a>.</p>
</blockquote>

