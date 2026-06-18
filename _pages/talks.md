---
title: "Talks"
permalink: /talks/
---

{% assign year_groups = site.data.talks | group_by_exp: "talk", "talk.date | date: '%Y'" %}
{% assign year_groups = year_groups | sort: "name" | reverse %}

<div class="talks-timeline">
{% for group in year_groups %}
<h2 class="talks-year">{{ group.name }}</h2>
<ul class="talks-list">
{% for talk in group.items %}
<li class="talk">
{% if talk.url %}<a class="talk-title" href="{{ talk.url }}">{{ talk.title }}</a>{% else %}<span class="talk-title talk-title--plain">{{ talk.title }}</span>{% endif %}
<span class="talk-meta"><span class="talk-date">{{ talk.date | date: "%d %b %Y" }}</span><span class="talk-conf">{{ talk.conference }}</span><span class="talk-lang lang-{{ talk.lang | downcase }}">{{ talk.lang }}</span></span>
</li>
{% endfor %}
</ul>
{% endfor %}
</div>
