## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)
![Lanes Image](./examples/example_output.jpg)

In this project, your goal is to write a software pipeline to identify the lane boundaries in a video, but the main output or product we want you to create is a detailed writeup of the project.  Check out the [writeup template](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) for this project and use it as a starting point for creating your own writeup.  

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

---

### 1 camera calibration

#### 1.1 camera matrix and distortion coefficients
Brief description how the camera matrix and distortion coefficients are computed. 

The code in following method: `detection_apis.cal_undistort`. The input image is as dimension (1280, 720, 3). No of corners by x and y are given to be assumed as (9, 6) 

Prepared "object points", that is (x, y, z) coordinates of chessboard corners. Below methos does the job: `detection_apis.get_img_obj_points`.

Assuming, chessboard is fixed on (x, y) plane with z=0, so object points will be same for every calibration image.  

Hence, `objp` will only be a replicated array of coordinates. `objpoints` will be appended with a copy every time all chessboard corners are successfully detected in a test image.  
`imgpoints` will be stored with the (x, y) pixel pos for each corner in image with every successful detection.  

`objpoints` and `imgpoints` are used to calculate camera calibration & distortion coefficients with `cv2.calibrateCamera()`.  distortion coeff is applied to test image with`cv2.undistort()` method. 


### 2 pipeline (test images)

#### 2.1 example of applying undistortion on image
Distortion correction has been correctly applied to every image. 

#### 2.2 color transformation
color transforms, gradients and/or methods to create a binary thresholded img

following thresholds 
* Absolute sobel threshold (x, y) direction  =>  Source:  `detection_apis.py#sobel_filter` 
* rgb threshold on red & green colors => source `detection_apis#rgb_filter` 
* yuv threshold on s-channel => source `detection_apis.py#yuv_filter` 
* hsv threshold on s-channel => source `detection_apis#hsv_filter`
* hls threshold on s-channel and l-channel => source `detection_apis.py#hls_filter`

#### 2.3 pipeline (test images)
Describtion about how a perspective transform is performed on image with included example

perspective transform method is in function `perspective_transform`
and it does a function call `warp_perspective`.  source points and destination points are hardcoded here.

```python
def get_source_points():
    return [[220,720], [1100, 720], [780, 470], [600, 470]]

def get_destination_points(width, height, fac=0.3):
    fac = 0.3
    p1 = [fac * width, height]
    p2 = [width - fac * width, height]
    p3 = [width - fac * width, 0]
    p4 = [fac * width, 0]
    destination_points = [p1,p2,p3,p4]
    return destination_points
```

It resulted in the below src and dest points:

| src        | dest   | 
|:-------------:|:-------------:| 
| 220, 720      | 384, 720        | 
| 1100, 720      | 896, 720      |
| 780, 470     | 896, 0      |
| 600, 470      | 384, 0        |


#### 2.4 Identifying lane line pixels and Polynomial fit

After applying calibration, threshold, perspective transform to a road image, output image with lines comes out clearly. 


#### 2.5 radius of curvature and center offset 
Description how radius of curvature is calculated and the position of the vehicle with respect to center.

* **Radius of curvature cal** - *source : detection_apis.py#get_curvature_radius*
 ```
    fit_cr = np.polyfit(ploty * ym_per_pix, x * xm_per_pix, 2)
    # Calculate the radius of curvature
    curverad = ((1 + (2 * fit_cr[0] * y_eval * ym_per_pix + fit_cr[1]) ** 2) ** 1.5) / np.absolute(2 * fit_cr[0])
```

* **Offset from center calculation**  detection_apis.py#get_offset_from_center*  
```
    lane_center = (right_x[height-1] + left_x[height-1]) / 2
    xm_per_pix = 3.7 / 700  # meters per pixel in x dimension
    img_center_offset = abs(width / 2 - lane_center)
    offset_metters = xm_per_pix * img_center_offset
    return offset_metters
```

#### 2.6 Example image of result

Rectified image is warped back to the orig image and plotted to get lane boundaries. It shows lane boundary was identified correctly.

### 3 Pipeline (video)

#### 3.1 Video Output
Please take a look at the vidoe output file attached with github code

---

### 4 Discussion

The most obvious and biggest fault with pipeline is it's very difficult have composition of thresholds on color spaces. Also, applying filters will not work well with all road conditions, lights, shadows, noise.

Problem is solved partially, if lane lines are not detected, lane lines are drawn in blind mode. Lane lines computed before are drawn unless you have enough data to represent lines.

We need better methods and strategy that have set of filters to learn and build outputs in better way.

Problen cannot be solved purely with computer vision and machine learning algorithm are needed to annotate lane lines. CNN like architectures can be deployed to label and train some lanes before testing and correcting the final ones

