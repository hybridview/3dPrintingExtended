# Bed Mesh in Klipper  


## TRANSLATED FROM: Vyper_Extended


## <u>Documentation</u> 
  
- https://www.klipper3d.org/Bed_Mesh.html  
  
---  
  
  
#############################################################################  
## Load saved mesh									     
#############################################################################
  
### <u>Include in start code</u>       
  
 `BED_MESH_PROFILE LOAD="Name"`    
  
*Example: BED_MESH_PROFILE LOAD="small" *  
  
---  
#############################################################################  
## Full mesh before printing								     
#############################################################################
  
### <u>Include in start code</u>
 
 `BED_MESH_CLEAR`
 `BED_MESH_CALIBRATE`  
  
*Example: BED_MESH_PROFILE LOAD="small" *

    
--- 
#############################################################################  
## Bed Mesh im Druckbereich "Area Bed Mesh"									     
############################################################################# 
   
Here is the documentation of the area mesh
https://github.com/cryd-s/Vyper_extended/tree/main/GCODES/printarea_mesh
