# SpotBugs
[SpotBugs][]是FindBugs的成功继任者，静态分析的工具，用来寻找Java代码中Bug。更详细的介绍请直接去官网看吧。

# requires
SpotBugs requires JRE (or JDK) 1.8.0 or later to run. However, it can analyze programs compiled for any version of Java, from 1.0 to 1.9.

# Using SpotBugs
SpotBugs can be used standalone and through several integrations, including:
* [Ant][]
* [Maven][]
* [Gradle][]
* [Eclipse][]

# Bug Descriptions
SpotBugs checks for more than 400 bug patterns. Bug descriptions can be found [here](https://spotbugs.readthedocs.io/en/latest/bugDescriptions.html).
无法连接外网的情况下，可以下载官方的[PDF文件](https://media.readthedocs.org/pdf/spotbugs/latest/spotbugs.pdf)。

以下从网络上搜集了别人的几个翻译，仅供参考，注意辨别真假。

* Malicious code vulnerability (MALICIOUS_CODE)恶意代码漏洞

  code that is vulnerable to attacks from untrusted code
代码有漏洞，可能被攻击。
必须要重要。

* Multithreaded correctness (MT_CORRECTNESS)多线程的正确性

 code flaws having to do with threads, locks, and volatiles
可能导致线程、死锁及不稳定的
必须很重要。

* Performance (PERFORMANCE)性能

 code that is not necessarily incorrect but may be inefficient
可能代码正确但是性能会有问题的地方
必须很重要。

* Security (SECURITY)安全

 A use of untrusted input in a way that could create a remotely exploitable security vulnerability.
非安全输入，可导致远程利用的安全漏洞。
很重要

* Bad practice (BAD_PRACTICE)低劣的代码实践，违反推荐和必要的编码惯例

 Violations of recommended and essential coding practice. Examples include hash code and equals problems, cloneable idiom, dropped exceptions, Serializable problems, and misuse of finalize.
不是特别重要。

* Correctness (CORRECTNESS)正确性，可能出现非预期的代码错误

 Probable bug - an apparent coding mistake resulting in code that was probably not what the developer intended.
逻辑大概就是：这个地方的正确性值得商榷，你最好来确认下会不会潜在的出现不是你预期的行为。
不是特别重要。

* Experimental (EXPERIMENTAL)实验性的

 Experimental and not fully vetted bug patterns
看官方文档里的子bug类别描述，大概意思是说老兄，这个地方本程序查不出来，要么是类太大了之类，bug匹配模式不太适用，你自己再确认下。
不太重要。

* Dodgy code (STYLE)糟糕的代码（风格）

 code that is confusing, anomalous, or written in a way that leads itself to errors. Examples include dead local stores, switch fall through, unconfirmed casts, and redundant null check of value known to be null. More false positives accepted.
不是很重要，看公司要求。

* Internationalization (I18N)国际化

 code flaws having to do with internationalization and locale
代码没有遵守必要的本地化和国际化
这个要看公司要求

* Bogus random noise (NOISE)假的随机噪音

 Bogus random noise: intended to be useful as a control in data mining experiments, not in finding actual bugs in software
据描述的意思，意思就是兄弟，你不用管，这是spotbugs自己搞事。
完全不重要


[SpotBugs]: https://spotbugs.github.io/
[Ant]: http://spotbugs.readthedocs.io/en/latest/ant.html
[Maven]: http://spotbugs.readthedocs.io/en/latest/maven.html
[Gradle]: http://spotbugs.readthedocs.io/en/latest/gradle.html
[Eclipse]: http://spotbugs.readthedocs.io/en/latest/eclipse.html
