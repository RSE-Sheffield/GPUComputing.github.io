---
layout: default
title: GPU Computing Seminars
permalink: /seminars/
---

This page provides a list of upcoming and previous seminars held by GPUComputing@Sheffield group. We aim to hold lunchtime seminars on the last Tuesday of every month, inviting speakers to talk about a wide range of topics that involves the application of GPUs in research and industry.

# Upcoming Senimars #

<section class="seminars">

	<article>
		<header>
			<h1><time>30/05/2017 1-2pm</time> Towards achieving GPU-native adaptive mesh refinement</h1>
			<div>Speaker: Ania Brown</div>
			<div>Location: Workroom 2 (G05), The Diamond</div>
		</header>
		<p>
		Modern simulations model increasingly complex multiscale systems, and the need to capture details at multiple length scales can lead to large memory requirements. Adaptive mesh refinement (AMR) is a method for reducing memory cost by varying the accuracy in each region to match the physical characteristics of the simulation, at the cost of increased data structure complexity. This complexity is a particular problem on the GPU architecture, which is most naturally suited to regular data sets. I will describe some of the optimisation and software challenges that need to be considered when implementing AMR on GPUs, based on my experience working on a GPU-native framework for stencil calculations on a tree-based adaptively refined mesh as part of my Master degree. Topics covered will include achieving coalesced access with the AMR data structure, memory defragmentation after grid changes and load balancing using space-filling curves.
		</p>
		<p>
		<a href="https://www.eventbrite.co.uk/e/talk-towards-achieving-gpu-native-adaptive-mesh-refinement-tickets-33614872990">Please register to attend with Eventbrite</a>
		</p>
	</article>

</section>
