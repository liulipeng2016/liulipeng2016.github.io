---
title: 写代码前先写 spec：OpenSpec × Superpowers × Spec-Kit 三方对比 + 决策矩阵
date: 2026-07-21T11:57:56.000Z
tags:
  - AI
  - 工具
categories: []
description: spec-driven development 工具对比：什么时候该上 OpenSpec / Superpowers / Spec-Kit，什么时候 Claude Code 或 Codex 的 plan mode 就够。配 4×4 决策矩阵 + 桥接器生态点。
---

<h2 id="问题"><a href="#问题" class="headerlink" title="问题"></a>一、问题</h2><p>Claude Code、Codex、Cursor 都内置 plan mode。再叠一层 spec 工具解决什么？两件事：</p>
<ol>
<li>跨会话持久化——plan mode 输出关掉会话就没了，spec 工具写到 <code>openspec/</code>、<code>.specify/</code>、<code>.superpowers/</code> 这些目录里</li>
<li>跨仓协作契约——一个 feature 跨多个仓，平台团队先发 spec，产品仓 agent 读 spec 干活</li>
</ol>
<p>这跟 <a href="/2026/07/21/ponytail-caveman/">上一篇</a> 聊的 ponytail × caveman 不冲突：那两个管"agent 写什么、怎么说"，spec 工具管"agent 跟人、跟别的 agent 怎么谈需求"。</p>
<h2 id="三个工具"><a href="#三个工具" class="headerlink" title="三个工具"></a>二、三个工具</h2>
<pre class="mermaid">flowchart LR
  A["<b>explore</b><br/>跟 agent 边聊边<br/>收敛需求"] --> B["<b>propose</b><br/>生成 proposal.md<br/>specs/ design.md<br/>tasks.md"] --> C["<b>apply</b><br/>按 tasks.md 干活"] --> D["<b>archive</b><br/>收档，specs<br/>合回主干"]
  A -.不写代码.-> A
  B -.人 review.-> B
  C -.每个 task 可原子提交.-> C
</pre>
<h3 id="2-1-OpenSpec"><a href="#2-1-OpenSpec" class="headerlink" title="2.1 OpenSpec"></a>2.1 OpenSpec</h3><p>目录级别的工作流。每个 change 一个目录，4 个 markdown 文件：</p>
<figure class="highlight bash"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div></div></td><td class="code"><pre><div class="line">/opsx:explore add-dark-mode   <span class="comment"># 跟 agent 边聊边收敛需求</span></div><div class="line">/opsx:propose add-dark-mode  <span class="comment"># 产出 proposal.md / specs/ / design.md / tasks.md</span></div><div class="line">/opsx:apply                  <span class="comment"># 按 tasks.md 干活</span></div><div class="line">/opsx:archive                <span class="comment"># 收档，specs 合回主干</span></div></pre></td></tr></table></figure>
<p>参数：</p>
<ul>
<li><code>npm i -g @fission-ai/openspec</code>，无 Python / uv</li>
<li>25+ agent 支持（Claude Code / Codex / Cursor / Copilot / Gemini / OpenCode）</li>
<li>specs 是 markdown：<code>## ADDED Requirements</code> + <code>#### Scenario: ...</code></li>
<li>2026 年出 <strong>Stores</strong>（beta）：specs 单独放一个仓，跨 repo 共享</li>
</ul>
<h3 id="2-2-Superpowers"><a href="#2-2-Superpowers" class="headerlink" title="2.2 Superpowers"></a>2.2 Superpowers</h3>
<pre class="mermaid">flowchart TD
  S1["<b>1. brainstorming</b><br/>苏格拉底式逼问<br/>敲成 design doc"] --> S2["<b>2. using-git-worktrees</b><br/>拉新分支建隔离工作区"]
  S2 --> S3["<b>3. writing-plans</b><br/>切 2-5 分钟一档任务<br/>每条带 路径+代码+验证"]
  S3 --> S4["<b>4. subagent-driven-development</b><br/>or<br/><b>executing-plans</b>"]
  S4 --> S5["<b>5. test-driven-development</b><br/>RED → GREEN → REFACTOR"]
  S5 --> S6["<b>6. requesting-code-review</b><br/>spec compliance + code quality<br/>critical 阻塞"]
  S6 --> S7["<b>7. finishing-a-development-branch</b><br/>verify · merge/PR/keep/discard"]
  S6 -.未通过.-> S4
