---
layout: distill
title: the nasa space robotics challenge
date: 2021-10-31
description: university of adelaide's solution to the nasa space robotics challenge
comments: true

authors:
  - name: Ravi Hammond
    url: "https://ravihammond.github.io"
    affiliations:
      name: AIML, Adelaide 

# Below is an example of injecting additional post-specific styles.
# If you use this post as a template, delete this _styles block.
_styles: >
  .padded {
    padding-top: 2rem;
    padding-bottom: 2rem;
    padding-left: 4rem;
    padding-right: 4rem;
  }

  .caption {
    line-height: 130%;
  }

  .verbatim {
    background: #f0f0f0;
  }

  .link_btn {
    border: 1px solid;
  }

bibliography: 2021-09-27-distill.bib
---

## Introduction

You know when an opportunity arises that will certainly take a lot out of you, but teach you invaluable life lessons at the same time? When Ragav Sachdeva contacted me in January 2021 about the NASA Space Robotics Challenge Phase 2 (NSRC), this was one of those times, and I knew I had to take the offer. As a true salesman, Ragav told me that “NASA was hosting a robotics competition, and it looks like we’ve got a good shot at winning it, do you want in?”. He explained the details of the project, and I immediately knew it was going to be big. I wanted in. This decision would consequently consume the next six months of my life. I signed up to co-lead the team alongside Ragav, and another friend, James Bockman. After half a year of high-energy team meetings, all nighters, and victorious high-fives, we programmed a team of rovers to autonomously find and extract resources from the surface of a simulated moon. Our efforts finally bore fruit in July 2021 when our team placed third overall, and received the additional innovation award.

<!--<embed youtube video here>-->

<!--<show blog contents page>-->

## What Exactly is the NASA Challenge?

The NSRC is the second installment in NASA’s attempt to progress space robotics research by casting a wide net across developers across the globe. In this round there were originally 114 teams registered, with 22 progressing to the finals. We were the only Australians to progress to the finals! Teams had to program a fleet of six autonomous rovers to find, extract, and retrieve resources from a simulated lunar environment. There were three different types of rovers to choose from; a scout with an attached resource sensor as, an excavator that could scoop the resources from the ground, and a hauler to bring resources back to base.

<!--<show picture of of the rover types>-->

Resources were scattered throughout a 200x200 meter map. Two base stations stood in the middle; A Processing Plant to receive excavated resources, and a recharge station to regenerate rover batteries. The rovers had to autonomously complete their tasks while avoiding obstacles and driving across rough, slippery terrain. Additional challenge that the simulator presented was a lack of GPS, and an extremely inaccurate IMU. Using these available onboard sensors, the rovers had to estimate their locations in the world by themselves.

<!--<show picture of the lunar surface with the>-->

The goal of the competition is to extract as many resources from the lunar surface as possible. The team that extracts the most resources wins.

## Creating the Base Framework

Our University had entered a different group of students in the qualifying round who had already written some existing code prior to our arrival. However, we quickly realised that we’d need to scrap everything, and write the entire thing completely from scratch. On top of this, we had never programmed robots before, and many aspects of the challenge were completely new to us! Our skill sets were primarily computer vision research and industry software engineering. 

In an effort to improve our chances of success, our University assigned sixty-four members to the team. An outrageous number! Brooke’s law states that “adding manpower to a late software project makes it later”, and our institution’s decision to bolster the team’s numbers would indeed prove a hindrance. Attempting to divide a four-person job across sixty-four people was a nightmare, and we knew that if everyone began programming at the same time there would be far too much confusion and incurred technical debt. Therefore, Ragav, James, and I put our heads together to create some infrastructure that would allow all team members to cleanly develop code in parallel.

The goal was to create a base framework that would allow members to easily “plug and play” different rover combinations, effortlessly swap out rover behaviours, and iterate quickly. To create this framework, we divided the work into three parts; state machine, rover equipment interface, and the ROS framework. Ragav developed shef, a beautifully simple system to easily swap out rover behaviours, James created a neat python API to interact with the rover’s on-board equipment, and I created the ROS framework that allowed multiple rovers to execute their behaviours at the same time.

<!--<insert diagram of our Hackathon framework>-->

Once complete, we wrote an internal wiki with in-depth documentation, recorded hour-long tutorial videos, and hosted a bootcamp to bring the rest of the team up to speed. Our efforts put into this framework allowed our large team of sixty-four to simultaneously develop throughout the entire competition.

