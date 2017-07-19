# **Finding Lane Lines on the Road**

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 9 steps to convert the original image:

![alt text](/readme/00_orig.png)

1. Detection of image shape and calulation of all values that are needed in the
   following (e.g. the polygon for masking) accordingly.
- Selection by color. Limits were iterately optimizing during debugging. If the
  pipeline failed, the color was determined using the pipette from gimp. Final
  values are `224, 137, 40` (RGB). From all values, a `safety = 10` is subtracted
  before applying to the image.
  
  ![alt text](/readme/02_color_filter.png)
- Conversion to grayscale to save computational power (one channel instead of three).

  ![alt text](/readme/03_grayscale.png)
- Application of Gaussian smoothing in order for small, unnecessary edges to
  vanish.
  
  ![alt text](/readme/04_smoothing.png)
- application of a poly mask to focus on important regions of the image. The
  polygon was defined dynamically based on the image dimensions.
  
  ![alt text](/readme/05_polygon.png)
- Canny edge detection to detect large changes in image lightness. These are
  the edges needed for further processing. `low_threshold = 70` and
  `high_threshold = 150` yielded best results.
  
  ![alt text](/readme/06_canny.png)
- Hough transform and line identification in hough space. Lines are searched in
  hough space since it's easier to identify similar lines and to aggregate them.
  
  ![alt text](/readme/07_hough.png)
- Lane identification (for details, see below).

  ![alt text](/readme/08_lanes.png)
- Final image.

  ![alt text](/readme/99_final.png)

In order to draw a single line on the left and right lanes, I implemented the
`draw_lane_lines()` function based on the `draw_lines()`. This is how it works:

- It uses all lines (defined as `slope` and `y-intercept`) that we're found in
  hough space.
- For each line it is decided, if it's part of a left lane marking (then the
  `slope` is smaller than 0) or a right lane (`slope` larger than 0, else).
- Detected hough lines are filtered for senseless slopes (left: `slope > 0.2`, right:
  `slope < 0.2`). Only points (two points form a line) that fulfill this criteria
  are considered in the following. This increases robustness.
- For each lane (right / left) all `(X/Y)` coordinates that form a hough line
  are collected - one line results into two `(X/Y)` pairs.
- Using the `np.polyfit()` function, two lanes are fitted based on all
  available points for the lane (`np.polyfit(left_lane_lines_x, left_lane_lines_y, 1)`).
- `np.polyfit` returns a linear and constant component. These are used to finally
  plot the lane. This methodology can be adopted to higher order polynoms easily.
- If no lanes are detected, requirements are lowered automatically. `min_line_len`
  is decreased by 3 and `max_line_gap` is increased on every iteration temporarily for the current image.

### 2. & [3.]: Shortcoming [and how to solve them]

- Pipeline only considers the current frame and neglects the historical information
    - Use past frames as well.
- Lanes are not well represtented when going around corners
    - Higher order polynoms to fit lane markings while going around corners better  would be nice.
- Current pipeline optimization is a painfull, manual process
    - All pipeline settings for all stages encapsuled in one object, automated
    optimization. A possible heuristic for optimization would be to penalize
    fast changes of lane markings.
- Lanemarks sometimes jump
    - Filtering lanemarkings with a kalman filter to avoid fast changes and/or jumps.
    - Averaging over multiple frames.
- pipeline is only applicable to the given camera angle.
  - Use of the car acceleration sensor to adjust the polygon of interest if
    camera angle changes (e.g. when the car drives over a speed bump).
