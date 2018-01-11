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

[image1]: ./media/chessboard_corners.png "Chessboard corners"
[image2]: ./media/undistorted_calibration_image.png "Undistorted image"
[image3]: ./media/undistorted_image.png "Undistorted image"
[image4]: ./media/combined_grad_threshold.png "Combined gradient threshold"
[image5]: ./media/hls_lane_mask.png "Yellow and white lane detection using HLS"
[image6]: ./media/color_thresholded.png "Color threshold using HLS"
[image7]: ./media/combined_gradient.png "Combined gradient"
[image8]: ./media/trapezoid.png "Trapezoid"
[image9]: ./media/bird_eye.png "Bird-Eye View"
[image10]: ./media/lane_lines.png "Lane lines"
[image11]: ./media/lane_overlay.png "Lane overlay"


---

### Camera Calibration

The code for this step is contained in the "Camera calibration using chessboard images" section of the [notebook](./project.ipynb)

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I used `cv2.findChessboardCorners()` function to identify all chessboard corners:
![alt text][image1]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image2]

### Pipeline (single images)

#### 1. Correct image distortion
Image distortion is done using `cv2.undistort()` function, wrapped in `undistort_image()` in my code. Here's an example of an undistorted test image:
![alt text][image3]

#### 2. Create a thresholded binary image


I used a combination of gradient and color thresholds to generate a binary image.

I used a combination of three gradient threshold methods: Sobel Threshold `abs_sobel_threshold()`, magnitude of gradient threshold `mag_threshold()`, and direction of the gradient `dir_threshold()`. All three methods are combined in the `combined_threshold()` method. You can see kernel and threshold settings in the `get_grad_threshold()` method. Example output:

![alt text][image4]

For the color threshold detection I converted images to the HLS colorspace and used masks to detect yellow and white lanes `select_white_yellow_hls()`:

![alt text][image5]

I then selected the S channel to obtain the final color gradient `get_color_threshold()`. Here's an example of a color thresholded image:

![alt text][image6]

After obtaining gradient and color thresholds, I combined them to obtain the final image `combine_grad_color_threshold()`. Here's the result:

![alt text][image7]

#### 3. Perform perspective transform

I used images of a straight road to manually identify a trapezoid that maps to lanes:

![alt text][image8]

 I used the trapezoid to perform the perspective transform. The code for my perspective transform includes a function called `get_warped()` , which takes as inputs an image `image` and uses the hardcoded source `src` and destination `dst` points that I derived from the trapezoid:

| Source        | Destination   |
|:-------------:|:-------------:|
| 280, 671      | 280, 671      |
| 584, 460      | 280, 0        |
| 702, 460      | 1031, 0       |
| 1031, 671     | 1031,671      |

I used `cv2.warpPerspective()` function to perform image transformation to get a bird's eye view of the road:

![alt text][image8]

#### 4. Detect lane lines

I used the histogram line detection algorithm to detect lanes on the image using sliding windows `find_lane_all_windows()`. I then fit the lines with the 2nd order polynomial `get_lane_data_overlay()`:

![alt text][image10]

#### 5. Calculate radius and curvature
I calculated radius of the curvature and converted it to meters. I also found the offset from the image center. `find_radius_meters()`

#### 6. Plot lane outline back on the image
I used two global variables `l_lane` and `r_lane` to track characteristics for each lane and to store data from 10 previous iterations in order to average lane fit. The variables are instances of the class `Lane`.

I output lane curvature and vehicle offset information unto the final image along with the projected lane:

![alt text][image11]

This pipeline is captured in `get_lane_data_overlay()`.

---

### Process video

Video processing is done using the `process_image()` function, which undistorts each frame, generates a combined threshold, applies perspective transformation, detects and fits lines, and generates the final output image.

Here's a [link to my video result](./output_images/project_video_processed.mp4)

---

### Discussion

Lane detection performed reasonably well on the project video using just the Sobel gradient threshold algorithm combined with the HLS algorithm. It did poorly on the challenge video failing to detect a lane that had an asphalt line in the middle. Switching to a combined Sobel, magnitude, and direction gradients addressed this problem, though the overall performance on the challenge video is not acceptable: [challenge video result](./output_images/challenge_video_processed.mp4).

Adding pre-processing of yellow and white lines using masks in the HLS colorspace improved performance in areas where asphalt transitions into concrete, though you can still see some hesitation.

Averaging line data from the previous 10 lines smoothed lane tracking.

## Additional improvements
I would like to spend more time:

* iterating on color thresholds to improve handling of shadows
* speeding up line detection by limiting search area to the previously found lines +/- some margin
* eliminating outliers when searching for lines by finding diffs with previous lines
* replace the hardcoded trapezoid with a calculated area
* try convolutional line detection
