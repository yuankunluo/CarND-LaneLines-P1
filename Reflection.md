# **Finding Lane Lines on the Road**

---

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

[//]: # (Image References)

[image1]: ./doc/output_1.challenge.jpg "Challenge"
[step1]: ./doc/step1.png  "Select Yellow Lane"
[step2]: ./doc/step2.png "Select White Lane"
[step3]: ./doc/step3.png "Combine selections"
[image2]: /doc/4.solidYellowCurve.jpg "solidYellowCurve"
[step7]: ./doc/step7.png "Hough-Transform"
[step8]: ./doc/step8.png "Draw Lane Lines"
[step9]: ./doc/step9.png "Final Step"
[lost]: ./doc/lost.jpg "Lost"
[wrong]: ./doc/wrong.jpg "Lost"
---

# Reflection

![Finding Lane Lines][image1]

## 1. Pipeline

Since there are yellow and white colored lane lines, to select both lane lines uniformly by grayscale is not good enough. We need to select both color range separately.

For this purpose I first select yellow lane in HLS color space, then select white lane in HSV color space.

![Select Yellow][step1]
![Select White][step2]

After selections I combine them into one selection.

![Combine Selections][step3]

The we apply Gaussian Blur and Canny Edge Detection to the combined image. The parameters to these 2 processes are the same as we learned in lectures.

To get ride of irrelevant parts, I use a function to get the vertices of ROI (Region of interest), which will returns the right vertices depends on the input image's size.  

The Hough-Transform's parameters have been tuned, because on some case, for Example the right lane on the solid yellow curve image has only very short dashed lines. We need more sensitive Hough-Transform-Process to get us more line segments, that will help us with determining the slopes for lane lines. As shown in the following figure (Debug Mode on), we need more detailed information on the right side.

![Hough-Transform][step7]

The Hough-Transform-Process give us only line segments, that is not enough for us to get lane lines, because there are dashed lines, more ever the left and right lane should be distinguished.

For every line segment, we firstly check its slope. It is a part of left lane line, if its slope is negative. Otherwise it belongs to right side.

For all left or right side segments, we collect their locations. Then use polyfit() to find the best 1-D polynomial that has the minimum squared error.

```
# Example for left side points
# We collect all xs and ys
# Then calling polyfit to find a polynomial that
# fits them

left_fit = np.polyfit(left_xs, left_ys, 1)
left_poly = np.poly1d(left_fit)
```

After finding a proper polynomial, we need to draw it on the image. To accomplish this goal, we first calling linspace() to return us some x points within the left or right part of the region of interests.

```
# Example for the left polynomial
# It has slope of -0.744 and intercept 658.1
left_poly  
-0.744 x + 658.1
```

![Draw Lane Lines][step8]

For every x the linespace() returned, we compute its coresponding y. These xs and ys are combined into locations. Then simply by calling cv2.polylines() function to draw a line of the polynomial depends on the locations we get in last step.

Because we already have points of polynomial to draw, so we only need to tune the draw_lines() function a little bit.

```
def draw_lines(img, points, color=[255, 0, 0], thickness=2):
  lanes_image = np.copy(img)
  # Calling polylines() to draw a line
  cv2.polylines(lanes_image, points,1,color=color, thickness=thickness)
  return lanes_image
```

The final step is just weighted images by calling the pre-defined helper function.

![final Step][step9]


## 2. Potential shortcomings

One potential shortcoming is, it is unable to detect shadowed white lane. Since we select white lane in HSL color space, it is hard to select the a little grayed shadowed white lane.

![lost][lost]

The detection on Video can some times be unstable. That lead to a unsmooth transformation on videos. I think this happens due to the polyfit() function, which I used to interpolate line segments.

![wrong][wrong]

This program can only detect road during day time, I don't know if it will still work for night version.

## 3. Suggest possible improvements to my pipeline

1. I need to continue working on the interpolate/average process. So that I can produce smoothly lane Detection.

2. To get rid of the influence of shadows, I could find a way to eliminate shadows like we do with Photoshop.
