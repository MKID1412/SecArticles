#  AI 已经能把漏洞串成利用链？Cloudflare 实测 Mythos Preview  
aeverj
                    aeverj  红队工坊   2026-11-23 02:32  
  
> 原文标题： Project Glasswing: what Mythos showed us  
  
  
  
![BLOG-3301 图 1](https://mmbiz.qpic.cn/mmbiz_jpg/iauzMfibvvF2fBAKjyfegwn7tDouRSflnyz4agHloSKzFEiaDWaC4zfw0mxPm3xibqpC8UUNHtUjxYSxviaAYYL2FHZCNicib1O4gBtcVAnjWCYsZs/640?wx_fmt=jpeg&from=appmsg "")  
  
BLOG-3301 图 1  
  
  
过去几个月，我们一直在自己的基础设施上测试一批面向安全的大语言模型（LLM）。这些模型能帮我们找出自家系统里的潜在漏洞，好让我们及时修复；同时，它们也让我们看清：攻击者拿到最新模型后能做什么。  
  
在这些模型里，最受关注的是 Anthropic 的 Mythos Preview。几周前，我们受邀以 [Project Glasswing]   
https://www.anthropic.com/glasswing  
 项目成员的身份使用 Mythos Preview。很快，我们把它指向了自己的 50 多个代码仓库，想看它能发现什么，也想看它是怎么工作的。  
  
本文分享我们的观察：这些模型擅长什么、不擅长什么，以及围绕它们的架构和流程需要怎么改，才能让这套东西真正规模化。  
## Mythos Preview 带来了什么变化  
  
Mythos Preview 是实打实的进步，这一点值得在最前面说清楚。我们让模型分析自己的代码已经有一段时间了，而从以往的通用前沿模型（frontier model）到今天的 Mythos Preview，这个跨度不只是对既有能力的打磨。  
  
它是一种不同类型的工具，做的是不同类型的工作，因此很难拿它和早期模型做干净的同类对比。所以与其拿通用前沿模型去给 Mythos Preview 打分，不如直接描述它实际能做什么。在与它合作的过程中，有两个特点格外突出：  
  
• **利用链构建（exploit chain construction）-**  
 真实攻击很少只靠一个漏洞。它会把若干小的攻击原语（primitive）串成一条可用的利用链。比如，它可能把一个释放后使用（use-after-free）漏洞变成任意读写原语，劫持控制流，再用返回导向编程（return-oriented programming，ROP）链彻底接管系统。Mythos Preview 能拿到其中几个原语，并推理如何把它们组合成可运行的验证。它一路展示的推理过程看起来像资深研究员的手笔，而不像自动扫描器的输出。  
  
• **证明生成（proof generation）-**  
 发现漏洞和证明漏洞可利用是两回事，而 Mythos Preview 两件都能做。它会写出能触发怀疑漏洞的代码，在临时环境里编译并运行。如果程序的行为符合模型的预期，那就是证明；如果不符合，模型会读取失败信息、调整假设、再试一次。这个循环和它找到的漏洞本身一样重要——没有可用证明的疑似缺陷只是猜测，而 Mythos Preview 能自己补上这段差距。  
  
上面描述的部分能力并非 Mythos Preview 独有。我们把其他前沿模型放进同一套框架里跑时，它们也找到了相当多相同的底层漏洞，在某些情况下推理水平也比我们预期的走得更远。它们欠缺的地方在于把碎片拼起来的那一刻：模型会识别出一个有意思的漏洞，写一段颇有见地的说明解释它为何重要，然后就停住了，留下的链条没走完，可利用性还是个问号。Mythos Preview 带来的变化是，模型现在能把那些低危漏洞（传统上会悄无声息地躺在待办队列里）串成一条更严重的完整利用链。  
## 正当漏洞研究中的模型拒答  
  
Anthropic 在 Project Glasswing 中提供的 Mythos Preview，并没有像 Opus 4.7 或 GPT-5.5 这类公开可用模型那样附带额外的安全防护措施。  
  
尽管如此，模型仍会自发地对某些请求表示抗拒——就像让它适合挖漏洞的网络能力一样，它也有自己涌现出来的护栏，有时会让它拒绝正当的安全研究请求。但我们发现，这些自发拒答并不稳定：同一个任务，换个说法或换个上下文呈现，结果可能完全不同，正如下面的例子所示。  
  
  
![BLOG-3301 图 1](https://mmbiz.qpic.cn/mmbiz_jpg/iauzMfibvvF2cOTD2obrNzVyAHn59RibmLBFwEflFsk8Jm4PWFLHPCoSSATvIV6dPkXCc0vU60ibzxcEK7Y960yoicJnUsWCyBx4MTqo9aM6EVgk/640?wx_fmt=jpeg&from=appmsg "")  
  
BLOG-3301 图 1  
  
  
Mythos Preview 拒绝构建可运行的概念验证的示例  
  
例如，模型一开始拒绝了对某个项目做漏洞研究，之后在对该项目环境做了无关改动后，又同意对同一份代码做同样的研究。而被分析的代码本身没有任何变化。  
  
另一个例子中，模型在一个代码库里发现并确认了若干严重的内存漏洞，随后却拒绝编写演示性利用。同样的请求换个说法，答案就不一样；而且由于模型的概率特性，同一个请求在不同运行之间也可能给出不同结果。语义等价的任务，可能仅仅因为呈现方式和时机不同，就得到相反的结果。  
  
这一点很重要：模型的这些自发拒答/护栏是真实存在的，但它们不够稳定，无法独自充当完整的安全边界。也正因如此，未来任何公开发布的、有能力的网络前沿模型，都必须在这层基线行为之上再加上额外防护——这样它才适合在 Project Glasswing 这类受控研究场景之外被更广泛地使用。  
## 信噪比难题  
  
漏洞分类研判（triage）中最难的部分之一，是判断哪些漏洞是真的、哪些是可利用的、哪些需要立刻修。这个问题在 AI 出现之前就很棘手。AI 漏洞扫描器和 AI 生成的代码让它变得更糟，为此我们在 Cloudflare 里建了多个后置验证阶段来应对。  
  
影响噪声率的因素主要有两个：  
  
• **编程语言（Programming language）-**  
 C 和 C++ 让你直接掌控内存，也带来了相伴的漏洞类型——缓冲区溢出、越界读写——而 Rust 这类内存安全语言在编译期就消除了这些问题。我们明显看到，用内存不安全语言写的项目误报更多。  
  
• **模型偏差（Model bias）-**  
 优秀的人类研究员会告诉你他发现了什么、有多大把握。模型不会。你让模型去找漏洞，它就会找到，不管代码里到底有没有。结论往往带着“可能”“或许”“理论上可以”这类含糊措辞，而这类含糊结论的数量远超扎实结论。对一个探索性工具来说，这种偏差是合理的；但对一个研判队列来说，它是灾难——每一条推测性结论都要耗费人的注意力和 token 才能否定，而这个成本在上千条结论上会不断累积。  
  
Mythos Preview 在这方面有明显改善，尤其是它串联原语的能力——把多个漏洞组合成一个可运行的验证，而不是孤立地报告它们。带着 PoC 送来的结论，才是你能据此行动的结论，也意味着花在“这到底是不是真的”上的时间少得多。  
  
我们的框架是刻意调成“宁可多报”的，这样能看到更多（也更少漏报），代价是噪声大增。但在研判阶段，Mythos Preview 的输出质量明显更高：含糊结论更少、复现步骤更清楚、走到“修还是不修”这个决定所花的工作更少。  
## 为什么把通用编程智能体直接丢给仓库行不通  
  
去年我们刚开始做 AI 辅助的漏洞研究时，直觉做法很直接：把一个通用编程智能体指向任意仓库，让它去找漏洞。这种做法“能用”——模型确实会产出结论——但它无法对真实代码库形成有意义的覆盖，也找不出有价值的结论。主要有两个原因：  
  
• **上下文（Context）-**  
 编程智能体是为单一聚焦的工作流调优的：实现一个功能、修一个漏洞、写一次重构。它们吞下大量源码，一次只握住一个假设并对它反复迭代。这恰恰是漏洞研究最不该有的形态——漏洞研究天生是窄而并行的。人类研究员会挑一件具体的事深挖：可能是某个复杂功能、某几个跨安全边界的转换，或者某一类漏洞，比如命令注入，也就是攻击者输入最终被当成 shell 命令执行。然后他们换个功能、换个安全边界、换个漏洞类型再来一遍，在整个代码库里重复几千次。面对十万行级别的仓库，单个智能体会话（即便带子智能体）也只能以有用的方式覆盖大约千分之一的面，随后模型的上下文窗口就满了、上下文压缩（compaction）启动——这可能把早先本来重要的发现丢掉。  
  
• **吞吐量（Throughput）-**  
 单流智能体一次只做一件事，而真实代码库需要同时对多个组件验证多个假设，并且在发现有趣线索时还能继续扇出。你可以把单个智能体压得更狠，但到某个点之后，限制你的就不再是模型，而是这种交互形态本身。把模型直接放进编程智能体里，对人工调查来说是合适的——当研究员已有线索、想多一双眼睛时很好用；但要拿到高覆盖率，它就是个错误的工具。一旦接受这一点，我们就不再让 Mythos Preview 干错位的活，而是转而围绕它搭建框架。  
## 框架到底解决了什么  
  
在大规模跑这套东西的过程中，我们得到四条经验，每一条都指向同一个需求：需要一个能管理整体执行的框架（harness）。  
  
• **范围收窄能产出更好的结论 -**  
 告诉模型“在这个仓库里找漏洞”，它会四处乱逛。告诉它“在这个具体函数里找命令注入，它的上方是这样的信任边界，这是架构文档，这是这片区域之前的覆盖情况”，它做的事就接近研究员真正会做的事。  
  
• **对抗式复核能降低噪声 -**  
 在初始结论和队列之间插入第二个智能体——不同的提示词、不同的模型、且它自己没有产出新结论的能力——能抓住大量第一个智能体自查时会漏掉的噪声。事实证明，让两个智能体刻意对立，比单纯叮嘱一个智能体“要小心”有效得多。  
  
• **把链条拆到多个智能体上能得到更好的推理 -**  
 “这段代码有 bug 吗？”和“攻击者真的能从系统外部触达这个 bug 吗？”是两个不同的问题。分开来问时模型对每一个都答得更好，因为每个问题都比合并版更窄。  
  
• **多个窄任务并行胜过一个大而全的智能体 -**  
 让很多智能体处理边界极紧的问题、事后再去重结果，覆盖率会比让一个智能体包干更好。  
  
上述每一条观察都关于模型行为，合在一起描述的东西已经不再是一个聊天界面了，而是一个帮你达成最终结果的框架。搭框架的头几步很简单——你可以让模型自己帮忙，我们就是这么做的。我们用 Mythos Preview 来构建、调整并改进我们原有的框架，让它贴合自身的长处。  
  
下面举一个框架在实际中长什么样的例子。  
## 我们的漏洞发现框架  
  
下面按阶段展示我们的漏洞发现框架。它被用来扫描我们运行时、边缘数据路径、协议栈、控制平面以及我们所依赖的开源项目中的实际代码。  
  
  
![BLOG-3301 图 2](https://mmbiz.qpic.cn/sz_mmbiz_jpg/iauzMfibvvF2eUjDHUIO33tZy817QHAjqZK3Vj8ADWFBmvXKF8IY5yFj2c4picFDIx6qh5zYS6iaSlTGEFFaSoh2TnEfTOia2lyZZW22YZCztx2E/640?wx_fmt=jpeg&from=appmsg "")  
  
BLOG-3301 图 2  
  
  
<table><thead><tr style="box-sizing: border-box;margin: 0;"><th align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;color: #f9f9f7;text-transform: uppercase;background-color: #111;font-size: .875em;font-weight: 700;"><strong style="box-sizing: border-box;margin: 0;color: #111;font-weight: 700;"><span leaf="">阶段</span></strong></th><th align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;color: #f9f9f7;text-transform: uppercase;background-color: #111;font-size: .875em;font-weight: 700;"><strong style="box-sizing: border-box;margin: 0;color: #111;font-weight: 700;"><span leaf="">它做什么</span></strong></th><th align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;color: #f9f9f7;text-transform: uppercase;background-color: #111;font-size: .875em;font-weight: 700;"><strong style="box-sizing: border-box;margin: 0;color: #111;font-weight: 700;"><span leaf="">为什么重要</span></strong></th></tr></thead><tbody><tr style="box-sizing: border-box;margin: 0;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><strong style="box-sizing: border-box;margin: 0;color: #111;font-weight: 700;"><span leaf=""><img data-src="https://mmbiz.qpic.cn/mmbiz_png/iauzMfibvvF2d313DlkxXicNjY9NPTtRJl3suO77hJQrZ2kS6CFAibLEMCdodcEf2lwd6FYdBVW9t1vPbEPGmQic8Wf70ibs9XhCINMibqnb4kqYMg/640?wx_fmt=png&amp;from=appmsg" alt="BLOG-3301_image7" class="rich_pages wxw-img" data-ratio="1" data-w="68" height="auto" style="box-sizing: border-box;border: 1px solid #111;max-width: 100%;margin: 0 auto;display: block;height: auto;" data-aistatus="1"/></span><span leaf=""><br/></span><span leaf=""> 侦察（Recon）</span></strong></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">一个智能体自上而下通读仓库，扇出到负责各子系统的子智能体，产出一份架构文档，涵盖构建命令、信任边界、入口点以及可能的攻击面。它还会生成下一阶段的初始任务队列。</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">为下游每个智能体提供共享上下文，减少“乱逛”问题。</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;background-color: #f5f5f3;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><strong style="box-sizing: border-box;margin: 0;color: #111;font-weight: 700;"><span leaf=""><img data-src="https://mmbiz.qpic.cn/sz_mmbiz_png/iauzMfibvvF2fGNEDtYc1xy554iaCahp9MnSPK9L64WsCbvGhu9OubPYxicEn5iau4wNnzdDkfUFRpFc1rd63LumNFcucrKbC3dyH5kicfjUwd9OY/640?wx_fmt=png&amp;from=appmsg" alt="BLOG-3301_image8" class="rich_pages wxw-img" data-ratio="1" data-w="68" height="auto" style="box-sizing: border-box;border: 1px solid #111;max-width: 100%;margin: 0 auto;display: block;height: auto;" data-aistatus="1"/></span><span leaf=""><br/></span><span leaf=""> 搜寻（Hunt）</span></strong></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">每个任务是一个攻击类型加上一条范围提示。猎手（真正去找漏洞的智能体）并发运行，通常一次约 50 个，每个再扇出到若干探索子智能体。每个猎手都能使用工具，在每个任务专属的临时目录里编译并运行概念验证代码。</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">大部分工作在这里完成。许多窄任务并行，而不是一个包干到底的智能体。</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><strong style="box-sizing: border-box;margin: 0;color: #111;font-weight: 700;"><span leaf=""><img data-src="https://mmbiz.qpic.cn/sz_mmbiz_png/iauzMfibvvF2dyURPg1T7DaKoo69K9Mdd1AIdFmKicQkKrhtSkfVy2StOGv06ibNzibcSLLAfHN6SWKTofZP4wniaLPmPTEvTXsMDanuFHGjWgkrw/640?wx_fmt=png&amp;from=appmsg" alt="BLOG-3301_image5" class="rich_pages wxw-img" data-ratio="1" data-w="68" height="auto" style="box-sizing: border-box;border: 1px solid #111;max-width: 100%;margin: 0 auto;display: block;height: auto;" data-aistatus="1"/></span><span leaf=""><br/></span><span leaf=""> 验证（Validate）</span></strong></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">一个独立智能体重读代码，并试图推翻原始结论。它使用不同的提示词，且没有自行产出新结论的能力。</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">能抓住相当一部分猎手自查时抓不到的噪声。</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;background-color: #f5f5f3;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><strong style="box-sizing: border-box;margin: 0;color: #111;font-weight: 700;"><span leaf=""><img data-src="https://mmbiz.qpic.cn/mmbiz_png/iauzMfibvvF2cN0NYcHbAcibZYO4ibpZuEOHicZkLp7giauW4pkzEffBMDY1MW5LqpibMcD5ibDkkRI28Hnj9baA3MHOGric3DicPoTBWd7rsWAd3m4eM/640?wx_fmt=png&amp;from=appmsg" alt="BLOG-3301_image11" class="rich_pages wxw-img" data-ratio="1" data-w="68" height="auto" style="box-sizing: border-box;border: 1px solid #111;max-width: 100%;margin: 0 auto;display: block;height: auto;" data-aistatus="1"/></span><span leaf=""><br/></span><span leaf=""> 补漏（Gapfill）</span></strong></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">猎手会标记出自己碰过但没有彻底覆盖的区域。这些区域将被重新排进队列再跑一遍。</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">抵消模型倾向于漂向自己已经成功过的攻击类型的毛病。</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><strong style="box-sizing: border-box;margin: 0;color: #111;font-weight: 700;"><span leaf=""><img data-src="https://mmbiz.qpic.cn/sz_mmbiz_png/iauzMfibvvF2fV8d4qofwxyicNibAia92K4Q3JxK1icc7jfjcrnF1FZmmhA9g4rUALUCprTYe8mMjAsWOngicz28NC0LEzN3VjXxjCwdc2AbsjIDpU/640?wx_fmt=png&amp;from=appmsg" alt="BLOG-3301_image4" class="rich_pages wxw-img" data-ratio="1" data-w="51" height="auto" style="box-sizing: border-box;border: 1px solid #111;max-width: 100%;margin: 0 auto;display: block;height: auto;" data-aistatus="1"/></span><span leaf=""><br/></span><span leaf=""> 去重（Dedupe）</span></strong></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">同一根因的结论合并为一条记录。</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">变体分析是一项能力，而不是给队列灌重复项的手段。</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;background-color: #f5f5f3;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><strong style="box-sizing: border-box;margin: 0;color: #111;font-weight: 700;"><span leaf=""><img data-src="https://mmbiz.qpic.cn/mmbiz_png/iauzMfibvvF2dot0icEQEqXPth6U5t2aS0e9xfQEUAFTqzGjuCp9KyodNvBglnwOibrZMBRUsMwYekuenIXvep3ibrvvjK3PNkRUXjD8LtdWZ83k/640?wx_fmt=png&amp;from=appmsg" alt="BLOG-3301_image3" class="rich_pages wxw-img" data-ratio="1" data-w="51" height="auto" style="box-sizing: border-box;border: 1px solid #111;max-width: 100%;margin: 0 auto;display: block;height: auto;" data-aistatus="1"/></span><span leaf=""><br/></span><span leaf=""> 追踪（Trace）</span></strong></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">对共享库中每条已确认结论，一个追踪智能体扇出（每个消费方仓库一个实例），借助跨仓库符号索引，判断攻击者控制的输入是否真的能从系统外部抵达该 bug。</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">把“这里有个缺陷”变成“这里有个可达的漏洞”。这是最重要的阶段。</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><strong style="box-sizing: border-box;margin: 0;color: #111;font-weight: 700;"><span leaf=""><img data-src="https://mmbiz.qpic.cn/mmbiz_png/iauzMfibvvF2ftGbfZ3fD43aVNhyoFWwibeDWg4Mr5r56XGENpTBMasgu0RibFPKWVjae5XeYSAXcZao5WF8Fq85Sboxczwb8FLDwriagkAvXRnc/640?wx_fmt=png&amp;from=appmsg" alt="BLOG-3301_image1" class="rich_pages wxw-img" data-ratio="1" data-w="51" height="auto" style="box-sizing: border-box;border: 1px solid #111;max-width: 100%;margin: 0 auto;display: block;height: auto;" data-aistatus="1"/></span><span leaf=""><br/></span><span leaf=""> 反馈（Feedback）</span></strong></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">可达的追踪结果会变成消费方仓库里的新搜寻任务——也就是该 bug 真正暴露的地方。</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">闭合成环。流水线越跑越好。</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;background-color: #f5f5f3;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><strong style="box-sizing: border-box;margin: 0;color: #111;font-weight: 700;"><span leaf=""><img data-src="https://mmbiz.qpic.cn/sz_mmbiz_png/iauzMfibvvF2eM4Sliak6HUShOvXFsIvpgpBlN2iaNT3DDWgSEkqkqRyQjmMvHZOw8X1oArn54PErTYdFQ7wQCHl9JbcWZmI5HWybLJT6osLibTw/640?wx_fmt=png&amp;from=appmsg" alt="BLOG-3301_image10" class="rich_pages wxw-img" data-ratio="1" data-w="51" height="auto" style="box-sizing: border-box;border: 1px solid #111;max-width: 100%;margin: 0 auto;display: block;height: auto;" data-aistatus="1"/></span><span leaf=""><br/></span><span leaf=""> 报告（Report）</span></strong></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">一个智能体按预定义 schema 写出结构化报告，自行修正对 schema 的校验错误，然后提交到采集 API。</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">输出是可查询的数据，而不是自由格式的长文。</span></span></td></tr></tbody></table>## 这对安全团队意味着什么  
  
其他安全负责人对 Mythos Preview 最强烈的反应都围绕速度——更快地扫描、更快地打补丁、压缩响应周期。我们交流过的团队里，不止一个如今已经在按“从 CVE 公布到生产环境打完补丁两小时”的服务水平目标运行。这种直觉可以理解：攻击方的时间线变短，防守方的时间线就得跟着变短。但光靠快是不够的，我们认为很多团队即将用大量时间、精力和金钱去痛苦地认识到这一点。  
  
打补丁更快，并不会改变产出补丁那条流水线的形状。如果回归测试要一整天，你想压到两小时的目标而不跳过它是不可能的；而你跳过回归测试所发出的 bug，往往比你想修的那个 bug 还糟。我们试过让模型自己写补丁，结果有几个补丁出去以后修好了原来的 bug，却悄悄弄坏了代码所依赖的其他东西——这就是上面那条经验的又一次重演。  
  
更难的问题是：围绕漏洞的架构应该长什么样。原则是——即便 bug 存在，也要让攻击者更难利用它，这样“漏洞被披露”和“它被修好”之间的这段空档就没那么要命。这意味着要有站在应用前面、挡住对 bug 访问路径的防御；意味着把应用设计成某处代码的缺陷无法让攻击者触及其他部分；也意味着能在同一时刻把修复推送到代码运行的每一个地方，而不是干等各个团队分别部署。  
  
我们也清楚这个话题是双向的。帮助我们找到自家代码 bug 的同一套能力，落到不该拿的人手里，会加速对互联网上每一个应用的攻击。Cloudflare 站在数百万这类应用的前面，而上面描述的架构原则，恰恰是我们的产品本就为客户去实现的原则。关于这对客户意味着什么，我们将在未来几周分享更多。  
  
如果你的团队也在做类似的工作，想交流心得，欢迎联系我们：  
security-ai-research@cloudflare.com  
。  
  
我们与 Mythos Preview 的研究是在受控环境中针对自家代码进行的；此项工作暴露出的每一个漏洞，都按 Cloudflare 正式的漏洞管理流程进行了分类研判、验证，并在需要时完成了修复。  
  
这项工作是一次团队协作。感谢 Albert Pedersen、Craig Strubhart、Dan Jones、Irtefa Fairuz、Martin Schwarzl 和 Rohit Chenna Reddy 对本文背后的研究、工程与分析所做的贡献。  
  
**如果你对网络安全、红队攻防技术充满热情，渴望学习更多实战技巧，例如渗透测试、自动化脚本编写、免杀技术等， 欢迎关注我的公众号**  
******在这里，我会持续分享更多高质量的技术文章，与你一同探索网络安全的奥秘，提升实战技能！ 让我们一起在队攻防的道路上，不断精进，突破边界！**  
> **免责声明：**  
 本文仅供安全技术研究与学习交流之用。 严禁将本文所提及的技术用于任何非法用途，包括但不限于未经授权的渗透测试、网络攻击、恶意代码传播等。  
  
  
