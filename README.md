# **Finding Lane Lines on the Road** 

## Goals/ Steps
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images_output/regioned.png "Region Filter"
[image2]: ./test_images_output/colored.png "Color Filter"
[image3]: ./test_images_output/edged.png "Canny Edge Filter"
[image4]: ./test_images_output/houghed.png "Hough Line Filter"
[image5]: ./test_images_output/lined.png "Extended Lines"
[image6]: ./test_images_output/overlayed.png "Final Weighted Image"


---

## Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

Pipeline: Region -> Color -> Canny Edge -> Hough Transform -> Extend lines -> Overlay

My pipeline consisted of four steps: a region mask, a color filter, canny edge detection,
and a hough line transform. It also remembers the previous frame's lane positions in case
the right, left, or both lane markers are not found in the image. Since the lanes do not move much between frames,
this was used a safeguard when the pipeline failed to find the lane markers; this seemed to only happen with shadows or a few times when the segmented lane markers were missed. I initially used a grayscale conversion,
but after some testing, I found out that just using the color mask for white and yellow
worked much better for the optional challenge than using grayscale. After each
step in the pipeline, an image is shown as an example, specifically, yellowcurve2.jpg.

I first started by using a region mask to black out everything not inside a quadrilateral.
I used percentages rather than pixels to make it more robust; which helped
for the optional challenge. I removed a small portion of the bottom to eliminate the car's
hood in the challenge video.

![alt text][image1]

Second, I applied a color filter to eliminate anything not yellow or white. This filter
helped with the challenge video as the change in road color was throwing off my previous
pipeline that used a grayscale conversion. Now with these two filters, the images were
just lane makers and the rest of the image was black. Also, without this filter being
second in the pipeline, the Canny Edge detector would output horizontal lines from the
shadows of the trees in the challenge video.

![alt text][image2]

Third, I used Canny Edge detection to find the edges of the lane marks. Using simple
statistics, I could the function choose appropriate parameters for cv2 Canny.
I calculated the average, and used a sigma of 0.25 in each direction to find a min and max.
From those values, I got a lower and upper threshold; 0.25 sigma seemed to work the best.

![alt text][image3]

Fourth, I used the Hough Transform to find the lanes in the images. I decided on a threshold
of 50 to include less designated lane markers (were due to shadows and road color). I set the
minimum line length to 10 so smaller lines would be discarded and only lane markers would be
outputted. Also, I set the maximum line gap to 2 (a small number) since some of the lane
markers were broken up into a couple lines with small pixels between them.

![alt text][image4]

To draw a single line on the left and right lanes, I modified the draw_lines() 
function by dividing the lines input into either left or right and then averaging the
values into one line. Now with an average for each side, I calculated the slope of each
line and then extended the line to the x intercept and to 65% of the vertical size. My
function calculated the x-intercepts for each line by manipulating the point-slope
equation, and then calculated the x value for the corresponding 65% y value. The
function could then send the two points for each line to cv2.line().

![alt text][image5]
![alt text][image6]

### 2. Identify potential shortcomings with your current pipeline


There are a few shortcomings to my pipeline. The first shortcoming would be that the
pipeline is tuned for yellow and white lane markers. This was used to discard any gray
or shadows that were causing issues for the Canny Edge detector. If the lane markers'
paint is wearing off or some exotic color for some reason, like blue, the pipeline
would filter them out.

Another shortcoming is that my pipeline latches the previous frame's lane positions
in case it cannot find lane markers for the current frame. Since the lanes do not
move much from frame to frame, this is usually fine. However, if the pipeline can
not find lanes in the first image, it wouldn't draw anything since there is no
previous lanes latched; this was not the case for any of the three videos.

Building on the previous shortcoming, the pipeline could produce incorrect lines if
it cannot find any new lanes for a non-negligible number of frames. For example, if
the pipeline could not find any new lines for five seconds or about 150 frames, it would
use the last lines found (five seconds ago) and these could be off enough to notice.
However, in practice, it would be hard for this case to happen.

Lastly, this whole pipeline assumes the camera is stationary and would not move. If
for some reason the camera were to move, the region mask would distort every subsequent
step in the pipeline.


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to adjust the image to remove shadows rather than
only allowing yellow and white colors for the case of wearing paint and exotic lane
marker colors. This would help eliminate many of the lines found by the Canny Edge
detector that were from shadows. Another way to remove these, would be to filter
out any horizontal lines since the lane markers should only be vertical (with some 
slant); so, filter out any lines not within a subset of slopes.

Another possible improvement would include making the latch mechanism more robust.
So, averaging the lane markers from previous frames with current rather than just
using the new frame's lanes would smooth out the lines superimposed on the videos to make it look less jerky.