## Planning Our Milestones

With a working framework, it was time to look at the bigger picture to figure out what our end goal was, and how to break up the work into manageable hurdles along the way. After long discussions, we settled on the following milestones: 

The purpose of the first milestone was to gain a high-level understanding of all moving parts. The goal was, using a cheat-GPS module, have the rovers find and extract resources one time. The scout would find a resource patch, the excavator dig it up, and the hauler would take them back to base, depositing at the processing plant. 

<!--<include gif or image of the task>-->

The second milestone was to extend milestone 1, where rovers had to continuously retrieve resources from the world - not just once. In this milestone the rovers would also be equipped to dodge obstacles. True obstacle locations were found using a cheat method. 
The third and final milestone was to repeat milestone 2, but without any cheats! This meant calculating the rover’s location, and nearby obstacle locations using only the rover’s onboard sensors. 

## Getting our First Points

Before we could reliably design the system for milestone 3, there were some major questions that needed answers, and we didn’t even know what some of those questions were! So we constructed a proof of concept goal to give us needed hands-on experience; have the rovers retrieve resources using as many cheats as needed.

The scout would drive to a known resource location (cheat), the excavator and hauler would drive to the scouts location (cheat), the excavator would dig resources and dump them into the hauler’s bin (cheat), and then the hauler would return the resources back to the processing plant’s known location (cheat). The rover and base station locations were already known, and the rovers drove in straight lines (not avoiding obstacles).

Completing this task was a huge achievement for the whole team as it taught us some crucial lessons, and gave us confidence that the task of this challenge was indeed possible. To motivate our group, and celebrate the completion of this first milestone, I recorded the demonstration video shown below.

<!--<insert of Youtube video of milestone 1>-->

## Dodging Obstacles On The Map

Whilst performing tasks for milestone 1, the rover’s drove in perfectly straight lines, directly to their destination. The issue with this approach is if they drove into rocks or craters, they could flip, permanently putting themselves out of action. As this event would be an unacceptable catastrophe, we needed our rovers to plan safe, collision-free paths.

We needed an algorithm to find paths throughout the area that would allow rovers to reach their destination safely. The first piece of this puzzle was to create a map consisting of all rocks and craters (cheat), and then calculating the space around these obstacles that was safe to drive. Using this obstacles map, we utilised the A\* search algorithm to calculate a set of waypoints that the rover could follow to reach its destination. The rover could drive in a straight line from each waypoint to the next, with assurance that it would not drive into any obstacle.

With the addition of this obstacle avoidance algorithm, milestone 2 could be completed. The rovers could continuously retrieve resources throughout the map whilst avoiding all obstacles. Keep in mind, however, that the system at this point was dependent on the cheaty obstacle map, and rover/base cheat locations from milestone 1. 

## Calculating the Rovers’ Locations 

Early iterations of our program depended on using the rovers' true locations that were extracted directly from the simulator. As submitting a solution using this method would disqualify us, we needed our rovers to estimate their own locations using on-board sensors. The previous qualifying team had attempted to use an off-the-shelf visual SLAM package to track the rover’s locations using their stereo cameras, but it wasn’t reliable within the high contrast lunar environment. So instead, we decided to use a classical approach to robot localisation - an Extended Kalman Filter (EKF). This algorithm allowed us to track a rover’s most-likely location by fusing it’s IMU and wheel odometry data.  

An issue with EKF is that the rover's estimated location will slowly accumulate error over time. A rover may think it’s at a certain location in the world, but in reality it’s actually ten meters away! These location estimates progressively get worse over time, caused by wheel slippages and noisy IMU readings. Our solution to this drift problem was to use the two base stations in the middle of the map as visual anchors. If a rover gets close enough to see the base stations, it can use the known station locations to figure out where it truly is - correcting the drift. We chose to use the base stations as visual “anchors”, because they were large, bright objects in the middle of the map, and they didn't move.

<!--<show PnP image>-->

This method allowed us to periodically reset the rover’s locations throughout the simulation run, ensuring that the calculated locations didn’t drift too much! Here is a plot showing the location of the hauler throughout a 2-hour time run. As you can see, the location of the hauler is successfully being reset at regular intervals throughout the simulation run.

<!--<show plot of hauler location resets>-->

## Visually Detecting Obstacles

