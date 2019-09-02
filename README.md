## Advanced Lane Finding Project


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

[image1]: ./output_images/calibration3.png "Finding Corners"
[image12]: ./output_images/undistort.png "Undistorted"
[image2]: ./output_images/channels.png "Selecting channels"
[image3]: ./output_images/thresholds.png "Thresholding"
[image4]: ./output_images/warped.png "Warp Example"
[image5]: ./output_images/hist.png "Histogram"
[image6]: ./output_images/boxes.png "Boundary Boxes"
[image7]: ./output_images/polyfit.png "Fit polynomials"
[image8]: ./output_images/fitaroundpoly.png "Fit around last poly"
[image9]: ./output_images/drawn.png "Final Output"
[video1]: ./output_project_video.mp4 "Video"

### Camera Calibration

First, I setup 2 arrays, objpoints and imgpoints. The first array contains points localisation disposed in a grid of 9 by 6.
Then I detected corners from a calibration image with OpenCV's findChessboardCorners() method. This corners are going to be the reference corners used in imgpoint.
The `cv2.calibrateCamera()` function is then used to compute the distortion from the imgpoints to the objpoints. I then apply this distortion to a test image and will then apply it to all camera images.

![alt text][image1]

### Pipeline (single images)

#### 1. Distortion-corrected image.

Here is the distortion applied to a regular camera image :
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I decided to split the image into 2 differents channels:
- A Gray scale version of the red channel
- The S channel of the HLS color space

![alt text][image2]

I then used thresholding to highlight only the relevant information from the image.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Then, to get a top down view. I used a straight line image and hand selected 4 points forming a rectangle shape.
src = np.float32([[589,457], [700,457], [1016,665], [287,665]])
I then selected points on the destination image with a rectangle shape too in the dst variable.

I then used cv2.getPerspectiveTransform(src, dst) and cv2.warpPerspective(img, M, img_size, flags=cv2.INTER_LINEAR) to warp the image accordingly to the selected points.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To detect the lane pixels, I ploted the white pixel of the first half on and histogram. It allows me to locate where the lines are starting.

![alt text][image5]

From this 2 points, i used boundary boxes to locate the white pixels situated inside the boxes. After collecting the points in an array, I average the x position of the points to place my next boxes.
If there isnt enough points in the current box, I will move the next box as much and in the same direction as the previous boxs moved. It allows me to more precisely guess where the next points will be and not miss them.

![alt text][image6]

After that I was able to fit 2 polynomials with the collected coordinates. I created another function to be able to look for white points around the last detected polynomial.

![alt text][image7]
![alt text][image8]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I first had to be able to convert pixels to meters so I took a straight ligne picture and assumed the lane measured 3.7m. I measured the lane in pixel, it was 775 pixels. So the x conversion is 3.7/775 .
 
To calculate the radius i had to fit 2 new polynomials converting the pixels to meters. These new polynomials were then used to calculate the radius with the formula :

((1 + (2 * left_fit_cr[0] * y_eval * ym_per_pix + left_fit_cr[1] )**2)**(3/2))/ abs(2 * left_fit_cr[0])

To calculate the offset from the center of the lane, I calculated the x positions at y=720 which is the bottom of the image. I subtracted the left x to the right x and divided by 2 (average of the 2 positions). I then substracted this position from the center of the image and converted to meter

I also calculated the size of the lane with a similar method.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I then unwarped the image and drawn everything on it.

![alt text][image9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

The pipeline works as follows:

- first, detect a line with boundary boxes
- if the line is detected correctly (more or less 3.7m at begining and end of the lane), append the polynomials to a record array and then draw them
- if it's not succesful it will skip to the next frame
- for the next frame it will use the last polynomials to look for the line around them
- if it doesnt work, it will try again with the boundary boxes
- if all of this fails, it will display an average of the 5 last polynomials recorded

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I didnt code the pipeline object oriented. I should have done that to be able to store values more easily instead of using global variables.

The pipeline will fails when it encounters a shadow, I did not find any thresholing values helping with the shadow problem.
