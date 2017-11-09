## Writeup

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

[undistorted]: ./examples/undistorted.jpg "Undistorted"
[test_image_undistorted]: ./examples/test_image_undistorted.jpg "Test image undistorted"
[test_image_combined]: ./examples/test_image_combined.jpg "Test image combined"
[test_image_warped]: ./examples/test_image_warped.jpg "Test image warped"
[test_image_sliding_window]: ./examples/test_image_sliding_window.jpg "Test image sliding window"
[test_image_area]: ./examples/test_image_area.jpg "Test image area"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cells in the "Camera calibration" section of the Jupyter notebook "./Advanced-Lane-Finding.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world.
Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each
calibration image.  Thus, `objp` is just a replicated array of coordinates, and `obj_points` will be appended with a copy
of it every time I successfully detect all chessboard corners in a test image. `img_points` will be appended with the
(x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output `obj_points` and `img_points` to compute the camera calibration and distortion coefficients using
the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the
`cv2.undistort()` function and obtained this result:

![alt text][undistorted]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][test_image_undistored]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.

The code for this step is contained in the code cells in the "Thresholding" section of the Jupyter notebook.

I used a combination of color and gradient thresholds to generate a binary image.

For the color thresholding I tried the HSV and HSL color space and ended up isolating the S channel in the HSL color space.
This is where I got the best result and both the white and yellow lines were crisp.

I also used the Sobel operator to find changes in color intensity in the image. The first step was to convert the image to grayscale.
Then I applied multiple variants of the Sobel operator and combined that with the color thresholding to detect lanes in a robust way.

Here's an example of my output for this step:

![alt text][test_image_combined]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for this step is contained in the code cells in the "Perspective transform" section of the Jupyter notebook.

The code for my perspective transform includes a function called `warp()` that takes an image as an input, as well as source (`src`) and destination (`dst`) points.
I hardcoded the source and destination points in the following manner:

```python
src = np.array([[195, bottom_y], [602, 445], [675, 445], [1110, bottom_y]], np.float32)
dst = np.array([[320, bottom_y], [320, 0], [960, 0], [960, bottom_y]], np.float32)
```

I chose this points by mesuring them directly on the image. I obtained better results with this method.

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image
and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][test_image_warped]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for this step is contained in the code cells in the "Pipeline" section of the Jupyter notebook (line 50).

The logic I used to detect lane lines was the one described in the course: sliding windows.

I first took an histogram along all the colums in the lower half of my image. This helps me finding the x position of the base
of the lane lines (the two most prominent peaks in the histogram). I then used a sliding window around the line centers to find
and follow the lane line until I reach the top of the frame.

I did not try to use convolutions because of time.

Here's an example of sliding windows applied to the test image:

![alt text][test_image_sliding_window]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for this step is contained in the code cells in the "Pipeline" section of the Jupyter notebook (line 151).

In order to calculate the curvature of the line we need to fit new polynomials to find x for y in the real world. This will make
use of some predefined pixels to meters conversion values.

After that we are able to compute the radius of the curvature in real world space using this two statements:

```python
left_curverad = ((1 + (2 * left_fit_cr[0] * y_eval * self.ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2 * left_fit_cr[0])
right_curverad = ((1 + (2 * right_fit_cr[0] * y_eval * self.ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2 * right_fit_cr[0])
```

We now have a radius of curvature in meters.

The last thing we need to do is calculate the position of the vehicle in the lane. We use the left and right lane polynomial to
calculate the center offset in pixels and then we convert it to meters.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code for this step is contained in the code cells in the "Pipeline" section of the Jupyter notebook (line 170).

The code for this step is actually quite simple once you have the x,y position of the lanes. The first step is to draw
the lane area on an empty RGB image of the same size of our original image. At that point the area drawn is warped so we
need to apply an inverse perspective transform before merging it with our original unwarped image.

Here is an example of my result on a test image:

![alt text][test_image_area]

Left curvature: 10350.204372m
Right curvature: 7674.50313469m
Centre offset: 0.2301133898 m

---

### Pipeline (video)

#### 1. Provide a link to your final video output. Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./test_videos_output/project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Given more time the first thing I would like to do is to implement a look-ahead filter in order to skip the sliding window step
once I know where the lines are, on subsequent frames. This would make the detection algorithm much faster and more robust as well.

The detection is also a bit wobbly sometimes. Implementing smoothing as well as the look-ahead filter described above would certainly
help with that.

Finally I would have liked to experiment with more colors spaces and thresholding parameters. In some of the frames the lines are
not as easily detectable as others because there is a lot of noise.
