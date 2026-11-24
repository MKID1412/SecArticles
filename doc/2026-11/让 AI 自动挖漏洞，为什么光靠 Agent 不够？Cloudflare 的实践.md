#  让 AI 自动挖漏洞，为什么光靠 Agent 不够？Cloudflare 的实践  
aeverj
                    aeverj  红队工坊   2026-11-24 02:33  
  
> 原文标题： Build your own vulnerability harness  
  
  
  
![BLOG-3344 1](https://mmbiz.qpic.cn/mmbiz_jpg/iauzMfibvvF2fL331CUicM4uRY8hAaviakzvE2CHiczkVcDZpE6X0rgqgpaZfPWIgMgJNd2LOnjfU53sesGZpR8sJRDebt3vM0jamicE8r4f1PYUA/640?wx_fmt=jpeg&from=appmsg "")  
  
BLOG-3344 1  
  
  
几周前，我们公布了「玻璃翼计划」（Project Glasswing）的初步发现，探讨了把前沿安全模型指向企业级代码库时会发生什么。我们还研究了防御体系如何调整，以[保护我们的基础设施和客户免受前沿 AI 带来的威胁]   
https://blog.cloudflare.com/frontier-model-defense/。自那以后，AI  
 生态继续快速变化——那些深度绑定单一模型的开发者，已经亲身体会到当初选中的模型下线、或者被更强的模型取代时是什么滋味。这些市场变动只是进一步印证了我们的核心判断：无论当下是哪个底层模型领跑，智能体（agentic）工作流的未来都不在独立模型、提示词或单智能体会话里。  
  
要从局部的安全"技能"（skill）升级为持续运行、覆盖整个代码库舰队（fleet）的扫描流水线，需要一种把模型当作可替换组件的架构。依赖单一模型会天然限制防御覆盖面，因为同一套系统往往会用完全相同的视角去看待代码路径。为对抗这一点，模型应当被频繁替换、交叉测试。让模型在整个流水线中轮换——比如用一个模型做初步发现、用另一个完全不同的模型做验证——就能确保漏洞被不同的逻辑体系交叉检查。此外，真正的企业级框架必须跳出孤立仓库，追踪跨仓库依赖中的漏洞，最终把成千上万的原始候选压缩成一条可信、经过分类的待修复队列。  
  
本文是一份实操视角的指南，讲如何构建这层模型无关（model-agnostic）的中间层，重点是我们如何管理状态控制、消除误报，以及在大规模下协调端到端的分类处置。  
## 先回应两个质疑  
  
上一篇文章论证了为什么通用编码智能体干不了这活。核心问题在于：智能体一次只持有一个假设，在只覆盖真实仓库一小部分时上下文窗口就满了，然后在上下文压缩过程中丢失信息。更多细节请[阅读那篇文章]   
https://blog.cloudflare.com/cyber-frontier-models/。  
  
在继续之前，我们想先回答两个可能出现的问题。  
  
**"为什么不用子智能体（subagent），而要搞一套框架？"**  
 子智能体很有用，也是不错的起点。但安全分析需要数百项彼此独立、能跨多次运行存活、不共享上下文窗口、并且事后还能被重新划定范围、交叉引用的调查。它需要持久化、去重、可恢复，最终还需要跨全舰队的依赖追踪。这是一个编排（orchestration）问题，靠提示词解决不了。  
  
**"这篇博客是不是在给前沿模型打广告？"**  
 不是。我们的方法核心是框架，而不是模型。在漏洞发现上，我们用它配上当前最适合我们需求的前沿模型来跑。当我们把不同模型指向同一目标时，每个模型找出的 bug 份额都不一样。真正持久的是框架。如果你要自建系统，从第一天起就把它设计成模型无关的。这样你才能自由选用任何模型，不受约束。  
## 一切始于一个技能  
  
我们最初写了一个约 450 行的 security-audit  
 技能，在单个仓库上运行，反复调整提示词，直到挖出真正的 bug。后来我们加上了编排层，它最终成了整个系统的管道。真正的价值藏在提示词里，而我们的提示词至今仍几乎原封不动地保留着最初那个技能里的攻击者场景、bug 类别和反模式检测规则。  
  
这个技能被设计成在一次会话里完成 7 个阶段的审计：  
  
• 三个并行的研究智能体做侦察，写出一份 architecture.md  
。  
  
• 每个攻击类别跑一个 **Hunter**  
（猎手）智能体，尝试攻破代码，而不是审阅代码。  
  
• 对抗性校验器（validator）尝试推翻每一条发现。  
  
• 幸存的发现被写成一份人类可读的漏洞报告。  
  
• 它们同时按 schema 输出为 findings.json  
，并用机械检查验证该文件。  
  
• 最后，一个全新的智能体独立复核每一条发现，对照源码重新验证。  
  
• 通过复核、存活下来的发现被提交到入库 API。  
  
那个最初的技能几乎直接映射到了后来的框架：  
  
<table><thead><tr style="box-sizing: border-box;margin: 0;"><th align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;color: #f9f9f7;text-transform: uppercase;background-color: #111;font-size: .875em;font-weight: 700;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">技能阶段</span></span></th><th align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;color: #f9f9f7;text-transform: uppercase;background-color: #111;font-size: .875em;font-weight: 700;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">框架阶段</span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;margin: 0;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">侦察智能体写出 architecture.md</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">侦察（Recon）</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;background-color: #f5f5f3;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">每个攻击类别跑 Hunter</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">狩猎（Hunt）</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">校验器推翻发现</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">验证（Validate）</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;background-color: #f5f5f3;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">幸存的发现变成报告</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">报告（Report）</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">findings.json 只做 schema 合规的机械校验，不校验正确性</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">对发现中的行号和函数做机械校验</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;background-color: #f5f5f3;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">全新智能体复核发现</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">独立验证</span></span></td></tr></tbody></table>  
技能管用，但很快暴露出局限。看覆盖率指标，单次运行只能找到多次运行累计能找到的约一半 bug。在我们的经验里，它找到的那些还偏向更简单、更不隐蔽的类型。一旦你的流程基本变成"跑十遍然后手工 diff"，你大概就该考虑上真正的框架了。  
  
在运行和微调这个技能的过程中，我们撞上了三堵墙：  
  
• **上下文耗尽**  
：一小时之后，上下文窗口就满了，模型开始蚕食自己的记忆，瞬间忘掉它花了一上午追查的 bug。我们通过把状态完全外部化、把大语言模型（LLM）当成无状态计算引擎，才突破了这个瓶颈。  
  
• **持久性**  
：运行中途崩溃就得从头再来。因为一次 AI 限流错误或连接抖动而丢掉数小时工作，是发现"你需要更好的架构"的一种代价极其高昂的方式。  
  
• **跨仓库推理**  
：一个只盯着单个仓库的会话，对它下游消费方的应用之间的关系完全失明；而当你检查组件之间的接口时冒出来的 bug 数量，可能比很多人预想的多。  
  
**建议：**  
 一个真实但最小化的框架，只需要把侦察、狩猎、验证三个阶段存进数据库，再配上一个不能自己提交发现的独立**校验器**  
。在你手上有超过一个真正重要的仓库之前，完全可以跳过跨仓库追踪。在真正被噪声淹没之前，也可以跳过专门的去重智能体。先从开发环境里的一个技能开始，把提示词打磨好，只有当"缺少某个架构阶段"成为具体拖慢你的原因时，再去建下一个阶段。  
## 把技能固化成流水线  
  
这个领域里多数 AI 安全文章讲的都是单个仓库或精心挑选的基准（benchmark）；像这样带着跨仓库追踪、跑整个代码库舰队，我们还没见过别处写过。我们的代码库横跨极其混杂的语言——Rust、Go、C、Lua、TypeScript 和 Python，再加上各种配置管理系统、静态配置以及各类额外上下文。所以我们不得不自创一套适合我们的东西。从第一次用斜杠命令跑起来，到能覆盖 128 个不同仓库、自动发现并盘问相关依赖的舰队扫描器，一共花了大约六周。固化过程基本是机械性的：我们把技能的每个阶段提升为独立智能体，在背后放一个数据库，前面加一个编排器。映射几乎是 1:1 的。  
  
整个舰队跑在一套统一的框架上，没有任何按语言做的调优，并且能追踪仓库之间的依赖。把语法解析交给模型，让系统变得语言无关；而真正的差异化能力，是它能追踪仓库之间  
的依赖。框架本身并不关心看到的是 C 指针还是 TypeScript 文件，它关注的是安全编排的更高层逻辑。这让我们能在数百个不同代码库上规模化扩展，而不用写任何定制化的语言解析器。  
## 两阶段的漏洞研究工作流  
  
我们的整个漏洞研究工作流建立在一个两阶段的操作框架上：**漏洞发现框架**  
（Vulnerability Discovery Harness，VDH）和**漏洞验证系统**  
（Vulnerability Validation System，VVS）。  
  
VDH 是我们的发现引擎，主动扫描代码库以浮出潜在安全问题。一旦 bug 进入 VVS（它允许多个框架同时汇入），它们就会依次经过去重、研判（Judgment），最后进入修复（Fixing）阶段，我们稍后会讲。  
  
VDH 用一个模型，VVS 用完全不同的另一个模型，于是两个模型实际上在互相检查。这里面有明显的好处：强迫模型 B（VVS）去评判模型 A（VDH）的产出，意味着这条发现会被一整套完全不同的逻辑权重和训练数据来评估——它扮演一个不带偏见、对抗性的第三方，唯一职责就是无情地压力测试模型 A 的假设。在运营层面，我们也受益于把模型供应商当成可互换的商品。供应商会随时间改变温度（temperature）、缓存和推理算力预算，甚至在同一模型版本内部也会变。我们没有去构建一个依赖模型长期行为可预测的系统，而是让框架能够吸收下游的波动而不崩掉。  
## 阶段一：漏洞发现框架（VDH）  
  
上一篇文章讲过了每个智能体/阶段是干什么的，所以这里我们讲它没讲的部分：阶段之间的胶水，以及那些决定整套东西能不能跑起来的关键细节。  
  
<table><thead><tr style="box-sizing: border-box;margin: 0;"><th align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;color: #f9f9f7;text-transform: uppercase;background-color: #111;font-size: .875em;font-weight: 700;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">智能体/阶段</span></span></th><th align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;color: #f9f9f7;text-transform: uppercase;background-color: #111;font-size: .875em;font-weight: 700;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">主要职责</span></span></th><th align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;color: #f9f9f7;text-transform: uppercase;background-color: #111;font-size: .875em;font-weight: 700;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">子智能体 / 工具</span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;margin: 0;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">侦察（Recon）</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">摸清目标架构，绘制潜在威胁向量</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">3 个并行的侦察子智能体写出 architecture.md</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;background-color: #f5f5f3;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">狩猎（Hunt）</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">按攻击类别执行攻击，编译代码片段，探测二进制</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">它会派生子智能体（视模型不同，承担全舰队 9% 到 20% 的任务）。它会访问并写入愿望清单（Wishlist）工具。</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">验证（Validate）</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">先机械检查发现，再做对抗性证伪</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">分两趟：纯代码先做 schema/路径检查，再由一个隔离的单一智能体尝试推翻该发现，之后才允许归档。</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;background-color: #f5f5f3;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">补缺（Gapfill）</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">为覆盖空白的格子生成新的狩猎任务</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">对仍然看起来偏薄、测试不足的（区域 × 攻击类别）格子，投入新的狩猎任务</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">去重（Dedup）</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">识别并合并重叠的发现</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">结合确定性代码与智能体，按根因聚类发现，实时折叠合并</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;background-color: #f5f5f3;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">追踪（Trace）</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">遍历依赖图；为消费方仓库派任务</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">遍历依赖图，在每个识别出的消费方仓库内加入狩猎任务，确保跨仓库 bug 被捕获</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">反馈（Feedback）</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">从已有报告学习，优化后续运行</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">收集验证失败、浅层运行和重复漏掉的案例，立刻改写队列中的提示词，让后续任务更锋利</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;background-color: #f5f5f3;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">报告（Report）</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">渲染人类可读报告</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">只是一个脚本，不需要模型</span></span></td></tr></tbody></table>  
表 1：漏洞发现框架（VDH）  
  
第四到第八个阶段作为一个持续的"生产者—消费者"循环运行。随着初始狩猎推进，**补缺、反馈和追踪**  
智能体生成新任务；**去重**  
把重叠的发现折叠回去，循环的其余部分继续消费队列。这确保了即使在周期后段才发现的漏洞，也仍能在同一次运行内被验证、被报告，并被拿去和其他代码比对，确认它们没有同样的 bug。  
  
这样切分流水线，保证了严格的上下文控制。上下文窗口一填满，模型就开始幻觉。我们把每个智能体的职责压得极其聚焦，让上下文占用保持在总窗口的 25% 以下。一个天真的"读完所有文件  
"式做法，每一次都会突破这个限制。  
  
有一点让我们栽过跟头：持久化必须在并行化之前就考虑进去。你绝不会想因为一个意外错误就丢掉一次五小时的运行。每个阶段都写入同一个按（run_id  
、repo  
、stage  
）键控的 SQLite 数据库。任何阶段都能恢复、重试，或被拉进后续运行，而不用重做工作。发现边产生边流式保存，所以崩溃的代价只是正在处理的那一个任务，别的都不受影响。  
  
**建议：**  
 有时一次临时的 API 错误会以文本形式出现在（200 OK  
）响应流里，而不是抛出代码异常。对编排器来说，这看起来和一个干净完成的任务一模一样。你必须显式地对响应文本做分类，而不能只信异常类型，否则你会把空跑当成成功记录在案。  
### 动态威胁建模  
  
在**侦察**  
阶段，威胁模型是智能体自己写出来的，而不是别人递给它的。除了大约十来个内置攻击类别（多种注入、内存破坏、协议解析、时序侧信道等等），**侦察**  
智能体还能当场发明仓库特有的类别，每一类都有自己的方法论。它会针对该代码库量身定制一套分类法，用来更精准地划定 **Hunter**  
 智能体的范围。  
  
光读源码，不足以理解代码在压力下的行为，尤其是 C 及其他底层语言里那些微妙的未定义行为 bug。**Hunter**  
 智能体要超越代码阅读，进入主动执行。它们编译代码片段、构建小版本，然后攻击它们。质量最大的一次跃升，来自给 **Hunter**  
 一个沙箱（基于 unshare  
）去把二进制跑崩。  
  
**建议：**  
 如果框架本身跑在 Docker 里，那个沙箱需要 seccomp=unconfined  
 和 apparmor=unconfined  
，否则它会静默启动失败。这一行修复能替你省下一天挠头——如果你和我们一样，不是嵌套容器化方面的专家。  
### 微分叉与愿望清单  
  
在核心流水线阶段之外，我们加了两个专门机制，让 **Hunter**  
 拥有很大的自主权，能在不中断正在进行的分析的前提下调整关注点、申请外部资源：  
  
**兄弟分叉（Sibling Forking）**  
：当一个 **Hunter**  
 智能体撞上一条超出当前范围的有趣代码路径时，这能确保它不会跑偏。它通过一次工具调用，带着精确的结构化种子（structural seed）去分叉出一个兄弟智能体。在全舰队范围内，这大约占 9% 的任务，但比例高度依赖模型——从接近零到大约五分之一不等，取决于当时是哪个模型在狩猎。  
  
**愿望清单（Wishlist）**  
：当一个智能体需要它手上没有的工具时——通常是某个**校验器**  
要确认概念验证（PoC），或某个 **Hunter**  
 想搭点东西（比如特定构建环境、一台虚拟机、或某些生产配置文件）——它就写入一张中心化的愿望清单，提供足够的上下文，让系统在人类补齐依赖后自动重跑那个确切的任务。其中一些还能半自愈：如果容器需要带着某些改动重建，可以在一轮运行结束后，由一个通用编码框架监控日志来自主完成。  
  
自愿望清单上线以来，128 个仓库累计写入 25,472 次，它是智能体和我们对话的主要方式。就在我们写这篇文章时又收到一条："我需要一台 FreeBSD 虚拟机来端到端确认这个 PoC。"  
### 全舰队跨仓库追踪  
  
在初始清理之后，一个**追踪器**  
（Tracer）智能体会检查不同软件组件之间是如何连接的。它寻找一条特定路径：潜在攻击者能否把有害输入从外部送到系统中有漏洞的部分？如果答案是肯定的，**追踪器**  
就会在消费方仓库里自动派发新的狩猎任务。要让这工作，你需要一个统一的跨仓库符号索引和一张准确的依赖图。这让你能挖出标准单仓库扫描会漏掉的深层、系统性问题。  
  
在整个仓库舰队上跑我们的框架，揭示了两个只有规模化之后才浮现的教训。  
  
第一，去重本身就是一个问题，大到需要专门的智能体。当你只扫几个仓库时，可以人工肉眼扫一遍重叠的 bug。简单的字符串匹配或文件路径检查在这里救不了你。判断两个复杂的逻辑缺陷是否真的是同一个根因 bug，听起来很轻松，其实不然。它需要的认知推理量之大，逼得我们部署专门的**去重**  
智能体来清理噪声，还得配上它们自己的启发式规则和削减工作量的办法。  
  
第二，不要早早接入静态分析。我们把 Semgrep 一路接进了流水线，结果**Hunter**  
 在一个月的运行里一次都没调用它。它们宁愿去读代码、跑代码。相比之下，愿望清单是整个系统里使用最频繁的工具。值得留意智能体实际伸手去拿的是什么，而不是你以为它们会想要什么。  
### 让发现变得可信  
  
智能体会去改源码，好让自己的漏洞利用跑通，然后得意地报告它刚刚亲手造出来的 bug。它会写一个完全同义反复的测试，比如"exec() 会执行东西，因此是严重漏洞  
"。或者它会搭一个能跑通但啥也证明不了的利用，因为背后的威胁模型是胡扯。如果你的框架不主动对抗这些，你造出来的只是一台更快生产垃圾的机器。  
  
**Hunter**  
 在允许归档任何东西之前，必须先陈述威胁模型。它必须精确界定攻击者是谁、这个漏洞跨越了哪条边界、或打破了哪个假设。输出的 schema 排序强制了这一点。这个要求消灭了那些空洞的发现，也就是"如果用户有数据库写权限，他就能往数据库里写  
"这类。  
  
每一条被确认的发现，都必须附上一个 PoC，写成针对原始、未被改动代码库运行的测试。这防止智能体去改源文件、强行让漏洞利用成功。如果没有可用的 PoC，我们就判定这条发现是假的。实践中，那意味着一个 **Hunter**  
 编译一个三十行的解析循环，开着内存保护跑它，并证明错误的读取步长来自栈地址而不是预期的消息体。你可以自己重跑一遍。此外，每一条被确认的发现还必须附带一份建议补丁。真正进入我们评审队列的，是一个已验证的 bug、一个能跑的测试和一份可用的 git diff，而不是一段含糊的文字描述。  
  
在一条利用路径存活之前，确定性代码（用普通代码写的，不是另一个模型）会机械地验证所引用的文件和路径确实存在，并确认补丁和测试都能正确解析。这个**校验器**  
不能自己记录发现；它唯一的职责是激烈地推翻 **Hunter**  
 的理论。如果允许 **Hunter**  
 给自己批作业，它会自信地把自己产出的一切都判定为有效。  
  
我们不声称系统有某个漏报率。代码库里每一条真实 bug 并没有一个带标注的完整集合，所以任何声称的召回率（recall）数字都纯属臆测。我们能观察的是：重复运行是否持续挖出新 bug（是的），以及覆盖是否仍在跨运行增长。这都只是代理指标——你无法确知单个代码库里到底有多少 bug——但它是衡量有效性的一个够用的方式。  
## 阶段二：漏洞验证系统（VVS）  
  
从框架出来的发现只是分类处置流程的开始，所有发现都汇入一个共享的 VVS，目前它一共在 145 个仓库里存有 13,841 条发现。对这么大的量做分类，本身就是一个巨大的工程问题，重要程度不亚于狩猎。那个分类引擎跑在与发现框架不同的模型上，拆成三项不同的工作。  
  
<table><thead><tr style="box-sizing: border-box;margin: 0;"><th align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;color: #f9f9f7;text-transform: uppercase;background-color: #111;font-size: .875em;font-weight: 700;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">智能体/阶段</span></span></th><th align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;color: #f9f9f7;text-transform: uppercase;background-color: #111;font-size: .875em;font-weight: 700;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">主要职责</span></span></th><th align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;color: #f9f9f7;text-transform: uppercase;background-color: #111;font-size: .875em;font-weight: 700;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">派生 / 子智能体 / 工具</span></span></th></tr></thead><tbody><tr style="box-sizing: border-box;margin: 0;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">去重（Dedup）</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">判断该漏洞是否已在系统中，或已作为内部 Jira 工单提出</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">确定性部分：普通代码在文件、函数、信任边界和稀有 token 上构建倒排索引，然后给每条发现一份简短的候选清单 </span></span><section><span leaf=""><br/></span><span leaf=""><br/></span><span style="box-sizing: border-box;margin: 0;"><span leaf="">概率性部分：去重智能体在这份短清单上推理，跨运行稳定的键会重新打开已有记录</span></span></section></td></tr><tr style="box-sizing: border-box;margin: 0;background-color: #f5f5f3;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">研判（Judgment）</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">生产可达性与验证</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">单一智能体——通过 MCP 服务器构建关于该 bug 的上下文，以了解该服务在生产环境里长什么样。它搜索 wiki、Jira、git、配置以及所有其他可用来源，试图弄清这个 bug 是否真的适用于我们的生产环境，并据此给漏洞打分。它还会对照源码验证该 bug 在最新 main 分支上是否仍然存在。</span></span></td></tr><tr style="box-sizing: border-box;margin: 0;"><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">修复（Fixing）</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">生成补丁，跑回归测试</span></span></td><td align="left" style="box-sizing: border-box;margin: 0;text-align: left;border: 1px solid #888;padding: .5em .75em;"><span style="box-sizing: border-box;margin: 0;"><span leaf="">打补丁前后各跑一次回归测试（过滤到受影响的测试；只有在无法按测试过滤时才跑全量）。它要求在目标测试上出现干净的失败→通过翻转才能过关。如果打补丁后的测试失败，或者全量运行检测到下游回归，提交会被自动阻断并标记给人工介入。</span></span></td></tr></tbody></table>  
表 2：漏洞验证系统（VVS）  
### 去重  
  
用 LLM 把每一条发现和其余每一条都比较，复杂度是 O(N²)，在规模上完全撑不住。为了让模型离开关键路径，确定性代码在结构化数据（涉及的文件/函数、信任边界、稀有 token）上构建倒排索引，生成一份真正的简短候选清单。只有到这一步，才由一个智能体去看这份短清单，判断一个修复是否能同时关掉其中好几条。跨运行稳定的键确保重复找到的 bug 会重新打开已有记录，而不是新开一条。  
### 上下文研判  
  
研判是对幸存者做的第二道独立关卡。智能体会重新核对最新信息，从部署、环境和配置上下文中提取内容，判断该代码路径在生产中是否可达，并识别仓库负责人。这个过程把"现在就可利用"从"真实但潜伏"、以及"真实但归错了组件"里筛出来。它把一堆混乱的发现转化为一个风险驱动的编排工作流。  
### 自动修复  
  
**修复器**  
（Fixer）拿到建议补丁和单元测试，按仓库风格重写它们，应用 diff，跑针对性测试。干净的失败→通过翻转是理想情况，也是唯一会自动清理的情形；打补丁后测试失败则阻断提交。**修复器**  
从不自行合并代码；必须有人审阅分支。这道关卡是不可妥协、人在环中的安全保障，它使得变更管理合规所需的清晰、不可破坏的密码学审计链成为可能。如果放任模型随意打补丁，它会很乐意在修掉一个安全 bug 的同时，悄悄弄坏一个不相关的功能，或者引入几十个新 bug。  
  
在这三项分类工作中，每个智能体都被限制在一个狭窄任务内，外面包着确定性的记账代码，而且任何东西在人类批准一次试运行（dry run）之前都不会写入生产。虽然这条流水线把工程瓶颈从"找 bug"挪到了"审查并落地修复"，**修复器**  
仍然是系统里最年轻、最慢的部分。  
## 成本几何  
  
在仓库舰队上跑数百个智能体并不便宜，但至少开销的形态是可预测的。几乎所有的算力预算都直接投进了狩猎阶段。这让**补缺**  
成为我们控制成本与覆盖的杠杆，因为每多跑一轮，成本大约只有初次狩猎的一半。  
  
由于每个仓库的成本差异极大，我们按仓库而不是按运行来编预算。我们对每个仓库设严格的任务上限，并拉起一个 50 到 200 个 worker 的池子。这样你就能把钱花在真正能挖到东西的仓库上，而不浪费在挖不到的仓库上。  
  
这也是为什么对我们来说，大扫描是周期性的存量清扫，而不是每次 PR 都查。一次复杂仓库的全量扫描可能要花数小时；最糟的一次跑了 14 小时出头。更便宜、更小的框架才适合那种逐 PR 的场景。  
## 我们如何判断它有效  
  
我们通过追踪自动流水线把刻意的工程噪声过滤成高质量、可执行发现（finding）的效率，来衡量系统有效性。由于我们有意把 **Hunter**  
 调成对可能被串联成更大攻击的细微原语过度上报，真正的成功指标是：在数据到达人类之前，我们能把这堆原始数据精炼到什么程度。  
  
为衡量这点，我们持续追踪有多少原始发现能通过每一道验证关卡。得益于侦察阶段更好的上下文注入，我们的初始验证拒绝率从 40% 降到 11%，而高完整性发现的占比从 35% 升到 58%（约代表 12,057 条生命周期内发现）。  
  
以下是本文写作时，从原始候选到可执行发现的全生命周期分解。  
  
  
![BLOG-3344 2](https://mmbiz.qpic.cn/sz_mmbiz_jpg/iauzMfibvvF2dsZhUHbiaIbq4LLte5PllXbMiaibDcNoyvfIgVBnDx9szibibcIfL650aACT9Qwf5Cyo9IhzHay25poa62L2FpLEico6COE1icZP1WB4/640?wx_fmt=jpeg&from=appmsg "")  
  
BLOG-3344 2  
  
  
**漏洞发现框架（VDH）**  
  
• **原始候选（Raw candidates）：**  
 发现框架在独立验证之前产出的全部内容。  
  
• **需要复现（Needs repro）：**  
 看起来合理、但需要人工复现才可信的发现。  
  
• **验证阶段被拒（Rejected at validation）：**  
 校验器推翻了威胁模型、利用路径、受影响代码或证据。  
  
• **重复（Duplicates）：**  
 被折叠到同一框架产生的另一条发现上的候选。  
  
• **通过验证（Survived validation）：**  
 通过独立验证关卡、进入 VVS 的发现。  
  
• **流向别处的 bug（Bugs that went elsewhere）：**  
 被有意绕开这条流程的发现。  
  
**漏洞验证系统（VVS）**  
  
• **另一个漏洞框架（Another vulnerability harness）：**  
 汇入同一验证系统的其他自动化来源。  
  
• **系统内 bug 总数（Total bugs in system）：**  
 入库后的合并池。  
  
• **重复（Duplicates）：**  
 去重环节判定为已被另一条标准发现或工单覆盖的发现。  
  
• **错误仓库 / 其他 / 非风险（Wrong repo / other / not a risk）：**  
 噪声桶：归错对象的发现、纵深防御项，或潜伏风险。  
  
• **发送给团队的 bug（Bugs sent to teams）：**  
 已定稿、干净、可送修（remediation）的发现。  
  
• **判定为互联网可利用（Judged Internet-exploitable）：**  
 高紧迫性发现，真实攻击者能在生产中触发。  
  
• **未判定为互联网可利用（Not judged Internet-exploitable）：**  
 紧迫性较低但仍可执行的 bug（生产问题、依赖风险或配置错误）。  
  
• **最终严重度划分（Final severity split）：**  
 用于给工程团队排优先级的分类方式。  
  
这套框架的核心指标不是某个臆测的召回分数——而是让摆在真人面前的未确认发现数量尽可能接近零。架构必须是一条不留情面的过滤漏斗。  
  
• 在 VDH 生成的 **20,799**  
 个原始候选中，只有约 12,057 个通过了验证。  
  
• 当它们被推入 VVS、与另一个框架的发现汇合后，中心池达到 **13,841**  
 条。  
  
• **去重**  
智能体折叠掉了 **5,442**  
 条重复发现。  
  
• **1,154**  
 条被以"错误仓库"或"低风险"路由进队列，并在合适的地方被回收进系统。  
  
• 最终留下 **7,245**  
 条可执行发现供工程团队处理。  
  
传统合规规则完全基于静态 CVSS 分数来规定任意的修复时限（例如"30 天内修完所有高危"）。我们的上下文研判层把这个合规复选框变成了真正的风险管理。  
  
这套架构能把发现追踪回它们的源头，意味着修复一个根因就能解掉一整簇发现，而不只是逐个打补丁。VDH 的系统表现还通过把仓库切分成（区域 × 攻击类别）格子来衡量，并迭代运行**补缺**  
智能体，直到它不再产出发现为止。每当我们更新某个底层提示词，我们会在一个留出的仓库上测试它，看覆盖格子的总数是否真的发生了变化。  
  
框架还接了自动化健康信号，用来在流水线早期捕获系统故障。如果一次狩猎结束得快得可疑，且没有派生出子狩猎或补缺任务，这通常意味着某个依赖崩了，而不是代码库很干净。为了补救，系统会把任何零发现收尾的 **Hunter**  
 智能体标记为"浅层（shallow）"，并立刻重新入队跑一次。  
  
最后，前面描述的独立分类关卡强化了系统的稳健性。用不同的模型和不同的逻辑权重重新研判所有提交，我们确保了一种不带偏见、对抗性的验证，它与用于发现的具体模型解耦，从而提供一层无论用哪个模型都持续有效的信任层。  
  
这一切都没做完。我们一直在改系统，它离完美科学差得远。但如今原始候选发现很便宜，唯一值得做的，是把它们变成可靠、可验证的代码修复。  
  
搭建你自己的框架，意味着接受 AI 模型是易变的，但你的编排层不必如此。通过把安全逻辑与任何单一供应商解耦、强制对抗性验证、自动化分类流水线，你就能把一座 LLM 噪声之山，变成一台可靠的、覆盖全舰队的防御引擎。  
### 我们的"北极星"指标：衡量真实世界的速度  
  
每个代码库都略有不同，为了展示这在现实中如何运作，我们基于一次标准仓库运行，画出了一份贴近实际的基准。请记住，这代表的只是单个仓库上的单次运行；随着时间推移，当持续的舰队级循环不断去重、过滤、回收发现，它会把生命周期内的候选总量削减大约 65%。  
  
**通过自动打补丁节省的工程小时数：**  
 我们不盯着静态基线，而是用技术吞吐量、处理速度，以及它消除人工分类瓶颈的能力来衡量流水线的健康度：  
  
• **初始验证裁剪：**  
 对一个标准仓库（约 3 万行代码），这会产出 100 条初始发现，完整运行耗时 3-4 小时，全程保持高度聚焦的上下文窗口。  
  
• **压缩：**  
 去重层与上下文研判层并行处理这些候选。在 3 小时内，系统把这一批发现从约 100 条原始候选压缩、精炼成 80 个不同的高保真 bug。  
  
• **修复：**  
 自动修复器以平均每个 bug 5 分钟的速度处理这 80 个不同 bug。总计下来，系统大约 14 小时内就能完成发现、验证、去重，并开出可用的拉取请求（pull request）。  
  
**缩短严重缺陷的平均解决时间（MTTR）：**  
 当然，你不能一次性把 80 个补丁全灌进生产而不出事。为保证部署安全，我们用分级放量：  
  
• **严重暴露遏制（Critical Exposure Containment）：**  
 系统把严重、高危、可利用的 bug（80 个里平均约 10 个）隔离出来。我们把这些快速通道交给人工审查，并引入发布周期，在 5 天内于生产环境完成修复。  
  
• **增量加固（Incremental Hardening）：**  
 其余潜伏风险、次要配置异常和低紧迫性 bug，在 15-20 天的窗口内增量滚动进生产，以保证平台稳定性。  
### 我们如何处理这一大堆补丁  
  
这些发现来自一个隔离、划界的研究实验，目的是对代码做压力测试。它们不代表我们线上生产环境中现存的、未修复的漏洞。  
  
由于框架在我们的测试环境里持续运行，等你读到这篇文章时，这些具体数字早就过时了。流水线浮出的每一个 bug 都附带一个能演示它的可运行测试用例和一份补丁草稿。我们的安全团队正在系统地处理这些报告并应用必要的修复，也就是说，你每天使用的 Cloudflare 产品已经针对这些向量进行了主动加固。  
  
随这篇博客一起，我们开源了当初用来开发这套框架的初始技能；发布前做了一点清理，让它更易理解和集成，但技能本身基本保持不变。希望框架本身不久后也能跟上。它可以成为你自己漏洞框架、你自己技能的起点，或者任何最适合你需求的起点：[github.com/cloudflare/security-audit-skill]   
https://github.com/cloudflare/security-audit-skill  
  
**如果你对网络安全、红队攻防技术充满热情，渴望学习更多实战技巧，例如渗透测试、自动化脚本编写、免杀技术等， 欢迎关注我的公众号**  
******在这里，我会持续分享更多高质量的技术文章，与你一同探索网络安全的奥秘，提升实战技能！ 让我们一起在队攻防的道路上，不断精进，突破边界！**  
> **免责声明：**  
 本文仅供安全技术研究与学习交流之用。 严禁将本文所提及的技术用于任何非法用途，包括但不限于未经授权的渗透测试、网络攻击、恶意代码传播等。  
  
  
