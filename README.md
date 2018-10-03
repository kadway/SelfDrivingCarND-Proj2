## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

[//]: # (Image References)

[image1]: ./camera_cal/calibration1.jpg "Original chessboard image"
[image2]: ./camera_chess_undist/chess_undist_calibration1.jpg "Undistorted chessboard image"
[image3]: ./test_images/straight_lines1.jpg "Test image"
[image4]: ./output_images/undistorted/straight_lines1.jpg "Undistorted test image"
[image5]: ./output_images/color_gradient_transformed/straight_lines1_combined_binary.jpg "Color and gradient transformed image"
[image6]: ./output_images/warped/_original_straight_lines1.jpg "Warped image test"
[image7]: ./output_images/fitted_poly/straight_lines1.jpg "Fitted poly"
[image8]: ./output_images/out_image/straight_lines1.jpg "Radious"
[video1]: ./output_videos/project_video.mp4 "Project video"

The Project
---

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

---

### Camera Calibration

The code for this step is contained in the IPython notebook located in `./camera_calibration.ipynb`.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

Original Chessboard
![alt text][image1]

"Undistorted Chessboard"
![alt text][image2]

### Pipeline (single images)

#### 1. Example of a distortion-corrected image.

Here is an example of the `straight_lines1.jpg` original test image.

![alt text][image3]

The undistortion of an image is performed by the `undistort(img, img_path=0)` function in the `Advanced_lane_finding.ipynb` notebook.
Firstly the camera calibration parameters are loaded and then feed in to the openCV function `cv2.undistort(img, mtx, dist, None, mtx)` which returns the undistorted image.
The `img_path` is the image path and is only passed on to the function for the purpose of saving a copy of the modified image to the respective folder in `/output_images/`.

After the distortion correction, the test image looks like this:

![alt text][image4]

#### 2. Color transforms and gradients to create a thresholded binary image.
I used a combination of color and gradient thresholds to generate a binary image.
The thresholding steps are performed in function `transform(image, img_path=0)` in `Advanced_lane_finding.ipynb`.

The undistorted image was converted to gray scale and the OpenCV Sobel() function was used to apply x and y gradient to the image.
Sobel x and y gradients thresholded images. The x and y gradients were then thresholded into a binary image. (function `abs_sobel_thresh(img, orient='x', sobel_kernel=3, thresh=(0, 255)):`)

The gradient magnitude and direction were calculated from the Sobel x and y gradients. 

Gradient magnitude is calculated with: `np.sqrt(sobelx**2 + sobely**2)`
And gradient direction: `np.arctan2(np.absolute(sobely), np.absolute(sobelx))`
​
The color channels S and L were also thresholded and combined into the final binary image.
The image was first converted to HLS with `cv2.cvtColor(img, cv2.COLOR_RGB2HLS)` and then the individual color channels were thresholded.

The combined binary image is performed by combining all the binary images in the following way:
`((gradx == 1) & (grady == 1)) | ((mag_binary == 1) & (dir_binary == 1)) | ((s_binary == 1) & (l_binary == 1))`

Here's an example of my output for this step using the `straight_lines1.jpg` test image.

![alt text][image5]

#### 3. Perspective transform.

The code for my perspective transform includes a function called `warper()`,  in the 4th code cell of the IPython notebook `Advanced_lane_finding.ipynb`.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. 
I chose to hardcode the source and destination points, according to the code provided in the writeup_template, in the following manner:

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

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image6]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image7]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image8]

---

### Pipeline (video)

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

