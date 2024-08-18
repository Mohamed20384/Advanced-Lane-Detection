# Advanced-Lane-Detection
Advanced Lane Segmentation - Computer vision

![ezgif](https://github.com/user-attachments/assets/28993e17-441f-4f34-82a2-9d2adb9588d1)

In Advanced Lane Segmentation, we apply computer vision techniques to augment video output with a detected road lane, road radius curvature and road center offset. 

Steps of this project are the following:
Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
Apply a distortion correction to raw images.
Use color transforms, gradients, etc., to create a thresholded binary image.
Apply a perspective transform to rectify binary image (“birds-eye view”).
Detect lane pixels and fit to find the lane boundary.
Determine the curvature of the lane and vehicle position with respect to center.
Warp the detected lane boundaries back onto the original image.
Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

Camera Calibration
Every camera has some distortion factor in its lens. The known approach to correct for that in (x,y,z) space is apply coefficients to undistort the image. To calculate this a camera calibration process is required.
It involves reading a set of warped chessboard images, converting them into grey scale images before using cv2.findChessboardCorners() to identify the corners as imgpoints.

![image](https://github.com/user-attachments/assets/c06c7291-d9f7-41ad-b56c-2e76893d0e04)

If corners are detected then they are collected as image points imgpoints along with a set of object points objpoints; with an assumption made that the chessboard is fixed on the (x,y) plane at z=0 (object points will hence be the same for each calibration image).
In the function camera_calibrate I pass the collected objpoints, imgpoints and a test image for the camera image dimensions. It in turn uses cv2.calibrateCamera() to calculate the distortion coefficients before the test image is undistorted with cv2.undistort() giving the following result.

![image](https://github.com/user-attachments/assets/b00cc303-9a5e-4b38-878c-fd0035da6f98)

Distortion corrected image
The undistort_image takes an image and defaults the mtx and dist variables from the previous camera calibration before returning the undistorted image.

![image](https://github.com/user-attachments/assets/3d3e0f7b-4a09-4270-bed5-a5ca6d0c7f91)

Threshold binary images
A threshold binary image, as the name infers, contains a representation of the original image but in binary 0,1 as opposed to a BGR (Blue, Green, Red) colour spectrum. The threshold part means that say the Red colour channel( with a range of 0-255) was between a threshold value range of 170-255, that it would be set to 1.
A sample output follows.

![image](https://github.com/user-attachments/assets/08bb71ce-f1e3-445a-9ee4-e6be56a461a5)

Initial experimentation occurred in a separate notebook before being refactored back into the project notebook in the combined_threshold function. It has a number of default thresholds for sobel gradient x&y, sobel magnitude, sober direction, Saturation (from HLS), Red (from RGB) and Y (luminance from YUV) plus a threshold type parameter (daytime-normal, daytime-bright, daytime-shadow, daytime-filter-pavement).
Whilst the daytime-normal threshold worked great for the majority of images there were situations where it didn't e.g. pavement colour changes in bright light and shadow.

![image](https://github.com/user-attachments/assets/7148e51c-f582-45cc-b9f3-d6abe99e8550)
Daytime Normal with noise bright light & pavement change
![image](https://github.com/user-attachments/assets/50163cbb-3a1e-4cdb-adac-aa852a93d1fb)
Daytime Normal with shadow
Perspective transform — birds eye view
To be able to detect the road lines, the undistorted image is warped. The function calc_warp_points takes an image's height & width and then calculates the src and dst array of points. perspective_transforms takes them and returns two matrixes M and Minv for perspective_warp and perpective_unwarp functions respectively. The following image, shows an undistorted image, with the src points drawn with the corresponding warped image (the goal here was straight lines)

![image](https://github.com/user-attachments/assets/2f880d2d-7721-4f44-9c25-a1d3ac137df8)

Lane-line pixel identification and polynomial fit
Once we have a birds eye view with a combined threshold we are in a position to identify lines and a polynomial to draw a line (or to search for points in a binary image).

![image](https://github.com/user-attachments/assets/975711d1-5609-4d0b-8160-019b9fb2cd67)

topdown warped binary image

A histogram is created via lane_histogram from the bottom third of the topdown warped binary image. Within lane_peaks, scipy.signal is used to identify left and right peaks. If just one peak then the max bin either side of centre is returned.
calc_lane_windows uses these peaks along with a binary image to initialise a left and right instance of a WindowBox class. find_lane_window then controls the WindowBox search up the image to return an array of WindowBoxes that should contain the lane line. calc_fit_from_boxes returns a polynomial or None if nothing found.
poly_fitx function takes a fity where fity = np.linspace(0, height-1, height) and a polynomial to calculate an array of x values.


The search result is plotted on the bottom left of the below image with each box in green. To test line searching by polynomial, I then use the left & right WindowBox search polynomials as input to calc_lr_fit_from_polys. The bottom right graphic has the new polynomial line draw with a blue search window (relates to polynomial used for the search from WindBoxes) that was used overlapping with a green window for the new.

![image](https://github.com/user-attachments/assets/712b249d-b103-469a-9794-64c76ee01c88)

Warped box seek and new polynomial fit
Radius of curvature calculation and vehicle from centre offset
In road design, curvature is important and its normally measured by its radius length. For a straight line road, that value can be quite high.
In this project our images are in pixel space and need to be converted into meters. The images are of US roads and I measured from this image the distance between lines (413 pix) and the height of dashes (275 px). Lane width in the US is ~ 3.7 meters and dashed lines 3 metres. Thus xm_per_pix = 3.7/413 and ym_per_pix = 3./275 were used in calc_curvature. The function converted the polynomial from pixel space into a polynomial in meters.
To calculate the offset from centre, I first determined where on the x plane, both the left lx and right rx lines crossed the image near the driver. I then calculated the xcentre of the image as the width/2. The offset was calculated such (rx - xcenter) - (xcenter - lx) before being multiple by xm_per_pix.

lane.result_decorated
![image](https://github.com/user-attachments/assets/c3c2e8b6-2589-4037-9da4-d520c989bc4b)



