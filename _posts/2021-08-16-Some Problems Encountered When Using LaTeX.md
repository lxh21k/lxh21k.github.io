---
title: Some Problems Encountered When Using LaTeX
date: 2021-08-16 15:40
--- 

在用 LaTeX 写毕设的时候遇到了一些问题，今天看到当时当时整理的一些，说不定再遇到就又忘掉了，干脆放在Blog持续更新。

- 段落会根据页面上下两端对其导致某些段间距过大

https://stackoverflow.com/questions/1363392/latex-avoid-that-page-text-stretches-over-whole-page-with-twoside-option

> Add `\raggedbottom` to the latex preamble that will solve it.

- 给图/表等环境设置特定序号而不采用自动序号

https://latex.org/forum/viewtopic.php?t=5852
https://blog.csdn.net/wkd22775/article/details/51791553

`\renewcommand\thetable{x.x}`

- 修改图/表环境的题注标题

`\renewcommand{\thealgorithm}{xxx}`

- 修改题注字号

`\captionsetup[algorithm]{font=small}`
