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


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  


### Project structure

In the IPython notebook of the project (named "AdvancedLinesDetection.ipynb") each of the goals previously mentioned are explicitly shown: for each one of them there is a cell that reads all the test images and plot the results side by side with the input image. 
The results of each section are also saved in the folder "output_images/" with the creation of a subfolder for each one of the steps.
The name of each subfolder will be specified in the next steps of this writeup.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the cell number 2 and 4 of the IPython notebook.

First the corner detection on all the chessboard images (in the folder 'camera_cal/) is computed with the OpenCV function `cv2.findChessboardCorners()`. The corners are then appended to a list of image points.
Then, also a list of object point is computed: they are same for each calibration image.
Based on these two vectors I could compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.

As a first test I applied the distortion correction to the image 'camera_cal/calibration1.jpg' to test that the algorithm was providing the expected results. The output is saved in the folder "output_images/" with the name "calibration1_undistorted.png".

Then, I applied the correction to all the test images in the folder "test_images/.jpg" and saved the comparison in the folder "output_images/undistorted_images" (see cell 6).


### Pipeline (single images)

At this point I have all the calibration and distortion coefficients available and I can procede with the processing of the images. All the undistorted images are also saved in a list named "thresholded_images".

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Here (cell number 7) I will use gradient and color techniques on our undistorted images in order to create a thresholded binary image. 
Here the functions I used for this step: 

- The absolute Sobel gradient along x 
- The direction of the gradient to highlight edges on a particular direction
- A color thresholding on the HLS space, in both S and L channels
- As a final step I defined a function called `thresholding_pipeline()` that combines all the binary output provided by the previous functions.

As the previous section, all the results are saved in the folder "output_images/thresholded_images/" where it is possible to find a comparison between all the undistorted images and the thresholded images (cell number 8).

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The third step of the algorithm is the perspective transform.

The code for my perspective transform is in the cell number 9 of the notebook and it calls the `cv2.getPerspectiveTransform()` function. The necessary inputs are the image that we want to transform, the source points and the destination points.
These points have been hardcoded in the function.
I chose the the source and destination points in the following manner:

    src = np.float32(
        [
        [central_column - 92, int(rows*0.63)],    # high left corner
        [central_column + 92, int(rows*0.63)],    # high right corner
        [central_column - 860, last_row],          # low left corner
        [central_column + 860, last_row]           # low right corner

    ])
        
    dst = np.float32(
        [
        [0, 0],                 # high left corner
        [last_column, 0],       # high right corner
        [0, last_row],          # low left corner
        [last_column, last_row] # low right corner
    ])

This resulted in the following values:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 548   453     |   0, 0        | 
| 732   453     |   1279, 0     |
| -220   719    |   0, 719      |
| 1500  719     |   1279, 719   |

This function also returns the inverse of the transformation matrix because it will be necessary later.

I verified that my perspective transform was working as expected by plotting side by side the input image with the one with the applied transform. All the images have been tested and the results are saved in the folder "output_images/transformed_images".
This part can be found in the cell number 10.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The next step of the pipeline is the identification of the lane line pixels on the image. Once the lines pixels have been found they are fit with a second order polynomial.

As learned during the classroom, I implemented this step (cell 11) defining the function `find_lane_pixels()` that expects as input the warped image from the previous step and uses the following techniques:

- Histogram technique: a histogram is defined here in order to find the starting point for the left and right lines close to the ego-vehicle

- Sliding window technique: a window that moves up in the lane-lines direction is defined first finding the window boundaries in x and y for boht right and left lines. Then the the nonzero pixels are identified and if a certain number of pixels is found then the next window will be centered on their mean position.

- Fitting a polynomial: once we have all the pixels of both lines they are fit with a second order polynomial by calling the function `np.polyfit()`.

The final part of this step is to check whether the results are satisfactory or not. In order to do so, in the cell 12 I am taking all the warped images and drawing on them the windows and the polynomial that has been calculated. All the images with the comparison are saved in the following folder 'output_images/windows_images/'.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

This section has been implemented in the cell number 13 of the notebook.

Here the task is to compute the curvature of the polynomial found in the previous section. To accomplish this, I used the  formula learned during the classroom that assumes that the curve of the road can be modelled as a circle. The main relevant step here is the definition of the conversions in x and y from pixels space to meters: 

    ym_per_pix = 30/720 # meters per pixel in y
    xm_per_pix = 3.7/700 # meters per pixel in x

The next part of this step is the calculation of the ego vehicle position with respect to the center of the lane. 
Here the assumption that the camera is mounted in the center of the car is made. In this was it is possible to compute the vehicle center simply by dividing by 2 the total number of columns of the image.
Then, the center of the lane is calculated starting by the left and right polyinomials representing the lane-lines.
Finally, the distance of the ego vehicle from the center line is the difference of these two last computed values.
Again here, I have to use the constant `xm_per_pix` to convert the calculation to meter.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

To complete this step I implemented the function `lane_area_identification()` that can be found at the cell number 15.

The main aim of this step is to plot the lane boundaries back to original image space. Note that for this step it necessary to use the inverse perspective matrix (Minv).
In this function it also computed the average radius (starting from the left and the right one). I decided to saturate the average radius to 5km in order to handle straight lines (where it tends to infinite).

In the cell 16 I am taking as input the original test images and showing on them the obtained results. The images can be found in the folder 'output_images/identified_area_images'

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here is it possible to find the link to the video output: 
https://github.com/marcorossi10/CarND-Advanced-Lane-Lines/blob/master/project_video_output.mp4

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The major challenge I faced was finding a lane-lines identification that was robust to light and shadow changes. 
The most difficult part of the video was when the street changes color to a lighter gray: here the original pipeline I implemented was failing and the lane identification was extending up to the road boundary on the left. 
To solve this issue I had to add a colour threshold on the L channel.

The pipeline could fail anytime that the colour and/or edge thresholding fails to find the lines and unfortunatetly this is quite likely to happen since the pipeline was tested only on a specific scenario. Thus, a good starting point to improve it would be to define a pipeline able to tune the parameters on a more extended set of scenarios.

The line identification is sometimes wobbling. This is mainly due to the fact that there is no memory used in the algorithm: a better results could be obtained by adding a sort of filter that takes into account previous information in order to smooth the current one. But overall, I am satisfied with my lane lines identification, even the radius estimation is providing values within an accettable range.
