# search_and_sample_return
Robotics Nanodegree Program Project Submission

I'll go through the basic implementation and the necessary steps for the project submission.

Notebook Analysis

   1. Obstacle and rock sample identification.
   
      Perspective Transform
      
      ![Alt text](/misc/example_grid1.jpg?raw=true "Title")
      
      I've used the example grid image above to choose source points for the
      grid cell in front of the rover (each grid cell is 1 square meter in the sim)
     
      ```
      def perspect_transform(img, src, dst):
           
         M = cv2.getPerspectiveTransform(src, dst)
         warped = cv2.warpPerspective(img, M, (img.shape[1], img.shape[0]))# keep same size as input image
         mask = cv2.warpPerspective(np.ones_like(img[:,:,0]), M, (img.shape[1], img.shape[0]))
         # Return the warped image and his mask
         return warped, mask
      ```
      
      These source and destination points are defined to warp the image
      to a grid where each 10x10 pixel square represents 1 square meter
      The destination box will be 2*dst_size on each side
      
      ```dst_size = 5```
      
      Setting the bottom offset
      
      ```bottom_offset = 6```
      
      ```
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
      
      For obstacles we can just invert the color selection that we used to detect ground pixels. 
      Everything above the threshold is navigable terrain, then everthing below the threshold must be an obstacle.
      
      Threshold of RGB > 160 does a nice job of identifying ground pixels only
      ```
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
         
         threshed = color_thresh(warped)
         obs_map = np.absolute(np.float32(threshed) - 1) * mask
      ```
      ![Alt text](/misc/threshold.png?raw=true "Title")
      
      For rocks, we think about imposing a lower and upper boundary in our color selection to be more specific about 
      choosing colors.
      
      RGB of 110,110 and 50 can detect the yellow color of the rocks within the images
      
      ```
      def find_rocks(img, levels=(110,110,50)):
         rockpix = (img[:,:,0] > levels[0]) \
                 & (img[:,:,1] > levels[1]) \
                 & (img[:,:,2] < levels[2])
            
         color_select = np.zeros_like(img[:,:,0])
         color_select[rockpix] = 1
    
         return color_select 
      ```
     
      ![Alt text](/misc/rock.png?raw=true "Title")
     
   2. ```process_image()``` analysis & worldmap creating
   
      Read in csv file as a dataframe
      ```
      import pandas as pd
      
      df = pd.read_csv('../test_dataset/robot_log.csv', delimiter=';', decimal='.')
      ```
      
      Create list of image pathnames
      ```
      csv_img_list = df["Path"].tolist()
      ```
      
      Read in ground truth map and create a 3-channel image with it
      ```
      ground_truth = mpimg.imread('../calibration_images/map_bw.png')
      ground_truth_3d = np.dstack((ground_truth*0, ground_truth*255, ground_truth*0)).astype(np.float)
      ```
      Creating a class to be the data container that read in saved data from csv file and populate this object.
      Worldmap is instantiated as 200 x 200 grids corresponding to a 200m x 200m space (same size as the ground truth
      map: 200 x 200 pixels) 
      ```
      class Databucket():
         def __init__(self):
            self.images = csv_img_list  
            self.xpos = df["X_Position"].values
            self.ypos = df["Y_Position"].values
            self.yaw = df["Yaw"].values
            self.count = 0 # This will be a running index
            self.worldmap = np.zeros((200, 200, 3)).astype(np.float)
            self.ground_truth = ground_truth_3d
        
      ```
      
      Instantiate a Databucket()
      ```
      data = Databucket()
      ```
      
      Apply perspective transform & mask
      ```
      warped, mask = perspect_transform(grid_img, source, destination)
      ```
      
      Apply color threshold to identify navigable terrain/obstacles
      ```
      threshed = color_thresh(warped)
      obs_map = np.absolute(np.float32(threshed) - 1) * mask
      ```
      
      Convert thresholded & obstacles image pixel values to rover-centric coords
      ```
      xpix, ypix = rover_coords(threshed)
      obsxpix, obsypix = rover_coords(obs_map)
      ```
      
      world_size is instantiated from worldmap
      ```
      world_size  = data.worldmap.shape[0]
      ```
      
      The destination box will be 2*dst_size on each side
      ```
      scale = 2*dst_size # 2*dst_size = 10
      ```
      
      Reading rover position and yaw angle from csv file
      ```
      xpos = data.xpos[data.count]
      ypos = data.ypos[data.count]
      yaw = data.yaw[data.count]
      ```
      
      Convert rover-centric pixel values to world coords
      ```
      x_world, y_world = pix_to_world(xpix, ypix, xpos, ypos,
                                       yaw, world_size, scale)
      obs_x_world, obs_y_world = pix_to_world(obsxpix, obsypix, xpos, ypos,
                                       yaw, world_size, scale)
      ```
      
      Update worldmap (to be displayed on right side of screen)
      ```
      data.worldmap[y_world, x_world, 2] = 255
      data.worldmap[obs_y_world, obs_x_world, 0] = 255
      nav_pix = data.worldmap[:,:,2] > 0
      data.worldmap[nav_pix, 0] = 0
      ```
      
      See if we can fins some rocks
      ```
      rock_map = find_rocks(warped, levels=(110,110,50))
      if rock_map.any():
         rock_x, rock_y = rover_coord(rock_map)
      ```
      
      Make a mosaic image
      
      Create a blank image
      ```
      output_image = np.zeros((img.shape[0] + data.worldmap.shape[0], img.shape[1]*2, 3))
      ```
      Put the original image in the upper left hand corner
      ```
      output_image[0:img.shape[0], 0:img.shape[1]] = img
      ```
      Add the warped image in the upper right hand corner
      ```
      output_image[0:img.shape[0], img.shape[1]:] = warped
      ```
      Overlay worldmap with ground truth map
      ```
      map_add = cv2.addWeighted(data.worldmap, 1, data.ground_truth, 0.5, 0)
      ```
      Flip map overlay so y-axis points upward and add to output_image 
      ```
      output_image[img.shape[0]:, 0:data.worldmap.shape[1]] = np.flipud(map_add)
      ```
      
      Then putting some text over the image
      ```
      cv2.putText(output_image,"Populate this image with your analyses to make a video!", (20, 20), 
                  cv2.FONT_HERSHEY_COMPLEX, 0.4, (255, 255, 255), 1)
      if data.count < len(data.images) - 1:
         data.count += 1 # Keep track of the index in the Databucket()
      ```
      
      Finaly ```return output_image```


Autonomous Navigation and Mapping 

   1. ```perception_step()``` and ```decision_step()``` explanation and why these functions were modified as they were.
   
      In the ```perception_step()```, we apply the same ```process_image()``` functions in succession and update the 
      Rover state accordingly where the camera images is coming to us in Rover.img
  
      The world_size is initialized from the Rover.worldmap
      ```
      world_size  = Rover.worldmap.shape[0]
      ```
   
      Update the Rover worldmap(to be displayed on right side of screen)
      ```
      Rover.worldmap[y_world, x_world, 2] += 10
      Rover.worldmap[obs_y_world, obs_x_world, 0] += 1
      ```
   
      Convert rover-centric pixel positions to polar coordinates
      ```
      dist, angles = to_polar_coords(xpix, ypix)
      ```
   
      Update the Rover.nav_angles
      ```
      Rover.nav_angles = angles
      ```
   
      See if we can find some rocks
      ```
      rock_map = find_rocks(warped, levels=(110,110,50))
      if rock_map.any():
          rock_x, rock_y = rover_coords(rock_map)
    
          rock_x_world, rock_y_world = pix_to_world(rock_x, rock_y, Rover.pos[0], Rover.pos[1],
                                                      Rover.yaw, world_size, scale)
                            
          rock_dist, rock_ang = to_polar_coords(rock_x, rock_y)
          rock_idx = np.argmin(rock_dist)
          rock_xcen = rock_x_world(rock_idx)
          rock_ycen = rock_y_world(rock_idx)
    
          Rover.worldmap[rock_ycen, rock_xcen, 1] = 255
          Rover.vision_image[:, :, 1] = rock_map * 255
      else:
          Rover.vision_image[:, :, 1] = 0
      ```
   
      Finally ```return Rover```
   
      Regarding the ```decision_step()```, having the Rover.nav_angles not None then we can go through the decision tree
      for determining throttle, brake and steer commands based on the output of the perception_step() function  
   
      If in a state where want to pickup a rock send pickup command
      ```
      if Rover.near_sample and Rover.vel == 0 and not Rover.picking_up:
           Rover.send_pickup = True
      ```
   
   2. Autonomousmode navigation results explanation and how we might improve them.
     
      After code modification, we can see the rover driving and the worldmap populating the colors while the rover moves 
      so we can see the vision image showing the navigable terrain and the obstacles and the worldmap getting updates at every
      frame. 
     
      So this basic and simple AI is doing a nice job with a 70% of fidelity but sometimes the rover become confused if 
      is stucked behind a rock or run in circles so we need to access the manual mode and take over to force the rover 
      to find a new direction.
     
      We can improve the result by managing better the threshold colors to increase the navigation fidelity so we can trace a
      better path history and prevent the rover from returning to the place so we can scan the hole wolrdmap and 
      find all the rocks quickly.
