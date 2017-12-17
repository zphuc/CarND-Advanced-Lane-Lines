
# Advanced Lane Finding Project

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

[image1]: ./results_images/undistort_output1.png "Undistorted"
[image2]: ./results_images/undistort_output2.png "Undistorted"
[image3]: ./results_images/binary_combo_example.png "Binary Example"
[image4]: ./results_images/warped_straight_lines.png "Warp Example"
[image5]: ./results_images/color_fit_lines-1.png "Fit Visual"
[image6]: ./results_images/color_fit_lines-2.png "Fit Visual"
[image7]: ./results_images/example_output.png "Output"
[image8]: ./results_images/failure_example-1.png "Failure"

## Rubric Points

#### Here I will consider the [rubric](https://review.udacity.com/#!/rubrics/571/view)  points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cell [1],[2] of the IPython notebook located in `P4.ipynb`.  

I start by preparing "object points" `objp` with size of nx*ny, which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The thresholding steps are contained in the code cells [3],[4],[5] of the IPhython notebook `P4.ipynb`. I did the following steps.

* Apply an image mask of the white and yellow color using the `cv2.inRange()`  to obtain a output image with much more interesting region of white and yellow lanes (by the `mask_white_yellow()` function in code cell [4]).
* Convert the image to HLS color space with a threshold range for S-Channel.
* Apply a gradient thresholds in X direction for the image.

Then, I used a combination of color and gradient thresholds to generate a binary image (code cell [5]).  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in the code cell [6] of the IPython notebook `P4.ipynb`.  The `warper()` function takes as inputs an image (`img`). A source (`src`) and destination (`dst`) points are defined as the global variables. I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 585, 460      | 320, 0        |
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image (test_images/straight_lines1.jpg) and its warped counterpart to verify that the lines appear parallel in the warped image (see the red lines in the warped result).

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

 These steps are contained in the code cells [7],[8] of the IPhython notebook `P4.ipynb`.

 I used the method and codes which were introduced in the "Finding the Lines" class.

After applying above transforms to obtain a warped binary image, I take a histogram along all the columns in the lower half of the image. Finding the peak of the left and right halves of the histogram, I can get the staring points for the left and right lines.

Implement the sliding windows from the staring points, I can get the "hot" pixels associated with the lane lines. Then fit the pixels with a second order polynomial to obtain the lane lines like this:

![alt text][image5]
![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

This step is contained in the code cells [9] of the IPhython notebook `P4.ipynb`.

 In here, I used the method and codes, which were introduced in the "Measuring Curvature" class, to calculate the radius of curvature of the lanes.

I calculate the center of the  lanes using the staring points of the left and right lane at the bottom of the image.
Supposed that the vehicle is in the center of the image, I can calculate the `offset` of the vehicle with respect to the center of the lanes.   

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lthe code cells [10] of the IPhython notebook `P4.ipynb` in the function `draw_detectedLane(img)`.  Here is an example of my result on the test images:

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's my results of the following video:
* project_video : [project_video_out.mp4](./project_video_out.mp4)
* challenge_video : [challenge_video_output.mp4](./challenge_video_output.mp4)
* harder_challenge_video : [harder_challenge_video_output.mp4](./harder_challenge_video_output.mp4)

Noted that, the `challenge_video` and `harder_challenge_video` have some failures that should be improved in the further study.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In the first step, I applied the color/gradient thresholds with some tunning of the threshold ranges. However, I found some failures and the detected region becomes unstable at some area of the strong solar radiation effect.

Here is an example of the fail image:  
(also see a fail video. [project_video_output-failure.mp4](./project_video_output-failure.mp4)):

 ![alt text][image8]

In order to improve the problem, I applied the `mask_white_yellow()` function (code cell [4] in `P4.ipynb`). An limitation range for finding the peak of the left and right halves of the histogram also applied in the `cal_leftright_coefs()` function (code cell [8] in `P4.ipynb`) as follows.

```python
midpoint = np.int(histogram.shape[0]//2)
ww=histogram.shape[0]
leftx_base = np.argmax(histogram[50:midpoint])
rightx_base = np.argmax(histogram[midpoint:(ww-50)]) + midpoint
```

After the improvement, I can get the better video result as show above.
