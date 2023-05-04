Download Link: https://assignmentchef.com/product/solved-cs421-assignment-2-customftp-server
<br>



<h1>I.      Introduction</h1>

In this programming assignment, you are asked to implement the CustomFTP server which will be able to serve multiple clients at once.

The goal of the assignment is to make you familiar with concurrent networking using threads and stateful servers. You must implement your program using the <strong>Java Socket API of the JDK</strong>. If you have any doubt about what to use or not to use, please contact your teaching assistant.

When preparing your project please keep in mind that your projects will be auto-graded by a computer program. Any problems in the formatting can create problems in the grading; while you will not get a zero from your project, you may need to have an appointment with your teaching assistant for a manual grading. Errors caused by <strong>incorrectly naming</strong> the project files and folder structure will cause you to lose points.




<h1>II.       Design Specifications a. CustomFTP Specifications</h1>

You will implement the server program which will handle CustomFTP request from a CustomFTP client we will provide. This CustomFTP protocol is very similar to the one we provided in the first programming assignment.

<h2>i. Control Connection</h2>

All commands are sent from the client (i.e., the program we provide) to the server (i.e., the program you will provide) through the control connection. After each command, the server sends a response to the client again using the same control connection. This control connection is persistent, i.e., it stays open throughout the session. The commands must be constructed as strings encoded in <strong>US-ASCII </strong>and have the following format:

&lt;CommandName&gt;&lt;Space&gt;&lt;Argument&gt;&lt;Space&gt;&lt;Data&gt;&lt;CR&gt;&lt;LF&gt;

<ul>

 <li>&lt;CommandName&gt; is the name of the command. For all possible commands and their explanations, see Table 1.</li>

 <li>&lt;Space&gt; is a single space character. Can be omitted if &lt;Argument&gt; is empty.</li>

 <li>&lt;Argument&gt; is the argument specific to the command. Can be empty if no argument is needed for the given command. See Table 1 for more information.</li>

 <li>&lt;Data&gt; is only used in PUT The data format is explained in Fig. 1</li>

 <li>&lt;CR&gt;&lt;LF&gt; is a carriage return character followed by a line feed character, i.e., “r
”.</li>

</ul>

<table width="589">

 <tbody>

  <tr>

   <td width="129"><strong>&lt;CommandName&gt;</strong></td>

   <td width="120"><strong>&lt;Argument&gt; </strong></td>

   <td width="184"><strong>Explanation</strong></td>

   <td width="156"><strong>Example Usage </strong></td>

  </tr>

  <tr>

   <td width="129">PORT</td>

   <td width="120">&lt;port&gt;</td>

   <td width="184">Send port number of the data connection that the client is bound to.</td>

   <td width="156">PORT 60001r
</td>

  </tr>

  <tr>

   <td width="129">GPRT</td>

   <td width="120">–</td>

   <td width="184">Get the port of the server through a data connection.</td>

   <td width="156">GPRTr
</td>

  </tr>

  <tr>

   <td width="129">NLST</td>

   <td width="120">–</td>

   <td width="184">Obtain the list of all files and directories in the current working folder of the server.</td>

   <td width="156">NLSTr
</td>

  </tr>

  <tr>

   <td width="129">CWD</td>

   <td width="120">&lt;child&gt;</td>

   <td width="184">Change the working directory to the one of the child directories of the current working directory of the server.</td>

   <td width="156">CWD imagesr
</td>

  </tr>

  <tr>

   <td width="129">CDUP</td>

   <td width="120">–</td>

   <td width="184">Go to the parent directory of the current working directory of the server.</td>

   <td width="156">CDUPr
</td>

  </tr>

  <tr>

   <td width="129">PUT</td>

   <td width="120">&lt;filename&gt; </td>

   <td width="184">Put the file &lt;filename&gt; to the current working directory of the server</td>

   <td width="156">PUTImage.jpgr
</td>

  </tr>

  <tr>

   <td width="129">MKDR</td>

   <td width="120">&lt;foldername&gt;</td>

   <td width="184">Create the new folder&lt;foldername&gt; in the current working directory of the server.</td>

   <td width="156">MKDRimagefolderr
</td>

  </tr>

  <tr>

   <td width="129">RETR</td>

   <td width="120">&lt;filename&gt;</td>

   <td width="184">Retrieve the file&lt;filename&gt; from the current working directory of the server.</td>

   <td width="156">RETRsample.txtr
</td>

  </tr>

  <tr>

   <td width="129">DELE</td>

   <td width="120">&lt;filename&gt;</td>

   <td width="184">Delete the file&lt;filename&gt; from the current working directory of the server.</td>

   <td width="156">DELEsample.txtr
</td>

  </tr>

  <tr>

   <td width="129">DDIR</td>

   <td width="120">&lt;foldername&gt;</td>

   <td width="184">Delete the folder&lt;foldername&gt; from the current working directory of the server.</td>

   <td width="156">DDIRimagefolderr
