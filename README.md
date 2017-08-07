# Advanced-Lane-Finding
Advanced Lane Finding project for Udacity's Self-Driving Car Nanodegree Program

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

![chess_undist][center]