</pre>
<p>14 个 skill，强制跑——agent 接任务前必须先 check 相关 skill：</p>
<ol>
<li>brainstorming——把粗想法敲成 design doc</li>
<li>using-git-worktrees——拉新分支建隔离工作区</li>
<li>writing-plans——切 2-5 分钟一档的任务，每条带文件路径 + 完整代码 + 验证步骤</li>
<li>subagent-driven-development / executing-plans——派 subagent 一条一条干，每条两段 review（先 spec compliance，再 code quality）</li>
<li>test-driven-development——RED-GREEN-REFACTOR</li>
<li>requesting-code-review——任务之间 review，critical 阻塞</li>
<li>finishing-a-development-branch——verify、merge / PR / keep / discard</li>
</ol>
<p>跟 OpenSpec 的区别：OpenSpec 出 proposal.md，Superpowers 出"被自动执行的工序"。OpenSpec 管 spec，Superpowers 管 TDD / worktree / code review / 收尾这一整套工程纪律。</p>
<h3 id="2-3-Spec-Kit"><a href="#2-3-Spec-Kit" class="headerlink" title="2.3 Spec Kit"></a>2.3 Spec Kit</h3>
<pre class="mermaid">flowchart TD
  K0["<b>constitution</b><br/>立项目宪法<br/>质量/测试/合规基线"] --> K1["<b>specify</b><br/>写 what &amp; why"]
  K1 -.clarify.-> K1
  K1 --> K2["<b>plan</b><br/>选技术栈 · 出架构"]
  K2 --> K3["<b>tasks</b><br/>切任务列表"]
  K3 -.analyze.-> K3
  K3 -.checklist.-> K3
  K3 --> K4["<b>implement</b><br/>按任务干活"]
  K4 -.taskstoissues.-> K3
  K4 -.converge.-> K3
