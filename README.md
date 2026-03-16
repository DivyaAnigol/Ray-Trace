# Ray-Trace
This work marks the transition of my ray-tracing study from ANSYS to MATLAB, using a fully custom heater-configuration model built from scratch. By recreating the geometry and control logic in MATLAB, the project now offers faster iteration, deeper customization, and a more flexible platform for analysing IR heater behaviour.

## Technical Content
**AIM:** To simulate IR radiation transport in an SLS chamber using geometric and Monte-Carlo ray tracing, capturing direct and reflected energy paths. The model converts ray-hit distributions into heat-flux and temperature fields to evaluate thermal uniformity and optimize heater performance.


**TOOL USED:** MATLAB


**CODE & LOGIC:**

**1. Material Properties (PA12 & Stainless Steel):** 
```
%% Material Properties
PA12 = struct('reflectivity', 0.05, 'emissivity', 0.46);
SS   = struct('reflectivity', 0.85, 'emissivity', 0.15);
```
**Physics relevance:**

Controls optical behavior upon ray interaction:
1. PA12 powder is mostly absorbing → converts radiation into heat
2. Stainless steel is highly reflective → drives multi-bounce behavior.
   
These parameters dictate how energy is redistributed in the chamber.


**2. Heater Geometry (Line/Cylinder Definition):**
```
heater_lines = [
    x1 y1 z1  x2 y2 z2;
    ...
];
```
**Physical Relevance:**

Represents IR heaters as finite-length line emitters, approximating real quartz tube IR lamps.

These heater geometries govern: emission origin, radiation direction, spatial heating uniformity.

Defines where and how radiation enters the system.


**3. Directional Emission Using Emission Cone:**
```
theta = deg2rad(emission_angle) * rand();
phi   = 2*pi*rand();
dir = [sin(theta)*cos(phi), sin(theta)*sin(phi), -cos(theta)];
```
**Physical Relevance:**

Models realistic infrared emission, where heaters radiate strongest perpendicular to their surface.

The emission cone captures: angular distribution of IR intensity, beam spreading, directional heating effects, 

This is crucial for accurate radiation mapping.


**4. Ray–Wall Reflection Physics:**
```
curr_dir = curr_dir - 2 * dot(curr_dir, wall.normal) * wall.normal;
```
**Physical Relevance:**

Implements specular reflection, the correct optical law for polished stainless steel.

Controls: secondary/tertiary radiation, indirect heating, spatial smoothing of energy

Reflections are a dominant factor in chamber thermal uniformity.


**5. Bed Intersection Calculation:**
```
t_bed = -curr_origin(3) / curr_dir(3);
hit_bed = curr_origin + t_bed * curr_dir;
```
**Physical Relevance:**

Determines where radiation actually lands on the powder bed surface.

This gives: energy deposition location, heat flux patterns, temperature rise predictions


**Main Logic for Ray Tracing**

**1. High-Resolution Ray Tracing**

**A. Very High Deterministic Ray Count**
```
n_rays = 10000;
```
**Physical relevance:**

Produces extremely dense ray sampling, creating a highly accurate irradiance map.

Improves resolution of: direct heater coverage, shadowing, reflection patterns.

**B. Ray Hit Counting on Powder Bed**
```
ray_hit_count(iy, ix) = ray_hit_count(iy, ix) + 1;
```
**Physical relevance:**

Counts every individual ray that reaches the bed. This transforms ray tracing from visualization → quantitative radiation mapping.

**C. Multiple High-Depth Reflections**
```
max_reflections = 5;
```
**Physical relevance:**

Captures multiple irradiance bounces, enabling realistic prediction of: indirect heating, wall-guided radiation, irradiance smoothing.

This is crucial in an SLS chamber where reflective walls dominate.

**D. Hit Map as Final Output**
```
surf(... ray_hit_count ...)
```
**Physical relevance:**

Outputs a ray intensity distribution, not temperature. This is a high-fidelity radiation-only model, perfect for optical validation.





+ <img width="465" height="465" alt="image" src="https://github.com/user-attachments/assets/df8e55bc-76ec-438b-8016-334ac0ff7b33" />
  
  **Fig.a. The plot visualizes high-density IR rays and multi-bounce reflections inside the SLS chamber, revealing the full spatial spread of direct and wall-reflected radiation onto the powder bed.**
  
  

+ <img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/ff2b263d-5035-4396-811d-1f016610ceb8" />

  **Fig.b. The plot shows the spatial distribution of IR ray impacts on the powder bed, revealing localized intensity peaks where direct and reflected radiation concentrates.**


  

 


**2. Ray Tracing by Monte Carlo Method**

**A. Monte-Carlo Emission**
```
theta = deg2rad(emission_angle) * rand();
phi = 2*pi*rand();
```
**Physical relevance:**

Introduces randomized angular emission, representing real-life IR heater photon distribution.

Enables: statistical sampling, physically correct probabilistic heat deposition.

**B. Energy Per Ray (Power Conservation)**
```
energy_per_ray = total_energy_J / total_rays;
```
**Physical relevance:**

Links actual heater power to each ray, ensuring energy is conserved. This converts ray hits → physically meaningful heat input.

**C. Heat Flux Computation (W/m²)**
```
heat_flux = ray_hits * energy_per_ray / area_per_cell;
```
**Physical relevance:**

Maps discrete ray arrivals into continuous heat flux, matching real thermal boundary conditions for SLS powder.

**D. Temperature Model**
```
temp_rise = (ray_hits * energy_per_ray) ./ (mass_per_cell * c);
```
**Physical relevance:**

Uses material properties to compute temperature rise from absorbed radiation. This connects optical behavior → thermal response.

**E. Conduction Smoothing**
```
T_next(i,j) = T_conduction(i,j) + r*(...)
```
**Physical relevance:**

Solves 2D heat diffusion, capturing in-plane powder conduction. Brings results closer to real preheating physics.



+ <img width="465" height="440" alt="image" src="https://github.com/user-attachments/assets/10a1a8e4-35fe-4f5e-91eb-27dc155eb1a9" />
  
  **Fig.a. The figure illustrates Monte-Carlo IR ray propagation from all heaters, showing direct and reflected energy paths that ultimately form the basis for computing heat flux and temperature rise on the powder bed.**
  

+ <img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/3b6a9ff2-f184-42a9-831d-10e529f2ef14" />

  **Fig.b. The heatmap captures the spatial density of IR ray arrivals on the powder bed, revealing the anisotropic energy distribution created by direct illumination and multi-bounce wall reflections.**
  

+ <img width="465" height="500" alt="image" src="https://github.com/user-attachments/assets/85f28b38-da42-4f0b-bb96-547f745ebcb8" />

  **Fig.c. This heat flux map translates ray-hit statistics into physical irradiance, revealing how combined direct and reflected IR energy distributes across the powder bed in W/m².**
  

+ <img width="500" height="400" alt="image" src="https://github.com/user-attachments/assets/a2753ba9-55ba-4e44-ab1d-d56e8cb031c8" />

  **Fig.d. This temperature map shows the early thermal response of the powder bed, where localized IR energy deposition converts into spatially varying temperature rise after one second.**
  






