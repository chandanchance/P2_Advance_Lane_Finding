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

[image1]: ./Outputs/undistort_output.png "Undistorted"
[image2]: ./Outputs/test1.png "Road Transformed"
[image3]: ./Outputs/binary_combo_example.png "Binary Example"
[image4]: ./Outputs/warped_straight_lines.png "Warp Example"
[image5]: ./Outputs/color_fit_lines.png "Fit Visual"
[image6]: ./Outputs/example_output.png "Output"
[video1]: ./Outputs/project_video_outputNew.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

Steps followed:
1.Camera calibration

2.Distortion correction

3.Perspective transform

4.Color/gradient threshold

5.Detect lane lines

6.Determine the lane curvature

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 2nd code cell of the IPython notebook located in "./Advanced Lane Finding Project.ipynb"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The `mtx` and `dist` from the camera calibration steps were used to undistort the images using `cv2.undistort()`. Here is an example of that:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps are in 13th code cell).  Here's an example of my output for this step. Combined pipeline is in the code cell 16.

1. gradx == x direction sobel filter with kernel size = 3, threshold =(10,235)

2. grady == y direction sobel filter with kernel size = 3, threshold =(50,255)

3. mag_binary == magnitude sobel filter with kernel size = 3, threshold =(40,200)

4. dir_binary == direction sobel filter, threshold =(0.7,1.2)

5. s_binary == s filter , threshold =(180,255)

NOTE: R filter was added to the pipeline as suggested in the previous review.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `get_perspective()`, which appears in lines 1 through 7 in the 7th code cell.  The `get_perspective()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src_left_top = [int(img_size[0] / 2 - 58), int(img_size[1] / 2 + 100)]
src_right_top = [int(img_size[0] / 2 + 65), int(img_size[1] / 2 + 100)]
src_right_bottom = [int(img_size[0] * 5 / 6) + 35, img_size[1]]
src_left_bottom = [int((img_size[0] / 6) ), img_size[1]]

perspective_src_vertices = np.float32([src_left_top, src_right_top, src_right_bottom, src_left_bottom])      

dst_left_top = [int(img_size[0] / 4), 0]
dst_right_top = [int(img_size[0] * 3 / 4), 0]
dst_right_bottom = [int(img_size[0] * 3 / 4), img_size[1]]
dst_left_bottom = [int(img_size[0] / 4), img_size[1]]

perspective_dst_vertices = np.float32([dst_left_top, dst_right_top, dst_right_bottom, dst_left_bottom])  
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 582, 460      | 320, 0        | 
| 705, 460      | 960, 0        |
| 213, 720      | 320, 720      |
| 1101, 720     | 960, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

The verification pipeline:

1. undistoring the loaded image

2. Cropping the image

3. Applying the perspective transform


![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Steps followed to get the lane lines and fitting a polynomial fit(Code is found in the 19th code cell of the jupyter notebook):

1. loading the image and undistorting the image using the previously found out `mtx` and `dist`.

2. using the `combined_edge_detection()` to detect the edges using color and gradient thresholds so that a binary image is generated.

3. `region_of_interest()` is used to mask the other parts of the image.

4. `get_perspective` is used to get the transform of the lane lines.

5. `fit_polynomial()` which inturn calls `find_lane_pixels()` is coded to get the required output.
    `find_lane_pixels()` does the following things:
    A) with the help of the hostogram `midpoint`, `leftx_base` and `rightx_base` are found out so that we know where to start the windowing.
    B) No of windows, height of each window are defined.
    C) `leftx_current` and `rightx_current` keep changing as the window rolls.
    D) Window is rolled to find out `leftx`, `lefty`, `rightx`, `righty`


![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

f(y) = Ay*y+By+C and curvature = (1+(2Ay+B)^2)^1.5/absolute(2A) is used in the cell 22. Function `measure_curvature_real()` is defined to find out the curvatures.

Note: `np.polyfit()` is used to fit the left and right lanes in `fit_poly()` in cell 21. 

NOTE: Calculation of the position of the car with respect to center has also been altered as suggested.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines cell 21 with function `pipelineNew()`. This function calls all other funtions to detect the lane of the road.
Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [video1](./Outputs/project_video_outputNew.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I have walked step by step as tought in the classroom videos to complete this project.
Tunning of the parameters according to the scene needs to be done to make the pipeline more solid. As I have used many filtering techniques and hard coded the filtering parameters, there are chances of pipeline going wrong if the scene is changed.

So in conclusion, I would say hardcoding the parameters which take the ROI or the filtering parameters for the filtering techniques need to be avoided.
