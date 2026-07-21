---
title: 让 AI 大模型少写、写短、更稳：ponytail × caveman 双开实战
date: 2026-07-21T10:58:39.000Z
tags: []
categories: []
description: 基于 ponytail 的 “YAGNI 阶梯” 与 caveman 的 “嘴小脑大”，把 “写什么” 与 “怎么说” 拆开治理：让 AI agent 少写代码、写短回复、不乱堆依赖、不丢安全。
---

      <h2 id="背景"><a href="#背景" class="headerlink" title="背景"></a>背景</h2><p>用大模型写代码久了，会撞上两个老毛病：</p>
<ol>
<li><strong>过度工程</strong>——让你加一个日期选择，agent 给你装了 <code>flatpickr</code>、写了 50 行 wrapper、讨论起时区；</li>
<li><strong>话太多</strong>——一个 bug 修复，前面先写三段“很高兴帮你排查”。</li>
</ol>
<p>前者污染仓库、后者吃 token。两件事表面无关，其实来自同一个根：<strong>没有人约束 agent 少写、写短</strong>。今天介绍的两个开源 skill，就是分别从“两端”切这个问题：</p>
<ul>
<li><a href="https://github.com/dietrichgebert/ponytail" target="_blank" rel="noopener">DietrichGebert/ponytail</a>：管<strong>“写什么”</strong>，强迫 agent 走 YAGNI 阶梯；</li>
<li><a href="https://github.com/JuliusBrussee/caveman" target="_blank" rel="noopener">JuliusBrussee/caveman</a>：管<strong>“怎么说”</strong>，把回复压成“嘴小脑大”的 caveman 语。</li>
</ul>
<p>组合起来，就是 agent 不乱写 + 输出更精炼。下面把两个工具拆开讲一遍。</p>
<h2 id="一、ponytail：管“写什么”"><a href="#一、ponytail：管“写什么”" class="headerlink" title="一、ponytail：管“写什么”"></a>一、ponytail：管“写什么”</h2><h3 id="1-1-它是谁"><a href="#1-1-它是谁" class="headerlink" title="1.1 它是谁"></a>1.1 它是谁</h3><p>ponytail 的口号是 <em>He says nothing. He writes one line. It works.</em>——一个留长马尾、戴椭圆眼镜的老 senior dev：你给他 50 行代码，他看一眼，换成一行。</p>
<p>它的 <code>AGENTS.md</code> 里写得很直白：<strong>写之前先走一遍 YAGNI 阶梯</strong>，停在第一个能站的台阶上：</p>
<figure class="highlight plaintext"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div><div class="line">7</div></pre></td><td class="code"><pre><div class="line">1. 这东西真的需要存在吗？   → 不需要：跳过（YAGNI）</div><div class="line">2. 这个仓库里已经有了吗？  → 复用，别重写</div><div class="line">3. 标准库能做吗？          → 用标准库</div><div class="line">4. 平台原生特性？          → 用原生</div><div class="line">5. 已装的依赖里能找到吗？  → 用依赖</div><div class="line">6. 一行能搞定吗？          → 一行</div><div class="line">7. 只有上面都不行，才写“最小可工作”的实现</div></pre></td></tr></table></figure>
<p>关键一句是 <strong>“lazy, not negligent”</strong>——验证、错误处理、安全、可访问性永远不上断头台。换句话说：</p>
<blockquote>
<p>少写 = 只删掉不需要的代码。<br>不能砍错的东西：trust-boundary 校验、数据丢失兜底、security、a11y。</p>
</blockquote>
<h3 id="1-2-量化效果"><a href="#1-2-量化效果" class="headerlink" title="1.2 量化效果"></a>1.2 量化效果</h3><p>ponytail 自己在 <a href="https://github.com/dietrichgebert/ponytail/blob/main/benchmarks/results/2026-06-18-agentic.md" target="_blank" rel="noopener">agentic benchmark</a> 里做了对照实验——同一个 Claude Code agent、有/无 ponytail skill、改 <a href="https://github.com/fastapi/full-stack-fastapi-template" target="_blank" rel="noopener">tiangolo/full-stack-fastapi-template</a>（真实 FastAPI + React 仓库）、12 个 feature ticket、n=4、Haiku 4.5：</p>
<table>
<thead>
<tr><th>vs 无 skill 基线</th><th style="text-align:right">LOC</th><th style="text-align:right">tokens</th><th style="text-align:right">cost</th><th style="text-align:right">time</th><th style="text-align:right">safe</th></tr>
</thead>
<tbody>
<tr><td><strong>ponytail</strong></td><td style="text-align:right"><strong>-54%</strong></td><td style="text-align:right"><strong>-22%</strong></td><td style="text-align:right"><strong>-20%</strong></td><td style="text-align:right"><strong>-27%</strong></td><td style="text-align:right">100%</td></tr>
<tr><td>caveman（控制组）</td><td style="text-align:right">-20%</td><td style="text-align:right">+7%</td><td style="text-align:right">+3%</td><td style="text-align:right">+2%</td><td style="text-align:right">100%</td></tr>
<tr><td>“YAGNI + one-liners”裸 prompt</td><td style="text-align:right">-33%</td><td style="text-align:right">-14%</td><td style="text-align:right">-21%</td><td style="text-align:right">-30%</td><td style="text-align:right">95%</td></tr>
</tbody>
</table>
<p>几个值得注意的点：</p>
<ul>
<li>ponytail 是唯一一个<strong>四个维度都压</strong>、安全还守住 100% 的方案；</li>
<li>越是有“过度工程陷阱”的任务（比如加个 date picker），削得越狠——benchmark 里 date picker 从 404 行降到 23 行、color picker 从 287 行降到 23 行；</li>
<li>已经在极限最小化的代码，ponytail 几乎不动；</li>
<li>裸 prompt（自己写一段“少写点”）能省代码，但安全性降到 95%，相当于把 validation / error handling 一起砍了。</li>
</ul>
<h3 id="1-3-怎么用"><a href="#1-3-怎么用" class="headerlink" title="1.3 怎么用"></a>1.3 怎么用</h3><p>安装一条命令：</p>
<figure class="highlight bash"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div></pre></td><td class="code"><pre><div class="line">codex plugin marketplace add DietrichGebert/ponytail</div><div class="line">codex plugin add ponytail@ponytail</div></pre></td></tr></table></figure>
<p>Claude Code / Copilot CLI / Gemini CLI / Pi / OpenCode / Hermes / CodeWhale / Swival / Devin / OpenClaw / Qoder 都在 README 里给了对应命令。一次性安装，每会话自动激活。</p>
<p>四个级别：</p>
<table>
<thead>
<tr><th>级别</th><th>适用</th></tr>
</thead>
<tbody>
<tr><td><code>lite</code></td><td>项目本来就干净，怕“压太狠”</td></tr>
<tr><td><code>full</code>（默认）</td><td>日常开发主力</td></tr>
<tr><td><code>ultra</code></td><td>code review、清理历史债、被仓库“冒犯”时</td></tr>
<tr><td><code>off</code></td><td>头脑风暴、教学、刻意留白</td></tr>
</tbody>
</table>
<p>切换命令是 <code>/ponytail &lt;level&gt;</code>，可通过 <code>PONYTAIL_DEFAULT_MODE</code> 环境变量锁定每会话默认级别。</p>
<p>配套的 skill 也值得用：</p>
<ul>
<li><code>/ponytail-review</code>：PR review 一句话一条，<code>L42: 🔴 bug: ...</code> 格式；</li>
<li><code>/ponytail-audit</code>：全仓盘点“可以删/简化/换成标准库”的点；</li>
<li><code>/ponytail-debt</code>：把所有 <code>ponytail:</code> 注释收回成一份债表，避免“later means never”；</li>
<li><code>/ponytail-gain</code>：打出实际收益成绩单（代码量/成本/速度），可直接对外汇报。</li>
</ul>
<p>整个仓库只有一个 <code>AGENTS.md</code> 加几个 skill——这就是 ponytail 的特点：<strong>规则用文字写清楚，工程量很小</strong>。</p>
<h2 id="二、caveman：管“怎么说”"><a href="#二、caveman：管“怎么说”" class="headerlink" title="二、caveman：管“怎么说”"></a>二、caveman：管“怎么说”</h2><h3 id="2-1-它是谁"><a href="#2-1-它是谁" class="headerlink" title="2.1 它是谁"></a>2.1 它是谁</h3><p>caveman 的口号是 <em>why use many token when few do trick</em>——脑大嘴小：</p>
<blockquote>
<p>Shrinks what the agent <strong>says</strong>, not what it <strong>knows</strong>.</p>
</blockquote>
<p>对比下面这两段（来自 README）：</p>
<table>
<thead>
<tr><th width="50%">🗣️ 普通 agent（69 tokens）</th><th width="50%">🦬 caveman agent（19 tokens）</th></tr>
</thead>
<tbody>
<tr><td>The reason your React component is re-rendering is likely because you’re creating a new object reference on each render cycle…</td><td>New object ref each render. Inline object prop = new ref = re-render. Wrap in <code>useMemo</code>.</td></tr>
<tr><td>Sure! I’d be happy to help you with that. The issue you’re experiencing is most likely caused by your authentication middleware…</td><td>Bug in auth middleware. Token expiry check use <code>&lt;</code> not <code>&lt;=</code>. Fix:</td></tr>
</tbody>
</table>
<p>同一个修复，三分之一字数，技术信息没掉一条。</p>
<h3 id="2-2-量化效果"><a href="#2-2-量化效果" class="headerlink" title="2.2 量化效果"></a>2.2 量化效果</h3><p>caveman 自己 benchmark 的 10 个任务、Claude API 实测：</p>
<table>
<thead>
<tr><th>指标</th><th style="text-align:right">数值</th></tr>
</thead>
<tbody>
<tr><td>Output tokens 平均减少</td><td style="text-align:right"><strong>65%</strong></td></tr>
<tr><td>Input tokens 减少</td><td style="text-align:right">0%</td></tr>
<tr><td>技术准确率</td><td style="text-align:right">100%（无技术丢失）</td></tr>
<tr><td>区间</td><td style="text-align:right">22%–87%</td></tr>
</tbody>
</table>
<p>另外一篇 2026-03 的论文 <a href="https://arxiv.org/abs/2604.00025" target="_blank" rel="noopener">Brevity Constraints Reverse Performance Hierarchies in Language Models</a>，测了 31 个模型，发现<strong>强制简短</strong>在某些 benchmark 上能让准确率提升约 <strong>26 个点</strong>。换句话说：</p>
<blockquote>
<p>对大模型来说，<em>短，往往比长更对</em>。</p>
</blockquote>
<h3 id="2-3-六个级别"><a href="#2-3-六个级别" class="headerlink" title="2.3 六个级别"></a>2.3 六个级别</h3>
<table>
<thead>
<tr><th>级别</th><th>举例</th></tr>
</thead>
<tbody>
<tr><td><em>normal agent</em></td><td>You should wrap the object in <code>useMemo</code>, since a new reference is created on every render.</td></tr>
<tr><td><code>lite</code></td><td>Wrap object in <code>useMemo</code>. New ref created every render.</td></tr>
<tr><td><code>full</code>（默认）</td><td>New ref each render. Wrap object in <code>useMemo</code>.</td></tr>
<tr><td><code>ultra</code></td><td>New ref/render. <code>useMemo</code> it.</td></tr>
<tr><td><code>wenyan-lite</code></td><td>每渲新建引用，<code>useMemo</code> 包之。</td></tr>
<tr><td><code>wenyan</code></td><td>每渲引用异，宜 <code>useMemo</code> 之。</td></tr>
<tr><td><code>wenyan-ultra</code></td><td>引异。宜 <code>useMemo</code>。</td></tr>
</tbody>
</table>
<p><code>wenyan</code> 系列是文言文——同一个意思字符更密，适合中文场景下追求极致 token 效率。caveman 不会翻译你的语言，只是压<em>风格</em>。</p>
<h3 id="2-4-byte-preserved"><a href="#2-4-byte-preserved" class="headerlink" title="2.4 byte-preserved"></a>2.4 byte-preserved</h3><p>这是 caveman 一个被低估的卖点：</p>
<ul>
<li>代码、命令、URL、错误信息<strong>逐字符保留</strong>，不会“优化”成错的；</li>
<li>不翻译语言，只压风格；</li>
<li>压缩是显式的，不是“悄悄改了”。</li>
</ul>
<p>所以你 review caveman 的输出，不会出现“明明写了 <code>useMemo</code>，压缩后变成了 <code>memo</code>”这种事故。</p>
<h3 id="2-5-配套工具"><a href="#2-5-配套工具" class="headerlink" title="2.5 配套工具"></a>2.5 配套工具</h3>
<ul>
<li><code>/caveman [lite|full|ultra|wenyan...]</code>：切换级别；</li>
<li><code>/caveman-commit</code>：Conventional Commit，subject ≤50 字；</li>
<li><code>/caveman-review</code>：PR review 一行一条；</li>
<li><code>/caveman-stats</code>：本会话真实 token 节省、累计节省、USD 折算；</li>
<li><code>/caveman-compress &lt;file&gt;</code>：把 <code>CLAUDE.md</code>、<code>AGENTS.md</code> 这类 memory 文件整体压 ~46%，<strong>每个后续 session 都省</strong>；</li>
<li><code>caveman-shrink</code>：MCP 中间件，包一层 MCP server，把工具描述也压掉。</li>
</ul>
<p>README 里对收益有个很诚实的补充：</p>
<blockquote>
<p>caveman 只压 output。input 不变，skill 本身每轮加 ~1–1.5k input tokens。所以极度简短任务可能净亏，但 readability 和速度都赢。</p>
</blockquote>
<p>这是关键判断依据——<strong>caveman 的真正价值是“更好读、更快”，省钱是副产品</strong>。</p>
<h2 id="三、双开组合拳"><a href="#三、双开组合拳" class="headerlink" title="三、双开组合拳"></a>三、双开组合拳</h2><p>两个 skill 做的事互不重叠：</p>
<table>
<thead>
<tr><th>skill</th><th>管的是</th><th>出问题</th></tr>
</thead>
<tbody>
<tr><td>ponytail</td><td>“这行<strong>该不该写</strong>”</td><td>代码量爆炸、依赖膨胀</td></tr>
<tr><td>caveman</td><td>“这行<strong>该怎么写</strong>”</td><td>回复啰嗦、token 浪费</td></tr>
</tbody>
</table>
<h3 id="3-1-推荐默认"><a href="#3-1-推荐默认" class="headerlink" title="3.1 推荐默认"></a>3.1 推荐默认</h3><p>主力档：<code>ponytail full</code> + <code>caveman full</code>。</p>
<ul>
<li>ponytail 把“能省就省”的代码压下去，但安全/校验不动；</li>
<li>caveman 把“能短就短”的回复压下去，但代码/URL 不动。</li>
</ul>
<p>重型 code review、清理历史债：<code>ponytail ultra</code> + <code>caveman ultra</code>。</p>
<h3 id="3-2-团队落地清单"><a href="#3-2-团队落地清单" class="headerlink" title="3.2 团队落地清单"></a>3.2 团队落地清单</h3>
<ol>
<li><strong>AGENTS.md 顶部贴 ponytail 阶梯</strong>：把 7 条 YAGNI ladder 放第一条 prompt 永远可见的位置，等于把规则“焊死”；</li>
<li><strong>PR 模板加 skill 引用</strong>：reviewer 勾选是否跑了 <code>/ponytail-review</code> 和 <code>/caveman-review</code>，挡掉“讲不清为什么有这段代码”的 PR；</li>
<li><strong>长 memory 文件定期 <code>/caveman-compress</code></strong>：CLAUDE.md / AGENTS.md 每长到一定行数压一次，长期看是 input 端最大的节省；</li>
<li><strong>CI 加 <code>/ponytail-audit</code> 兜底</strong>：每周跑一次全仓盘点，发现“可以删的代码”开 issue；</li>
<li><strong>新人 onboarding 第一天</strong>：告诉他这两条 prompt，<code>/ponytail</code> + <code>/caveman</code>，比让 senior 带看一周更高效。</li>
</ol>
<h2 id="四、什么时候不双开"><a href="#四、什么时候不双开" class="headerlink" title="四、什么时候不双开"></a>四、什么时候不双开</h2><p>不是所有场景都适合双开，给三个判断：</p>
<ul>
<li><strong>一次性实验 / 教学</strong>：单 caveman 即可，让 agent 解释清楚，ponytail 反而挡思路；</li>
<li><strong>安全 / 合规重写</strong>：单 ponytail，让结构清晰优先，避免“短到看不清”；</li>
<li><strong>闲聊 / 头脑风暴</strong>：两者都关（<code>/ponytail off</code> + <code>/caveman off</code>），保留发散空间。</li>
</ul>
<h2 id="五、结语"><a href="#五、结语" class="headerlink" title="五、结语"></a>五、结语</h2><p>少写 ≠ 偷工减料，写短 ≠ 说不出话。</p>
<p>ponytail 让 agent 回到 <em>该不该</em>，caveman 让 agent 回到 <em>够不够</em>。两个 skill 加起来，就是让 agent 输出回归“刚好够用”——这是 senior dev 该有的样子：</p>
<blockquote>
<p>He says nothing. He writes one line. It works.</p>
</blockquote>
<p>参考资料：</p>
<ul>
<li><a href="https://github.com/dietrichgebert/ponytail" target="_blank" rel="noopener">DietrichGebert/ponytail</a></li>
<li><a href="https://github.com/dietrichgebert/ponytail/blob/main/benchmarks/results/2026-06-18-agentic.md" target="_blank" rel="noopener">ponytail agentic benchmark</a></li>
<li><a href="https://github.com/JuliusBrussee/caveman" target="_blank" rel="noopener">JuliusBrussee/caveman</a></li>
<li><a href="https://arxiv.org/abs/2604.00025" target="_blank" rel="noopener">Brevity Constraints Reverse Performance Hierarchies in Language Models</a></li>
</ul>

