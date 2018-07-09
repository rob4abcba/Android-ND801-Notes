
**



See this post on StackOverflow for a good discussion on different methods of doing this. 
Also, if you want more information on the okhttp, you can find it here.





**



If you have a java.io.InputStream object, how should you process that object and produce a String?

Suppose I have an InputStream that contains text data, and I want to convert it to a String, so for example I can write that to a log file.

What is the easiest way to take the InputStream and convert it to a String?

public String convertStreamToString(InputStream is) { 
    // ???
}
java string io stream inputstream
shareedit
edited Jun 24 at 1:20

Mahozad
35216
asked Nov 21 '08 at 16:47

Johnny Maelstrom
16.6k51617
712
Boy, I'm absolutely in love with Java, but this question comes up so often you'd think they'd just figure out that the chaining of streams is somewhat difficult and either make helpers to create various combinations or rethink the whole thing. – Bill K Nov 21 '08 at 17:16
20
The answers to this question only work if you want to read the stream's contents fully (until it is closed). Since that is not always intended (http requests with a keep-alive connection won't be closed), these method calls block (not giving you the contents). – f1sh Jul 14 '10 at 13:32
12
You need to know and specify the character encoding for the stream, or you will have character encoding bugs, since you will be using a randomly chosen encoding depending on which machine/operating system/platform or version thereof your code is run on. That is, do not use methods that depend on the platform default encoding. – Christoffer Hammarström Dec 17 '10 at 13:50 
2
Just to have fun with my own comment from 9 years ago, these days I use Groovy's "String s=new File("SomeFile.txt").text" to read an entire file all at once and it works great. I am happy with using groovy for my non-production (scripting) code and--well honestly forcing you to deal with encoding and extremely long files the way java does is a really good idea for production code anyway so it works for it's purpose, Groovy works for quick scripts which java isn't great at--Just use the right tool for the job and it all works out. – Bill KNov 1 '17 at 23:58 
show 1 more comment
57 Answers
activeoldestvotes
1 2 next
up vote
2016
down vote
accepted
A nice way to do this is using Apache commons IOUtils to copy the InputStream into a StringWriter... something like

StringWriter writer = new StringWriter();
IOUtils.copy(inputStream, writer, encoding);
String theString = writer.toString();
or even

// NB: does not close inputStream, you'll have to use try-with-resources for that
String theString = IOUtils.toString(inputStream, encoding); 
Alternatively, you could use ByteArrayOutputStream if you don't want to mix your Streams and Writers

shareedit
edited May 21 at 13:09

Marko Zajc
6411
answered Nov 21 '08 at 16:54

Harry Lime
21.2k32030
42
For android developers, seems like android does not come with IOUtils from Apache. So you might consider referring to other answers. – Chris.Zou Aug 1 '14 at 3:57
26
This is an incredibly old question at this point (it was asked in 2008). It is worth your time to read through more modern answers. Some use native calls from the Java 8 library. – Shadoninja Mar 28 '16 at 3:13
20
This answer is heavily outdated and one should be able to mark it as such (sadly this is not possible atm). – codepleb Apr 20 '16 at 13:00
show 10 more comments

up vote
2080
down vote
Here's a way using only standard Java library (note that the stream is not closed, YMMV).

static String convertStreamToString(java.io.InputStream is) {
    java.util.Scanner s = new java.util.Scanner(is).useDelimiter("\\A");
    return s.hasNext() ? s.next() : "";
}
I learned this trick from "Stupid Scanner tricks" article. The reason it works is because Scanneriterates over tokens in the stream, and in this case we separate tokens using "beginning of the input boundary" (\A) thus giving us only one token for the entire contents of the stream.

Note, if you need to be specific about the input stream's encoding, you can provide the second argument to Scanner constructor that indicates what charset to use (e.g. "UTF-8").

Hat tip goes also to Jacob, who once pointed me to the said article.

EDITED: Thanks to a suggestion from Patrick, made the function more robust when handling an empty input stream. One more edit: nixed try/catch, Patrick's way is more laconic.

shareedit
edited Sep 2 '17 at 1:27
answered Mar 26 '11 at 20:40

Pavel Repin
25.9k12737
8
Thanks, for my version of this I added a finally block that closes the input stream, so the user doesn't have to since you've finished reading the input. Simplifies the caller code considerably. – user486646 Apr 21 '12 at 17:07 
4
@PavelRepin @Patrick in my case, an empty inputStream caused a NPE during Scanner construction. I had to add if (is == null) return ""; right at the beginning of the method; I believe this answer needs to be updated to better handle null inputStreams. – CFL_Jeff Aug 9 '12 at 13:36 
95
For Java 7 you can close in a try-with: try(java.util.Scanner s = new java.util.Scanner(is)) { return s.useDelimiter("\\A").hasNext() ? s.next() : ""; } – earcam Jun 13 '13 at 5:24 
3
Unfortunately this solution seems to go and lose the exceptions thrown in my underlying stream implementation. – Taig Jul 16 '13 at 7:59
11
FYI, hasNext blocks on console input streams (see here). (Just ran into this issue right now.) This solution works fine otherwise... just a heads up. – Ryan Feb 24 '14 at 5:36 
show 10 more comments
up vote
1600
down vote
Summarize other answers I found 11 main ways to do this (see below). And I wrote some performance tests (see results below):

Ways to convert an InputStream to a String:

Using IOUtils.toString (Apache Utils)

String result = IOUtils.toString(inputStream, StandardCharsets.UTF_8);
Using CharStreams (guava)

String result = CharStreams.toString(new InputStreamReader(
      inputStream, Charsets.UTF_8));
Using Scanner (JDK)

Scanner s = new Scanner(inputStream).useDelimiter("\\A");
String result = s.hasNext() ? s.next() : "";
Using Stream Api (Java 8). Warning: This solution convert different line breaks (like \r\n) to \n.

String result = new BufferedReader(new InputStreamReader(inputStream))
  .lines().collect(Collectors.joining("\n"));
Using parallel Stream Api (Java 8). Warning: This solution convert different line breaks (like \r\n) to \n.

String result = new BufferedReader(new InputStreamReader(inputStream)).lines()
   .parallel().collect(Collectors.joining("\n"));
Using InputStreamReader and StringBuilder (JDK)

final int bufferSize = 1024;
final char[] buffer = new char[bufferSize];
final StringBuilder out = new StringBuilder();
Reader in = new InputStreamReader(inputStream, "UTF-8");
for (; ; ) {
    int rsz = in.read(buffer, 0, buffer.length);
    if (rsz < 0)
        break;
    out.append(buffer, 0, rsz);
}
return out.toString();
Using StringWriter and IOUtils.copy (Apache Commons)

