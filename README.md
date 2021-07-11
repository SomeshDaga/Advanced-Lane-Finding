# Advanced-Lane-Finding

Advanced Lane Finding project for Udacity's Self-Driving Car Nanodegree Program

![final-result](lane_finding.gif)

[//]: # (Image References)

[chess_undist]: ./images/Chessboard_undist.png "Distorted vs Undistorted Chessboard"
[threshold]: ./images/combined_threshold.png "Combined Thresholding Effect"
[lane_undist]: ./images/lane_undist.png "Distorted vs Undistorted Lanes"
[perspective]: ./images/perspective.png "Overhead View"
[perspective_lanes]: ./images/perspective_lanelines.png "Perspective Lane Lines"
[result]: ./images/result.png "Final Result"

---

# Goals of the Project

The aim of this project was to trace out the lane lines viewed through the windshield from a provided video, while estimating the radius of the curvature of the road and the offset of the driver from the center of the lane

# Implementation Details

## Camera Calibration

The camera matrix and distortion coefficients were obtained through the use of multiple calibration images of a chessboard, and OpenCV's `findChessboardCorners` and `calibrateCamera` functions. Each chessboard provides a 9x6 grid with corners which are to be matched up against a user-defined set of coordinates for the chessboard corners. 

Below is an image of the result of the undistortion of one of the provided calibration images:
![][chess_undist]

## Image Processing Pipeline

I shall now go through the various steps that were taken to isolate the lane lines through computer vision and superposing them on the frames of the provided video.

### Undistortion

On the assumption that the camera position stays fixed throughout the duration of the video, the same camera matrix and distortion coefficients were used as those obtained during the calibration. Here is its result applied to one of the test images given:
![][lane_undist]

### Thresholding

Two primary types of thresholding were considered in processing the images:
* Gradient (Sobel) Thresholding
* Color Thresholding

It was found that the L & B channels of the LAB color space was most effective at isolating the yellow component (lane) from the frames of the video. The Y channel of the YUV color space was found to work best for the white line, resulting in an almost error free image. While gradient thresholding was attempted, it proved to only eliminate useful points from the resulting combined threshold of the color spaces and hence was not used in the final solution. So, the net combination of thresholding only involved ORing the LAB thresholding for the yellow lane with the YUV thresholding for the white lane. A region of interest mask was applied to the result to eliminate other stray detections.

The code for the thresholding can be found between the start of the `processImage` function and the `Combined Thresholding` comment in the jupyter notebook, `solution.ipynb`

The following is a result of the combined thresholding:
![][threshold]

### Perspective Transform

The next step was to create a perspective transform of the provided image/thresholded bitmask, in order to be able to later fit curves to the lane lines and determine parameters like the radius of curvature and distance from the lane center.

In order to this, four points on the road surface (pixel locations) were manually determined using the notebook backend for the matplotlib library, to be able to interactively check pixel location by hovering over plotted images. This was how I determined the so-called `source` points of the perspective transform. The `destination` points were chosen in order to have the lane cover most of the image window. The OpenCV function `getPerspectiveTransform` was used to obtain the transformation matrix (only evaluated once), and used subsequently to transform all the frames of the video (using the `warpPerspective` OpenCV function). The source code can be found at the start of the `Image Pipeline` section.

Below is a result of the perspective transform on the undistorted image and the thresholded image:
![][perspective]

### Identification of Lane Line Pixels and Curve Fitting

To determine the lane line pixels, the following steps were performed:
* A historgram of summed pixel values along the columns for the bottom half of the image were evaluated and the peak positions on either side of the center of the image were used as the base of the lane lines
* The image was divided into horizontally into a set of windows, with a set width and mean position based on the mean of the pixel positions on the prior window (given that a minimum no. of pixels were found).
* The non-zero pixel values in the windows were then used as points to plot a best-fit curve through

The source code can be found in the `processImage` function, between the `# Lane Finding` and the `# Plot lanes lines on undistorted image in driver's view` comments.

A second order polynomial was used to fit the curves onto the non-zero pixel locations found. The y values were initialized using numpy's `linspace` function to create the y-values in one pixel increments. The x-values were fitted using the polynomial coefficients determined through numpy's `polyfit` function.

Below is an image of the identification and plotting of the lane lines (after the perspective transform):
![][perspective_lanes]

### Calculation of Radius of Curvature and Lane Offset

To calculate the radius of curvature of the lane lines, the curve fits were utilized for both sides of the lane. The first and second derivates were calculated using the polynomial coefficients at the location of the driver (i.e. y value at the base of the image). To make the Radius of Curvature measurement slightly more robust, an average across the previous 5 measurements were calculated and reported (as with the lane offset distances explained below). Given that the lane was 3.7 m across, the pixel to meter conversion was obtained by finding the width of the lane in each frame of the video (in pixels) by taking the mean of the different in the x positions on the left and right curve fits. The pixel to meter conversion factor was then (3.7 / mean_pixel_width).

To calculate the lane offset, the mean pixel location of the center of the lane was calculated by taking the average of the right and left curve fits at the base of the image, and subtracting the center pixel of the image (1280/2 = 640) from this value. The conversion to meters was done based on the conversion factor obtained previously.

The source code can be found under the `Calculate Radius of Curvature (at user's location) and Distance from lane center` comment in the `processImage` function.

### Result

Below is an image of the lane lines drawn in the driver's view:
![][result]

The result on the provided test video can be seen by playing the `solution.mp4` file provided in this repository.

## Discussion

Issues faced include:
* Stray artifacts detected in the thresholding/lane finding from surrounding images that have similar characteristics to the lane lines e.g. white wheel of another car moving past in the neighbouring lane, dirt on the side of the road that follows the curvature of the lane lines etc
* At times, poor detection of the intermittent white lane line as compared to yellow line (just based on no. of pixels detected for both)
* High Fluctuation in radius of curvature measurements

Although the radius of curvature measurement showed huge fluctation (and are ultimately unreliable), they did exhibit the correct pattern in that when going around a bend, the ROC was smaller than going along a straight stretch of road. Along the straight stretches, the ROC fluctuated in the range of hundreds of meters (in the same ballpark of the 1 km ROC track that was used in filming the video). To be able to make this more robust, will depend on being able to make the lane detection better such that the lane line positions do not change significantly between frames.

Due to personal time constraints, a better sliding window approach was not implemented between frames of the video to make it more robust, but it is a direction that I would definitely take if I were to do it again. 



