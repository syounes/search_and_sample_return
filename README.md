# search_and_sample_return
Robotics Nanodegree Program Project Submission

I'll go through the basic implementation and the necessary steps for the project submission.

I got throught the environment setup with the python starterkit and finally I run the Rover_Project_Test_Notebook.ipynb from the project repository folder under the RoboND environmnet and Jupyter Notebook.

Notebook Analysis

   1. Obstacle and rock sample identification.
   
      Perspective Transform
      
      In the perspetive transform image we've mapped the navigable terrain, the obstacles that appears in 
      dark color and we identified the field of vew of camera by creating a mask
      ```
      # I've used the example grid image above to choose source points for the
      # grid cell in front of the rover (each grid cell is 1 square meter in the sim)
      def perspect_transform(img, src, dst):
           
         M = cv2.getPerspectiveTransform(src, dst)
         warped = cv2.warpPerspective(img, M, (img.shape[1], img.shape[0]))# keep same size as input image
         mask = cv2.warpPerspective(np.ones_like(img[:,:,0]), M, (img.shape[1], img.shape[0]))
         
         return warped, mask
        
      # These source and destination points are defined to warp the image
      # to a grid where each 10x10 pixel square represents 1 square meter
      # The destination box will be 2*dst_size on each side
      dst_size = 5 
      # Set a bottom offset to account for the fact that the bottom of the image 
      # is not the position of the rover but a bit in front of it
      # this is just a rough guess, feel free to change it!
      bottom_offset = 6
      source = np.float32([[14, 140], [301 ,140],[200, 96], [118, 96]])
      destination = np.float32([[image.shape[1]/2 - dst_size, image.shape[0] - bottom_offset],
                        [image.shape[1]/2 + dst_size, image.shape[0] - bottom_offset],
                        [image.shape[1]/2 + dst_size, image.shape[0] - 2*dst_size - bottom_offset], 
                        [image.shape[1]/2 - dst_size, image.shape[0] - 2*dst_size - bottom_offset],
                        ])
      warped, mask = perspect_transform(grid_img, source, destination)
      ```
      
      ![Alt text](/misc/perspective.png?raw=true "Title")

      Color Thresholding
      
      ```
      # Identify pixels above the threshold
      # Threshold of RGB > 160 does a nice job of identifying ground pixels only
      def color_thresh(img, rgb_thresh=(160, 160, 160)):
         # Create an array of zeros same xy size as img, but single channel
         color_select = np.zeros_like(img[:,:,0])
         # Require that each pixel be above all three threshold values in RGB
         # above_thresh will now contain a boolean array with "True"
         # where threshold was met
         above_thresh = (img[:,:,0] > rgb_thresh[0]) \
                     & (img[:,:,1] > rgb_thresh[1]) \
                     & (img[:,:,2] > rgb_thresh[2])
         # Index the array of zeros with the boolean array and set to 1
         color_select[above_thresh] = 1
         # Return the binary image
         return color_select      
      ```
      
     
   2. ```process_image()``` analysis & worldmap creating

Autonomous Navigation and Mapping 

   1. ```perception_step()``` and ```decision_step()``` explanation and why these functions were modified as they were.
   
   2. Autonomousmode navigation results explanation and how we might improve them.