StringWriter writer = new StringWriter();
IOUtils.copy(inputStream, writer, "UTF-8");
return writer.toString();
Using ByteArrayOutputStream and inputStream.read (JDK)

ByteArrayOutputStream result = new ByteArrayOutputStream();
byte[] buffer = new byte[1024];
int length;
while ((length = inputStream.read(buffer)) != -1) {
    result.write(buffer, 0, length);
}
// StandardCharsets.UTF_8.name() > JDK 7
return result.toString("UTF-8");
Using BufferedReader (JDK). Warning: This solution convert different line breaks (like \n\r) to line.separator system property (for example, in Windows to "\r\n").

String newLine = System.getProperty("line.separator");
BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
StringBuilder result = new StringBuilder();
String line; boolean flag = false;
while ((line = reader.readLine()) != null) {
    result.append(flag? newLine: "").append(line);
    flag = true;
}
return result.toString();
Using BufferedInputStream and ByteArrayOutputStream (JDK)

BufferedInputStream bis = new BufferedInputStream(inputStream);
ByteArrayOutputStream buf = new ByteArrayOutputStream();
int result = bis.read();
while(result != -1) {
    buf.write((byte) result);
    result = bis.read();
}
// StandardCharsets.UTF_8.name() > JDK 7
return buf.toString("UTF-8");
Using inputStream.read() and StringBuilder (JDK). Warning: This solution has problem with Unicode, for example with Russian text (work correctly only with non-Unicode text)

int ch;
StringBuilder sb = new StringBuilder();
while((ch = inputStream.read()) != -1)
    sb.append((char)ch);
reset();
return sb.toString();
Warning:

Solutions 4, 5 and 9 convert different line breaks to one.

Solution 11 can't work correctly with Unicode text

Performance tests

Performance tests for small String (length = 175), url in github (mode = Average Time, system = Linux, score 1,343 is the best):

              Benchmark                         Mode  Cnt   Score   Error  Units
 8. ByteArrayOutputStream and read (JDK)        avgt   10   1,343 ± 0,028  us/op
 6. InputStreamReader and StringBuilder (JDK)   avgt   10   6,980 ± 0,404  us/op
10. BufferedInputStream, ByteArrayOutputStream  avgt   10   7,437 ± 0,735  us/op
11. InputStream.read() and StringBuilder (JDK)  avgt   10   8,977 ± 0,328  us/op
 7. StringWriter and IOUtils.copy (Apache)      avgt   10  10,613 ± 0,599  us/op
 1. IOUtils.toString (Apache Utils)             avgt   10  10,605 ± 0,527  us/op
 3. Scanner (JDK)                               avgt   10  12,083 ± 0,293  us/op
 2. CharStreams (guava)                         avgt   10  12,999 ± 0,514  us/op
 4. Stream Api (Java 8)                         avgt   10  15,811 ± 0,605  us/op
 9. BufferedReader (JDK)                        avgt   10  16,038 ± 0,711  us/op
 5. parallel Stream Api (Java 8)                avgt   10  21,544 ± 0,583  us/op
Performance tests for big String (length = 50100), url in github (mode = Average Time, system = Linux, score 200,715 is the best):

               Benchmark                        Mode  Cnt   Score        Error  Units
 8. ByteArrayOutputStream and read (JDK)        avgt   10   200,715 ±   18,103  us/op
 1. IOUtils.toString (Apache Utils)             avgt   10   300,019 ±    8,751  us/op
 6. InputStreamReader and StringBuilder (JDK)   avgt   10   347,616 ±  130,348  us/op
 7. StringWriter and IOUtils.copy (Apache)      avgt   10   352,791 ±  105,337  us/op
 2. CharStreams (guava)                         avgt   10   420,137 ±   59,877  us/op
 9. BufferedReader (JDK)                        avgt   10   632,028 ±   17,002  us/op
 5. parallel Stream Api (Java 8)                avgt   10   662,999 ±   46,199  us/op
 4. Stream Api (Java 8)                         avgt   10   701,269 ±   82,296  us/op
10. BufferedInputStream, ByteArrayOutputStream  avgt   10   740,837 ±    5,613  us/op
 3. Scanner (JDK)                               avgt   10   751,417 ±   62,026  us/op
11. InputStream.read() and StringBuilder (JDK)  avgt   10  2919,350 ± 1101,942  us/op
Graphs (performance tests depending on Input Stream length in Windows 7 system)
enter image description here

Performance test (Average Time) depending on Input Stream length in Windows 7 system:

 length  182    546     1092    3276    9828    29484   58968

 test8  0.38    0.938   1.868   4.448   13.412  36.459  72.708
 test4  2.362   3.609   5.573   12.769  40.74   81.415  159.864
 test5  3.881   5.075   6.904   14.123  50.258  129.937 166.162
 test9  2.237   3.493   5.422   11.977  45.98   89.336  177.39
 test6  1.261   2.12    4.38    10.698  31.821  86.106  186.636
 test7  1.601   2.391   3.646   8.367   38.196  110.221 211.016
 test1  1.529   2.381   3.527   8.411   40.551  105.16  212.573
 test3  3.035   3.934   8.606   20.858  61.571  118.744 235.428
 test2  3.136   6.238   10.508  33.48   43.532  118.044 239.481
 test10 1.593   4.736   7.527   20.557  59.856  162.907 323.147
 test11 3.913   11.506  23.26   68.644  207.591 600.444 1211.545
shareedit
edited Jun 27 '17 at 21:50

Sean Bright
87.2k12109120
answered Feb 17 '16 at 0:58

Viacheslav Vedenin
27.6k123153
13
As you're writing the "summary answer", you should note that some solutions automatically convert different linebreaks (like \r\n) to \n which might be undesired in some cases. Also it would be nice to see the additional memory required or at least allocation pressure (at least you may run JMH with -prof gc). For the really cool post it would be great to see the graphs (depending on string length within the same input size and depending on the input size within the same string length). – Tagir Valeev Feb 17 '16 at 4:28 
11
Upvoted; the funniest thing is that results are more than expected: one should use standard JDK and/or Apache Commons syntactic sugar. – mudasobwa Apr 15 '16 at 9:17
14
Amazing post. Just one thing. Java 8 warns against using parallel streams on resources that will force you to lock and wait (such as this input stream) so the parallel stream option is rather cumbersome and not worth it no? – mangusbrother May 7 '16 at 20:20
4
Does the parallel stream actually maintain the line ordering? – Natix Sep 29 '16 at 18:43
3
What is reset() for in example 11 ? – Rob Stewart Apr 13 '17 at 22:12
show 10 more comments
up vote
789
down vote
Apache Commons allows:

