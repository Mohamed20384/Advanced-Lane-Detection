# 🚦 Advanced Lane Detection — Road Segmentation with Computer Vision  

![ezgif](https://github.com/user-attachments/assets/28993e17-441f-4f34-82a2-9d2adb9588d1)  

**Advanced Lane Detection** is a computer vision project that transforms raw driving footage into an **augmented video stream**, overlaying detected lane boundaries, radius of road curvature, and vehicle offset from the center.  

This system combines **camera calibration, image transformations, thresholding, perspective warping, and polynomial fitting** to deliver real-time, reliable lane detection for autonomous driving and driver-assist systems.  

---

## 🔑 Pipeline Overview  

The project follows a robust step-by-step approach:  

1. **Camera Calibration**  
   - Estimate camera calibration matrix & distortion coefficients from chessboard images.  
   - Remove lens distortion to get clean, undistorted frames.  

2. **Image Undistortion**  
   - Apply calibration results to raw road images.  
   - Correct for distortion artifacts for more accurate processing.  

3. **Binary Thresholding**  
   - Use color transforms + gradient thresholds to extract lane markings.  
   - Output: binary (0/1) images highlighting only lane features.  

4. **Perspective Transform (Bird’s-Eye View)**  
   - Warp road images into a top-down perspective.  
   - Straightens road lines to simplify detection.  

5. **Lane Pixel Detection & Polynomial Fitting**  
   - Identify lane line pixels using histogram + sliding windows.  
   - Fit a 2nd-order polynomial to map lane curvature.  

6. **Curvature & Vehicle Offset Calculation**  
   - Convert pixel-space to meters using real-world scaling.  
   - Estimate lane curvature radius and car’s lateral position relative to road center.  

7. **Final Overlay**  
   - Warp lane boundaries back onto original frame.  
   - Annotate video with lane boundaries, curvature radius, and vehicle offset.  

---

## 📸 Camera Calibration  

Every camera introduces lens distortion. To correct this, we:  

- Detect chessboard corners (`cv2.findChessboardCorners`).  
- Build **object points** (3D real-world reference) and **image points** (2D projection).  
- Calibrate camera with `cv2.calibrateCamera`.  
- Undistort frames with `cv2.undistort`.  

✅ Result: crisp, distortion-free road images ready for processing.  

![Calibration](https://github.com/user-attachments/assets/b00cc303-9a5e-4b38-878c-fd0035da6f98)  

---

## 🎨 Thresholded Binary Images  

Binary thresholding highlights lane markings using:  

- Sobel Gradients (X, Y, Magnitude, Direction)  
- HLS Saturation Channel  
- RGB Red Channel  
- YUV Luminance  

Different threshold profiles handle **daylight, shadows, and pavement variations**.  

![Binary](https://github.com/user-attachments/assets/08bb71ce-f1e3-445a-9ee4-e6be56a461a5)  

---

## 🕹 Perspective Transform (Bird’s Eye View)  

- Define **source (src)** trapezoid around lane lines.  
- Map to **destination (dst)** rectangle for top-down view.  
- Warp/unwarp using transformation matrices `M` and `Minv`.  

This straightens lane lines, making detection more robust.  

![Warp](https://github.com/user-attachments/assets/2f880d2d-7721-4f44-9c25-a1d3ac137df8)  

---

## 📐 Lane Detection & Polynomial Fit  

1. Build lane histogram from bottom third of binary warped image.  
2. Detect peaks → initialize sliding windows.  
3. Follow lane pixels vertically with **WindowBox** search.  
4. Fit left/right lane lines with a 2nd-order polynomial.  

![Fit](https://github.com/user-attachments/assets/712b249d-b103-469a-9794-64c76ee01c88)  

---

## 🛣 Curvature & Vehicle Position  

- Convert pixels → meters using:  
  ```python
  xm_per_pix = 3.7 / 413   # meters per pixel in X  
  ym_per_pix = 3.0 / 275   # meters per pixel in Y  
