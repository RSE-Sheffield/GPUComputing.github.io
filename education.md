---
layout: default
title: Education Training and Course Material
permalink: /education/
---

The GPUComputing@Sheffield group provides regular training for GPU programming and use of GPU accelerated software. Training is always announced first via the [GPUComputing@Sheffield mailing-list](.\home) (giving priority to members). Training is also announced via RSE-Sheffield and disseminated more widely based on availability of places (which is usually restricted). 

This page provides a number of permanently available resources and self paced labs which are the result of previous training sessions delivered at the University of Sheffield. For questions relating to the content in this training material please raise a [Github issue](./todo).

<div class="griditem row">
{% include education_tags.html %}
{% for i in site.data.education %}
<div class="griditem-item col-md-4 col-sm-6 col-xs-12" data-tags='{{ i.tags | jsonify | downcase }}'>
	<div class="well griditem-outer">
		<div class="griditem-inner">
			
			
			<div class="griditem-img bordered" style="background-image: url({% if i.image contains 'http' %}'{{ i.image }}'{% else %}'/static/img/{{ i.image }}'{% endif %});"></div>
			
			<h3 class="griditem-headlines">{{ i.name }}</h3>
			<p>{{ i.length }}</p>
			
			<div class="griditem-content">
				<div class="tag-holder">
				{% if i.tags %} 
					{% for j in i.tags %}
					<span class="label tags tag-filter" data-tag="{{ j | downcase }}">{{ j }}</span>
					{% endfor %}
				{% endif %}
				</div>		
    		</div>
    			<div class="griditem-footer"> 
    				{% if i.url %} 
    				<a href="{{ i.url }}" class="btn btn-info btn-raised btn-sm griditem-link">{% if i.url contains 'http'%}External Link{% else %}More{% endif %}</a>
    				{% endif %}
    			</div>
		</div>
	</div>
	</div>
{% endfor %}
</div>
 