String myString = IOUtils.toString(myInputStream, "UTF-8");
Of course, you could choose other character encodings besides UTF-8.

Also see: (Docs)

shareedit
edited Jan 14 '16 at 0:48

Daniel Alexiuc
9,10184669
answered Dec 8 '08 at 20:13

Chinnery
8,70421725
1
Also, there is a method that only take a inputStream argument, if you are find with the default encoding. – Guillaume Coté Feb 3 '11 at 16:07
12
@Guillaume Coté I guess the message here is that you never should be "fine with the default encoding", since you cannot be sure of what it is, depending on the platform the java code is run on. – Per Wiklander Feb 3 '11 at 21:54
6
@Per Wiklander I disagree with you. Code that is going to work on a single could be quite sure that default encoding will be fine. For code that only open local file, it is a reasonable option to ask them to be encoded in the platform default encoding. – Guillaume Coté Feb 4 '11 at 15:56
32
To save anyone the hassle of Googling - <dependency> <groupId>org.apache.commons</groupId> <artifactId>commons-io</artifactId> <version>1.3.2</version> </dependency> – Chris Mar 9 '12 at 12:04 
6
Also little improvement would be to use apache io (or other) constant for character encoding instead of using plain string literal - eg: IOUtils.toString(myInputStream, Charsets.UTF_8); – user1018711 Jan 13 '14 at 12:35
show 3 more comments
up vote
261
down vote
Taking into account file one should first get a java.io.Reader instance. This can then be read and added to a StringBuilder (we don't need StringBuffer if we are not accessing it in multiple threads, and StringBuilder is faster). The trick here is that we work in blocks, and as such don't need other buffering streams. The block size is parameterized for run-time performance optimization.

public static String slurp(final InputStream is, final int bufferSize) {
    final char[] buffer = new char[bufferSize];
    final StringBuilder out = new StringBuilder();
    try (Reader in = new InputStreamReader(is, "UTF-8")) {
        for (;;) {
            int rsz = in.read(buffer, 0, buffer.length);
            if (rsz < 0)
                break;
            out.append(buffer, 0, rsz);
        }
    }
    catch (UnsupportedEncodingException ex) {
        /* ... */
    }
    catch (IOException ex) {
        /* ... */
    }
    return out.toString();
}
shareedit
edited Jul 15 '15 at 10:23
community wiki
11 revs, 8 users 39%
Paul de Vrieze
8
This solution uses multibyte characters. The example uses the UTF-8 encoding that allows expression of the full unicode range (Including Chinese). Replacing "UTF-8" with another encoding would allow that encoding to be used. – Paul de Vrieze Dec 9 '11 at 23:11
27
@User1 - I like using libraries in my code so I can get my job done faster. It's awesome when your managers say "Wow James! How did you get that done so fast?!". But when we have to spend time reinventing the wheel just because we have misplaced ideas about including a common, reusable, tried and tested utility, we're giving up time we could be spending furthering our project's goals. When we reinvent the wheel, we work twice as hard yet get to the finish line much later. Once we're at the finish line, there is no one there to congratulate us. When building a house, don't build the hammer too – jmort253 Jan 20 '12 at 1:05 
10
Sorry, after re-reading my comment, it comes off a little arrogant. I just think it's important to have a good reason to avoid libraries and that the reason is a valid one, which there very well could be :) – jmort253 Jan 20 '12 at 1:35
4
@jmort253 We noticed performance regression after updating some library in our product for several times. Luckily we are building and selling our own product so we don't really have the so called deadlines. Unfortunately we are building a product that is available on many JVMs, databases and app servers on many operation systems so we have to think for the users using poor machines... And a string operation optimizing can improve the perf by 30~40%. And a fix: In our product, I even replaced should be 'we even replaced'. – coolcfan May 9 '12 at 8:39
9
@jmort253 If you would already use apache commons I would say, go for it. At the same time, there is a real cost to using libraries (as the dependency proliferation in many apache java libraries shows). If this would be the only use of the library, it would be overkill to use the library. On the other hand, determining your own buffer size(s) you can tune your memory/processor usage balance. – Paul de Vrieze May 22 '12 at 9:16
show 7 more comments
up vote
224
down vote
How about this?

InputStream in = /* your InputStream */;
StringBuilder sb=new StringBuilder();
BufferedReader br = new BufferedReader(new InputStreamReader(in));
String read;

while((read=br.readLine()) != null) {
    //System.out.println(read);
    sb.append(read);   
}

br.close();
return sb.toString();
shareedit
edited Jan 10 '16 at 3:58

hidro
8,81143644
answered Aug 4 '11 at 8:29

sampathpremarathna
3,21351733
8
The thing is, you're first splitting into lines, and then undoing that. It's easier and faster to just read arbitrary buffers. – Paul de Vrieze Apr 20 '12 at 18:36
17
Also, readLine does not distinguish between \n and \r, so you cannot reproduce the exact stream again. – María Arias de Reyna Domínguez Sep 10 '12 at 8:08
1
very inefficient, as readLine read character by character to look for EOL. Also, if there is no line break in the stream, this does not really make sense. – njzk2 Apr 18 '14 at 18:05
1
This isn't the best answer because it's not strictly byte in byte out. The reader chomps newlines, so you have to be careful to maintain them. – Jeffrey Blattman Jan 18 '16 at 19:16
show 3 more comments
up vote
151
down vote
If you are using Google-Collections/Guava you could do the following:

InputStream stream = ...
String content = CharStreams.toString(new InputStreamReader(stream, Charsets.UTF_8));
Closeables.closeQuietly(stream);
Note that the second parameter (i.e. Charsets.UTF_8) for the InputStreamReader isn't necessary, but it is generally a good idea to specify the encoding if you know it (which you should!)

shareedit
edited Jan 30 '13 at 16:35

ralfoide
84911119
answered Jul 13 '10 at 15:56

Sakuraba
2,17811313
2
@harschware: Given the question was: "If you have java.io.InputStream object how should you process that object and produce a String?" I assumed that a stream is already present in the situation. – Sakuraba Apr 13 '11 at 9:41 
1
+1 for guava, -1 for not specifying the encoding of the input stream. eg. new InputStreamReader(stream, "UTF-8") – andras Jul 6 '12 at 11:01
show 6 more comments
up vote
104
down vote
This is my pure Java & Android solution, works well...

public String readFullyAsString(InputStream inputStream, String encoding)
        throws IOException {
    return readFully(inputStream).toString(encoding);
}    

public byte[] readFullyAsBytes(InputStream inputStream)
        throws IOException {
    return readFully(inputStream).toByteArray();
}    

private ByteArrayOutputStream readFully(InputStream inputStream)
        throws IOException {
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    byte[] buffer = new byte[1024];
    int length = 0;
    while ((length = inputStream.read(buffer)) != -1) {
        baos.write(buffer, 0, length);
    }
    return baos;
}
shareedit
edited Mar 25 '16 at 7:54
answered May 8 '12 at 20:24

TacB0sS
5,42584893
2
Works well on Android in comparison with other answers which work only in enterprise java. – vorrtex Jan 14 '13 at 19:30
1
Quick note: The memory footprint of this is maxed by 2*n, where n is the size of the stream, as per the ByteArrayInputStream auto-growing system. – njzk2 Apr 18 '14 at 18:07
1
Unnecessarily doubles memory usage, that is precious on mobile devices. You'd better use InputStreamReader and append to StringReader, the byte to char conversion will be done on the fly, not in bulk at the end. – OlivJan 22 '15 at 8:33 
show 2 more comments
up vote
55
down vote
Here's the most elegant, pure-Java (no library) solution I came up with after some experimentation:

public static String fromStream(InputStream in) throws IOException
{
    BufferedReader reader = new BufferedReader(new InputStreamReader(in));
    StringBuilder out = new StringBuilder();
    String newLine = System.getProperty("line.separator");
    String line;
    while ((line = reader.readLine()) != null) {
        out.append(line);
        out.append(newLine);
    }
    return out.toString();
}
shareedit
edited Dec 7 '13 at 16:39
answered Jan 1 '13 at 3:43

Drew Noakes
173k103503589
4
Isn't there a reader.close() missing? Ideally with try/finally... – Torben Kohlmeier Jun 2 '13 at 13:50
6
@TorbenKohlmeier, readers and buffers don't need to be closed. The provided InputStream should be closed by the caller. – Drew Noakes Jun 3 '13 at 11:37
5
Don't forget to mention that there's a more preferable constructor in InputStreamReader that takes a CharSet. – jontejj Jun 27 '13 at 12:36
4
why do people keep using readLine? if you don't use the lines per se, what good is it (except being very slow?) – njzk2 Apr 18 '14 at 18:07
3
@voho, if one line is that long, then there's no way to allocate the return value anyway which must be equal or greater in size to that line. If you're dealing with files that large, you should stream them. There are plenty of use cases for loading small text files into memory though. – Drew Noakes Aug 7 '14 at 12:16
show 4 more comments
up vote
54
down vote
How about:

import java.io.BufferedInputStream;
import java.io.ByteArrayOutputStream;
import java.io.InputStream;
import java.io.IOException;    

public static String readInputStreamAsString(InputStream in) 
    throws IOException {

    BufferedInputStream bis = new BufferedInputStream(in);
    ByteArrayOutputStream buf = new ByteArrayOutputStream();
    int result = bis.read();
    while(result != -1) {
      byte b = (byte)result;
      buf.write(b);
      result = bis.read();
    }        
    return buf.toString();
}
shareedit
answered Jun 10 '09 at 21:07

Jon Moore
1,094911
2
This one is slow, because reads byte by byte. – Daniel De León May 8 '15 at 21:56
2
@EJP I've found it to be slower than using the BufferedInputStream and reading into a byte array buffer instead of one byte at a time. Example: 200ms vs 60ms when reading a 4.56 MiB file. – jk7 Mar 17 '17 at 23:13
show 1 more comment
up vote
40
down vote
For completeness here is Java 9 solution:

public static String toString(InputStream input) throws IOException {
    return new String(input.readAllBytes(), StandardCharsets.UTF_8);
}
The readAllBytes is currently in JDK 9 main codebase, so it likely to appear in the release. You can try it right now using the JDK 9 snapshot builds.

shareedit
answered Sep 2 '15 at 11:50

Tagir Valeev
62.9k10137223
2
@ChristianHujer, I don't see it in the latest jdk8u commit. AFAIK new methods are never introduced in Java updates, only in major releases. – Tagir Valeev Feb 1 '16 at 7:54
2
@ChristianHujer, the question was about InputStream, not about Path. The InputStream can be created from many different sources, not only files. – Tagir Valeev Feb 2 '16 at 12:02
1
This was written a year ago, so to update, I confirm that this method is indeed in the public release JDK 9. Furthermore, if your encoding is "ISO-Latin-1" then this will be extremely efficient since Java 9 Strings now use a byte[] implementation if all characters are in the first 256 code points. This means the new String(byte[], "ISO-Latin-1") will be a simple array copy. – Klitos Kyriacou Nov 17 '17 at 10:21
show 5 more comments
up vote
33
down vote
I'd use some Java 8 tricks.

public static String streamToString(final InputStream inputStream) throws Exception {
    // buffering optional
    try
    (
        final BufferedReader br
           = new BufferedReader(new InputStreamReader(inputStream))
    ) {
        // parallel optional
        return br.lines().parallel().collect(Collectors.joining("\n"));
    } catch (final IOException e) {
        throw new RuntimeException(e);
        // whatever.
    }
}
Essentially the same as some other answers except more succinct.

shareedit
edited Jul 15 '15 at 11:03

Ian2thedv
1,9231632
answered Jul 17 '14 at 17:58

Simon Kuang
2,17421747
5
Would that return null ever get called? Either the br.lines... returns or an exception is thrown. – Holloway Jul 23 '14 at 9:13
3
@Khaled A Khunaifer: yes, pretty sure... maybe you should have a look here: docs.oracle.com/javase/tutorial/essential/exceptions/…. What you wrongly edited is a "try-with-resources" statement. – jamp Feb 5 '15 at 13:13
10
Why do you call parallel() on the stream? – robinst Apr 20 '15 at 5:13
3
This would not result in an honest copy of the data if the source stream used windows line endings as all \r\n would end up getting converted into \n... – Lucas Aug 13 '15 at 18:30
1
You can use System.lineSeparator() to use the appropriate platform-dependent line-ending. – Steve KSep 28 '15 at 23:25
show 1 more comment
up vote
26
down vote
I ran some timing tests because time matters, always.

I attempted to get the response into a String 3 different ways. (shown below)
I left out try/catch blocks for the sake readability.

To give context, this is the preceding code for all 3 approaches:

   String response;
   String url = "www.blah.com/path?key=value";
   GetMethod method = new GetMethod(url);
   int status = client.executeMethod(method);
1)

 response = method.getResponseBodyAsString();
