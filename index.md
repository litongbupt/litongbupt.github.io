---
layout: page
title: 技塑人生
---
{% include JB/setup %}


<!-- Tab panes -->
<div class="tab-content col-sm-9">
    <div class="tab-pane active" id="posts" >
      <ul class="list-unstyled">
		  {% for post in site.posts %}
		    <li style="line-height: 35px;"><span><a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></span> - {{ post.date | date_to_string }}</li>
		  {% endfor %}
      </ul>
    </div>
</div>

<ul class="posts">

</ul>



