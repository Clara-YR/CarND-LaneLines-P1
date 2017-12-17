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

### 1. Process and Function Descriptions of My Pipeline

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
- `min_line_len = 40` and `max_line_gap = 150` -- increase the `min_line_len` to draw lines long enough so that most short edges of other objects, and improve the `max_line_gap` to draw single lines rather than segments.

I change the thickness in `draw_line` from 2 to 4 and the to draw bolder lines.

Moreover, I add a slope condition as a filter to preclude lines with slope far away from the lane lines.
![alt test][image5]


Step 6, implement `weighted_img` to draw the lines on the initial image and save the output in directory test_images_output. The parameter `β` in `weighted_img` is modified from 1.0 to 0.6 to draw half transparent lane line.
![alt test][image6]



### 2. Identify Potential Shortcomings with My Current Pipeline


One potential shortcoming would that my pipeline wouldn't function normally when the lane lines are not located in the polygon region exactly. In this situation, the function `region_of_interest` may not satisfy our expectation.

Another shortcoming could be my pipeline can only detect straight lane lines. It will be out of order in front of curved lane lines. The slope condition in `draw_lines` is an unignorable contribution.


### 3. Suggest Possible Improvements to My Pipeline

A possible improvement would be to modified the parameters, especially the low and high Canny thresholds as well as the Hough lines parameters.

Another potential improvement could be to teach my pipeline to detect edges in a colorful image rather than grayscale image. Thus I can add a condition to only detect edges of objects only with a color of white and yellow.


### 4. Change in Help Function

1. The thickness in `draw_lines` is changed from 2 to 4 in order to draw bolder lines.

2.  Moreover, I add a slope condition as a filter to preclude lines with slope far away from the lane lines.

```
            slope = (y2-y1)/(x2-x1)
            if slope**2 > 0.3 and slope < 1:
```

3.  β in `weighted_img` is changed from 1.0 to 0.6 to draw half transparent lines.