2)

InputStream resp = method.getResponseBodyAsStream();
InputStreamReader is=new InputStreamReader(resp);
BufferedReader br=new BufferedReader(is);
String read = null;
StringBuffer sb = new StringBuffer();
while((read = br.readLine()) != null) {
    sb.append(read);
}
response = sb.toString();
3)

InputStream iStream  = method.getResponseBodyAsStream();
StringWriter writer = new StringWriter();
IOUtils.copy(iStream, writer, "UTF-8");
response = writer.toString();
So, after running 500 tests on each approach with the same request/response data, here are the numbers. Once again, these are my findings and your findings may not be exactly the same, but I wrote this to give some indication to others of the efficiency differences of these approaches.

Ranks:
Approach #1
Approach #3 - 2.6% slower than #1
Approach #2 - 4.3% slower than #1

Any of these approaches is an appropriate solution for grabbing a response and creating a String from it.

shareedit
edited Oct 17 '17 at 8:55

martijnn2008
2,30331834
answered Oct 12 '11 at 17:23

Brett Holt
985914
2
2) contains an error, it adds always "null" at the end of the string as you are always makeing one more step then necessary. Performance will be the same anyway I think. This should work: String read = null; StringBuffer sb = new StringBuffer(); while((read = br.readLine()) != null) { sb.append(read); } – LukeSolar Oct 21 '11 at 13:32 
show 2 more comments
up vote
24
down vote
Pure Java solution using Streams, works since Java 8.

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.stream.Collectors;

