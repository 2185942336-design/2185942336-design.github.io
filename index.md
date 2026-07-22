---
layout: default
title: 我的 CTF 知识库
---

# 🚩 欢迎来到我的 CTF 笔记库

这里记录了我的网络安全学习之路，包括 AI 安全、Web 渗透、工具使用等实战笔记。

## 📚 笔记目录

<ul>
{% for file in site.static_files %}
    {% if file.path contains '.md' and file.path != '/index.md' and file.path != '/README.md' %}
        <li style="margin-bottom: 10px; font-size: 1.1em;">
            <a href="{{ site.baseurl }}{{ file.path }}" style="text-decoration: none; color: #0366d6; font-weight: bold;">
                📄 {{ file.name | replace: '.md', '' | replace: '-', ' ' }}
            </a>
        </li>
    {% endif %}
{% endfor %}
</ul>

---
*Generated automatically from repository files.*
