## Advanced Lane Finding

[//]: # (Image References)

[dist_undist]: ./doc_images/dist_undist.png "Distorted and undistorted images after chessboard calibration"

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


