## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/calibration1.jpg.jpg "Distortion Correction"
[image2]: ./output_images/undistorded.jpg "Undistortion Example"
[image3]: ./output_images/gradien_distribution.jpg "Thresholding Analysis"
[image4]: ./output_images/combined_binary.jpg "Binary Output"
[image5]: ./output_images/transformation_region.jpg "Warping Region"
[image6]: ./output_images/birdeye_view.jpg "Warp Example"
[image7]: ./output_images/sliding_window.jpg  "Sliding Window Search"
[image8]: ./output_images/fitted_polyline.jpg    "Polynomial fitted line"
[image9]: ./output_images/final_image.jpg   "Output"
[video1]: ./output_video/project_video1.mp4 "Video"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

This document is writeup document for this project. Code work for this project is saved as AdvanceLaneFinding.ipynb. All the images for testing pipeline are saved in folder output_images and videos are saved in folder output_video

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the cell just below the heading "Calculate camera calibration".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using undistort_image() function which basically uses the `cv2.undistort()` function. The result of distortion correction on chess board images look like this

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I used function undistort_image() described in section with heading Undistort image function. Undistorted image example is attached here 
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image defined in function get_image_binary() under heading Image thresholding. I used S-channel for colour thresholding and also used S-channel for gradient thresholding. I used soble in x direction for magnitude threholding and direction thresholding. After apllying thresholding, all three thresholding are plotted as individual channel in image here
![alt text][image3]

The final binary image after thresholding appears as 
![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `transform_image()` under heading Transform image, which take input image and source and destination points and transform image using function cv2.warpPerspective().

Source point image is trapezoid over lane marking in straight raod and destination point are rectangle. Points, I have considered as following 
src_corners = np.array([[592,450], [690,450], [1070,700], [245,700]])
    dst_corners = np.array([[200,100], [1000,100], [1000,650], [200,650]])
| Source        | Destination   | 
|:-------------:|:-------------:| 
| 592,450       | 200,100       | 
| 690,450       | 1000,100      |
| 1070,700      | 1000,650      |
| 245,700       | 200,650       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image5]
![alt text][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Once I have obtained warped image. I use Lane class object which has flag when lane is detected. when lane is not detected I use sliding winodw search for detecting lane pixels after histogram searcj for best place to start. Sliding search sppears like this

![alt text][image7]

when pixel for lanes are found, a second order polynomial fitted on pixels. An example of fitted line is attached here
![alt text][image8]

when leane is detected then pixels around polynomial is searched and ne line is fitted on that curve. if lane are not conclusive then skip the step. if lane are not found for 10 continuous frames then sliding window searched is performed. Criteria for lane to be conclusive are
average distance  between two lane should be within margin
curvature of two line should not be too much difference
and curvature should not be too high.

function process_image() includes all the code. 

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.
To calculate radius of curvature, polynomial coefficients on pixel world is converted into real world polynomial coefficients. Code for the coverting polynomial into realworld included in section Convert pixel polynomial into real world polynomial. 

once polyniomial is converted into real world then curvature is calculated using function meas_curvature()

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

 Here is an example of my result on a test image:

![alt text][image9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_video/project_video1.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I managed to achieve line dtection reasonably well but there are challenges/shortcomings which need to be addressed. I will try to discuss challenges step by step

Image thresholding: 
I used combinationation of s-channes, r-channel and b-channel and gradient magnitude and direction threshold. despite using this, lane pixels may be missed in some of following scenario
   1. lane marking is covered with snow or any other things
   2. Snow is making line al shadowongside of lane marking
   3. shadow of trees, overbridges
   4. Sharp bend on the road etc.
   
Perspective transformation:
Perpective transformation works well on considerably flat roads but I think image transformation may not be that good if road is steep specially when road is bending.

Sliding window search:
Sliding window is dependant on rectangle size. if rectangle is too big, it may contain too many outliers and lane may deviate its path. and if rectangle is small then pixels may be missed.  In some cases, binary thresholding picks up pixels which fall into far end of rectangle and results into deviation in path.

Polynomial fit:
Polynomial fit try to fit all the pixels which contains outliers and some time lane is fitted in wrong way.

Search around polynomial:
Search around polynomial may not consider points in other direction and lane ditection may not work properly

 
