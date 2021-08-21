---
author:
  name: "CJ Williams"
date: 2018-12-01
linktitle: Code For Good 2019
type:
- post
- posts
title: Visualising Youth Unemployment - J.P. Morgan's Code For Good 2019
weight: 10
series: 
draft: false
---

In November, I was fortunate enough to be invited to participated in the J.P. 
Morgan's Code for Good Hackathon. The event connects teams of students with 
several charities, each with a specific problem they hope to solve. 

My team choose to work with the charity Movement to Work - a UK charity that 
aims to tackle youth unemployment. Movement to Work wanted a tool that would 
help them to explore the levels of youth unemployment across the UK. Specifically, 
they wanted an interactive map that would display the Claimant Count 
(a measure of the number of people claiming the Job Seeker's Allowance) in every 
electoral ward for 18 to 24 year olds. 

After a long night we had managed to hack together a rough working prototype. 
While we were roughly able to visualise the data, we were plauged with inconsistencies 
between the boundaries and the unemployent data, causing holes in our choropleth. 
We also struggled with computational load of rendering such a large dataset, meaning 
sacrifices in boundary accuracy had to be sacrificed. 

{{<image src="/cfg-19.png" alt="Our web app" position="center" style="border-radius: 8px;" >}}

Despite these obstacles and the fact that we didn't win, there was still plenty to be proud of. 
Many of us in the team started the event with little knowledge of things like GeoJSON or Mapbox, 
and we all had to learn quickly and collaborate in a sleep-deprived environment. 

Thank you to everyone who helped to make the event the success that it was!