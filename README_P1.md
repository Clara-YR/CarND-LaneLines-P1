# **Project 1  Finding Lane Lines on the Road** 




[//]: # (Image References)

[image1]: ./examples/gray.jpg "Grayscale"
[image2]: ./examples/blur_gray.jpg "Gaussian blurring"
[image3]: ./examples/canny_edges.jpg "Canny"
[image4]: ./examples/masked_edges.jpg "Mask"
[image5]: ./examples/line_img.jpg "Hough Function"
[image6]: ./examples/lines_edges.jpg "Draw lines"

---

### Reflection

### Part 1. Process and Function Descriptions of My Pipeline

My pipeline consisted of 6 steps in total.

Step 1, I converted the images to grayscale via `grayscale()`.
![alt text][image1]


Step 2, then I use `gaussian_blur()` to smooth the images. By using [Gaussian function](https://en.wikipedia.org/wiki/Gaussian_blur) Gaussian blur filters the image and transforms the image pixel function much more continuous and less discrete so that it'll be more convenient for us to differentiate it.
By using Gaussian function Gaussian blur filters the image and transforms the image pixel function much more continuous and less discrete so that it'll be more convenient for us to differentiate it.
![alt test][image2]


Step 3, the Canny function `canny()` is executed to detect the edges. I set the low and high threshold as 100 and 120 respectively, which have a smaller range than the recommended value (50 and 150) in the previous exercise, to get rid of edges of other objects as much as possible.
![alt test][image3]


Step 4, I comply the `region_of_interet()` to keep the polygon range containing the lane line and cut off the rest. The polygon is defined by four points `[(0, imshape[0]), (460, 320), (520, 320), (imshape[1], imshape[0])]`.
![alt test][image4]


Step 5 , applying Hough Transform and iterate over the output "lines" as well as draw lines on a blank image. Run `hough_lines()` and `draw_lines` is run in the former. 

- `rho = 1` and `theta = np.pi/180` -- 1 pixle distance and 1 radians angular resolution of Hough grid
- `threshold = 50` -- intersections to be choose should have at least 50 votes, improve this parameter can exlude disturbance from other blur edges. 
- `min_line_len = 100` and `max_line_gap = 150` -- increase the `min_line_len` to draw lines long enough so that most short edges of other objects, and improve the `max_line_gap` to draw single lines rather than segments.

I change the thickness in `draw_line` from 2 to 4 and the to draw bolder lines.

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by drawing lines while the slope > 0 and < 0 respectively.

Moreover, I add an intersection point (x0, y0) to extend the lane line through the entire frame length of the video. See more details in Part 5.
![alt test][image5]


Step 6, implement `weighted_img` to draw the lines on the initial image and save the output in directory test_images_output. The parameter `β` in `weighted_img` is modified from 1.0 to 0.6 to draw half transparent lane line.
![alt test][image6]



### Part 2. Identify Potential Shortcomings with My Current Pipeline


One potential shortcoming would that my pipeline wouldn't function normally when the lane lines are not located in the polygon region exactly. In this situation, the function `region_of_interest` may not satisfy our expectation.

Another shortcoming could be my pipeline can only detect straight lane lines. It will be out of order in front of curved lane lines. The slope condition in `draw_lines` is an unignorable contribution.


### Part 3. Suggest Possible Improvements to My Pipeline

A possible improvement would be to modified the parameters, especially the low and high Canny thresholds as well as the Hough lines parameters.

Another potential improvement could be to teach my pipeline to detect edges in a colorful image rather than grayscale image. Thus I can add a condition to only detect edges of objects only with a color of white and yellow.


### Part 4. Changes in Help Function in 1st submission

1. The thickness in `draw_lines` is changed from 2 to 4 in order to draw bolder lines.

2.  Moreover, I add a slope condition as a filter to preclude lines with slope far away from the lane lines.

```
slope = (y2-y1)/(x2-x1)
if slope**2 > 0.3 and slope < 1:
```

3.  β in `weighted_img` is changed from 1.0 to 0.6 to draw half transparent lines.


### Part 5. Changes in Help Function in 2nd submission

1. The `min_line_len` is changed from 40 to 100 as recommended.

2. `import math` to use `math.`

2. I discuss whether the slope is positive or negative respectively to draw single right and single left lane line:

```
# draw the right lane line
if slope > 0.5 and slope < 1:
    cv2.line(img, (x1, y1), (x0, y0), color, thickness)
# draw the left lane line
elif slope < -0.5 and slope > -1:
    cv2.line(img, (x0, y0), (x2, y2), color, thickness)
```
3. What's more I calculate the point (x0, y0), which is the intersection point of the screen edge and the line decided by (x1, y1) and (x2, y2), by taking use of the relationship: (y0 - y1)/(x0 - x1) = (y2 - y1) / (x2 - x1).

__NOTE:__There're three conditions of the intersection point, on the bottom or the right or the left edge of the screen, it's necessary to discuss the possible situations one by one to get the valid values of (x0, y0). If I take (x0, y0) = (x_bottom, y_bottom) as default, it may report error “cannot convert float NaN to integer” if the intersection point is actually on the right or the left edge of the screen.

```
# get the intersection point (x0, y0) of the line and the screen edge
# (x_bottom, y_bottom) is the potential intersection point on the bottom screen edge
y_bottom = img.shape[0]
x_bottom = (y_bottom - y1)//slope + x1
if not math.isnan(x_bottom):
    x0 = int(x_bottom)
    y0 = int(y_bottom)
else:
    # (x_right, y_right) is the potential intersection point on the right screen edge
    x_right = img.shape[1]
    y_right = (x_right - x1)*slope + y1
    if not math.isnan(y_right):
        x0 = int(x_right)
        y0 = int(y_right)
    else:
        # the potential intersection point on the left screen edge
        x0 = 0
        y0 = int((0 - x1)*slope + y1)
```