For the rovers to traverse the map without colliding with obstacles, the path planning algorithm needs to know the precise location of all surrounding obstacles such as rocks, craters, base stations, and other rovers. Our previous approach knew these locations by accessing the simulation directly. This was against the rules, and would disqualify us if we tried submitting this solution. We needed to figure out a different method of detecting these obstacles, so we decided to use the rovers’ onboard stereo cameras.

In order to create a map of surrounding obstacles, we needed to extract two pieces of information from the video feed; where in the image are these obstacles, and how far away are they? To figure out where in the image the obstacles were, we used a neural network called YOLO that can find bounding boxes around objects we care about. To train this YOLO model we hand-labelled over 9000 images, drawing boxes around all the obstacles in each image. 

<!--<Show image of yolo>-->

To figure out how far away each obstacle is, we used a technique called stereo depth. A technique that takes two side-by-side cameras, and calculates the distance of all pixels in the image.

<!--<Show image of stereo depth>-->

With this information, we can calculate the location of all obstacles within the rovers field-of-view, and add them to an “obstacle map”. The map continuously updated as the rovers drive around, detecting more obstacles.

<!--<Show video of rover driving with map viewer>-->

Here is a video that shows our path planning and obstacle detection in action. The scout is detecting objects with its camera, and calculating the safest route to traverse the area. As it drives it sees more obstacles, and if needed, will re-calculate the shortest path to dodge any new obstacles that obstruct its path. The mini-map you see in the corner is a custom-built map we created to visualise the obstacle map.

## The Final Push

When there was only one month left of the competition, we realised if we didn’t dedicate more time to the project, we were likely not going to complete the challenge. Contributing to the slow speed of progress was that most of our team were working separately from home, so our university set up a private space to work on the project full-time.

Another painful thorn in our side was that the simulator took fifteen hours to complete a single run, which made it exceptionally hard to iterate quickly. Every change required an entire day to test! In need of a more efficient testing method, we set up 20 cloud computers to run multiple instances of our program overnight, which was a monumental productivity boost for our team. Gathering such an enormous amount of data each night helped us iron out dozens of bugs each week, and whilst these computers ran our tests, we were able to simultaneously develop and fix bugs. 

One week before submission, catastrophe struck. A covid-19 outbreak occurred in our city, and our entire state went into immediate lockdown, which meant not leaving our homes for a week. Our workstations were set up at university, and working separately at such a late stage in the competition could jeopardise our chances of success. So we had to think of a solution, and we needed to think of one fast. Thankfully, one of our team members, Alec Arthur, offered us his house. His family moved out, and the team moved in. 

<!--<Alec’s house image>-->

We turned his living room into our own private office and worked on the challenge all day and night. Whilst working in the same house, we were able to iron out many more bugs and implement even more last-minute features. Everything was going smoothly, the due date was 8am the following morning, it was midnight, and we were set to submit our solution. As we ran our program for the last time before submitting, we noticed a grave error. The rovers weren’t acting as they were supposed to. 

Something was wrong, and in an exhausted rush of adrenaline, Ragav and I scoured the code to figure out a solution, whilst James and Alec started falling asleep at their desks. By 6am we had finally implemented the solution, and some last-minute tests seemed to show that everything was working as expected. However, as we were about to submit the code, half of us got cold feet. “We haven’t tested this latest change overnight, so how do we know that it truly works, and hasn’t broken something else?”. These team members with cold feed wanted to submit a previous version of our code that had been tested on the 20 cloud computers.

We were all deliriously tired, stressed, and worried that this latest change could contain unseen problems. Do we submit an older version? Or submit this new code? In the end it came down to a blind vote. We closed our eyes and stuck out our hands, and the latest change won the vote -  we ended up going with the hotfix. We knew that we had to “risk it to get the biscuit”, and if we wanted a shot at winning this competition, we had to take the chance. As it turned out the risk was worth it, because our program ran smoothly, and we placed 3rd worldwide, and we also received the innovation award!

## Conclusions

Opportunities like the NASA Space Robotics challenge don’t come around often. To be given the chance to lead a team of engineers, whilst competing against highly skilled professionals around the world, will remain a deeply treasured honour. The challenge was a welcomed event that reminded me of why I love what I do. Leading a team of skilled engineers, developing in high-intensity environments, solving obscure problems, and delivering solutions. If I’m able to spend the rest of my life working on projects like these, I’ll retire an immensely happy man with no regrets. If opportunities like these arise in your life, don’t think twice. Swoop them up before someone else does.

