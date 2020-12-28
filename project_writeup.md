# **Advanced Lane Lines Project Writeup**

### This project's goal is to code a software pipeline to identify the lane boundaries in a video. In order to do that, we tested different thresholds into the gradients, sobel and color channels with the different images provided by Udacity in the starter repo, once they have passed at the same time for a transformation process, regarding undistortion and warp (Eagle-eye perspective).

---

I delivered the expected rubric points as follow:

* Project Readme

* A jupyter notebook containing detailed steps followed to approach the solution.

    * **Camera calibration**: extract matrix and distortion coefficients. Provided distortion correction to raw images 
    
    [Tested Image](./output_images/test_undist.jpg)

    * **Sobel, gradients and color transform** to create a thresholded binary image, obtained through the function helpers properly identified at the notebook. Conveniently displayed for each method, including the resulting conbination of the final pipeline.

    * **Perspective transformation** is also applied and tested *(Please refer to the writeup to see example)*

    * Method used to **identify lane line pixels** *(See notebook for detailed guide through)*

    * Determine radius of **curvature** of the lane and the position of the vehicle with respect to center.

    * **Warp** the detected lane boundaries back onto the original image.

    * Output **visual display** of the lane boundaries and numerical estimation of lane curvature and vehicle position. Resulting **pipeline application** to the road images, displaying successfully lane line recognition.

* Final [video output](output_video/project_video.mp4)

* Project [writeup](project_writeup.md)



[//]: # (Image References)

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_testing.png "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/sliding_windows.png "Fit Visual"
[image6]: ./examples/resulting_display.jpg "Resulting Display"
[image7]: ./examples/line_drawing.png "lines drawing"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/README.md) is a template writeup for this project you can use as a guide and a starting point.  


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained from cell 2 to 4 in the jupyter notebook provided `advanced_lane_lines.ipynb` located in this repo. Also an testing image prooving undistortion can be found in ('output_images/test_undist.jpg')

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

Also available at the cell 5 in the notebook `advanced_lane_lines.ipynb`

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (Helpef function and thresholding steps at cells 10 through 18 in `advanced_lane_lines.ipynb`). 

```# Analyze images resulting

titles = ['SobelX', 'SobelY', 'Magnitude', 'Direction', 'Combined'] # Sobel, Gradient Magnitude, Gradient Direction, Combination
results = list(zip( result_sobel_X, result_sobel_Y, result_magnitude, result_direction, result_combined ))
results_title = list(map(lambda images: list(zip(titles, images)), results))[3:6]
flatten_results = [item for sublist in results_title for item in sublist]

```

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I performed a perspective transformation from line 23 to 27 in `advanced_lane_lines.ipynb`, basically using source and destination points, the `warper()` code takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([ 
    [left2_x, left2_y],
    [right1_x, right1_y],
    [right2_x, right2_y],
    [left1_x, left1_y]])
dst = np.float32([
    [offset, 0],
    [img_size[0]-offset, 0],
    [img_size[0]-offset, img_size[1]], 
    [offset, img_size[1]]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 466      | 200, 0        | 
| 705, 466      | 1080, 0       |
| 1130, 720     | 1080, 720     |
| 190, 720      | 200, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]
![alt text][image7]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I applied the helper functions related to Finding Lane Lanes in cell 28 in the notebook `advanced_lane_lines.ipynb`, First I define the conversion (pixels to meters), `ym_per_pix = 30/720` and `xm_per_pix = 3.7/700` and used the following functions:

* `find_lines`, find the polynomial representation of the lines.
* `visualize_lanes`, visualize the windows and fitted lines for `image`.
* `lanes_on_image`, display `images` on a [`cols`, `rows`] subplot grid.

** *Note*: ** *Please refer to the cell #29 to display the related polynomail application.*
```# Sliding window
images_poly = lanes_on_image(test_images)
```


![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I applied the helper functions related to calculate the radius of the curvature of the lane and the position of the vehicle with respect to the center in cell 28 in the notebook `advanced_lane_lines.ipynb`, and used the following functions:

* `calculate_curvature`, returns the curvature of the polynomial `fit` on the y range `yRange`.
* `draw_lines`, draw the lane lines on the image `img` using the poly `left_fit` and `right_fit`.
* `draw_lane_image`, find and draw the lane lines on the image `img`.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I applied the helper functions related to result of the whole **pipeline** in cell 28 in the notebook `advanced_lane_lines.ipynb`, and used the following functions:

* `pipeline`, find and draw the lane lines on the image `img`.

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Please refer to the functions *describing text* in cell 34 in the notebook `advanced_lane_lines.ipynb` for details in how I applied the pipeline in the video.

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Was pretty interesting to me to play around all the gradients, channel colors, and apropiate thresholds for each type of image and challenging to find a *common* approach to the final binary **combined** result. In deed, it took me a while that part, and when failing the first times, was the first thing to double check without success, since the lane lines finding issues I faced were related to properly identify the area of interest and that influenced directly my scaled window approach. I guess for now, I am satisfied and wanting to test it with the challenge videos and my own. 
