---
title: hexo-theme-melody v1.5 supports slides & iframe
date: 2018-12-19 14:37:01
tags: hexo
layout: slides
categories: Hexo系列
---

<section>
<h3 style="color:#fff">写博客远比搭建博客难得多~</h3>
<h3 style="color:#fff">竟然开始了，就要先坚持几天吧。</h3>
</section>

---

<section data-background-color="#C7916B">
<h2 style="color:#fff">Steps</h2>
<h3 style="color:#fff">1. Add a slides page</h3>
<pre style="color:#fff"><code>hexo new page slides
cd ./source/slides
</code></pre>
</section>

---

<section data-background-color="#00C4B6">
<h3 style="color:#fff">2. Add the layout type</h3>
<pre><code class="lang-bash hljs">vim index.md
</code></pre>
<p style="color:#fff">Add a type called <code>slides</code>：</p>
<pre><code class="lang-yaml hljs">title: slides
date: 2018-03-06 20:24:48
type: slides
</code></pre>
</section>

---

<section data-background-color="#1B9EF3">
<h3  style="color:#fff">3. Modified the melody.yml</h3>
<p style="color:#fff">Add slides default config:</p>
<pre><code class="lang-yaml hljs">slide:
  separator: whatever you like
  separator_vertical: whatever you like
  charset: utf-8
  theme: black
  mouseWheel: false
  transition: slide
  transitionSpeed: default
  parallaxBackgroundImage: ''
  parallaxBackgroundSize: ''
  parallaxBackgroundHorizontal: null
  parallaxBackgroundVertical: null
</code></pre>
<blockquote>
<p style="color:#fff">See reveal.js config</p>
</blockquote>
</section>

---

<section data-background-color="#F47466">
<h3 style="color:#fff">4. Write a md file with slides layout</h3>
<p style="color:#fff">In _posts folder, add a md file.</p>
<p style="color:#fff">For example:</p>
<pre><code class="hljs yaml">title: hexo-theme-melody v1.5 supports iframe & slides
date: 2018-03-06 19:57:52
layout: slides

// balalala...
</code></pre>
<p style="color:#fff">Then you will get a post of slides type.</p>
</section>

---

<section>
<h2 style="color:#fff">Slides layout with iframe</h2>
<p style="color:#fff">If you want to add a website whatever you like within an iframe, try this:</p>
<p style="color:#fff">In _posts folder, add a md file.</p>
<pre><code class="hljs yaml">title: hexo-theme-melody v1.5 supports iframe & slides
date: 2018-03-06 19:57:52
layout: slides
iframe: the-url-whatever-you-like
</code></pre>
<p style="color:#fff">Then you will get a post of iframe.</p>
</section>

---

<section data-background-color="69C282">
<pre><code>// 启动hexo
hexo serve
// 切换主题
git clone github网址

// 添加文章
hexo new "you article title"

// 发布
hexo clean
hexo g -d
</code></pre>
</section>