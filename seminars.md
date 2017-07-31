---
layout: default
title: GPU Computing Seminars
permalink: /seminars/
---

This page provides a list of upcoming and previous seminars held by GPUComputing@Sheffield group. We aim to hold lunchtime seminars on the last Tuesday of every month, inviting speakers to talk about a wide range of topics that involves the application of GPUs in research and industry.

If you would like to recommend a speaker then please contact [Dr Paul Richmond or Dr Mozhgan Kabiri Chimeh from the RSE Group](http://rse.shef.ac.uk/contact).


{% if site.data.seminars.size > 0 %}
# Upcoming Senimars #

<section class="seminars">

	{% for sem in site.data.seminars %}
	<article>
		<header>
			<h1><time>{{sem.date | date: "%d/%m/%Y"}}  {{sem.time}}</time> {{sem.title}}</h1>
			<div>Speaker: {{sem.speaker}}</div>
			<div>Location: {{sem.location}}</div>
		</header>
		<p>
		{{sem.description}}
		</p>
		{% if sem.notes %}
		<p class="notes">
		{{sem.notes}}
		</p>
		{% endif %}
		<p>
		<a class="btn btn-info btn-raised btn-sm griditem-link" href="{{sem.register_link}}">Register to attend the event</a>
		</p>
	</article>
	{% endfor %}

</section>
{% endif %}

{% if site.data.previous_seminars.size > 0 %}
# Previous Seminars #

<section class="seminars">
	{% for sem in site.data.previous_seminars %}
	<article>
		<header>
			<h1><time>{{sem.date | date: "%d/%m/%Y"}}  {{sem.time}}</time> {{sem.title}}</h1>
			<div>Speaker: {{sem.speaker}}</div>
			<div>Location: {{sem.location}}</div>
		</header>
		<p>
		<img src="{{sem.image_link}}" alt="{{sem.title}}" />
		</p>
		<p>
		{{sem.description}}
		</p>
		<p>
		{% if sem.slide_link %}
		<a href="{{sem.slide_link}}">Presentation slides</a>
		{% endif %}
		</p>
	</article>
	{% endfor %}

</section>
{% endif %}
