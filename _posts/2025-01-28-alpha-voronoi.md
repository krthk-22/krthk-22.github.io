---
layout: post
title: α Voronoi Diagrams
categories:
- Projects
- Computational Geometry
tags:
- Voronoi Diagrams
- Computational Geometry
description: An interesting project which I enjoyed a lot working on.
math: true
hidden: true
comments: true
date: 2025-01-28 20:47 +0530
---
This is one of very interesting projects that I have done during my B.tech at IIT Madras. I learnt [Voronoi Diagrams](https://en.wikipedia.org/wiki/Voronoi_diagram)(VD) as a part of [Algorithms in Computational Geometry](https://ed.iitm.ac.in/~raman/algcomp.html) course, they fascinated me and then I decided to explore more variations of the same. I asked this question "How will the voronoi diagram look like if we consider some ratio between the distances of the points instead of them being always Equidistant?" and then I started exploring the same.

## Journey
### Initial Exploration
I started with two points with ratio between the distances to be some ratio α and from the well-known Appolonius Circle theorem, I was able to figure out that the new VD is infact a circle. Then the immediate thought was to extend it to more than 2 points. This is where the real challenge and a lot of questions started to arise.

These are a few important questions struck me:
1. [How will I define the ratio between the distances? It can be from either of the points.](#defining-the-ratio)
2. [What is the physical significance of the ratio?](#physical-significance)
3. [How can I precisely define the VD for more than 2 points?](#defining-for-more-than-2-points)
4. [Once I answer the above questions, how can I compute the VD?](#computing-the-vd)

I am going to answer the above questions below and my experience in solving them.

### Defining the Ratio
This was the first question that came to my mind and I thought okay for now let me give some ordering and see how the VD looks and I proceeded. As I mentioned I arrived at a Circular Locus. After this I thought may be I can give an ordering for all pairs of points and then I will be able to find the VD. This created problems later on as I will explain in the upcoming sections.

This made me realise not to make any unnatural assumptions while solving a problem and I should always try to understand the problem in a more deeper sense.

### Physical Significance
I tried to give a physical significance to the ratio α and I realised that I want this ratio to be a measure of willingness of a point compared to other point. This is a very natural occuring scenario in real life. Think about a coffee shop which is very close to your house but not great and some other coffee shop which is a bit far but serves the best coffee. You will be willing to travel to the second coffee shop because of the quality of the coffee. This is the same thing that I wanted to capture in the ratio α.

But this created some more problems because of the ordering that I gave to the points. Say I have a cycle formed then the willingness of the points will be cyclically greater leading to contradiction.

### Defining for More than 2 Points
After asking the physical significance of the ratio, I realised that the ratio should naturally come up as mentioned in the example of coffee shops. If you observe carefully the ratio should come from the property of the points (the taste in case of coffee shops) and not from any ordering.

After the above realisation, I was able to define the ratio α between two points as ratio of weights (which indicate the willingness) of the points. This is a very natural way of defining the ratio and it also solved the cyclic ordering problem. Also it gave a way to define the VD for more than 2 points.

### Computing the VD
After coming up with the definitions and formulation of the problem, I tried to compute the VD for small set of points using some self-made simulations.

The simulations are made as follows: Take a grid and mark $$n$$ points on each axis. Now for each point compare the weights and distances and allot the point to whichever point it is willing to go. This is a very simple simulation but it gave me a lot of insights about the problem.
![Desktop View](/assets/img/dominant_regions.webp)

## Learning
After working with the simualations and a lot of thinking and observations, I was able to come up with a way to compute the VD for any set of points with weights. You can find the details of the final algorithm [here](/assets/pdfs/Voronoi.pdf) (which was inspired from the Fortune's Algorithm).

## Conclusion
This is the project that I enjoyed a lot working and brainstorming on. This gave me a flavor of how research goes on and how to think about a problem in a more deeper and meaningful sense. This project also made me realise not to make any unnatural assumptions. I am very happy to come up with a solution to the problem and I am looking forward to explore more such problems in the future and contribute to the field of Computational Geometry.