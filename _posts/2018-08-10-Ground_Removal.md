---
layout: post
title: Ground Removal
excerpt: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
categories: [detection]
comments: true
---
 
## Sliding Window's method

1. push all data into a 0.1×0.1×0.1 $m^3$ per cell
   1. sub item one
   2. sub item two
   3. sub item three
2. sort cells from high to low
3. use sliding window with hight threshold
   1. compute height difference of this cell
   2. also consider neighbour cells


## plane point ransac filter
1. choose approximate ground data
2. use PCL's ransac segmentation model

```c++
    pcl::ModelCoefficients::Ptr coefficients (new pcl::ModelCoefficients);
    pcl::PointIndices::Ptr inliers (new pcl::PointIndices);
    // Create the segmentation object
    pcl::SACSegmentation<pcl::PointXYZ> seg;
    // Optional
    seg.setOptimizeCoefficients (true);
    // Mandatory
    seg.setModelType (pcl::SACMODEL_PLANE);
    seg.setMethodType (pcl::SAC_RANSAC);
    seg.setDistanceThreshold (0.01);

    seg.setInputCloud (cloud);
    seg.segment (*inliers, *coefficients);
```


### tricks 1

![ground point Image](/img/ground1.png)


&emsp;&emsp;for one frame lidar data, we may choose the downside lidar(the red curve), this will save a lot of time.

### tricks 2

```c++
    seg.setDistanceThreshold( 0.05 );
    seg.setAxis( axis );
    seg.setEpsAngle( pcl::deg2rad( 10.0 ) );
```

&emsp;&emsp;the plane may vertical to vector(0,0,1), so we'd better to set axix to be (0,0,1) as well as the Eps angle.

### tricks 3
&emsp;&emsp;what if we may encounter two planes, like this:
![ground point Image](/img/ground2.png)

&emsp;&emsp;and if this time it's not good we may seg it again(with last time's coeef),Its interface is like this:

```c++
    void seg_ground_cloud( pcl::ModelCoefficients::Ptr& coefficients,
                          Eigen::Vector3f& axis,
                          int threshold = 0.05 );
```

## plane point angle_method1
we have one frame data like this:
![ground point Image](/img/ground3.png)
for every column,data like this:
![ground point Image](/img/ground4.png)
we use this stratege, like this:
![ground point Image](/img/ground5.png)
if the angle we get is above threshold, we think it's not ground point, and cotinue.And main precedure is under below:
![ground point Image](/img/ground6.png)

## plane point angle_method1

**main difference**: not only consider vertical angle, but also horizontal angle .The author use BFS method to compute which are ground points, the algorithm is like this: 
![ground point Image](/img/ground7.png)

ref: [PRBonn/depth_clustering]((http://www.google.com/))