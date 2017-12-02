## Advanced Lane Finding

[//]: # (Image References)

[dist_undist]: ./doc_images/dist_undist.png "Distorted and undistorted images after chessboard calibration"
[mask_1]:       ./doc_images/mask_3.png "Image with color mask applied"
[mask_2]:       ./doc_images/mask_4.png "Image with color mask applied"
[roi_1]:        ./doc_images/roi_3.png "Image with color mask applied for the region of interest"
[roi_2]:        ./doc_images/roi_4.png "Image with color mask applied for the region of interest"
[warp_1]:       ./doc_images/warp_3.png "Image with perspective transform applied"
[warp_2]:       ./doc_images/warp_4.png "Image with perspective transform applied"
[warp_mask_1]:  ./doc_images/warp_mask_3.png "Image with perspective transform & color mask applied"
[warp_mask_2]:  ./doc_images/warp_mask_4.png "Image with perspective transform & color mask applied"
[sliding_win]:  ./doc_images/polyline.png "Initial search for lane line with sliding windows"
[plot_0]:       ./doc_images/plot_0.png
[plot_1]:       ./doc_images/plot_1.png
[plot_2]:       ./doc_images/plot_2.png
[plot_3]:       ./doc_images/plot_3.png
[plot_4]:       ./doc_images/plot_4.png
[plot_5]:       ./doc_images/plot_5.png
[plot_6]:       ./doc_images/plot_6.png
[plot_7]:       ./doc_images/plot_7.png

The goal of this project is to create a software pipeline to identify lane boundaries in a video by:
1. Computing camera calibration matrix and distortion coefficients
2. Applying a distortion correction to images.
3. Using color transforms, gradients, etc to create a thresholded binary image
4. Apply a perspective transform to rectify the image and obtain a birds-eye view
5. Detect lane pixels to determine the lane boundary
6. Determine the curvature of the boundary
7. Warp the detected lane boundaries back onto the original image
8. Display the lane boundaries on the video.

This is the 4th project in Udacity's Self-Driving Car Nanodegree.

# Pipeline
A set of images with every step of the pipeline is shown at the end of the discussion of the pieces of the pipeline. The code for the pipeline is contained in the [Pipeline.ipynb](./Pipeline.ipynb) file. All other ipynb were used for testing out functionality of code.

## Calibration Matrix and Distortion Coefficients
`chessboard_calibrate()` function in Pipline.ipynb

Calibration matrices and distortion coefficients were calculated using 20 10x7 given chessboard images from the cars camera. The camera identified all of the internal corners of the chessboards (9x6) in 17 of the 20 images. The other 3 were identified as 9x5 (1) and 8x6 (2). 

## Correcting Distortion
The calibration matrix and distortion coefficients from the 9x6 set were used to distort the following images. The distorted images are on the left and the undistorted images are on the right.

![Distorted and undistorted images][dist_undist]



## Finding corners
`corners_roi()` function in Pipeline.ipynb

In order to perform a perspective transform, object points from a straight lane line image were chosen such that a perspective transform to a 720x1280 image would produce straight lines. This is a sub-optimal solution. It would be better to find the vanishing point of the image to determine the trapezoid. 

## Perspective Transform
`warp_lane_lines()` function in Pipeline.ipynb

The below images are the straight line images after a perspective transform is applied.

![Straight line 1 with perspective transform][warp_1]
![Straight line 2 with perspective transform][warp_2]

## Failed Attempt

The following two steps were attempted, but they did not produce satisfactory results.

---
### Color Mask
`color_mask()` and `apply_mask()` in Pipeline.ipynb

A color mask was created to show the lane lines and hide everything else. Our goal is to be robust to changes in brightness (such as shadows).

* In order to identify colored lane lines the image is changed to HSV color-space
* Once the image is in HSV color-space, all pixels between a narrow band of colors at a set hue are filtered.
* Lane lines are white and yellow, so a filter for yellow and a filter for white are created
* The filters are combined with bitwise or
* The filters are combined with the original image to display only the lane lines.

The below images are the straight line images after the color mask is applied. A second image is used to show the region of interest created by our corners.

![Straight line 1 with color mask][mask_1]
![Region of interest 1 with color mask][roi_1]

![Straight line 2 with color mask][mask_2]
![Region of interest 2 with color mask][roi_2]

### Transform & Mask
`warp_lane_lines()` function in Pipeline.ipynb

The below are the results of applying the perspective transform and color mask

![Warped and masked image 1][warp_mask_1]
![Warped and masked image 2][warp_mask_2]

### Thoughts
The mask looks for a specific hue range while allowing saturation and value to vary. Saturation is a more important characteristic of the lane lines than hue especially as the lightness or value changes from shaded to unshaded regions.

---

## Color threshold
`color_threshold()` function in Pipeline.ipynb

The image was converted from RGB color space to HSL. The saturation channel was isolated and a lower bound was chosen so that most of the scene besides the lane lines were ignored.

## Sobel threshold
`sobel_threshold()` function in Pipeline.ipynb

The image was converted from RGB to grayscale. A sobel edge detector in the x direction was used to find lane lines.

## Combined threshold
Lines 12-13 in `pipeline()` function in Pipeline.ipynb

The color and sobel thresholds were combined with a binary_or type of opertion in order to capture the results of both thresholds when fitting a polynomial to the lane lines.

## Perspective transform
`warp_lane_lines()` function in Pipeline.ipynb

A perspective transform is performed after the thresholds are combined. The corners are defined by corners_roi() function.

## Polynomial fit
`polynomial_fit()` function in Pipeline.ipynb

Points from the bottom half of the warped image were used to determine the most likely position of the base. From that base points within a 100px wide 80px high rectangle were averaged to determine the x position of the lane lines and follow the lane line. The points were based to the lane line class where a polynomial was fit for the current set of points and averaged with previous detections to determine the radius of curvature and position relative to center.

Once a fit was determined, all points within a set distance of the curve were captured to calculate the next fit. Below is an image with a set of windows drawn over it to show how the initial sliding window works.

![Sliding window][sliding_wins]

## Final image
Line 16-17 of `pipeline()` function in Pipeline.ipynb

The polynomial was projected back onto the undistorted image by reversing the perspective transform from the previous step.

![Pipeline plot][plot_0]

---

![Pipeline plot][plot_1]

---

![Pipeline plot][plot_2]

---

![Pipeline plot][plot_3]

---

![Pipeline plot][plot_4]

---

![Pipeline plot][plot_5]

---

![Pipeline plot][plot_6]

---

![Pipeline plot][plot_7]

---
## Lane Line class
A lane line class was defined that kept the last 10 good results in order to better fit the lane lines. Lane lines on the left and right were tracked separately and only compared afterwards. Lane lines that differed substantially from the previous fits were thrown out. After 60 fits were thrown out the best fit is assumed to be bad. A double-ended queue was used for storing fits since it's faster for FIFO operations.

Radius of curvature and line from center were calculated. In the event that the radius of curvature of the left and right lane lines were significant different (defined as one lane line curvature being twice the other), then the confidence of the lane lines was used to determine the better radius to use.The confidence of the lane line fits was determined by the number of good fits in the recent good fits queue. If the queue was full then the lane was at maximum confidence. If the lane lines were equally confident, then their radius of curvature was averaged. If one lane line was more confident than the other, then that radius of curvature was used and the other was ignored.

![Radius of curvature and car position][roc_pos]

# Video
The [linked video](./example_vid.mp4)is the result of the pipeline on project_video.mp4. Challenge videos are also included though they are not successful, primarily because the perspective transform of the lane lines is hard coded from the project video.

# Improvements
Three areas for improvement are:
1. *Automatically detecting the vanishing point.* Using linear algebra it should be possible to determine the vanishing point of the scene by finding all lines in the scene and seeing where they cross. This would make the pipeline robust to hills and valleys where the horizon would rise and dip.
2. *A more robust method of determining thresholds.* Values for the saturation threshold and sobel edge detection were selected by plotting out a bunch of images with different thresholds and selecting the best looking results. I think it would be useful to come up with a metric to determine how well the thresholds are working so that they can be automated, but I don't have great ideas about how to do this.
3. *Fixing the projection of the lane lines back onto the original image* The red and green lines were plotted on the warped image and then transformed back onto the original image. Due to how the transformation works, the horizontal flat top of the transformed green box was transformed back onto the image as not horizontal sometimes. This looks a little off and could be fixed with more time. 
