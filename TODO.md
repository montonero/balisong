## Mar 10, 2015

	* Let the binvox loader works
	* Create a new voxel format (.svo)
		* 1 contains the structure, the other the material information
	* Sparse Octree optimizer
		* If the 8 children of the voxel contains the same material (Color, normal, specularity, refraction). Store this information to the parent. Recursive apply the same logic, bottom-up.
	
### Memory Optimization
	* Data can be optimized by separating the material structure from the Octree.
	* Octree only provides the relative location of voxels
	* Normals will be on separate file, will be stored linearly
	* Material can just be a lookup, stored with respect to octree structure

### File streaming
	* Create an array of lookup table, all the possible material used.
	* For each LOD, make a grid, which stores the material indexes, arranged in morton encode and compressed using RLE (run-length-encoding).
	* Normals are stored in this arrangement as well
	* Objects can have same material but of different colors (i.e stained glass, erroded concrete, painted plates) or any discolored materials due to environmental factors. Technically, the discoloration are just different atoms attached to the bigger material, but are thier mass is negligible to be considered as another set of material
	

## March 12, 2015

	* Try to trace each pixel using threads		
	
## March 16, 2015
	 
	 * Add normals, injecting luminance from light source
	 * Make an efficient conversion algorithmn for determining the next voxel to evaluate rather than relying on location::from_xyz(lod, x,y,z) which performs a lot of calculations
	 * Implement a one world octree, which all object that are to be viewed will be on the octree. The world octree contains the camera location. Based on the camera location objects at minimum required LOD, will be loaded to this octree. The closer it is to the camera, the more details will be fetched. The loaded octree will be merged to the one world octree. There will be merging implementation of the octree.

	 * Merge point, Scale, rotation, translation, transformation lattice
	 
## March 17, 2015
	
	* Implement a non-recursive function for get_tree in octree module, this will be translated to opencl version to allow real time rendering
	* Implement an own ply to voxel + normal + color converter
	* Implement a function to generate a normal of a voxel based on the neighbors
		* Get all the neighbors for this voxel, then take 3 at a time, generate normals then get the average of the normals
		* Neighbors of 0 is 1,2,3,4,5,6,7 at the same parent.
		* while the children (4,5,6,7) of the neighbor parent can also be a neighbor of this 0, but not (0,1,2,3) 
	
	
## March 19, 2015
	* location module needs to have "common_parent" function which tells what is the location of the common parent of two locations, This can be done by comparing at which part of the location array it starts to differ the values. Then we can then ignore the parent location path in the array, and do the calculations at the local level 
	
## March 23, 2015
    * Create normals based on voxel formation
        * Each voxel has 6 face neighbors, 12 edge neighbors and 8 vertex neighbors, at total of 26 neighbors. Calculated as (3^3-1)
        * Getting the neighboring voxel at 0,0,0

            ================
             6 face neighbors          
            ================
             0  0  1
             0  1  0
             1  0  0
             0  0 -1
             0 -1  0
            -1  0  0
              
             
            ================
             8 vertex neighbors         
            ================        	
            -1 -1 -1 
            -1 -1  1
            -1  1 -1
            -1  1  1
             1 -1 -1
             1 -1  1
             1  1 -1	
             1  1  1 
             
             ================
             12 edge neighbors       
             ================   
             
             0  1  1
             1  0  1
             1  1  0
             
             0 -1 -1
            -1  0 -1
            -1 -1  0


            -1  1  0
             0 -1  1
             0  1 -1

             1  0  1
             1 -1  0
             1  0 -1
         
##March 24, 2015
	* Calculation of normals on a different approach
		* Calculate the normals based on empty sides of the voxel data
		* If there is no occluded neighbor, we can use the empty voxel away from  the center as the point of reference
		* For all empty voxels (may exclude those which are close to non-empty) get the vector to this empty voxels, then get the average. It will then be use to approximate the normal
		* Do an averaging of neighboring voxel to smoothen the normal distribution
		* The holes is caused mainly of occluded points that is somehow hit on the ray traced, not sure whether a neighbor algorithm octree bug or raytracing bug
		

##March 25, 2015
	* Fix camera and lightning orientation, right now it doesn't feel right.
	* Use quarternion for camera
	
##March 29, 2015
	* Make the decision of which occluded/empty voxel is the best candidate. Best Occuded voxel candidate is along with the center of mass of the object. Best empty voxel candidate is away from the object center of mass.
		* The vectors are along when their dot product is close to 1.
		* The vectors are away when their dot product is close to -1.
		
##April 2, 2015
	* Implement the octree to use 64bit bitset value. That is 1 voxel will be subdivided into 64 sub voxels
	
##April 3, 2015 
	* Calculate only the normal of the point when it is hit by the ray.
	* Calculate only the points when it is hit by the ray (equation based rendering)
	* Traversal of voxtree to calculate the points only that are part in the node/geometry
	* Research on arranging the bitsets in an array to optimize memory consumption
	
##April 5, 2015
    * Fast counting of bitsets
    * http://stackoverflow.com/questions/109023/how-to-count-the-number-of-set-bits-in-a-32-bit-integer	
    
##April 6, 2015
	* Make an implementation to convert obj to voxel format
		* Use https://github.com/PistonDevelopers/wavefront_obj
		* and http://fileadmin.cs.lth.se/cs/Personal/Tomas_Akenine-Moller/code/tribox3.txt
	* This is necessary to extract material(specular maps), color, even normals

	
##April 8, 2015
	*Rewrite the loops to use iterator rather than accessing via index (http://doc.rust-lang.org/book/iterators.html).
		* Accessing via index has extra bounds check thus a performance penalty which unlike iterator bounds check is not needed.

##April 10, 2015
	* There are around 4000 delta length ray marched until an object is hit
	* Reduce the number of delta point computation by making a low LOD traversal when no object is found. LOD is increased when a point might have hit.
	* Alogrithm: start from highest LOD, if no hit, increment direction length++, if hit LOD++ until no hit or hit && LOD == required LOD 
	
	fn hit(ray, LOD, node){
		let current_lod = 1;
		let length = 0;
		let photon = ray.at_length(length);
		loop {
			let location = location::from_xyz(current_lod, photon.x, photon.y, photon.z);
			let hit = node.is_location_occupied(&location);
			if hit{
				current_lod+=1;
				if current_lod == required_lod{
					break;
					//return location;
				}
			}
			else{
				//location.remove(last);
				length+=1;//a length should be moving from 1 voxel to next
				if length > max_distance{
					break;
					//return None;
				}
			}
		}
	}
	
	
	* Decreasing the LOD is just simple as ommiting the last element of the location.
	* Then determine the next element of the location with a direction 

##April 11,2015
	* Stucked with optimizing the LOD traversal
	* Another attempt:
		* make a ray box intersection to fast remove rays that has no hope of hitting the box
		* for all the other rays that may hit the box, do an adaptive LOD search
			* Starting at a point of increment with 1 voxel distance at higher LOD. 	 

##May 5, 2015
	* Progresssive ray marching, radius of intersection is shortened when there is an intersection, advances to next length when none.		