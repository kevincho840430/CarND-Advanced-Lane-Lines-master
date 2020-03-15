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

---
In this case we use lots of CV algorithm to achieve the goal of fitting road.
### Camera Calibration

First I use chessboard to calibration camera,I start by preparing `object points`, which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. Thus, `objp` is just a replicated array of coordinates, and objpoints will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. imgpoints will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

![](https://i.imgur.com/VCTVjHg.png)

## Apply a distortion correction to raw images
And then I used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the cv2.calibrateCamera() function. I applied this distortion correction to the test image using the cv2.undistort() function and obtained this result:
![](https://i.imgur.com/0FU9Msr.png)

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![](https://i.imgur.com/JOmGCcQ.png)



#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

After undistorted image,we get the correct view on cameras.Next I used a combination of color and gradient thresholds to generate a binary image.
The result looks like:

![](https://i.imgur.com/1bwnTyF.png)

I combine all of the thresholds to get the final color sapce image.

![](https://i.imgur.com/Xu21hSj.png)

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

To compute the perspective transform, i'm using cv2.getPerspectiveTransform() function, and cv2.warpPerspective() to generate a warped image given the perspective transform. In this case we're mapping the perspective view to a flat plane, so we need four source points and four targe points.


The code for my perspective transform includes a function called `warper()`, which appears in `transform_perspective` function.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.array([[(width*0.4, height*0.65),
                 (width*0.6, height*0.65),
                 (width, height),
                 (0, height)]], 
                 dtype=np.float32)
dst = np.array([[0,0], 
                [img.shape[1], 0], 
                [img.shape[1], img.shape[0]],
                [0, img.shape[0]]],
                dtype = 'float32')
```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |
I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![](https://i.imgur.com/lVen7s3.png)

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

The algorithm calculates the histogram on the X axis. Finds the picks on the right and left side of the image, and collect the non-zero points contained on those windows. When all the points are collected, a polynomial fit is used (using np.polyfit) to find the line model. On the same code, another polynomial fit is done on the same points transforming pixels to meters to be used later on the curvature calculation. The following picture shows the points found on each window, the windows and the polynomials:

![](https://i.imgur.com/LfNPWH2.png)

![](https://i.imgur.com/6VD9me1.png)



#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

On the step 4 a polynomial was calculated on the meters space to be used here to calculate the curvature. The formula is the following:

`((1 + (2*fit[0]*yRange*ym_per_pix + fit[1])**2)**1.5)/np.absolute(2*fit[0])`
where fit is the the array containing the polynomial, yRange is the max Y value and ym_per_pix is the meter per pixel value.

To find the vehicle position on the center:

- Calculate the lane center by evaluating the left and right polynomials at the maximum Y and find the middle point.
- Calculate the vehicle center transforming the center of the image from pixels to meters.
- The sign between the distance between the lane center and the vehicle center gives if the vehicle is on to the left or the right.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

To display the lane lines on the image, the polynomials where evaluated on a lineal space of the Y coordinates. The generated points where mapped back to the image space using the inverse transformation matrix generated by the perspective transformation.

![](https://i.imgur.com/yoCUueP.png)


---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

The project video was processed and the results at `video_output`

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?
- When i do the perspective transform,it could not to find the lane clearly
- the challenge videos are not good enough to fit the line
- maybe can use better algorithm to fit the lane