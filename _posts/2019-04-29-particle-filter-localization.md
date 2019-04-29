---
layout: post
title: "Particle Filter Localization"
date: 2019-04-29
excerpt: "A SLAM project for ECE5242: Intelligent Autonomous System"
tags: [Project]
comments: true
---

## Introduction

The project aims to implement the structure of mapping and localization in an indoor environment using information from IMU and Lidar range sensors. In the following implementation of SLAM, it mainly uses the vehicle encoder information to get the odometry and relocate itself based on a particle filter that computes its correlation with local map.

## Dataset

### Encoder Data

The Encoders data has timestamp and the count data of 4 wheels. Each revolution of wheel is equally divided to 360 counts. The Encoders data gives the number of counts within each interval of sampling.

![Encoders Data](https://user-images.githubusercontent.com/11435445/56913568-fc6e7900-6a7f-11e9-8656-b62e16c3ec1e.png)
*Figure 1: Encoders data, Wheel Count vs. Timestamp*

### Lidar Scan Data

Lidar Scan data gives 1080 samples on a 270 degrees range (rotation is counterclockwise from -135 to 135 degrees) at each sample.

![Lidar Scan Data](https://user-images.githubusercontent.com/11435445/56913717-6555f100-6a80-11e9-93c6-3235c40e359c.png)
![Lidar Scan Data](https://user-images.githubusercontent.com/11435445/56913825-9504f900-6a80-11e9-8118-00533251c7b9.png)
*Figure 2: Lidar Scan Data*

### IMU Data

The third type of data is sampled from Inertial Measurement Unit (Unit).It gives the accelerator and gyroscope’s data on 3 dimensions.

## Data Preprocessing

### Encoders Accumulation

All the values in the encoders data are sampled within each interval, in order to calculate its odometry, the count of each wheel are first converted to a sum of all the previous counts at each sample. Therefore, the distance and direction the robot traveled in the interval of any two samples could be simply re-sampled by subtracting the accumulated counts.

### Data Alignment

The data of encoders and lidar are not perfectly aligned for each sample. In order to align these two data, for each sample of the re-sampled encoders data, the sample of lidar which has the closest timestamp to it is extracted and aligned with it.

## Technical Approach

### Mapping with Odometry

![Odometry Calculation](https://user-images.githubusercontent.com/11435445/56914209-81a65d80-6a81-11e9-8498-9778669d1f0a.png)
*Figure 3: Odometry Calculation*

Since the accumulated counts of wheels at each sample has been calculated, the odometry of robot within any two samples could be calculated using the above equations.

In order to get the odometry, some config data of the robot and lidar needs to be calculated at first.

$$Vehicle Width = \frac{(Inner Width + Outer Width)}{2} = \frac{(311.15 + 476.25)}{2}$$

$$Wheel Perimeter = \pi \cdot d_w = \pi \cdot 254.0$$

$$Dist Per Count = \frac{Wheel Perimeter}{360}$$

Say $$f_r^i, f_l^i, r_r^i, r_l^i$$ as the accumulated counts at sample i, the travel distance
∆s could be calculated as the following,

$$\Delta s_{r}^i = \frac{(f_r^i - f_r^{i-1}) + (r_r^i - r_r^{i-1})}{2}$$

$$\Delta s_{l}^i = \frac{(f_l^i - f_l^{i-1}) + (r_l^i - r_l^{i-1})}{2}$$

$$\Delta s^i = \frac{\Delta s_{r}^i+\Delta s_{l}^i}{2}$$

The sheer angle $$\Delta \theta$$ and the decomposing distance of $$\Delta s$$ on it, $$\Delta x$$ and $$\Delta y$$ could then be represented as the following,

$$\Delta \theta^i = \frac{\Delta s_{r}^i - \Delta s_{l}^i}{1.85 \cdot Wheel Width}$$

$$\Delta x^i = \Delta s^i cos(\Delta \theta^{i-1} + \frac{\Delta \theta^i}{2})$$

$$\Delta y^i = \Delta s^i sin(\Delta \theta^{i-1} + \frac{\Delta \theta^i}{2})$$

Then, the following odometry of the robot over the entire data set could be plotted using $$\Delta x$$ and $$\Delta y$$

![Odometry Results](https://user-images.githubusercontent.com/11435445/56914786-fcbc4380-6a82-11e9-92f3-0c21e487a4cf.png)
*Figure 4: Odometry Results: 3D(x,y vs. time) and 2D(x,y)*

### Particle Filter

The procedure of particle filter based localization is the following,

1. Initialize N particles
2. For each sample, update the particles witodometry data and add noise
3. Comparing correlation with the map at the currenposes, and re-weight the particles according to theicorrelation
4. Comparing correlation with the map at the currenposes, and re-weight the particles according to theicorrelation
5. Calculate the number of particles which has weight greater than a threshold $$th_w$$ and if thinumber if less than $$\alpha N$$, then re-sample accordinto their weights
6. Update the map based on the particle with thlargest weight using the lidar's data
7. Repeat

In this implementation, some of the above parameters are set as the following,

Number of Particles: $$N = 30$$

Particle Resample Threshold: $$th_w = 30$$, $$\alpha = 0.5$$

### Occupancy Grid Mapping

The occupancy grid ($$m_{x,y}$$) is a map representing the likelihood of whether the position x, y is occupied or free by accumulating the measurement $$(z = 1,−1)$$’s log-odd as follows:

$$\begin{aligned} \operatorname{log odd} &=\log \frac{p(z | m_{x, y}=1) p\left(m_{x, y}=1\right)}{p(z | m_{x, y}=0) p\left(m_{x, y}=0\right)}=\log \frac{p(z | m_{x, y}=1)}{p(z | m_{x, y}=0)}+\log \frac{p\left(m_{x, y}=1\right)}{p\left(m_{x, y}=0\right)} \end{aligned}$$

$$\operatorname{logodd}_{occ}=\log \frac{p(z=1 | m_{x y}=1)}{p(z=1 | m_{x y}=0)}$$

$$\operatorname{logodd}_{free} =\log \frac{p(z=-1 | m_{x y}=0)}{p(z=-1 | m_{x,y}=1)}$$

$$\operatorname{logodd}=\left\{\begin{array}{ll}{\text {logodd}+\text {logodd}_{\text {occ}},} & {\text { if } z=1} \\ {\text {logodd}+\text {logodd}_{\text {free}},} & {\text { if } z=-1}\end{array}\right.$$

## Results

For each map, two maps are plotted for comparison. The LHS is mapping with odometry only. The RHS is the result after introducing particle filter.

## Discussions

It could be seen that there is a huge improvement on the quality of mapping after using particle filer for localization. The improvement could be simply explained as the following.

In the corners of the hallway, the sheer of robot would usually yield a remarkable deviation. In most cases, this error would map the same hallway with some shift at each time of passing through. After implementing particle filter, it helps the robot to re-localize itself when turning back to and passing through the same hallway. Some particles would correlate very well with the occupied grids within the map. Therefore, the particle filter would drag the robot back to the previous track.

Another threat on the accuracy and quality of mapping is the movement on the z-axis. In the current implementation of odometry calculation and particle filter, it only consider the movement on the 2D plane. Therefore, in many cases, where z-axis movements are involved remarkably for instance, rough roads with pits everywhere or slopes with curve, would cause some significant errors on mapping and localizing. In this case, IMU data, which provided but not used in this implementation, would be useful to obtain the information on the z-axis. It could be done in the future to further improve the performance of this SLAM implementation.