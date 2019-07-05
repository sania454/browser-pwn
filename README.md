Browser-Pwn
===

The world of Browsers is dominated by 4 major players:
*   Chromium/Chrome (Blink-Engine)
*   Firefox (Gecko-Engine)
*   Safari (WebKit-Engine)
*   Edge (Blink-Engine (former EdgeHTML-Engine)

The following is split into two parts:
-  Information that helps to understand their architecture and implementation and how to build them from sources
-  Information that helps finding their calculator popping feature


#  Table of Contents

1.  Engines
      *  [Overview](#engine-overview)
      *  [Chromium](#chromium-blink)
      *  [Firefox](#firefox-gecko)
      *  [Safari](#safari-webkit)
      *  [Edge](#edge-blinkedgehtml)
2. Exploitation
      *  [Overview](#exploitation-overview)
      *  [Chromium](#chromium-pwn)
      *  [Firefox](#firefox-pwn)
      *  [Safari](#safari-pwn)
      *  [Edge](#edge-pwn)
3. [Tools](#tools)
4. [JavaScript Docs](#javascript-ecmascript-docs)
      




# Engines

## Engine-Overview
* [Javascript Engine Fundamentals: the good, the bad, and the ugly](https://slidr.io/bmeurer/javascript-engine-fundamentals-the-good-the-bad-and-the-ugly)
* [Javascript Engine Fundamentals: Shapes and Inline Caches](https://mathiasbynens.be/notes/shapes-ics)
* [JavaScript Engines - how do they even (Video)](https://www.youtube.com/watch?v=p-iiEDtpy6I)

### Browse the Sources
Of course you can use you're own favorite setup to browse the sources.
However, those repos are relatively large and I tried a couple different setups until I found something that worked for me.
So if you don't have good setup already, here are a couple of my experiences that might help you:
*    [CTags](https://chromium.googlesource.com/chromium/src/+/master/docs/linux_eclipse_dev.md) (+Vim): Works well with following references and calls. If you're used to navigate through large source-trees with this puristic setup, it can be a good option for you.
The downside being of course the lack of the features most of the big IDEs come with nowadays.
*    [CLion](https://www.jetbrains.com/clion/): I use JetBrain products for a lot of my coding activities, but CLion didn't work well for me, especially following references. Of course this might be due to setup issues.
*    [Eclipse](https://www.eclipse.org/): I haven't used it in a while, but this turned out to be a good option. Unfortunately, it takes a lot of resources for the indexer to run through the code.
     * [Here](https://chromium.googlesource.com/chromium/src/+/master/docs/linux_eclipse_dev.md) is a setup description for the Chromium-Project, but it works similarly for the other projects as well.
*    [ccls](https://github.com/MaskRay/ccls)+[VSCode](https://code.visualstudio.com/) This is the best option for me so far. ccls is very fast with indexing the repos and works great with VSCode. You can also combine it with other editors and IDEs see https://github.com/MaskRay/ccls/wiki/Editor-Configuration

## Chromium (Blink)

[Project](https://www.chromium.org/blink) |
[GitHub](https://github.com/chromium/chromium)

Articles:
*  [What is Chromium? (DE)](https://www.heise.de/newsticker/meldung/Chrome-und-Chromium-Was-sind-eigentlich-die-Unterschiede-4245456.html)

The JavaScript-Engine of Blink is V8.

### V8

[Project](https://v8.dev/) |
[GitHub](https://github.com/v8/v8) |
[Source](https://cs.chromium.org/chromium/src/v8/src/) |
[How2Build](https://v8.dev/docs/build)


Build (Ubuntu 18.04):

```
$ git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
$ export PATH=$PATH:./depot_tools
$ gclient
$ mkdir ./v8 && cd ./v8
$ fetch v8 && cd v8
$ git pull
$ gclient sync
$ ./build/install-build-deps.sh
$ tools/dev/gm.py x64.release
$ out/x64.release/d8
```

Useful flags:

*    `--print-opt-code`: code generated by optimizing compiler
*    `--print-byte-code`: bytecode generated by interpreter
*    `--trace-ic`: different object types a call site encouters
*    `--trace-opt` and `--trace-deopt`: which functions are (de)optimized
*    `--trace-turbo`: TurboFan traces for the Turbolizer visualization

Articles:
*  [A tour of V8](http://www.jayconrod.com/posts/51/a-tour-of-v8--full-compiler)

#### JIT-Compiler: TurboFan

[Docs](https://v8.dev/docs/turbofan) |
[Blog](https://v8.dev/blog/turbofan-jit)

V8 provides a visualization for TurboFan called [Turbolizer](https://github.com/v8/v8/tree/master/tools/turbolizer)

Articles:

* [Introduction to TurboFan](https://doar-e.github.io/blog/2019/01/28/introduction-to-turbofan/)

##### Turbolizer usage:
1.    Run v8 with `--trace-turbo`: `d8 --trace-turbo foo.js`
2.    Generates json files e.g. `turbo-foo-0.json`
3.    Goto `v8/tools/turbolizer` and install with npm as described in `README.md`
4.    Serve directory e.g. `python -m SimpleHTTPServer 8000`
5.    Browse to `localhost:8000` and open `turbo-foo-0.json`



## Firefox (Gecko)

[Project](https://developer.mozilla.org/en-US/docs/Mozilla/Gecko) |
[GitHub](https://github.com/mozilla/gecko-dev)


The JavaScript-Engine of Gecko is Spidermonkey.

### Spidermonkey

[Project](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey) |
[Source](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey/Getting_SpiderMonkey_source_code) |
[How2Build](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey/Build_Documentation)

##### Source
*    [builtin](https://github.com/mozilla/gecko-dev/tree/master/js/src/builtin)


Build (Ubuntu 18.04):

```
$ wget -O bootstrap.py https://hg.mozilla.org/mozilla-central/raw-file/default/python/mozboot/bin/bootstrap.py && python bootstrap.py
$ git clone https://github.com/mozilla/gecko-dev.git && cd gecko-dev
$ cd js/src
$ autoconf2.13

# This name should end with "_DBG.OBJ" to make the version control system ignore it.
$ mkdir build_DBG.OBJ
$ cd build_DBG.OBJ
$ ../configure --enable-debug --disable-optimize
# Use "mozmake" on Windows
$ make -j 6
$ js/src/js
```


#### JIT-Compiler: IonMonkey

[Project](https://wiki.mozilla.org/IonMonkey)

Spidermonkey provides a visualization for IonMonkey called [IonGraph](https://github.com/sstangl/iongraph)

##### Source
*    [jit](https://github.com/mozilla/gecko-dev/tree/master/js/src/jit)


## Safari (Webkit)

[Project](https://webkit.org/) |
[GitHub](https://github.com/WebKit/webkit)


The JavaScript-Engine of Webkit is JavaScriptCore (JSC).

### JavaScriptCore

[Project](https://developer.apple.com/documentation/javascriptcore) |
[Wiki](https://trac.webkit.org/wiki/JavaScriptCore) |
[Source](https://github.com/WebKit/webkit/tree/master/Source/JavaScriptCore)

Articles:
*    http://www.filpizlo.com/papers.html

##### Source
*    Runtime: [Source/JavaScriptCore/runtime](https://trac.webkit.org/browser/webkit/trunk/Source/JavaScriptCore/runtime)

#### Build (Ubuntu 18.04):

```
# sudo apt install libicu-dev python ruby bison flex cmake build-essential ninja-build git gperf
$ git clone git://git.webkit.org/WebKit.git && cd WebKit
$ Tools/gtk/install-dependencies
$ Tools/Scripts/build-webkit --jsc-only --debug
$ cd WebKitBuild/Release
$ LD_LIBRARY_PATH=./lib bin/jsc
```


#### JIT-Compiler: LLInt+ Baseline JIT + DFG JIT + FTL JIT

WebKit has a 4-Layer JIT-Compiler system, representing the tradeoff between overhead performance cost and performance benefit.

Articles:
*    [Introduction to Webkit's JavaScript JIT Optimizations](https://webkit.org/blog/3362/introducing-the-webkit-ftl-jit/)
*    [Introducing the B3 JIT Compiler](https://webkit.org/blog/5852/introducing-the-b3-jit-compiler/)

##### Source
*    [LLInt (Low Level Interpreter)](https://trac.webkit.org/browser/trunk/Source/JavaScriptCore/llint)
*    [Baseline JIT](https://trac.webkit.org/browser/trunk/Source/JavaScriptCore/jit)
*    [DFG JIT (Data Flow Graph JIT)](https://trac.webkit.org/browser/trunk/Source/JavaScriptCore/dfg)
*    [FTL JIT (Faster Than Light Just In Time compiler)](https://trac.webkit.org/browser/trunk/Source/JavaScriptCore/ftl)





## Edge (Blink/EdgeHTML)

[Project](https://www.microsoft.com/en-us/windows/microsoft-edge) |
[GitHub](https://github.com/MicrosoftEdge)


Since Edge switched to Blink and the Chromium Project as its Rendering-Engine, Edge is using v8.
Originally, Edge had is own Rendering-Engine called EdgeHTML, which used the ChakraCore JavaScript-Engine.

### ChakraCore

[GitHub](https://github.com/Microsoft/ChakraCore) |
[How2Build](https://github.com/Microsoft/ChakraCore/wiki/Building-ChakraCore#linux)

#### Docs
*    [Architecture Overview](https://github.com/Microsoft/ChakraCore/wiki/Architecture-Overview)

#### Source
*    Runtime:  [Types](https://github.com/Microsoft/ChakraCore/tree/master/lib/Runtime/Types)
*    Runtime: [Language](https://github.com/Microsoft/ChakraCore/tree/master/lib/Runtime/Language)


#### Build (Ubuntu 18.04):

```
# To build ChakraCore on Linux: (requires Clang 3.7+ and Python 2)
$ apt-get install -y git build-essential cmake clang libicu-dev libunwind8-dev
$ git clone https://github.com/Microsoft/ChakraCore && cd ChakraCore
$ ./build.sh --cc=/usr/bin/clang-3.9 --cxx=/usr/bin/clang++-3.9 --arch=amd64 --debug
$ out/Debug/ch
```

# Exploitation

## Exploitation-Overview

*    Saelo: [Attacking JavaScript-Engines](http://www.phrack.org/papers/attacking_javascript_engines.html)
*    [Awesome-Browser-Exploitation](https://github.com/Escapingbug/awesome-browser-exploit)
*    [Attacking WebKit applications (Slides)](https://cansecwest.com/slides/2015/Liang_CanSecWest2015.pdf)
*    Saelo: Attacking Client-Side JIT Compilers - BlackHat 2018
     * [Video](https://youtu.be/emt1yf2Fg9g)
     * [Slides](https://saelo.github.io/presentations/blackhat_us_18_attacking_client_side_jit_compilers.pdf)
*    j0nathanj: From Zero to ZeroDay (Finding a Chakra Zero Day)
     *    [Video](https://media.ccc.de/v/35c3-9657-from_zero_to_zero_day)
          [Slides](https://github.com/j0nathanj/Publications/tree/master/35C3_From_Zero_to_Zero_Day)
*    Saelo: Fuzzili - (Guided-)fuzzing for JavaScript engines
     * [Video](https://www.youtube.com/watch?v=OHjq9Y66yfc)
     * [Slides](https://saelo.github.io/presentations/offensivecon_19_fuzzilli.pdf)



## Chromium Pwn
### CTF-Challenges
* 34c3: v9
    *   [Sources](https://github.com/saelo/v9)
    *   [WriteUp](https://gist.github.com/itsZN/9ae6417129c6658130a898cdaba8d76c) (Exploit-Script)
* 35c3: Krautflare
    *   [Files](https://abiondo.me/assets/ctf/35c3/krautflare-33ce1021f2353607a9d4cc0af02b0b28.tar)
    *   [WriteUp](https://abiondo.me/2019/01/02/exploiting-math-expm1-v8/)
    *   [WriteUp](https://www.jaybosamiya.com/blog/2019/01/02/krautflare/)
* CSAW-Finals-2018: ES1337
     *  [Files+WriteUp](https://github.com/osirislab/CSAW-CTF-2018-Finals/tree/master/pwn/ES1337)
* Plaid CTF 2018: Roll a dice
     *  [Files](https://github.com/m1ghtym0/write-ups/tree/master/browser/plaid-2018-roll-a-dice)
     *  [WriteUp](https://gist.github.com/saelo/52985fe415ca576c94fc3f1975dbe837)
     *  [WriteUp](https://ctftime.org/writeup/9999)
* Google CTF Finals 2018: Just In Time
     *  [Files+WriteUp](https://github.com/google/google-ctf/tree/master/2018/finals/pwn-just-in-time)
     *  [Slides](https://github.com/google/google-ctf/blob/master/2018/finals/solutions.pdf)
     *  [WriteUp](https://xz.aliyun.com/t/3348)
* *CTF 2019: oob-v8
     *  [Files](https://github.com/Changochen/CTF/raw/master/2019/*ctf/Chrome.tar.gz)
     *  [WriteUp](https://changochen.github.io/2019-04-29-starctf-2019.html)
     *  [WriteUp](https://github.com/vngkv123/aSiagaming/blob/master/Chrome-v8-oob/README.md)
     *  [WriteUp](https://github.com/alstjr4192/BGazuaaaaa/blob/master/*CTF%202019%20oob/pwn.js)
                
### RealWorld
* [MobilePwn2Own 2013 - Chrome on Android](https://docs.google.com/document/d/1tHElG04AJR5OR2Ex-m_Jsmc8S5fAbRB3s4RmTG_PFnw/edit)
* https://halbecaf.com/2017/05/24/exploiting-a-v8-oob-write/
* niklasb: Chrome IPC Exploitation
     * [Video](https://www.youtube.com/watch?v=MMxtKq8UgwE)
     * [Slides](https://github.com/phoenhex/files/blob/master/slides/chrome_ipc_exploitation_offensivecon19.pdf)
* [CVE-2019-5782 Write-Up](https://github.com/vngkv123/aSiagaming/tree/master/Chrome-v8-906043)
* [CVE-2019-5790](https://labs.bluefrostsecurity.de/blog/2019/04/29/dont-follow-the-masses-bug-hunting-in-javascript-engines/)
* saelo: Exploiting Logic Bugs in JavaScript JIT Engines
     * [Phrack Article](http://phrack.org/papers/jit_exploitation.html)
     * [41con 19' slides](https://saelo.github.io/presentations/41con_19_jit_exploitation_tricks.pdf)

### Hardening & Mitigations
* [Heap-hardening](https://struct.github.io/oilpan_metadata.html)



## Firefox Pwn
### Articles

* [Playing around with SpiderMonkey](https://vigneshsrao.github.io/play-with-spidermonkey/)
* [OR'LYEH? The Shadow over Firefox](http://www.phrack.org/issues/69/14.html)

### CTF-Challenges
* 33c3: Feuerfuchs
    *   [Sources](https://github.com/saelo/feuerfuchs)
    *   [WriteUp](https://bruce30262.github.io/Learning-browser-exploitation-via-33C3-CTF-feuerfuchs-challenge/)
    *   [WriteUp+Build](https://github.com/m1ghtym0/write-ups/tree/master/browser/33c3ctf-feuerfuchs)
* Blaze 2018: blazefox
    *   [Sources](https://ctftime.org/task/6000)
    *   [WriteUp](https://devcraft.io/2018/04/27/blazefox-blaze-ctf-2018.html)
    *   [WriteUp](https://gist.github.com/niklasb/4bddc9e8f32c3bd277ed26d66d488834) (Exploit-Script)
    *   [WriteUp](https://github.com/Jinmo/ctfs/blob/master/2018/blaze/pwn/blazefox.html) (Exploit-Script)
    *   [Build+WriteUp](https://github.com/m1ghtym0/write-ups/tree/master/browser/blaze-ctf-2018-blazefox)
* 35c3 FunFox
    *   [Sources](https://github.com/bkth/35c3ctf/tree/master/funfox) 
### RealWorld
* Use-after-free in Spidermonkey (Beta 53)
     * [Article](https://phoenhex.re/2017-06-21/firefox-structuredclone-refleak#turning-a-use-after-free-into-a-readwrite-primitive)
     * [Talk](https://www.youtube.com/watch?v=D_9EFWYnBik)
     * [Slides](https://grehack.fr/data/2017/slides/GreHack17_Get_the_Spidermonkey_off_your_back.pdf)
* https://saelo.github.io/posts/firefox-script-loader-overflow.html
* Use-after-free in SpiderMonkey (64.0a1)
     * [Article](https://www.zerodayinitiative.com/blog/2019/7/1/the-left-branch-less-travelled-a-story-of-a-mozilla-firefox-use-after-free-vulnerability)


## Safari Pwn
### CTF-Challenges
* RealWorldCTF 2018: Engine for Neophytes
    *   [Files](http://mightym0.de/ctf/rwctf-2018/allForPlayers.zip)
* 35c3: WebKid
    *   [Sources](https://github.com/saelo/35c3ctf/tree/master/WebKid)
    *   [WriteUp](https://github.com/LinusHenze/35C3_Writeups/tree/master/WebKid)
### RealWorld
* http://www.phrack.org/papers/attacking_javascript_engines.html
     *    [Source](https://github.com/saelo/jscpwn)
     *    [WriteUp+Build](https://github.com/m1ghtym0/write-ups/tree/master/browser/CVE-2016-4622)
* [Fuzzing Webkit and analysis of CVE-2019-8375](https://www.inputzero.io/2019/02/fuzzing-webkit.html)
* [CVE-2017-2446 WriteUp](https://doar-e.github.io/blog/2018/07/14/cve-2017-2446-or-jscjsglobalobjectishavingabadtime/)
* https://saelo.github.io/posts/jsc-typedarray.slice-infoleak.html
* https://github.com/saelo/pwn2own2018/tree/master/stage0
* https://github.com/LinusHenze/WebKit-RegEx-Exploit
* https://github.com/W00dL3cs/exploit_playground/tree/master/JavaScriptCore

### Hardening & Mitigations
* [Heap-hardening](https://labs.mwrinfosecurity.com/blog/some-brief-notes-on-webkit-heap-hardening/)
* CagedPtr [Source](https://github.com/WebKit/webkit/blob/master/Source/WTF/wtf/CagedPtr.h) & [ArrayBuffer Example](https://bugs.webkit.org/show_bug.cgi?id=175515)



## Edge Pwn
### CTF-Challenges
* Plaid 2017: chakrazy
    * [WriteUp](https://bruce30262.github.io/Chakrazy-exploiting-type-confusion-bug-in-ChakraCore/)
* N1CTF 2018: Chakra
     * [Files](https://github.com/Nu1LCTF/n1ctf-2018/tree/master/challenges/pwn/Chakra)
### RealWorld
* bkth: Attacking Edge Through the JavaScript-Compiler
     * [Video](https://www.youtube.com/watch?v=r4J7Zu1RV40)
     * [Slides](https://github.com/bkth/Attacking-Edge-Through-the-JavaScript-Compiler)

# Tools

## Libraries:
* [pwnjs (A Javascript library for browser exploitation) ](https://github.com/theori-io/pwnjs)

## Utils
  * [int64.js](https://github.com/saelo/jscpwn/blob/master/int64.js)
  * [utils.js](https://github.com/saelo/jscpwn/blob/master/utils.js)
 
## Debugging
*    [shadow](https://github.com/CENSUS/shadow) jemalloc heap exploitation framework (heap allocator used by Firefox)
# JavaScript (ECMAScript) Docs

* [Types&Values](http://www.ecma-international.org/ecma-262/6.0/#sec-ecmascript-data-types-and-values)
* [Objects](http://www.ecma-international.org/ecma-262/6.0/#sec-objects)
* [Object-Properties](https://tc39.github.io/ecma262/#sec-property-attributes)
