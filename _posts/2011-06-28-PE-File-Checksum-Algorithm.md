---  
layout: post
title: PE File Checksum Algorithm    
---
这两天闲来无事，在看PE文件相关的文章，我们知道PE文件是有校验和算法的，而且像驱动，或者系统DLL之类的特殊执行文件，对校验和的要求是极其高的，如果没算对，就认为是非法文件。微软SDK里提供一个API，叫MapFileAndCheckSum，这个可以计算出一个PE文件的校验和，该API的相关分析，可以参考这篇文章[《An Analysis of the Windows PE Checksum Algorithm》](http://www.codeproject.com/KB/cpp/PEChecksum.aspx)。关于校验和算法，还有一个比较有意思的算法是IP协议的校验和算法，可以看[RFC 1071](http://datatracker.ietf.org/doc/rfc1071/?include_text=1)。还有WIKI中的这篇文章[《Error detection and correction》](http://en.wikipedia.org/wiki/Redundancy_check#Error_detection_schemes)。  
我感觉这个应用层实现的PE CHECK SUM算法太过恶心了..就看了下系统内核PE加载器的内核实现，发现校验和算法部分和应用层的完全不同，比应用层的简洁多了，直接上代码:  
<div id="gist-3747774" class="gist">
    

        <div class="gist-file">
          <div class="gist-data gist-syntax">
              <div class="gist-highlight"><pre><div class="line" id="LC1"><span class="kt">unsigned</span> <span class="kt">short</span> <span class="nf">ChkSum</span><span class="p">(</span><span class="kt">unsigned</span> <span class="kt">int</span> <span class="n">CheckSum</span><span class="p">,</span> <span class="kt">void</span> <span class="o">*</span><span class="n">FileBase</span><span class="p">,</span> <span class="kt">int</span> <span class="n">Length</span><span class="p">)</span>  </div><div class="line" id="LC2"><span class="p">{</span>  </div><div class="line" id="LC3"><br></div><div class="line" id="LC4">&nbsp;&nbsp;&nbsp;&nbsp;<span class="kt">int</span> <span class="o">*</span><span class="n">Data</span><span class="p">;</span></div><div class="line" id="LC5">&nbsp;&nbsp;&nbsp;&nbsp;<span class="kt">int</span> <span class="n">sum</span><span class="p">;</span></div><div class="line" id="LC6"><br></div><div class="line" id="LC7">&nbsp;&nbsp;&nbsp;&nbsp;<span class="k">if</span> <span class="p">(</span> <span class="n">Length</span> <span class="o">&amp;&amp;</span> <span class="n">FileBase</span> <span class="o">!=</span> <span class="nb">NULL</span><span class="p">)</span></div><div class="line" id="LC8">&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">{</span></div><div class="line" id="LC9">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">Data</span> <span class="o">=</span> <span class="p">(</span><span class="kt">int</span> <span class="o">*</span><span class="p">)</span><span class="n">FileBase</span><span class="p">;</span></div><div class="line" id="LC10">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="k">do</span></div><div class="line" id="LC11">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">{</span></div><div class="line" id="LC12">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">sum</span> <span class="o">=</span> <span class="o">*</span><span class="p">(</span><span class="kt">unsigned</span> <span class="kt">short</span> <span class="o">*</span><span class="p">)</span><span class="n">Data</span> <span class="o">+</span> <span class="n">CheckSum</span><span class="p">;</span></div><div class="line" id="LC13">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">Data</span> <span class="o">=</span> <span class="p">(</span><span class="kt">int</span> <span class="o">*</span><span class="p">)((</span><span class="kt">char</span> <span class="o">*</span><span class="p">)</span><span class="n">Data</span> <span class="o">+</span> <span class="mi">2</span><span class="p">);</span></div><div class="line" id="LC14">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">CheckSum</span> <span class="o">=</span> <span class="p">(</span><span class="kt">unsigned</span> <span class="kt">short</span><span class="p">)</span><span class="n">sum</span> <span class="o">+</span> <span class="p">(</span><span class="n">sum</span> <span class="o">&gt;&gt;</span> <span class="mi">16</span><span class="p">);</span></div><div class="line" id="LC15">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">}</span></div><div class="line" id="LC16">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="k">while</span> <span class="p">(</span> <span class="o">--</span><span class="n">Length</span> <span class="p">);</span></div><div class="line" id="LC17">&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">}</span></div><div class="line" id="LC18"><br></div><div class="line" id="LC19">&nbsp;&nbsp;&nbsp;&nbsp;<span class="k">return</span> <span class="n">CheckSum</span> <span class="o">+</span> <span class="p">(</span><span class="n">CheckSum</span> <span class="o">&gt;&gt;</span> <span class="mi">16</span><span class="p">);</span></div><div class="line" id="LC20"><span class="p">}</span></div><div class="line" id="LC21"><br></div><div class="line" id="LC22"><span class="kt">unsigned</span> <span class="kt">int</span> <span class="nf">PECheckSum</span><span class="p">(</span><span class="kt">void</span> <span class="o">*</span><span class="n">FileBase</span><span class="p">,</span> <span class="kt">unsigned</span> <span class="kt">int</span> <span class="n">FileSize</span><span class="p">)</span>  </div><div class="line" id="LC23"><span class="p">{</span>  </div><div class="line" id="LC24"><br></div><div class="line" id="LC25">&nbsp;&nbsp;&nbsp;&nbsp;<span class="kt">void</span> <span class="o">*</span><span class="n">RemainData</span><span class="p">;</span></div><div class="line" id="LC26">&nbsp;&nbsp;&nbsp;&nbsp;<span class="kt">int</span> <span class="n">RemainDataSize</span><span class="p">;</span></div><div class="line" id="LC27">&nbsp;&nbsp;&nbsp;&nbsp;<span class="kt">unsigned</span> <span class="kt">int</span> <span class="n">PeHeaderSize</span><span class="p">;</span></div><div class="line" id="LC28">&nbsp;&nbsp;&nbsp;&nbsp;<span class="kt">unsigned</span> <span class="kt">int</span> <span class="n">HeaderCheckSum</span><span class="p">;</span></div><div class="line" id="LC29">&nbsp;&nbsp;&nbsp;&nbsp;<span class="kt">unsigned</span> <span class="kt">int</span> <span class="n">PeHeaderCheckSum</span><span class="p">;</span></div><div class="line" id="LC30">&nbsp;&nbsp;&nbsp;&nbsp;<span class="kt">unsigned</span> <span class="kt">int</span> <span class="n">FileCheckSum</span><span class="p">;</span></div><div class="line" id="LC31">&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">PIMAGE_NT_HEADERS</span> <span class="n">NtHeaders</span><span class="p">;</span></div><div class="line" id="LC32"><br></div><div class="line" id="LC33">&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">NtHeaders</span> <span class="o">=</span> <span class="n">ImageNtHeader</span><span class="p">(</span><span class="n">FileBase</span><span class="p">);</span></div><div class="line" id="LC34">&nbsp;&nbsp;&nbsp;&nbsp;<span class="k">if</span> <span class="p">(</span> <span class="n">NtHeaders</span> <span class="p">)</span></div><div class="line" id="LC35">&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">{</span></div><div class="line" id="LC36">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">HeaderCheckSum</span> <span class="o">=</span> <span class="n">NtHeaders</span><span class="o">-&gt;</span><span class="n">OptionalHeader</span><span class="p">.</span><span class="n">CheckSum</span><span class="p">;</span></div><div class="line" id="LC37">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">PeHeaderSize</span> <span class="o">=</span> <span class="p">(</span><span class="kt">unsigned</span> <span class="kt">int</span><span class="p">)</span><span class="n">NtHeaders</span> <span class="o">-</span> <span class="p">(</span><span class="kt">unsigned</span> <span class="kt">int</span><span class="p">)</span><span class="n">FileBase</span> <span class="o">+</span></div><div class="line" id="LC38">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">((</span><span class="kt">unsigned</span> <span class="kt">int</span><span class="p">)</span><span class="o">&amp;</span><span class="n">NtHeaders</span><span class="o">-&gt;</span><span class="n">OptionalHeader</span><span class="p">.</span><span class="n">CheckSum</span> <span class="o">-</span> <span class="p">(</span><span class="kt">unsigned</span> <span class="kt">int</span><span class="p">)</span><span class="n">NtHeaders</span><span class="p">);</span></div><div class="line" id="LC39">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">RemainDataSize</span> <span class="o">=</span> <span class="p">(</span><span class="n">FileSize</span> <span class="o">-</span> <span class="n">PeHeaderSize</span> <span class="o">-</span> <span class="mi">4</span><span class="p">)</span> <span class="o">&gt;&gt;</span> <span class="mi">1</span><span class="p">;</span></div><div class="line" id="LC40">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">RemainData</span> <span class="o">=</span> <span class="o">&amp;</span><span class="n">NtHeaders</span><span class="o">-&gt;</span><span class="n">OptionalHeader</span><span class="p">.</span><span class="n">Subsystem</span><span class="p">;</span></div><div class="line" id="LC41">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">PeHeaderCheckSum</span> <span class="o">=</span> <span class="n">ChkSum</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="n">FileBase</span><span class="p">,</span> <span class="n">PeHeaderSize</span> <span class="o">&gt;&gt;</span> <span class="mi">1</span><span class="p">);</span></div><div class="line" id="LC42">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">FileCheckSum</span> <span class="o">=</span> <span class="n">ChkSum</span><span class="p">(</span><span class="n">PeHeaderCheckSum</span><span class="p">,</span><span class="n">RemainData</span><span class="p">,</span> <span class="n">RemainDataSize</span><span class="p">);</span></div><div class="line" id="LC43"><br></div><div class="line" id="LC44">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="k">if</span> <span class="p">(</span> <span class="n">FileSize</span> <span class="o">&amp;</span> <span class="mi">1</span> <span class="p">)</span></div><div class="line" id="LC45">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">{</span></div><div class="line" id="LC46">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">FileCheckSum</span> <span class="o">+=</span> <span class="p">(</span><span class="kt">unsigned</span> <span class="kt">short</span><span class="p">)</span><span class="o">*</span><span class="p">((</span><span class="kt">char</span> <span class="o">*</span><span class="p">)</span><span class="n">FileBase</span> <span class="o">+</span> <span class="n">FileSize</span> <span class="o">-</span> <span class="mi">1</span><span class="p">);</span></div><div class="line" id="LC47">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">}</span></div><div class="line" id="LC48">&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">}</span></div><div class="line" id="LC49">&nbsp;&nbsp;&nbsp;&nbsp;<span class="k">else</span></div><div class="line" id="LC50">&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">{</span></div><div class="line" id="LC51">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">FileCheckSum</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span></div><div class="line" id="LC52">&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">}</span></div><div class="line" id="LC53"><br></div><div class="line" id="LC54">&nbsp;&nbsp;&nbsp;&nbsp;<span class="k">return</span> <span class="p">(</span><span class="n">FileSize</span> <span class="o">+</span> <span class="n">FileCheckSum</span><span class="p">);</span></div><div class="line" id="LC55"><span class="p">}</span></div><div class="line" id="LC56"><br></div><div class="line" id="LC57"><span class="kt">void</span> <span class="nf">TestPEFile</span><span class="p">(</span><span class="kt">void</span><span class="p">)</span>  </div><div class="line" id="LC58"><span class="p">{</span></div><div class="line" id="LC59"><br></div><div class="line" id="LC60">&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">TCHAR</span> <span class="o">*</span><span class="n">FileName</span> <span class="o">=</span> <span class="n">_T</span><span class="p">(</span><span class="s">"C:</span><span class="se">\\</span><span class="s">windows</span><span class="se">\\</span><span class="s">system32</span><span class="se">\\</span><span class="s">drivers</span><span class="se">\\</span><span class="s">wimmount.sys"</span><span class="p">);</span></div><div class="line" id="LC61"><br></div><div class="line" id="LC62">&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">HANDLE</span> <span class="n">FileHandle</span> <span class="o">=</span> <span class="n">CreateFile</span><span class="p">(</span><span class="n">FileName</span><span class="p">,</span></div><div class="line" id="LC63">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">GENERIC_READ</span><span class="p">,</span></div><div class="line" id="LC64">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">FILE_SHARE_READ</span> <span class="o">|</span> <span class="n">FILE_SHARE_WRITE</span><span class="p">,</span></div><div class="line" id="LC65">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="nb">NULL</span><span class="p">,</span></div><div class="line" id="LC66">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">OPEN_EXISTING</span><span class="p">,</span></div><div class="line" id="LC67">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="mi">0</span><span class="p">,</span></div><div class="line" id="LC68">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="nb">NULL</span><span class="p">);</span></div><div class="line" id="LC69">&nbsp;&nbsp;&nbsp;&nbsp;<span class="k">if</span> <span class="p">(</span><span class="n">FileHandle</span> <span class="o">!=</span> <span class="n">INVALID_HANDLE_VALUE</span><span class="p">)</span></div><div class="line" id="LC70">&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">{</span></div><div class="line" id="LC71">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">HANDLE</span> <span class="n">FileMapHandle</span> <span class="o">=</span> <span class="n">CreateFileMapping</span><span class="p">(</span><span class="n">FileHandle</span><span class="p">,</span></div><div class="line" id="LC72">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="nb">NULL</span><span class="p">,</span></div><div class="line" id="LC73">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">PAGE_READONLY</span><span class="p">,</span></div><div class="line" id="LC74">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="mi">0</span><span class="p">,</span><span class="mi">0</span><span class="p">,</span><span class="nb">NULL</span><span class="p">);</span></div><div class="line" id="LC75">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="k">if</span> <span class="p">(</span><span class="n">FileMapHandle</span> <span class="o">!=</span> <span class="nb">NULL</span><span class="p">)</span></div><div class="line" id="LC76">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">{</span></div><div class="line" id="LC77">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="kt">unsigned</span> <span class="kt">short</span> <span class="o">*</span><span class="n">FileBase</span> <span class="o">=</span> <span class="p">(</span><span class="kt">unsigned</span> <span class="kt">short</span> <span class="o">*</span><span class="p">)</span><span class="n">MapViewOfFile</span><span class="p">(</span><span class="n">FileMapHandle</span><span class="p">,</span><span class="n">FILE_MAP_READ</span><span class="p">,</span><span class="mi">0</span><span class="p">,</span><span class="mi">0</span><span class="p">,</span><span class="mi">0</span><span class="p">);</span></div><div class="line" id="LC78">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="k">if</span> <span class="p">(</span><span class="n">FileBase</span> <span class="o">!=</span> <span class="nb">NULL</span><span class="p">)</span></div><div class="line" id="LC79">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">{</span></div><div class="line" id="LC80">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">DWORD</span> <span class="n">FileSize</span> <span class="o">=</span> <span class="n">GetFileSize</span><span class="p">(</span><span class="n">FileHandle</span><span class="p">,</span><span class="nb">NULL</span><span class="p">);</span></div><div class="line" id="LC81"><br></div><div class="line" id="LC82">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">DWORD</span> <span class="n">HeaderCheckSum</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span></div><div class="line" id="LC83">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">DWORD</span> <span class="n">CheckSum</span> <span class="o">=</span> <span class="mi">0</span><span class="p">;</span></div><div class="line" id="LC84">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">MapFileAndCheckSum</span><span class="p">(</span><span class="n">FileName</span><span class="p">,</span><span class="o">&amp;</span><span class="n">HeaderCheckSum</span><span class="p">,</span><span class="o">&amp;</span><span class="n">CheckSum</span><span class="p">);</span></div><div class="line" id="LC85"><br></div><div class="line" id="LC86">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">DWORD</span> <span class="n">sum</span> <span class="o">=</span> <span class="n">PECheckSum</span><span class="p">(</span><span class="n">FileBase</span><span class="p">,</span><span class="n">FileSize</span><span class="p">);</span></div><div class="line" id="LC87">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="k">if</span> <span class="p">(</span><span class="n">sum</span> <span class="o">==</span> <span class="n">HeaderCheckSum</span><span class="p">)</span></div><div class="line" id="LC88">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">{</span></div><div class="line" id="LC89">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">printf</span><span class="p">(</span><span class="s">"Valid Pe File</span><span class="se">\n</span><span class="s">"</span><span class="p">);</span></div><div class="line" id="LC90">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">}</span></div><div class="line" id="LC91">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="k">else</span></div><div class="line" id="LC92">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">{</span></div><div class="line" id="LC93">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">printf</span><span class="p">(</span><span class="s">"Invalid Pe File</span><span class="se">\n</span><span class="s">"</span><span class="p">);</span></div><div class="line" id="LC94">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">}</span></div><div class="line" id="LC95"><br></div><div class="line" id="LC96">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">UnmapViewOfFile</span><span class="p">(</span><span class="n">FileBase</span><span class="p">);</span></div><div class="line" id="LC97">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">}</span></div><div class="line" id="LC98"><br></div><div class="line" id="LC99">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">CloseHandle</span><span class="p">(</span><span class="n">FileMapHandle</span><span class="p">);</span></div><div class="line" id="LC100">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">}</span></div><div class="line" id="LC101"><br></div><div class="line" id="LC102">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="n">CloseHandle</span><span class="p">(</span><span class="n">FileHandle</span><span class="p">);</span></div><div class="line" id="LC103">&nbsp;&nbsp;&nbsp;&nbsp;<span class="p">}</span></div><div class="line" id="LC104"><br></div><div class="line" id="LC105">&nbsp;&nbsp;&nbsp;&nbsp;<span class="k">return</span><span class="p">;</span></div><div class="line" id="LC106"><span class="p">}</span>  </div></pre></div>
          </div>

          <div class="gist-meta">
            <a href="https://gist.github.com/raw/3747774/1dcb9b28eaaf1e5ff935a468da217204bd0df2b3/pe_checksum.c" style="float:right;">view raw</a>
            <a href="https://gist.github.com/3747774#file_pe_checksum.c" style="float:right;margin-right:10px;color:#666">pe_checksum.c</a>
            <a href="https://gist.github.com/3747774">This Gist</a> brought to you by <a href="http://github.com">GitHub</a>.
          </div>
        </div>
</div>