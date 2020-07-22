## Project: 3D Motion Planning
![Quad Image](./misc/enroute.png)

---


# Required Steps for a Passing Submission:
1. Load the 2.5D map in the colliders.csv file describing the environment.
2. Discretize the environment into a grid or graph representation.
3. Define the start and goal locations.
4. Perform a search using A* or other search algorithm.
5. Use a collinearity test or ray tracing method (like Bresenham) to remove unnecessary waypoints.
6. Return waypoints in local ECEF coordinates (format for `self.all_waypoints` is [N, E, altitude, heading], where the drone’s start location corresponds to [0, 0, 0, 0].
7. Write it up.
8. Congratulations!  Your Done!

## [Rubric](https://review.udacity.com/#!/rubrics/1534/view) Points
### Here I have considered the rubric points individually and described how I addressed each point in my implementation.  

---
### Writeup / README
Below I describe how I addressed each rubric point and where in my code each point is handled.

### Explain the Starter Code

#### 1. Explain the functionality of what's provided in `motion_planning.py` and `planning_utils.py`
These scripts contain a basic planning implementation that includes A* algorithm.
In plan_path function, obstacle map data is loaded and grid+offsets are created inside create_grid function by providing map, target altitude and safety margin.
Start and goal positions are planned as arbitrary positions on grid. Further, A* algorithm is called to plan the path from start to goal position.
A* works with heuristic (eucledean distance) for path planning. A* takes start position, checks for every possible valid action and required cost.
It creates every action as a node and associated cost to reach, chooses the actions which is taking towards goal while avoiding obstacle.  
Once it reached to goal as current node, it traces back the exact path from start position and gives back that path, later these nodes in path converted to waypoints.
...And here is image from Udacity!
![Top Down View](./misc/high_up.png)

### Implementing Your Path Planning Algorithm

#### 1. Set your global home position
I have used np.loadtxt with datatype as S15 (considers it as string), skiprow as 0 and maxrow as 1. So that it reads only 1 row from that file.
Further, to extract longitude and latitude , I have broke the string array using split function and converted the string to float.

Here is code for that:

    data = np.loadtxt('colliders.csv',delimiter= ',',dtype='|S15',skiprows=0,max_rows=1)
    self.set_home_position(float(data[1].split()[1]),float(data[0].split()[1]),0)
#### 2. Set your current local position
Here I have used global_to_local and stored the values in NED format.

    global_current_position = [self._longitude,self._latitude, self._altitude]

    local_position = global_to_local(global_current_position,self.global_home)

#### 3. Set grid start position from local position
grid start is updated with current local position with respect to offset.

    grid_start = (int(local_position[0]) -north_offset, int(local_position[1]) -east_offset)

#### 4. Set grid goal position from geodetic coords
Grid goal position is modified to accept longitude, latitude , then it converts into local and adds to offsets to find local goal position.

    grid_goal_global = [-122.397745, 37.793837, self._altitude]
    grid_goal_local = global_to_local(grid_goal_global,self.global_home)
    print(grid_goal_local)
    grid_goal = (int(-north_offset + grid_goal_local[0]), int(-east_offset + grid_goal_local[1]))

#### 5. Modify A* to include diagonal motion (or replace A* altogether)
Actions and valid action functions are updated to do the diagonal movements. diagonal movements will be having both values (-1 or 1, not 0).
They are added with cost of sqrt(2) as shown below.

    NORTH_WEST = (-1,-1,np.sqrt(2))
    WEST_SOUTH = (1,-1,np.sqrt(2))
    SOUTH_EAST = (1,1,np.sqrt(2))
    EAST_NORTH = (-1,1,np.sqrt(2))
    
And to check validity of these actions, both side conditions are applied to them.

    if x - 1 < 0 or y - 1 < 0 or grid[x - 1, y - 1] == 1:
        valid_actions.remove(Action.NORTH_WEST)
    if x + 1 > n or y - 1 < 0 or grid[x + 1, y - 1] == 1:
        valid_actions.remove(Action.WEST_SOUTH)
    if x + 1 > n or y + 1 > m or grid[x + 1, y + 1] == 1:
        valid_actions.remove(Action.SOUTH_EAST)
    if x - 1 < 0 or y + 1 > m or grid[x - 1, y + 1] == 1:
        valid_actions.remove(Action.EAST_NORTH)

#### 6. Cull waypoints 
For this I used collinearity test method. The idea was simply to prune the path of unnecessary waypoints. Below is the code:

def prune_path(path):
    pruned_path = [p for p in path]

    i = 0
    while i < len(pruned_path) - 2:
        p1 = point(pruned_path[i])
        p2 = point(pruned_path[i+1])
        p3 = point(pruned_path[i+2])

        if collinearity_check(p1, p2, p3):
            pruned_path.remove(pruned_path[i+1])
        else:
            i += 1

    return pruned_path

take found path from A* algorithm and take 3 points at a time, then run collinarity check on those 3 points , if points are collinear then remove the middle point.
Run the collinarity check on every points to remove unnecessary waypoints.

### Execute the flight
#### 1. Does it work?
It works!

### Double check that you've met specifications for each of the [rubric](https://review.udacity.com/#!/rubrics/1534/view) points.
  
# Extra Challenges: Real World Planning

For an extra challenge, consider implementing some of the techniques described in the "Real World Planning" lesson. You could try implementing a vehicle model to take dynamic constraints into account, or implement a replanning method to invoke if you get off course or encounter unexpected obstacles.


