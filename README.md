## Advanced Lane Finding

[//]: # (Image References)

[dist_undist]: ./doc_images/dist_undist.png "Distorted and undistorted images after chessboard calibration"
[mask_1]: ./doc_images/mask_1.png "Image with color mask applied"
[mask_2]: ./doc_images/mask_2.png "Image with color mask applied"
[roi_1]: ./doc_images/roi_1.png "Image with color mask applied for the region of interest"
[roi_2]: ./doc_images/roi_2.png "Image with color mask applied for the region of interest"
[warp_1]: ./doc_images/warp_1.png "Image with perspective transform applied"
[warp_2]: ./doc_images/warp_2.png "Image with perspective transform applied"
[warp_mask_1]: ./doc_images/warp_mask_1.png "Image with perspective transform and color mask applied"
[warp_mask_2]: ./doc_images/warp_mask_2.png "Image with perspective transform and color mask applied"

The goal of this project is to create a software pipeline to identify lane boundaries in a video by:
1. Computing camera calibration matrix and distortion coefficients
2. Applying a distortion correction to images.
3. Using color transforms, gradients, etc to create a thresholded binary image
4. Apply a perspective transform to rectify the image and obtain a birds-eye view
5. TODO - Detect lane pixels to determine the lane boundary
6. TODO - Determine the curvature of the boundary
7. TODO - Warp the detected lane boundaries back onto the original image
8. TODO - Display the lane boundaries and an estimation of curvature on the video.

This is the 4th project in Udacity's Self-Driving Car Nanodegree.

## Calibration Matrix and Distortion Coefficients
Calibration matrices and distortion coefficients were calculated using 20 10x7 given chessboard images from the cars camera. The camera identified all of the internal corners of the chessboards (9x6) in 17 of the 20 images. The other 3 were identified as 9x5 (1) and 8x6 (2). 

## Correcting Distortion
The calibration matrix and distortion coefficients from the 9x6 set were used to distort the following images. The distorted images are on the left and the undistorted images are on the right.

![Distorted and undistorted images][dist_undist]



## Finding corners
In order to perform a perspective transform, object points from a straight lane line image were chosen such that a perspective transform to a 720x1280 image would produce straight lines. This is a sub-optimal solution. It would be better to find the vanishing point of the image to determine the trapezoid. 

## Perspective Transform
The below images are the straight line images after a perspective transform is applied.

![Straight line 1 with perspective transform][warp_1]
![Straight line 2 with perspective transform][warp_2]

## Color Mask
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

## Transform & Mask
The below are the results of applying the perspective transform and color mask

![Warped and masked image 1][warp_mask_1]
![Warped and masked image 2][warp_mask_2]