</td>

  </tr>

  <tr>

   <td width="129">QUIT</td>

   <td width="120">–</td>

   <td width="184">Tell server to end the session and shutdown.</td>

   <td width="156">QUITr
</td>

  </tr>

 </tbody>

</table>




Your program must send response messages which are encoded in US-ASCII.

Responses have the following format:

&lt;Code&gt;&lt;CR&gt;&lt;LF&gt;

<ul>

 <li>&lt;Code&gt; is either 200 (success) or 400 (failure), for the sake of simplicity. Your server should only send the success or failure codes and nothing else.</li>

 <li>&lt;CR&gt;&lt;LF&gt; is a carriage return character followed by a line feed character, i.e., “r
”. The specifications which show when your program should send failure is described in Table 2.</li>

</ul>

<table width="586">

 <tbody>

  <tr>

   <td width="196"><strong>&lt;CommandName&gt; </strong></td>

   <td width="390"><strong>Fails if: </strong></td>

  </tr>

  <tr>

   <td width="196">PUT</td>

   <td width="390">File with same name already exists.</td>

  </tr>

  <tr>

   <td width="196">MKDR</td>

   <td width="390">Folder with same name already exists.</td>

  </tr>

  <tr>

   <td width="196">DELE</td>

   <td width="390">File does not exist in the current working directory.</td>

  </tr>

  <tr>

   <td width="196">DDIR</td>

   <td width="390">Folder does not exist in the current working directory.</td>

  </tr>

  <tr>

   <td width="196">CWD</td>

   <td width="390">Folder does not exist in the current working directory.</td>

  </tr>

  <tr>

   <td width="196">CDUP</td>

   <td width="390">Current working directory is the root folder.</td>

  </tr>

 </tbody>

</table>

<h1>          ii.      Data Connection</h1>

While commands and responses are exchanged through a persistent control connection, the data (e.g., files obtained using RETR command or list of files and directories obtained using NLST command) is sent through a non-persistent connection named <em>data connection</em>. That is, the connection is opened just before the data transmission and closed once the transmission is complete.

CustomFTP uses what is called <em>active mode </em>in FTP. In active mode, the client binds to an available port for the data connection and sends this port to the server using the PORT command, at the beginning of the session. Then, when a data transfer is about the begin, the server connects to the client at the specified port and the transmission begins.

For data transmission mode, CustomFTP uses <em>block mode</em>. In block mode, the data is sent in the form of a block with a 16-bit header in addition to the data. This header indicates the size of the data to be transmitted (see Figure 1). Byte order of the header is in big-endian format, i.e., the most significant byte is sent first, the least significant byte is sent last.

<table width="543">

 <tbody>

  <tr>

   <td width="373">


    <table width="364">

     <tbody>

      <tr>

       <td width="231">Data Length (2 Bytes)</td>

       <td width="133">Data</td>

      </tr>

     </tbody>

    </table></td>

   <td width="75"></td>

   <td width="95">


    <table width="89">

     <tbody>

      <tr>

       <td width="89"> </td>

      </tr>

     </tbody>

    </table></td>

  </tr>

 </tbody>

</table>

<em>Figure 1: Illustration of data transmission in block mode. Note that the maximum data length is 65535 Bytes</em>




<h1><sup>b.</sup> CustomFTPServer Design Specifications</h1>

The client program provided for you, CustomFTPClient, is a simple client which attempts to connect and create a pre-determined folder structure in the folder which the

CustomFTPServer is running. In the test program, multiple clients will connect to your CustomFTP server simultaneously and each will create their folder structures. The clients will utilize the commands described here so your server must implement them correctly to pass the test routines. Below there are detailed explanations of the important commands and expected behavior from your server program.

<h2>i.  GPRT</h2>

The GPRT command retrieves the data socket of your server program for the client since the client has to use your data-port for PUT command. The format of the expected data response is the same as all data connections (2-byte header and data). You should however send the port number encoded as US-ASCII string, e.g., port 60000 is 5 bytes of ASCII.

The general format of the message is in the following form: &lt;portNumber&gt;

<ul>

 <li>&lt;portNumber&gt; is the randomly selected port of the server.</li>

</ul>

<h3>          ii.        NLST</h3>

The NLST command retrieves the contents of the current working directory as a US-ASCII encoded string using the data connection in the following form:

&lt;name1&gt;:&lt;type1&gt;&lt;CR&gt;&lt;LF&gt;&lt;name2&gt;:&lt;type2&gt;&lt;CR&gt;&lt;LF&gt;…&lt;nameN&gt;:&lt;typeN&gt;

