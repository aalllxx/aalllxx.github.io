---
layout: post
title:  "Ball Tracking for Squash"
date:   2016-08-16 16:00:00 -0400
categories: "projects"
---

A MATLAB Computer Vision program

[GitHub repository](https://github.com/aalllxx/Squash-Project)


Overview
--------
Ball tracking in sports has been around for over 15 years and I implemented a version that works for squash. I explain my process in depth in this post, but I figure I'll give away the result at the top. (I'm really proud of it). Check out how the curves in the output match the ball path in the input.

**Input:**

![Animated Squash Clip](https://media.giphy.com/media/3o6Zt89eg01OUTaD16/giphy.gif )

**Output:**

![Output image](http://i.imgur.com/20CSBmj.jpg)

The ball is hard to make out in the gif of the input video so I added a yellow highlight before posting this. It has nothing to do with my program.


Introduction
------------

With the Rio Olympics underway, everybody seems to be excited about odd sports. I got into squash last August and spent all year obsessed. I played squash, I watched squash, I tried to get my friends into squash. This summer, it didn't take long to realize I wanted to create a program for squash, but the specifics were much harder to flesh out.

Inspiration
-----------

I found two videos on Facebook that gave me my original direction and motivation. The first was of the German company [Fun With Balls GmbH](http://funwithballs.com/) and their [Interactive Squash](http://interactivesquash.com/) project. They project animated games onto the front wall of a squash court and detect where the ball hits. They support simple games like tic-tac-toe as well as training exercises which can test a player's ability to hit a particular spot on the wall. I was certainly not aiming to create a project that user-oriented or marketable, but identifying where a ball hit the front wall remained a primary goal of mine for quite a while.

The second video blew my mind entirely and motivated me to keep with sports oriented computer vision. [SPORTLOGiQ](http://sportlogiq.com/), a Montreal based sports computer vision company,  made [this video](https://vimeo.com/133304995) showing off their hockey technology and it was on an entirely different level than what I had thought was possible. HawkEye, SportsVision, and others have made their own splashes in the industry, and maybe it is just excellent marketing, but SPORTLOGiQ seems far ahead.

What's so captivating about their product is not it's beautiful UX design or perfect player/puck tracking. I was in awe of it's ability to understand what was happening in a game. They track statistics like loose puck recoveries and can determine which players contribute to goals. That understanding comes at a higher level than tracking objects and I never considered that a machine could figure it out. I've always been interested in sports technology, be it first down lines, baseball pitch tracking, or tennis in/out decisions, but SPORTLOGiQ's video gave me new inspiration.

**Starting to program**

Early in development, I was overflowing with inspiration, but seriously lacked direction. My original goal was still to detect where on the front wall a ball hit, and by the time I had developed decent ball detection in each frame (more on that later), I was painfully aware of how hard it would be to pull that off in 2D.

I had also been messing around extensively in MATLAB. This was my first time working in MATLAB and I was impressed with the documentation available on all built-in functions. My first real taste of computer vision came from reading the help articles on functions with input arguments I couldn't understand. I remember struggling with the [Kalman Filter](http://www.mathworks.com/help/vision/examples/using-kalman-filter-for-object-tracking.html) for a day or two and slowly understanding the difference between detection and tracking.

To keep at bay my growing anxiety that my project may fail, I binge-watched a  [Udacity course on computer vision](https://www.udacity.com/course/introduction-to-computer-vision--ud810) with Professor Aaron Bobick formerly of Georgia Tech. His course is excellent. I learned about [Fourier Transforms](https://classroom.udacity.com/courses/ud810/lessons/3515968538/concepts/34851485780923#), and [stereo geometry](https://classroom.udacity.com/courses/ud810/lessons/2965048666/concepts/29469186670923#), and a million other fascinating topics, but at the end of the day, none of it was really applicable to the basic tracking problem I was looking to solve.

In my hunt for scholarly help I found **[Computer Vision in Sports](http://www.springer.com/us/book/9783319093956)** (Springer International, 2014) which, after a week in interlibrary transit from the University of Wisconsin, changed my whole direction for the project.

**Computer Vision in Sports**

*CV in Sports* is a collection of scholarly papers published by a number of researchers on advances in CV as they apply to various sports. After drooling over the introductory chapter I had a handle on the four areas of focus the book explores.

1. Ball tracking
2. Player tracking
3. Determining what sport is being played
4. What is happening within a game

The whole book is astounding, but as soon as I saw this image I knew what I was going to build.

![Clipping from CV in Sports](http://i.imgur.com/5ik9fWq.png)

*Source: Computer Vision in Sports*, p. 35

**On to the code**

That's about all I the introduction I have to offer. I'll note that I pushed aside most of my worries about time efficiency. My goal was to create a program which could produce the results I wanted without any other caveats. I didn’t care if it worked quickly or was especially user friendly, so long as it worked well. I'll point out a few instances where I clearly chose very slow algorithms and something quicker might have been nearly as good.

Finally, if you want a quick introduction to squash, [this video](https://www.youtube.com/watch?v=3gQsAKZ71tU) does a mediocre job of explaining the rules, but has good illustrations. Onward!

`Blob-It-Up` - Ball detection
-----------------------------
What I enjoyed most about this project and CV in general is the way they make me think about breaking down everyday tasks into clear sets of instructions. The way I thought about detecting the ball in each frame, one of the first tasks in the project, is a clear example of this.

The first piece of code I wrote for this project was designed to use a built in MATLAB [Viola Jones](https://en.wikipedia.org/wiki/Viola%E2%80%93Jones_object_detection_framework) object detector. I would detect the ball in each frame by taking each frame and looking for a small, black circle - the ball. This, of course, didn't work at all. Small black circles are about as hard to detect as objects come. The best MATLAB documentation I found was for self-driving car programs - moving cameras looking for consistent and unique stationary objects. It took me a few failures to implement one of those algorithms to realize how my task was unique.

![Moving Box](http://lewisdartnell.com/motion/motion_images/Anim_1.gif)

*Source: LewisDartnell.com*

Like spotting camouflaged birds when they fly away, my squash ball was only distinct from background noise in that it moved. It didn't just bounce around randomly either - it followed paths. A few weeks after figuring this out, *CV in Sports* would give me the tools to quantify the motion, but when building my detector I was only thinking about the difference between frames.

My first example of slow code is `toSmoothedDiff`, the way I created a video of the difference between frames. I spent a long time tuning this code to work with my test clips. I used a GoPro wide angle shot at 720p and 120fps and attached the camera to the glass at the back of the court. With a different camera in a different position all of these tunings would need to be adjusted. This **is not** a scaleable program. `toSmoothedDiff` could certainly be optimized and made easier to modify, but I choose to keep it well tuned and left it as is. The only bit of efficiency I introduced at this stage was working with grayscale and then logical black and white images. I'll discuss using color at the end of the post, but long story short, it's not very useful.

The hardest part about generating clean detections is that there is too much noise, especially around the players. Because of the camera placement, the ball can be close to camera and large, or speeding into the front wall and only a few pixels wide. Just for an idea of the scale, my blobs were between 100px<sup>2</sup> and 750px<sup>2</sup>. Limbs and racquets often split into odd shapes in the difference video and even when I sampled at 120fps the ball would stretch into two disconnected blobs when hit quickly. Even with my finely tuned difference video it was still not possible to reliably detect the ball in each frame.

My solution to this problem is horribly inefficient and it pains me to think about. I took a second difference video with a four frame difference and used that help remove slower moving blobs. The result is a very clean image which allowed me to easily detect the ball.

![shoD1](https://media.giphy.com/media/l2SpTmHEvaCRmYamA/giphy.gif)

*Difference of one frame*

![shoD2](https://media.giphy.com/media/3o6Ztd0iCk0O3SNhfy/giphy.gif)

*Difference of four frames*

![noPeople](https://media.giphy.com/media/3o6Zth2jI3MSl2FXXi/giphy.gif)

*Result of `noPeople` with above as inputs*

Look at how much brighter the four frame difference video is, especially around the players. The biggest problem I had with detection was that racquets or limbs would often separate from the large *person* blob and get detected as a possible ball. By subtracting at a further difference, I was able to more clearly figure out where the whole player was and exclude that when searching for the ball.

`noPeople` was very hard to optimize properly, and `blobItUp` only uses it in noisy frames with more than 2 detections, but in the noisiest sections it works wonders at cleaning up a frame. At the end of the day though, I hope that in future versions I'll do away with the two difference videos.

Once I had well tuned difference videos the real challenge started. What unique characteristics does a squash ball have that noise or players do not? Through fine tuning of `blobItUp` I was able to isolate objects of the right size, but I was still a long way away from generating paths.

My tuning process was simple trial and error. I would change the size of my morphological dilation brush and minimum and maximum blob size. If no detections were found in a frame where there should be, I would redo the dilation with a larger brush. If too many detections were found, I would use `noPeople` and the second difference video to figure out which detections could be ignored.

At that point my detections were reliable and I was ready to solve my original problem - finding where on the front wall the ball made contact. A simple solution to this problem is possible with my two dimensional setup. When the ball changes direction and speed in a pixel on top of the front wall I could locate that point in the 2D space of the front wall. There are many problems with this setup and to do it right, I would need at least 2 cameras. In any case, without a setup to track the ball, only to detect it, my task was not feasible.

![Detections](http://i.imgur.com/dCz5Dps.png)
*All detections from `blobItUp`*

With accurate ball detection in ~85% of frames without total ball occlusion the results from `blobItUp` are consistently excellent. I can practically draw the curves already! With this as a foundation, I turned to object tracking.

Layered Data Association
------------------------
The 2D ball tracking chapter of *CV in Sports* described a three part process called Layered Data Association. For this section, looking at [my code](https://github.com/aalllxx/Squash-Project) is the most useful way to see how I implemented each element. Here I’ll try to explain the process in plain English

Layered Data Association(as described in *CV in Sports*) begins with a clip of a whole game of tennis with detections in each frame and outputs all of the paths from all of the rallies in the game. Each of the three levels works at a different level of abstraction as follows:

**Candidate Level**

`generatePoints`, `generateTracklets`, `improveTracklets`

This is the first abstraction away from detections. A game becomes a series of functions.

I generate tracklets from detections and check a circular window around each detection for other detections. If in the previous and following frames there are detections within a radius, a tracklet is formed. The three points become supports, and a second order function is fitted.

Tracklets are iteratively improved. I predict where the next detection should be based on the model in each tracklet and if detections lie within a threshold of that point, it is added to the tracklet and the fitted function is updated. This iteratively repeats until the overall fit stops improving, or no new supports are found.

**Tracklet Level**

`connectTracklets`

In this process shortest paths are found between each tracklet generated above. This abstracts away from individual functions and pieces together whole rallies.

If the models were not similar I extended temporally adjacent curves to their intersection and drew it out. Due to occlusion or racquet noise, detections near players are typically worse. Because curves typically end around players (when they hit the ball), extending curves this way is especially important. I drew those sections in yellow in the original output image.

**Path Level**

Shortest paths are found between all tracklets, not just adjacent ones. This shows which tracklets are connected (in a single rally) and which are distinct. It abstracts a third time and builds an entire game from a collection of rallies.

I didn't implement Path Level Association but I certainly intend to in the future. I hope to write more about it then.

**Endpoint/Midpoint Curves vs MATLAB fitObjects**

*CV in Sports* was my guide for implementing Layered Data Association and it steered me in the right direction time and time again. The one decision where I really took a different route than the book is how I created my models.

The book writes out the basics physics equations deriving acceleration, velocity, and position that I learned in high school.

![Equations](http://i.imgur.com/rlXxehY.png)

*Equations from CV in Sports. **p** is position, **v** is velocity, **a** is acceleration, **k** is frame.*

I had coded my curves using each tracklet's first and last supports and by choosing a middle support between the two. These equations work alright when there are a small number of points, but because they ignore all internal supports they are a bad choice for modeling long paths. If the ball path and all detections were perfectly regular a function based on three points would work, but with even a small amount of deviation the models produced are poor. Once I made the switch to using MATLAB's built in `fit()` function my results improved dramatically.

The only downside was that whereas I had been storing values in [xValue, yValue] arrays, I needed two fit objects stored in a struct for my tracklet models. One function mapped time to X position and the other mapped time to Y position. Position was dependent and frame number was independent.

Again, for all the details of my implementation, please take a look at [the code](https://github.com/aalllxx/Squash-Project).

Explaining my output
--------------------
After discussing my implementation it’s time to explain the image I posted at the start of this post. There are dots and lines in four colors and it's not so intuitive to read.

![Output image](http://i.imgur.com/20CSBmj.jpg)

**Black dots**
Black dots represent all detections from my candidate level association. They are primarily in a line around the path of the ball, but are also scattered around the court.

**Red dots**
Red dots are detections which had supports on either side. I formed tracklets with models from these.

**Blue curves**
Blue curves show tracklets which had more than 4 supports. `improveTracklets` takes each tracklet with supports and scoops up nearby detections which are within a threshold of where the tracklets model predicts. It adds these points to its own supports while deleting them from their old tracklet. If a tracklet has zero supports it is considered dead. In this way, longer tracklets grow and form single functions along a path.

**Yellow curves**
Yellow curves are extensions of a tracklet's model to the intersection with the neighboring tracklet. This is especially important because the racquet head will always occlude the ball when contact is made.
When the intersection is outside both curves (like in the long top right example) both curves are drawn. When the intersection is within one curve (bottom left) only the extension from one curve is drawn. In this case the curves don't line up because the ball traveled across the player's body and was occluded from view.

I was surprised by how true the ball travels to 2nd order (constant acceleration) polynomial curves. In the top-right section of the output there are long yellow extensions. My blue curves end 275px and 75px from the predicted intersection and the error, which I measured by hand from the actual frame of contact, was 20px. For such a long distance I am happy with that margin of error. With better tuning to my `scoopItUp` code I'm sure that error would come down.

**It's far from perfect**
My program works fairly well, but by taking a close look at a second clip I'll can show some areas that desperately need improvement.

![Second Clip Highlighted](http://i.giphy.com/3o6Zt2HBg6ECJP0dvW.gif)

*A second clip I worked with. Highlights added by hand before posting to blog.*

![Second Clip Output](http://i.imgur.com/xOYnt7m.png)

*Output of second clip*

The first thing you'll notice is my error on the forehand volley drop. I need to be quicker to the ball and put it away.

After getting over that, look at how the output is segmented in strange places. I don't really know why my blue tracklets are broken up in those places. That could be a small issue with how I tuned `improveTracklets` or it could indicate some larger bug. In any case, `connectTracklets` is sometimes able to compensate for this problem and draw yellow lines between the disconnected segments.

The next issue has more to do with how I created this output. `improveTracklets` is meant to iteratively run the `scoopUp` method, but I didn't fully implement the loop. In theory, tracklets should be improved until the next iteration doesn't find any new supports or the model starts to fit the supports worse than the last iteration. I would do this by checking `numSupports` and `meanDistance` after each iteration, but the real trouble comes in determining what threshold to use at each iteration.

When I made this output I ran two iterations of `scoopUp`, first with a distance threshold (how far a point can be from a prediction) of 5px and then of 3px. I tried it with smaller and larger thresholds and both produced worse results. I know how to implement this aspect, but ran out of time in the summer. For now, this all needs to be done manually. It also means that `doItAll`, my function which runs all the pieces in order, will not produce results as solid as doing it by hand and tweaking the threshold.

At the end of the day, this is a strong proof-of-concept, but needs lots of work to do anything useful.

Other thoughts
--------------

**A rant about OOP in MATLAB**

I've done most of my serious programming in Java so once I had a clear idea of what I needed, I transitioned straight into an Object-Oriented design. MATLAB seriously disappoints in its OOP practicality, and if I was doing this again I would certainly keep everything procedural. In essence I did make all my methods procedural by passing index values to my class methods and returning my modified elements from those methods. Passing by reference is not simple in MATLAB.

The only real difference between my implementation and a purely procedural one is that MATLAB displays the values of 1 dimensional structs when inspecting variables, but does not display class attributes. Passing values was a small nuisance, but the poor debugger design frustrated me to no end.

**Why squash is different**

I mentioned at the top of this post that ball tracking, even real-time tracking, has been around for a long time. This program is far from groundbreaking. That doesn't much bother me (it's new to me), but even if I was upset to be recreating an ancient technology, I could take solace in some of the unique aspects of squash.

My second clip shows this to some degree, but squash in general, and especially professional squash, is very chaotic. Watch [this clip](https://www.youtube.com/watch?v=IpApHbJpCy8) and pay attention to the dramatic changes in ball speed, the odd angles on boasts, the tightness of shots to the side walls, the reflections in the glass, etc. Whereas tennis has long and clean curves, squash rallies are all over the place. I am very happy that my program can detect strange angles and instant changes of speed.

**What I learned**

* MATLAB
* CV / Image processing
* Working on a long-term project
* How to explain a CS project to friends
* Markdown
* GitHub
* Blogging with Jekyll

Future plans
------------

**Player tracking**

After tracking the ball, tracking players feels like the next natural step. There are already a number of programs which accomplish this. In 2001, Pers et al. created a [A Low-Cost Real-Time Tracker of Live Sport Events](https://www.researchgate.net/profile/Stanislav_Kovacic/publication/3905802_A_low-cost_real-time_tracker_of_live_sport_events/links/0c960533ac2da56f04000000.pdf) and more recently, two undergrads at the University of Pennsylvania put together [a program](https://www.cis.upenn.edu/current-students/undergraduate/courses/documents/HonorspaperforEAS499_Wu_Judd.pdf) which does just that with a single camera above the court. Like my 2D ball tracking, that technology is interesting, but by itself mostly useless.

**Stereo vision**

I literally dream of stereo tracking. I think about how adding a second camera to my setup would solve my biggest problems and pave the way to new and useful functionality. All of the big innovations which I want to see for squash tracking need 3D.

Drawing the path of a squash ball is nice, but is hardly useful in any practical way. By introducing stereo footage, video I could make a 3D hull rendering of the squash court. I would know where on each wall the ball bounced and the speed of the ball at each frame. Adding a second camera would also help with the regular occlusions that a single camera has trouble with.

**Use color (or maybe not)**

My program totally ignores color and immediately converts videos to grayscale. In my implementation that is fine, but for player detection or to detect reflections on glass courts, color could be useful. In general, what I've seen is that detection isn't very difficult in a controlled environment like a squash court. The marginal benefits of color are likely not ever going to be worth the triple computing time.

**Better camera positioning**

This was my first time using a GoPro and I was shocked with the ease of use and quality of video produced. I clipped a tiny box to the back wall and it recorded in HD at 120fps. In previous setups I had been hanging tripods off of railings and attaching cameras with large lenses to them. Even then, the field of view and frame rate didn't compare with the GoPro's.

The GoPro's wide angle shot did cause problems for my ball detection and curve fitting. In my main test clip most of the action took place in the front of the court and when I tested rallies that were played further back they were much less smooth. This is partially because the ball changes size dramatically between the front and back of the court. Changes in position are also more extreme when closer to the camera and that causes the ball path to deviate from perfect 2nd order polynomials. Using a camera position similar to broadcast would help my program perform.

![zoomed camera](http://i.giphy.com/l2Sqi70EikLX7kt1u.gif)

*Source: PSA Squash TV*

**Player skeleton tracking**

Like Microsoft's Kinect showed, tracking a human skeletal structure is a feasible and effective way to track a person’s movements. Implementing this for a squash player could reveal interesting information about the technique of squash players. It could be used for professional player statistics or for analysis and coaching.

![skeleton tracking](https://answers.unrealengine.com/storage/temp/18510-kinect_bones.gif)

*Source: answers.unrealengine.com*

**Big data**

In a fully featured system I would be sure to track as many statistics as possible. A squash tracking system would be the total package if it could record all of the information I mention above. I'm excited to think about how huge amounts of data could change how players, especially amateur players, are coached. I'd also like to see quantitative data on how [Egyptian](https://www.youtube.com/watch?v=p1YhN8WTStI) and [British](https://www.youtube.com/watch?v=HtwckCfSIHU) play styles are different. A program could even learn which shots a particular player likes to use in different scenarios.

The sky will be the limit when it becomes easy to measure shot selection, racquet speed, and quantify technique. More than inclusion in the  olympics or improved racquet design, I want to see squash as a pioneer in computer vision. I hope I'm not the only one.