// ...
public static String inputStreamToString(InputStream is) throws IOException {
    try (BufferedReader br = new BufferedReader(new InputStreamReader(is))) {
        return br.lines().collect(Collectors.joining(System.lineSeparator()));
    }
}
As mentioned by Christoffer Hammarström below other answer it is safer to explicitly specify the Charset. I.e. The InputStreamReader constructor can be changes as follows:

new InputStreamReader(is, Charset.forName("UTF-8"))
shareedit
edited May 23 '17 at 12:34

Community♦
11
answered Feb 26 '15 at 18:39

czerny
4,09433051
10
Instead of doing Charset.forName("UTF-8"), use StandardCharsets.UTF_8 (from java.nio.charset). – robinst Apr 20 '15 at 5:12
add a comment
up vote
22
down vote
I did a benchmark upon 14 distinct answers here(Sorry for not providing credits but there are too many duplicates)

The result is very surprising. It turns out that Apache IOUtils is the slowest and ByteArrayOutputStream is the fastest solutions:

So first here is the best method:

public String inputStreamToString(InputStream inputStream) throws IOException {
    try(ByteArrayOutputStream result = new ByteArrayOutputStream()) {
        byte[] buffer = new byte[1024];
        int length;
        while ((length = inputStream.read(buffer)) != -1) {
            result.write(buffer, 0, length);
        }

        return result.toString(UTF_8);
    }
}
Benchmark results, of 20MB random bytes in 20 cycles
Time in milliseconds

ByteArrayOutputStreamTest: 194
NioStream: 198
Java9ISTransferTo: 201
Java9ISReadAllBytes: 205
BufferedInputStreamVsByteArrayOutputStream: 314
ApacheStringWriter2: 574
GuavaCharStreams: 589
ScannerReaderNoNextTest: 614
ScannerReader: 633
ApacheStringWriter: 1544
StreamApi: Error
ParallelStreamApi: Error
BufferReaderTest: Error
InputStreamAndStringBuilder: Error
Benchmark source code
import com.google.common.io.CharStreams;
import org.apache.commons.io.IOUtils;

import java.io.*;
import java.nio.ByteBuffer;
import java.nio.channels.Channels;
import java.nio.channels.ReadableByteChannel;
import java.nio.channels.WritableByteChannel;
import java.util.Arrays;
import java.util.List;
import java.util.Random;
import java.util.stream.Collectors;

/**
 * Created by Ilya Gazman on 2/13/18.
 */
public class InputStreamToString {


    private static final String UTF_8 = "UTF-8";

