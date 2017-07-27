---
layout: post
title:  "Exam Coversheet Reader"
date:   2017-07-27 16:00:00 -0400
categories: "projects"
---

A Python/openCV computer vision project

[GitHub repository](https://github.com/aalllxx/Exam_Coversheet_Reader)

# Exam Coversheet Reader
-----------
## Description
To save time when grading a large class, entering grades on paper and scanning them into a database can save time. This code does that using bubbles which can be filled in. 

Sample coversheet:

![sample coversheet](http://i.imgur.com/B2Qzszl.png)

Images are precisely cropped using bracket markings in each corner of the document. Regions of interest (ROIs) are identified and a number is extracted from each. This is written to a file.

There are three types of ROIs supported currently - 3x3 grids, 10x2 grids, and 10x3 grids. The 3x3 are intended for a three digit unique student ID which students will fill out when taking the exam. The 10x2 is intended to score an individual problem with value between 0-99. 10x3 is for a final grade and supports values up to 999.

Creating a template for a region of interest in handled in the program. If it detects a file named ```ROIs.csv``` it will read the ROIs from that. Otherwise it will assist the user in creating one using a GUI.

Data are output to a file called ```grades.csv``` with row containing a unique student ID each column a value associate with that student.

Grading output:

![grading output](http://i.imgur.com/NWif246.png)

GUI to build template:

![gui to build template](http://i.imgur.com/3gJgafZ.png)