<ul>

 <li>&lt;nameX&gt; is the name of the X<sup>th</sup> directory/file in the list.</li>

 <li>&lt;typeX&gt; is the type of the X<sup>th</sup> directory/file in the list. It is d if X is a directory, f if it is a file.</li>

 <li>&lt;CR&gt;&lt;LF&gt; is a carriage return character followed by a line feed character, i.e., “r
”.</li>

 <li>Your program should connect to the client using its port number previously received when the connection was first established.</li>

 <li>Your program should terminate this connection after sending the data.</li>

</ul>

Note that there is no extra &lt;CR&gt;&lt;LF&gt; at the end. Thus, if the current directory is empty, NLST should not send anything but a 16-bit header of zeros. Some example strings that will be sent by your program after receiving an NLST command are as follows:

cheesecake:dr
futon.txt:fr
target.jpg:fr
pocket:d research:d bear.png:f

<h1>          iii.    RETR</h1>

When your program receives the RETR command, it should try to connect to the client with its address and the port number it received with the PORT command. For example, consider the client is operating on localhost and has sent a PORT command with 60002 as its argument when the connection was first established. When your program receives the command RETR note.txt, it will first connect to the client at localhost:60002 and it will send the file using the <em>block mode</em>, i.e., with 2-byte data-length header and the data following the header.

<ul>

 <li>Your program should terminate the connection after sending the data. <strong>iv.</strong> <strong>PUT </strong></li>

</ul>

The put command is used for transferring a file from the client to the server. An example of a PUT command is below.

PUT image.jpg&lt;CR&gt;&lt;LF&gt;

When a PUT command is received, your server should start to listen to its data port (similar to RETR and NLST for client). Do not forget to send responses for the commands prior to the data connection.




<h1>c. Running your Server</h1>

We provide a testing program called ServerTester.py which tests your CustomFTP server for conformity with the specifications. You must execute this program while your CustomFTP server is running. For testing, we provide a sample directory structure, which your CustomFTP server will “serve”. You must run your server at the root folder of this folder structure in the folder called “Testing”.

When started, the clients we provide will automatically try to access and modify the folder contents; for tests we will use multiple clients.

Your program should run with the command

java CustomFTPServer &lt;Addr&gt; &lt;ControlPort&gt;

where “&lt; &gt;” denotes command-line arguments. These command-line arguments are:

<ul>

 <li>&lt;Addr&gt; [Required] The IP address of the server. Since you will be running both your program and the server program on your own machine, you should use 0.0.1 or localhost for this argument.</li>

 <li>&lt;ControlPort&gt; [Required] The control port to which the server will bind. Your program should connect to the server from this port to send the control commands.</li>

 <li>You must run your server before the test program.</li>

 <li>You must run your server in the provided</li>

</ul>




<h2>d. Running the Tests</h2>

The test program called CS421PA2Tester.py is provided to you. These programs will try to perform certain operations using your CustomFTPServer.

You should run the testing program with the following command:

python CS421PA2Tester.py

This test program will spawn 2 CustomFTPClients which would try to use your server simultaneously.

<ul>

 <li>You must run this test program in the provided TestFolder.</li>

 <li>For your convenience we provide another program called “CustomFTPConsole” which you can use to debug your program. This is a small “REPL” interface for the CustomFTP. Note that it’s a dummy interface, the commands PUT and RETR do not alter anything on the machine.</li>

 <li>Test programs         are       expecting        your     servers            control            port     to         bind     to localhost:60000.</li>

</ul>




<h1>Final Remarks</h1>

<ul>

 <li>Please contact your teaching assistant if you have any doubt about the assignment.</li>

 <li>Note that we have omitted the USER and PASS commands from the previous assignment. ● The main goal of this project is for you to provide a CustomFTP server with multithreading. Therefore, you can assume that the main part of the grade will come from this aspect. As a guideline, prioritize according to these: (from most important to least important) multithreading, correct behavior for correct usage, error handling for incorrect usage, and boundary cases.</li>

 <li><strong>You can and should refer to PA1 for more detailed explanations of the previous behavior. </strong></li>

 <li>You can modify the source code of the clients for experimental purposes. However, do not forget that your projects will be evaluated based on the version we provide.</li>

 <li>We have tested that these programs work with the discussed Java-Python combination.</li>

 <li>You might receive some socket exceptions if your program fails to close sockets from its previous instance. In that case, you can manually shut down those ports by waiting for them to timeout, restarting the machine, etc.</li>

 <li>Remember that all the CustomFTP commands must be constructed as <strong>strings </strong>and encoded with <strong>US-ASCII </strong></li>

 <li>Remember that all data (i.e., directory listings and files) are sent in block mode with a <strong>16bit (2 bytes)</strong> header in <strong>big-endian</strong></li>

 <li>Remember that the control connection and data connection are separate. All commands and responses are exchanged using the control connection, which stays active throughout</li>

</ul>

the session, while all data is sent through the data connection which is closed immediately after the data transfer is completed.

<strong> </strong>

<h2></h2>