</pre>
<p>GitHub 2025 下半年推的。命令链：</p>
<figure class="highlight bash"><table><tr><td class="gutter"><pre><div class="line">1</div><div class="line">2</div><div class="line">3</div><div class="line">4</div><div class="line">5</div><div class="line">6</div></div></td><td class="code"><pre><div class="line">/speckit.constitution   <span class="comment"># 立项目宪法</span></div><div class="line">/speckit.specify        <span class="comment"># 写 what &amp; why</span></div><div class="line">/speckit.plan           <span class="comment"># 选技术栈、出架构</span></div><div class="line">/speckit.tasks          <span class="comment"># 切任务</span></div><div class="line">/speckit.implement      <span class="comment"># 干活</span></div><div class="line">/speckit.taskstoissues  <span class="comment"># 推 GitHub Issues</span></div></pre></td></tr></table></figure>
<p>可选命令：<code>clarify</code>（补 spec 漏洞）、<code>analyze</code>（跨产物一致性）、<code>checklist</code>（出 quality checklist）、<code>converge</code>（盘点未完成项追加为新 task）。</p>
<p>Spec Kit 的杀手锏是三层扩展：</p>
<ul>
<li><strong>Extensions</strong>——加新命令、新工作流（接 Jira、V-Model 测试追踪）</li>
<li><strong>Presets</strong>——换 spec / plan / task 的模板（DDD 术语、合规章节）</li>
<li><strong>Bundles</strong>——一组 extension + preset + workflow 打包，角色化一键装（PM / 安全 / 业务分析 / 开发）</li>
</ul>
<p>OpenSpec / Superpowers 没这一层。30+ agent 集成。Python + uv 装。</p>
<h2 id="三-agent-视角"><a href="#三-agent-视角" class="headerlink" title="三 agent 视角"></a>三、agent 视角</h2><h3 id="3-1-Claude-Code"><a href="#3-1-Claude-Code" class="headerlink" title="3.1 Claude Code"></a>3.1 Claude Code</h3><p>ExitPlanMode 单会话内有效，关闭就丢。装 spec 工具等于把 plan 模式做成可复用资产：<code>/opsx:propose</code> 像加强版 plan mode，先写 proposal.md，等你 review 再继续。</p>
<h3 id="3-2-Codex"><a href="#3-2-Codex" class="headerlink" title="3.2 Codex"></a>3.2 Codex</h3><p>Codex CLI 的 <code>--plan</code> / <code>--apply</code> 跟 Claude Code 一样，plan 是会话内资产。但 Codex 没有 Claude Code 那种 always-on hooks，所以装 OpenSpec / Superpowers 在 Codex 上特别有用——是"流程上锁"的几乎唯一手段。Superpowers 对 Codex 是 marketplace 一等公民。</p>
<h3 id="3-3-Cursor"><a href="#3-3-Cursor" class="headerlink" title="3.3 Cursor"></a>3.3 Cursor</h3><p>Plan Mode 是 Composer 的开关，更碎片化：跟着当前 Composer 窗口走，切 tab 就丢。在 Cursor 上跑 spec 工具的收益最大——Cursor 原生 plan 状态最容易散，<code>.superpowers/</code> 目录会替你把"刚才 plan 到哪儿了"持久化。</p>
<h3 id="3-4-小结"><a href="#3-4-小结" class="headerlink" title="3.4 小结"></a>3.4 小结</h3><p>Plan mode 是短时记忆，spec 工具是长期档案 + 团队契约。一次性活、不跨仓、不跨会话，plan mode 够。要"三周后能复现"或"三个仓照着做"，上 spec 工具。</p>
<h2 id="四-决策矩阵"><a href="#四-决策矩阵" class="headerlink" title="四、决策矩阵"></a>四、决策矩阵</h2><p>横轴：变更频率。纵轴：团队规模。格子里的推荐：</p>
<table>
<thead>
<tr><th>规模 ↓ \ 频率 →</th><th style="text-align:center">一次性</th><th style="text-align:center">偶发 feature</th><th style="text-align:center">持续迭代</th><th style="text-align:center">大爆发</th></tr>
</thead>
<tbody>
<tr><td><strong>小（1-2 人）</strong></td><td style="text-align:center">不需要</td><td style="text-align:center">ponytail + caveman</td><td style="text-align:center">OpenSpec</td><td style="text-align:center">OpenSpec + Superpowers</td></tr>
<tr><td><strong>中（3-10 人）</strong></td><td style="text-align:center">OpenSpec</td><td style="text-align:center">OpenSpec</td><td style="text-align:center">OpenSpec + Superpowers</td><td style="text-align:center">Spec Kit</td></tr>
<tr><td><strong>大（10+ 人）</strong></td><td style="text-align:center">Spec Kit</td><td style="text-align:center">Spec Kit</td><td style="text-align:center">Spec Kit + bundle</td><td style="text-align:center">Spec Kit + spec-superflow</td></tr>
<tr><td><strong>多仓</strong></td><td style="text-align:center">Spec Kit + stores</td><td style="text-align:center">Spec Kit + stores</td><td style="text-align:center">OpenSpec stores</td><td style="text-align:center">spec-superflow + stores</td></tr>
</tbody>
</table>
<p>几个边角：</p>
<ul>
<li>小 × 持续 → OpenSpec 单跑。给三个月后的自己留记号</li>
<li>中 × 大爆发 → Spec Kit 而不是 OpenSpec+Superpowers。constitution + extension 扛得住三个月重构</li>
<li>多仓 × 大爆发 → 2026 才成立。"开源的 spec 层 + 纪律层 + 跨仓"完整栈是新东西</li>
</ul>
<h2 id="五-桥接器生态"><a href="#五-桥接器生态" class="headerlink" title="五、桥接器生态"></a>五、桥接器</h2><p>"OpenSpec 控需求、Superpowers 控质量"这条线，2026 年社区开始有源码级融合：</p>
<ul>
<li><a href="https://github.com/MageByte-Zero/spec-superflow" target="_blank" rel="noopener">MageByte-Zero/spec-superflow</a>（580★）—— npm 包，17 平台，9 skill，<code>ssf</code> CLI 自带 state machine / handoff / checkpoint</li>
<li><a href="https://github.com/rihebty/flow-kit" target="_blank" rel="noopener">rihebty/flow-kit</a>（367★）—— 纯 markdown，融合 bmad / spec-kit / OpenSpec / GSD / superpowers 等多家，<code>@</code> 引用即用</li>
<li><a href="https://github.com/fastknifes/openflow" target="_blank" rel="noopener">fastknifes/openflow</a>（200★）—— OpenCode 专属</li>
<li><a href="https://github.com/JiangWay/openspec-schemas" target="_blank" rel="noopener">JiangWay/openspec-schemas</a>（194★）—— 第一个 superpowers-bridge schema</li>
</ul>
<p>两个走向：spec-superflow / openflow 是"融合成一个新东西"（省事，版本绑死）；flow-kit / openspec-schemas 是"拼成 markdown"（灵活，不自动执行）。</p>
<h2 id="六-什么时候不需要"><a href="#六-什么时候不需要" class="headerlink" title="六、什么时候不需要"></a>六、什么时候不需要</h2><ol>
<li>写一个一次性 bash 脚本（扫 <code>node_modules</code> 列大文件 / 改一行 config 重启服务）——任务 &lt; 5 分钟，plan mode 出来的 3-5 步够细，再写 proposal.md 是流程上锁</li>
<li>自己一个人写的 500 行个人项目——没跨会话需求、没团队契约需求，<a href="/2026/07/21/ponytail-caveman/">ponytail + caveman</a> 风格 skill 组合就是天花板</li>
<li>纯调研 / 学习 / 探索型 prompt——输出是知识不是代码。spec 工具副作用是让 agent 把"调研方向"也写一遍 proposal</li>
</ol>
<p>共性：无法承受"写 spec 的时间"时，agent plan mode 杠杆更大。</p>
<h2 id="七-一句话带走的"><a href="#七-一句话带走的" class="headerlink" title="七、一句话带走的"></a>七、一句话带走的</h2><p>工具 ROI 跟团队纪律成反比——纪律越差，工具越值钱；纪律越好，工具越不重要。Spec Kit + constitution 给差纪律用，OpenSpec + Superpowers 给中等纪律用，ponytail + caveman 给好纪律用。</p>
<p>参考资料：</p>
<ul>
<li><a href="https://github.com/Fission-AI/openspec" target="_blank" rel="noopener">Fission-AI/OpenSpec</a> · <a href="https://github.com/obra/superpowers" target="_blank" rel="noopener">obra/superpowers</a> · <a href="https://github.com/github/spec-kit" target="_blank" rel="noopener">github/spec-kit</a></li>
<li><a href="https://github.com/MageByte-Zero/spec-superflow" target="_blank" rel="noopener">MageByte-Zero/spec-superflow</a> · <a href="https://github.com/rihebty/flow-kit" target="_blank" rel="noopener">rihebty/flow-kit</a> · <a href="https://github.com/fastknifes/openflow" target="_blank" rel="noopener">fastknifes/openflow</a> · <a href="https://github.com/JiangWay/openspec-schemas" target="_blank" rel="noopener">JiangWay/openspec-schemas</a></li>
</ul>
<script src="https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.min.js"></script>
<script>
  if (typeof mermaid !== 'undefined') {
    mermaid.initialize({ startOnLoad: true, securityLevel: 'loose', theme: 'default' });
  }
</script>


