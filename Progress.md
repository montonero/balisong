##March 13, 2015
* Binvox loader works
* Carving the voxels to exclude voxels that are completely included to save memory consumption.

##March 14, 2015
* Converted implementation from morton sorted structure (searched using binary search) to Octrees.
* Tracing is now done in threads, 1 thread per core delivers the maximum performance

##March 17,2015
* Light calculation with normals now work

##March 18, 2015
* Change the xyz orientation, pattern to unreal, blender, by using Z as up (previously Y as up)	

##March 24, 2015
* Derivation of normals from voxel structure itself made progress, initial calculation. Looks ugly
* Made the generated normals look smooth already. Tried 1st pass and 2nd pass smoothing.

##March 30, 2015
* Implemented finding the neighbor voxel using towards center of the model.
	* This can be improved by using the local geometric center of the parent models

##April 3, 2015
* Updated code to remove rust deprecated modules such as old_io

##April 4, 2015
* Renamed octree to voxtree since the number of children is not limited to 8 anymore, 64 children is optimal option for optimizing memory usage.
* Added blending of colors and light intensity.
* Tried recalculating of normals only when the point is hit in the computation of rays

##April 6, 2015
* Eliminated error calculation in normals which resulted in Normal(0,0,0) by substituting it with normals dervied from the center of the object

##April 8, 2015
* Reduced memory size from 6.4 GB(fully solid) to 5.3 GB(fully solid, ommited booleans) to 900 MB(carved out) to 713 MB(carved out normals only) by carving out completely occluded voxels. Using empty voxtree saves up to 1.1GB of unrequired contents
	* Fully solid (6.4 GB) - 26,637,838 solid voxels, occluded included  (2 % of the maximum space)
	* Carved out (720 MB)  - 2,102,146 surface voxels (8% of the total solid) (0.20 % of the maximum space)
	
	
	*KTM - 10,275,626 solid voxels, carved 1,681,220 voxels  - leafnodes: 3,362,440, it is 2 * carved. Need to eliminate putting a children voxel on the leaf nodes
* Created a 2 separate implementation for setting the path and setting the leaf

##April 14, 2015
*Made non-recursive functions in voxtree, this is for easily creating a version for OpenCL.

##April 22,2015
*Refactored Voxtree into Traits, with 2 concrete classes Voxtree and Voxbit
*Voxbit is just a path structure bitsets which constains only bitset and the childrent bitsets, this is a hope for minimizing memory usage
*Reduced 6.4 GB to 5.3 GB by using Voxbit

##May 6, 2015
*Able to move out scale independent of the ray and model LOD's, this will be the key to dynamic LOD adjustment	

##May 7, 2015
*Now able to render a smoother lower leve LOD but seems the continuity of the voxel is a bit off. Maybe needs to put back morton encoding.
	