---
layout: default
title: xi4nyu'blog
---
<div id="home" class="page-content wc-container">
	<div class="posts">
  		{% for post in site.posts limit:10 %}
  			<div class="post">
    			<h3 class="post-title">
			      <a href="{{ post.url | prepend: site.baseurl | prepend: site.url }}">
			        {{ post.title }}
			      </a>
    			</h3>
                <p class="post-meta">
                    <span class="post-date">
                    {{ post.date | date: "%Y-%m-%d" }} 
                    </span>
                </p>

  				<p>
	  				{{ post.excerpt }}	
  				</p>
  				<p>
  					<a href="{{ post.url | prepend: site.baseurl | prepend: site.url }}" title="{{ post.title}}">
  						Read More
					</a>
  				</p>
  			</div>
  		{% endfor %}
	</div>
    <!--
	<div class="post-footer">
	</div>
    -->
</div>
