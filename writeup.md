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

As a first test I applied the distortion correction to the image 'camera_cal/calibration1.jpg'to test that the algorithm was providing the expected results. The output is saved in the folder "output_images/" with the name "calibration1_undistorted.png".

Then, I applied the correction to all the test images in the folder "test_images/.jpg" and saved the comparison in the folder "output_images/undistorted_images" (see cell 6).


### Pipeline (single images)

At this point I have all the calibration and distortion coefficients available and i can procede with the processing of the images. All the undistorted images are also saved in a list named "thresholded_images".

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
        [central_column - 90, int(rows*0.63)],    # high left corner
        [central_column + 90, int(rows*0.63)],    # high right corner
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

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 550  453      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
