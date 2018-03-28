---
layout: home
---

<div class="index-content tourism">
    <div class="section">
        <ul class="artical-cate">
            <li><a href="/"><span>Note</span></a></li>
            <li class="on" style="text-align:center"><a href="/project"><span>Project</span></a></li>
            <li style="text-align:right"><a href="/tourism"><span>Tourism</span></a></li>
        </ul>

        <div class="cate-bar"><span id="cateBar"></span></div>

        <ul class="artical-list">
        {% for post in site.categories.tourism %}
            <li>
                <h2>
                    <a href="{{ post.url }}">{{ post.title }}</a>
                </h2>
                <div class="title-desc">{{ post.description }}</div>
            </li>
        {% endfor %}
        </ul>
    </div>
    <div class="aside">
    </div>
</div>
