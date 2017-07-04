# search_and_sample_return
Robotics Nanodegree Program Project

Notebook Analysis

1. Here is the code for identifying obstacles and a binary image of the rock_img.
        ```python
        #This function will give a binary image for obstacles given an image
        def find_obstacle(img, rgb_thresh=(160, 160, 160)):
            # Create an array of zeros same xy size as img, but single channel
            color_select = np.zeros_like(img[:,:,0])
            # Require that each pixel be below all three threshold values in RGB
            # below_thresh will now contain a boolean array with "True"
            # where threshold was met
            below_thresh = (img[:,:,0] < rgb_thresh[0]) \
                        & (img[:,:,1] < rgb_thresh[1]) \
                        & (img[:,:,2] < rgb_thresh[2])
            # Index the array of zeros with the boolean array and set to 1
            color_select[below_thresh] = 1
            # Return the binary image
            return color_select
        ``` 