    public static void main(String... args) {
        log("App started");
        byte[] bytes = new byte[1024 * 1024];
        new Random().nextBytes(bytes);
        log("Stream is ready\n");

        try {
            test(bytes);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void test(byte[] bytes) throws IOException {
        List<Stringify> tests = Arrays.asList(
                new ApacheStringWriter(),
                new ApacheStringWriter2(),
                new NioStream(),
                new ScannerReader(),
                new ScannerReaderNoNextTest(),
                new GuavaCharStreams(),
                new StreamApi(),
                new ParallelStreamApi(),
                new ByteArrayOutputStreamTest(),
                new BufferReaderTest(),
                new BufferedInputStreamVsByteArrayOutputStream(),
                new InputStreamAndStringBuilder(),
                new Java9ISTransferTo(),
                new Java9ISReadAllBytes()
        );

        String solution = new String(bytes, "UTF-8");

        for (Stringify test : tests) {
            try (ByteArrayInputStream inputStream = new ByteArrayInputStream(bytes)) {
                String s = test.inputStreamToString(inputStream);
                if (!s.equals(solution)) {
                    log(test.name() + ": Error");
                    continue;
                }
            }
            long startTime = System.currentTimeMillis();
            for (int i = 0; i < 20; i++) {
                try (ByteArrayInputStream inputStream = new ByteArrayInputStream(bytes)) {
                    test.inputStreamToString(inputStream);
                }
            }
            log(test.name() + ": " + (System.currentTimeMillis() - startTime));
        }
    }

    private static void log(String message) {
        System.out.println(message);
    }

    interface Stringify {
        String inputStreamToString(InputStream inputStream) throws IOException;

        default String name() {
            return this.getClass().getSimpleName();
        }
    }

    static class ApacheStringWriter implements Stringify {

        @Override
        public String inputStreamToString(InputStream inputStream) throws IOException {
            StringWriter writer = new StringWriter();
            IOUtils.copy(inputStream, writer, UTF_8);
            return writer.toString();
        }
    }

    static class ApacheStringWriter2 implements Stringify {

        @Override
        public String inputStreamToString(InputStream inputStream) throws IOException {
            return IOUtils.toString(inputStream, UTF_8);
        }
    }

    static class NioStream implements Stringify {

        @Override
        public String inputStreamToString(InputStream in) throws IOException {
            ReadableByteChannel channel = Channels.newChannel(in);
            ByteBuffer byteBuffer = ByteBuffer.allocate(1024 * 16);
            ByteArrayOutputStream bout = new ByteArrayOutputStream();
            WritableByteChannel outChannel = Channels.newChannel(bout);
            while (channel.read(byteBuffer) > 0 || byteBuffer.position() > 0) {
                byteBuffer.flip();  //make buffer ready for write
                outChannel.write(byteBuffer);
                byteBuffer.compact(); //make buffer ready for reading
            }
            channel.close();
            outChannel.close();
            return bout.toString(UTF_8);
        }
    }

    static class ScannerReader implements Stringify {

        @Override
        public String inputStreamToString(InputStream is) throws IOException {
            java.util.Scanner s = new java.util.Scanner(is).useDelimiter("\\A");
            return s.hasNext() ? s.next() : "";
        }
    }

    static class ScannerReaderNoNextTest implements Stringify {

        @Override
        public String inputStreamToString(InputStream is) throws IOException {
            java.util.Scanner s = new java.util.Scanner(is).useDelimiter("\\A");
            return s.next();
        }
    }

    static class GuavaCharStreams implements Stringify {

        @Override
        public String inputStreamToString(InputStream is) throws IOException {
            return CharStreams.toString(new InputStreamReader(
                    is, UTF_8));
        }
    }

    static class StreamApi implements Stringify {

        @Override
        public String inputStreamToString(InputStream inputStream) throws IOException {
            return new BufferedReader(new InputStreamReader(inputStream))
                    .lines().collect(Collectors.joining("\n"));
        }
    }

    static class ParallelStreamApi implements Stringify {

        @Override
        public String inputStreamToString(InputStream inputStream) throws IOException {
            return new BufferedReader(new InputStreamReader(inputStream)).lines()
                    .parallel().collect(Collectors.joining("\n"));
        }
    }

    static class ByteArrayOutputStreamTest implements Stringify {

        @Override
        public String inputStreamToString(InputStream inputStream) throws IOException {
            try(ByteArrayOutputStream result = new ByteArrayOutputStream()) {
                byte[] buffer = new byte[1024];
                int length;
                while ((length = inputStream.read(buffer)) != -1) {
                    result.write(buffer, 0, length);
                }

                return result.toString(UTF_8);
            }
        }
    }

    static class BufferReaderTest implements Stringify {

        @Override
        public String inputStreamToString(InputStream inputStream) throws IOException {
            String newLine = System.getProperty("line.separator");
            BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
            StringBuilder result = new StringBuilder(UTF_8);
            String line;
            boolean flag = false;
            while ((line = reader.readLine()) != null) {
                result.append(flag ? newLine : "").append(line);
                flag = true;
            }
            return result.toString();
        }
    }

    static class BufferedInputStreamVsByteArrayOutputStream implements Stringify {

        @Override
        public String inputStreamToString(InputStream inputStream) throws IOException {
            BufferedInputStream bis = new BufferedInputStream(inputStream);
            ByteArrayOutputStream buf = new ByteArrayOutputStream();
            int result = bis.read();
            while (result != -1) {
                buf.write((byte) result);
                result = bis.read();
            }

            return buf.toString(UTF_8);
        }
    }

    static class InputStreamAndStringBuilder implements Stringify {

        @Override
        public String inputStreamToString(InputStream inputStream) throws IOException {
            int ch;
            StringBuilder sb = new StringBuilder(UTF_8);
            while ((ch = inputStream.read()) != -1)
                sb.append((char) ch);
            return sb.toString();
        }
    }

    static class Java9ISTransferTo implements Stringify {

        @Override
        public String inputStreamToString(InputStream inputStream) throws IOException {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            inputStream.transferTo(bos);
            return bos.toString(UTF_8);
        }
    }

    static class Java9ISReadAllBytes implements Stringify {

        @Override
        public String inputStreamToString(InputStream inputStream) throws IOException {
            return new String(inputStream.readAllBytes(), UTF_8);
        }
    }

}
shareedit
answered Feb 13 at 21:30

Ilya Gazman
14.1k1280136
add a comment
up vote
20
down vote
Here's more-or-less sampath's answer, cleaned up a bit and represented as a function:

String streamToString(InputStream in) throws IOException {
  StringBuilder out = new StringBuilder();
  BufferedReader br = new BufferedReader(new InputStreamReader(in));
  for(String line = br.readLine(); line != null; line = br.readLine()) 
    out.append(line);
  br.close();
  return out.toString();
}
shareedit
edited Sep 12 '12 at 18:31
answered Mar 30 '12 at 19:52

TKH
599310
add a comment
up vote
18
down vote
If you were feeling adventurous, you could mix Scala and Java and end up with this:

scala.io.Source.fromInputStream(is).mkString("")
Mixing Java and Scala code and libraries has it's benefits.

See full description here: Idiomatic way to convert an InputStream to a String in Scala

shareedit
edited May 23 '17 at 12:18

Community♦
11
answered Mar 7 '12 at 7:32

Jack
9,214874144
1
Nowadays simply this works fine: Source.fromInputStream(...).mkString – KajMagnus Jul 30 '15 at 23:32 
add a comment
up vote
17
down vote
If you can't use Commons IO (FileUtils/IOUtils/CopyUtils) here's an example using a BufferedReader to read the file line by line:

public class StringFromFile {
    public static void main(String[] args) /*throws UnsupportedEncodingException*/ {
        InputStream is = StringFromFile.class.getResourceAsStream("file.txt");
        BufferedReader br = new BufferedReader(new InputStreamReader(is/*, "UTF-8"*/));
        final int CHARS_PER_PAGE = 5000; //counting spaces
        StringBuilder builder = new StringBuilder(CHARS_PER_PAGE);
        try {
            for(String line=br.readLine(); line!=null; line=br.readLine()) {
                builder.append(line);
                builder.append('\n');
            }
        } catch (IOException ignore) { }
        String text = builder.toString();
        System.out.println(text);
    }
}
or if you want raw speed I'd propose a variation on what Paul de Vrieze suggested (which avoids using a StringWriter (which uses a StringBuffer internally) :

public class StringFromFileFast {
    public static void main(String[] args) /*throws UnsupportedEncodingException*/ {
        InputStream is = StringFromFileFast.class.getResourceAsStream("file.txt");
        InputStreamReader input = new InputStreamReader(is/*, "UTF-8"*/);
        final int CHARS_PER_PAGE = 5000; //counting spaces
        final char[] buffer = new char[CHARS_PER_PAGE];
        StringBuilder output = new StringBuilder(CHARS_PER_PAGE);
        try {
            for(int read = input.read(buffer, 0, buffer.length);
                    read != -1;
                    read = input.read(buffer, 0, buffer.length)) {
                output.append(buffer, 0, read);
            }
        } catch (IOException ignore) { }

        String text = output.toString();
        System.out.println(text);
    }
}
shareedit
edited May 18 '10 at 13:05
answered May 18 '10 at 12:57

DJDaveMark
1,271920
show 1 more comment
up vote
16
down vote
This is an answer adapted from org.apache.commons.io.IOUtils source code, for those who want to have the apache implementation but do not want the whole library.

private static final int BUFFER_SIZE = 4 * 1024;

public static String inputStreamToString(InputStream inputStream, String charsetName)
        throws IOException {
    StringBuilder builder = new StringBuilder();
    InputStreamReader reader = new InputStreamReader(inputStream, charsetName);
    char[] buffer = new char[BUFFER_SIZE];
    int length;
    while ((length = reader.read(buffer)) != -1) {
        builder.append(buffer, 0, length);
    }
    return builder.toString();
}
shareedit
edited Oct 10 '15 at 4:37
answered Aug 3 '14 at 9:47

Dreaming in Code
2,3622236
add a comment
up vote
16
down vote
Make sure to close the streams at end if you use Stream Readers

private String readStream(InputStream iStream) throws IOException {
    //build a Stream Reader, it can read char by char
    InputStreamReader iStreamReader = new InputStreamReader(iStream);
    //build a buffered Reader, so that i can read whole line at once
    BufferedReader bReader = new BufferedReader(iStreamReader);
    String line = null;
    StringBuilder builder = new StringBuilder();
    while((line = bReader.readLine()) != null) {  //Read till end
        builder.append(line);
        builder.append("\n"); // append new line to preserve lines
    }
    bReader.close();         //close all opened stuff
    iStreamReader.close();
    //iStream.close(); //EDIT: Let the creator of the stream close it!
                       // some readers may auto close the inner stream
    return builder.toString();
}
EDIT: On JDK 7+, you can use try-with-resources construct.

/**
 * Reads the stream into a string
 * @param iStream the input stream
 * @return the string read from the stream
 * @throws IOException when an IO error occurs
 */
private String readStream(InputStream iStream) throws IOException {

    //Buffered reader allows us to read line by line
    try (BufferedReader bReader =
                 new BufferedReader(new InputStreamReader(iStream))){
        StringBuilder builder = new StringBuilder();
        String line;
        while((line = bReader.readLine()) != null) {  //Read till end
            builder.append(line);
            builder.append("\n"); // append new line to preserve lines
        }
        return builder.toString();
    }
}
shareedit
edited Feb 24 '17 at 19:28
answered Nov 17 '12 at 12:39

Thamme Gowda
5,41712736
1
You're right about closing streams, however, the responsibility for closing streams is usually with the stream constructor (finish what you start). So, iStream should really rather be closed by the caller because the caller created iStream. Besides, closing streams should be done in a finally block, or even better in a Java 7 try-with-resources statement. In your code, when readLine() throws IOException, or builder.append() throws OutOfMemoryError, the streams would stay open. – Christian Hujer Jan 31 '16 at 22:01
add a comment
up vote
14
down vote
Here is the complete method for converting InputStream into String without using any third party library. Use StringBuilder for single threaded environment otherwise use StringBuffer.

public static String getString( InputStream is) throws IOException {
    int ch;
    StringBuilder sb = new StringBuilder();
    while((ch = is.read()) != -1)
        sb.append((char)ch);
    return sb.toString();
}
shareedit
edited Dec 16 '15 at 9:20

rtruszk
3,525132847
answered Apr 9 '14 at 10:37

laksys
2,19831528
2
In this method there is no encoding applied. So let's say the data received from the InputStream is encoded using UTF-8 the output will be wrong. To fix this you could use in = new InputStreamReader(inputStream) and (char)in.read(). – Frederic Leitenberger Nov 4 '14 at 12:21
2
and memory-inefficient as well; I believe I tried using this before on a large input and StringBuilder ran out of memory – gengkev Nov 18 '14 at 3:37
1
There is another similar answer which uses a char[] buffer and is more efficient and takes care of charset. – Guillaume Perrot Apr 27 '15 at 21:39 
show 1 more comment
up vote
13
down vote
Here's how to do it using just the JDK using byte array buffers. This is actually how the commons-io IOUtils.copy() methods all work. You can replace byte[] with char[] if you're copying from a Reader instead of an InputStream.

import java.io.ByteArrayOutputStream;
import java.io.InputStream;

...

InputStream is = ....
ByteArrayOutputStream baos = new ByteArrayOutputStream(8192);
byte[] buffer = new byte[8192];
int count = 0;
try {
  while ((count = is.read(buffer)) != -1) {
    baos.write(buffer, 0, count);
  }
}
finally {
  try {
    is.close();
  }
  catch (Exception ignore) {
  }
}

String charset = "UTF-8";
String inputStreamAsString = baos.toString(charset);
shareedit
edited Aug 13 '14 at 4:30

Matt
52229
answered Nov 2 '12 at 12:37

Matt Shannon
13912
1
Please give a description on what you are trying to accomplish. – Ragunath Jawahar Nov 2 '12 at 12:58
add a comment
up vote
11
down vote
Kotlin users simply do:

println(InputStreamReader(is).readText())
whereas

readText()
is Kotlin standard library’s built-in extension method.

shareedit
answered Feb 4 '15 at 1:12

Alex
3,92083141
add a comment
up vote
10
down vote
Another one, for all the Spring users:

import java.nio.charset.StandardCharsets;
import org.springframework.util.FileCopyUtils;

public String convertStreamToString(InputStream is) throws IOException { 
    return new String(FileCopyUtils.copyToByteArray(is), StandardCharsets.UTF_8);
}
The utility methods in org.springframework.util.StreamUtils are similar to the ones in FileCopyUtils, but they leave the stream open when done.

shareedit
edited Jul 21 '17 at 10:24
answered Jul 29 '16 at 20:58

James
6,54323060
add a comment
up vote
10
down vote
Use the java.io.InputStream.transferTo(OutputStream) supported in Java 9 and the ByteArrayOutputStream.toString(String) which takes the charset name:

public static String gobble(InputStream in, String charsetName) throws IOException {
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
    in.transferTo(bos);
    return bos.toString(charsetName);
}
shareedit
edited Nov 28 '17 at 14:47
answered Jan 28 '16 at 15:55

jmehrens
6,22811832
show 3 more comments
up vote
9
down vote
This one is nice because:

Hand safety the Charset.
You control the read buffer size.
You can provision the length of the builder and can be not exactly.
Is free from library dependencies.
Is for Java 7 or higher.
What the for?

public static String convertStreamToString(InputStream is) {
   if (is == null) return null;
   StringBuilder sb = new StringBuilder(2048); // Define a size if you have an idea of it.
   char[] read = new char[128]; // Your buffer size.
   try (InputStreamReader ir = new InputStreamReader(is, StandardCharsets.UTF_8)) {
     for (int i; -1 != (i = ir.read(read)); sb.append(read, 0, i));
   } catch (Throwable t) {}
   return sb.toString();
}
shareedit
edited Apr 20 '15 at 18:17
answered Jun 8 '14 at 7:46

Daniel De León
8,48545650
show 1 more comment
up vote
6
down vote
Here's my Java 8 based solution, which uses the new Stream API to collect all lines from an InputStream:

public static String toString(InputStream inputStream) {
    BufferedReader reader = new BufferedReader(
        new InputStreamReader(inputStream));
    return reader.lines().collect(Collectors.joining(
        System.getProperty("line.separator")));
}
shareedit
answered Sep 2 '15 at 11:19

Christian Rädel
2911412
1
Seems that you did not actually read all the answers posted before. Stream API version was already here at least two times. – Tagir Valeev Sep 2 '15 at 11:55
1
You're not reading the original file, you're converting whatever line endings the file has to whatever line endings the OS has, possibly changing file contents. – Christian Hujer Jan 31 '16 at 22:07
show 1 more comment
up vote
6
down vote
The easiest way in JDK is with the following code snipplets.

String convertToString(InputStream in){
    String resource = new Scanner(in).useDelimiter("\\Z").next();
    return resource;
}
shareedit
answered Aug 9 '16 at 20:18

Raghu K Nair
2,2061927
add a comment
up vote
4
down vote
Guava provides much shorter efficient autoclosing solution in case when input stream comes from classpath resource (which seems to be popular task):

byte[] bytes = Resources.toByteArray(classLoader.getResource(path));
or

String text = Resources.toString(classLoader.getResource(path), StandardCharsets.UTF_8);
There is also general concept of ByteSource and CharSource that gently take care of both opening and closing the stream.

So, for example, instead of explicitly opening a small file to read its contents:

String content = Files.asCharSource(new File("robots.txt"), StandardCharsets.UTF_8).read();
byte[] data = Files.asByteSource(new File("favicon.ico")).read();
or just

String content = Files.toString(new File("robots.txt"), StandardCharsets.UTF_8);
byte[] data = Files.toByteArray(new File("favicon.ico"));
shareedit
edited Jul 15 '15 at 10:50

Ian2thedv
1,9231632
answered Jul 9 '15 at 13:20

Vadzim
13.2k469105
show 1 more comment
up vote
4
down vote
Raghu K Nair Was the only one using a scanner. The code I use is a little different:

String convertToString(InputStream in){
    Scanner scanner = new Scanner(in)
    scanner.useDelimiter("\\A");

    boolean hasInput = scanner.hasNext();
    if (hasInput) {
        return scanner.next();
    } else {
        return null;
    }

}
About Delimiter


# OkHTTP

OkHttp
Download v3.10.0GitHubSquare
An HTTP & HTTP/2 client for Android and Java applications
Overview
HTTP is the way modern applications network. It’s how we exchange data & media. Doing HTTP efficiently makes your stuff load faster and saves bandwidth.

OkHttp is an HTTP client that’s efficient by default:

HTTP/2 support allows all requests to the same host to share a socket.
Connection pooling reduces request latency (if HTTP/2 isn’t available).
Transparent GZIP shrinks download sizes.
Response caching avoids the network completely for repeat requests.
OkHttp perseveres when the network is troublesome: it will silently recover from common connection problems. If your service has multiple IP addresses OkHttp will attempt alternate addresses if the first connect fails. This is necessary for IPv4+IPv6 and for services hosted in redundant data centers. OkHttp initiates new connections with modern TLS features (SNI, ALPN), and falls back to TLS 1.0 if the handshake fails.

Using OkHttp is easy. Its request/response API is designed with fluent builders and immutability. It supports both synchronous blocking calls and async calls with callbacks.

OkHttp supports Android 2.3 and above. For Java, the minimum requirement is 1.7.

Examples
GET A URL
This program downloads a URL and print its contents as a string. Full source.

OkHttpClient client = new OkHttpClient();

String run(String url) throws IOException {
  Request request = new Request.Builder()
      .url(url)
      .build();

  Response response = client.newCall(request).execute();
  return response.body().string();
}
POST TO A SERVER
This program posts data to a service. Full source.

public static final MediaType JSON
    = MediaType.parse("application/json; charset=utf-8");

OkHttpClient client = new OkHttpClient();

String post(String url, String json) throws IOException {
  RequestBody body = RequestBody.create(JSON, json);
  Request request = new Request.Builder()
      .url(url)
      .post(body)
      .build();
  Response response = client.newCall(request).execute();
  return response.body().string();
}
Download
↓ v3.10.0 JAR

You'll also need Okio, which OkHttp uses for fast I/O and resizable buffers. Download the latest JAR.

The source code to OkHttp, its samples, and this website is available on GitHub.

MAVEN
<dependency>
  <groupId>com.squareup.okhttp3</groupId>
  <artifactId>okhttp</artifactId>
  <version>3.10.0</version>
</dependency>
GRADLE
compile 'com.squareup.okhttp3:okhttp:3.10.0'
Contributing
If you would like to contribute code you can do so through GitHub by forking the repository and sending a pull request.

When submitting code, please make every effort to follow existing conventions and style in order to keep the code as readable as possible. Please also make sure your code compiles by running mvn clean verify.

Before your code can be accepted into the project you must also sign the Individual Contributor License Agreement (CLA).

License
Copyright 2016 Square, Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
