# Realistic N-Body orbital Mechanics in Unity.

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/174beaf3-5455-43b3-a879-f221fb0a1de4)

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/e26cf605-19a7-4018-b004-d6659c37ef93)

# The Planetarium Module

The planetarium module contains all orbital mechanics as well as all celestial bodies. Bodies can follow N-body simulation like spacecrafts while celestial bodies with relatively stable and defined orbits can follow classical 2 body orbits.

# Keplerian Component
	
●	Reference frames

●	Converts a set of keplerian elements to state vectors to initialize positions of objects

●	Converts a set of state vectors back to keplerian parameters

●	Keep track of time.

●	Floating origin

Component Source Location

●	Planetarium.cs

●	PlanetariumSingleton.cs

●	DrawOrbitLines.cs

# Orbit Component

● N-body Orbital Integrator	using runge kutta RK4 and RK8 


●	Classical 2 body orbit solver

Component Source Location

●	Orbit.cs

# Keplerian Component - Reference Frame

Reference frame is important for all keplerian orbits. 

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/602f9fdb-90b9-42aa-b5a6-962243bcebc9)

Given the example above heliocentric inertial frame , let's consider the blue giant orbiting the star, notice that the white gridlines are parallel to the horizontal axis of gamespace, this shall represent the parent body’s ecliptic plane, this is the plane at which the orbit of the blue giant resides. Other planes can be defined, the vernal and equinox is equivalent to the red arrow , this is also where the blue giant’s equatorial plane (same as its rings) intersects the ecliptic plane of its parent. The blue arrow represents the vector that points to the parent or solstices. Definitions of the ecliptic planes are located in Orbit.cs and are constantly updated based on the local forward transform of the parent body or equatorial plane.

# State vector reference frames

For each celestial object or satellite object, there are 2 main reference frames, the parent reference frame, and heliocentric reference frame. Parent reference frame stores position and velocity relative to the object's orbiting body. E.g the moon’s parent is earth. Heliocentric reference frame stores position and velocity relative to the geometric center of the sun or heliocentric body (this can also be the blue giant). In Orbit.cs, position and velocity refers to the “parent centric“ frame and helio represents the heliocentric frame.

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/04f437fc-12f3-4884-9692-66048c76a908)

# Keplerian Component - Orbit Elements to state vector (Position and velocity)

Orbit.cs defines all keplerian elements as shown on the left. These parameters are calculated in real time. 

●	Eccentricity - the shape of the orbit ellipse, an eccentricity of 0 represents a perfectly circular orbit while an eccentricity of 1 or more represents a hyperbolic orbit or escape orbit. For reference our moon’s orbit is quite circular around earth and is around e = 0.055

● Semi Major Axis - The max “height” of the orbiting body relative to the geometric center of the parent body. (In reality the parent body wobbles so it's more of the “Averaged center”)

●	Semi Minor Axis - The smallest “Height” of the orbiting body relative to the geometric center of the parent body

●	Apoapsis - semi major axis minus planetary radius. Often used to measure altitude adobe ground level (AGL)

●	Periapsis - semi minor axis minus planetary radius , or closest distance an object will be relative to the parent’s AGL

●	Semi Major Axis Vector - semi major axis in vector form.

●	Semi minor Axis Vector - semi minor axis in vector form.

●	Inclination - the angle of the orbiting body’s orbital plane relative to the parent’s equatorial plane. Where 0 means the object orbits perfectly around the parent’s equator, 90 where the orbit passes perfectly around the parent’s poles.

●	Longitude of ascending Node (LAN) - In reality a place can be tilted as described by inclination but it can also yaw around the polar axis of the parent, this yaw angle is the LAN. The reference direction is represented by the parent’s ecliptic right (red arrow).  

●	Ascending Node - vector of each orbit pointing in the direction where it intersects with the parent’s equatorial plane. 

●	Argument of Periapsis (AOP) - Suppose the eccentricity is ~0.2, which means an egg shape like orbit. The highest and lowest point of the orbit can only be in one direction, or angle. That direction at which the periapsis is located relative to the ascending node is AOP. 

●	True Anomaly - The angle E of the elliptical orbit. X being the reference direction as mentioned above.

●	Mean Anomaly - Similar to true anomaly but the elliptical orbit is “squished” into a circular orbit.

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/aec7c870-b636-4bf1-81d3-57bd0ac8f739)

# Keplerian Component - State vector from Orbit Parameters

For each celestial object as well as satellite objects, we need to define an initial starting condition for it. While we can just put them in the scene, by defining the orbit parameters, we can be confident that the initialized state vectors will be accurate. These inputs are shown on the left for each object in Orbit.cs.

Below in sequence is the mathematical determination of state vectors from the above input. 

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/51a870ac-2b90-4e8b-92d3-3e604a281665)

Step 1 : Determine the standard gravitational parameter (μ) of the parent body.

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/a7fca85d-5b68-4ff9-b619-201aaadbaae4)

Where:
G is the gravitational constant (.0667430e-11)
M is the mass in KG of the parent body.

Step 2 : Calculate Semi Minor axis from Semi Major axis

If e < 1

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/b5c91b09-aaad-4df2-ad6e-0cbdad5c5b32)

Where:
a is the semi major axis input.
e is the eccentricity input.

If e > 1

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/accf02a9-6b76-4aa4-8707-ea6296b5234d)

Where:
a is the semi major axis input.
e is the eccentricity input.Where:

Step 3 : Get ascending node and orbit normal directions

This is done by calculating the vector of the parent's local red arrow or the ecliptic right. Given the longitude of the ascending node input and the normal vector of the parent, we rotate the red arrow by angle (LAN) around the normal vector to obtain an ascending node. 

This is done by a custom modified function that applies a rotation angle , v is the vector to rotate e.g ecliptic right, and rotate around is vector n or the normal vector.

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/ee9d583b-bbc3-41ab-973d-e67ebfb9d7c5)

In the rotation matrix below, vector R is the result rotation matrix, u is the axis, and theta is the angle. 

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/fb16f696-b397-4e64-a13f-47dd82058edb)

To obtain the result vector, we take the rotation R DOT product by the input vector. The same concept applied to inclination to obtain orbit normal using inclination input.

Note : Orbit normal dot ecliptic normal used to determine the direction of orbit. 

Step 4 : Determine orbit period 

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/7015c79c-b4d8-4687-9c5a-148751d789bf)

Where:
a is the semi minor axis as determined before.
T orbit period in seconds.

Step 5 : Determine Semi major / minor basis vectors.

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/b5021f8f-e114-4a7b-a867-6c4139766fa4)

Using the above convention, we can determine both vectors by rotating the ascending node vector by the argument of periapsis angle AOP input angle into theta 2 , then semi major axis basis is simply the opposite direction (Or cross product of periapsis and  parent normal axis)

Note that the semi major basis and minor basis is a direction vector only . 

We can obtain vector form of semi major and minor axis by:

Semi major Axis vector  = Semi major Axis Basis (direction)  * semi major axis (scalar km)
Semi minor Axis vector  = Semi minor Axis Basis (direction)  * semi minor axis (scalar km)

Apoapsis  (Scalar km) = Semi major Axis vector.magnitude -   parent body radius; 
periapsis  (Scalar km) = Semi minor Axis vector.magnitude -   parent body radius; 

* where Conditions based on eccentricity 
●	If the eccentricity is < 1, then the semi major axis axis and apoapsis.
●	If the eccentricity is => 1, then the semi major axis axis and apoapsis do not exist.

Step 6 : Calculate eccentric anomalies from mean anomaly input.

*If the eccentricity is < 1, a elliptical orbit, we use mean anomaly

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/d8808aa4-f6ed-494b-9616-5756235c0789)

Where:
V is the input true anomaly (radians)
e is eccentric.
E is an eccentric anomaly.

Since the user inputs a true anomaly, we have to convert it to an eccentric anomaly. However looking at the formula we can see that there’s no straightforward way to solve this since there are 2 values of E.

In order to solve this we have to use Newton-Raphson iterative method. Given a Epsilon or uncertainty, we can iterate on the formula below to get results that are within the epsilon to the actual value. 

In practice we can calculate n or iterations based on eccentricity. Based on some sources, a reasonable iteration n would be 2 for orbits that are circular and 6 for orbits that are close to hyperbolic.

ref: https://scicomp.stackexchange.com/questions/40700/solving-keplers-equation-with-newton-raphson-method

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/bff3cdb1-ff78-4606-9c05-1c06711e6995)

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/7319e4ec-c6db-40fe-a480-dcb66b455ee2)

Where:
En is the eccentric anatomy for each iteration
M is the mean anomaly.

*If the eccentricity is < 1, a hyperbolic orbit, we use hyperbolic anomaly, where the hyperbolic anomaly is defined as area F as shown below : 

For hyperbolic case, 

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/e2862f29-58dc-4eb4-9a05-1a21faf73f85)

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/4edcb003-7c00-4ec8-b186-1700a2a2d238)

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/a19b5e20-bf41-49e3-a9dc-002d8ed9a8d6)

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/f36b3079-9a2c-480e-90af-f6abd4eaa79f)

Where:
F        is the hyperbolic anomaly area.
Mh     is the mean anatomy at iteration h
Delta  is the epsilon value. Or accuracy. Typically epsilon 1e-8.

Step 7 : Obtain position from eccentric anomaly.

By applying pythagoras theorem, we can obtain position using eccentric anomaly and semi major axis. 

We can calculate the length of ea since we know that the distance from F to A is simply the semi minor axis and the distance from O to A is simply half of the sum of the semi major axis and semi minor axis. 

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/b7ea4b1e-780c-4eb3-b9b4-a4993a03b9c7)

Where:
O is the center focus.
F is the origin of the parent body.
a is semi major axis
|OA| and |ea| are the lengths
ℓ is eccentricity
E is eccentric vector
P’ is the eccentric position
P is the position
R is the radius vector
To obtain the position relative to parent, imagine where F = 0, then the vector position P’ will simply be the semi major axis basis (vector pointing from F to P) multiplied by scalar r.

Step 8 : Obtain velocity from true anomaly.

Velocity calculation uses true anomaly. We need to Convert eccentric anomaly to true. This can be done using the formula:

If the orbit is ellipse:

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/644a9984-3361-4b8a-8adb-0f743566be5e)

Where : 
E is the eccentric anomaly.
e is the eccentricity

*If the orbit is hyperbolic simply use hyperbolic trig functions.

Velocity can be obtained easily from the following formula below (Code is outdated not shown):

Perifocal reference frame (PQW):

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/4cd40776-ef14-44d4-947b-1c46049f2c07)


Where:
V is the velocity vector
f is the true anomaly
p is the semi-latus rectum
Μ is standard gravitational parameter of parent (GM)
e is the eccentricity

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/3997a7c9-ba6a-4a7c-9922-158dfc358c16)

# State vector to Orbital Elements

The same operation has to be performed in reverse where an object's state vector, its position and velocity are converted to orbital parameters. This is primarily used by N-body simulations, whereupon each integration step, the resultant position and velocity converted back to orbital parameters for display or trajectory planning. The steps used to obtain the keplerian parameters are displayed below. For now we only investigate the 

Step 1: Obtaining Eccentric Vector

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/05389577-38ae-4e56-a1d7-12e773a9d54b)

The eccentric vector as shown below is a vector that points from the parent body’s center to the periapsis of the orbit. This vector would allow us to determine various orbital parameters like the argument of periapsis angle as well as direction to apoapsis if the direction is inverted. The reason to find an eccentric vector first as this vector is a function of the position vector and velocity vector, by finding the angular momentum vector. 

Consider the equation below for eccentric vector e:
*Note that position and velocity are relative to the parent orbiting body only.

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/4d3b22b9-f19b-44fa-bf67-c206ea820413)

Where:
e is the eccentric vector pointing to periapsis
|e| the magnitude of e is eccentricity
R is the position vector
V is the velocity vector
h is the specific angular momentum vector (where h = the cross product of position and velocity)
Mu is the equal to GM of parent (standard gravitational parameter)

Step 2: Obtaining Semi major and minor
*Focal parameters are no longer needed in this section.

We need to firstly obtain the focus position of the orbit as shown below. Determination of the focal Parameter (distance from the directrix to the focus point). 

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/a28b105b-a949-4e6c-a0d2-a2e1cc794eb0)

*Focal parameters are no longer needed in this section.

The semi-minor axis basis or the direction to vector b can be determined by taking the cross product of the eccentric vector (same direction from c to f1, going left) and the angular momentum vector (which points downwards in the picture on the right). Since the angular momentum vector points down given the orbit is clockwise, the resultant b will be pointing downwards. To get the upwards b vector, we have to inverse either angular momentum vector or eccentric eccentric vector.  The semi major basis or the direction of vector a can be determined by rotating vector b by 90 degrees with the axis pointing upwards, this axis would be the orbit normal.

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/f37d6fea-c5fd-4742-8c08-73a4ca802504)

The semi major and minor vector can be hence determined by taking the semi major and minor basis multiplied by the scalar distance. To get the semi minor axis, we need to find the size of the ellipse, or distance from apoapsis and periapsis divided by half, and multiplied by the direction vector semi major basis. To find b, we need to know the shape of the ellipse, the compression ratio, where a compression ratio of 1 means a perfectly circular orbit, the more elliptical the orbit the lower the compression ratio. (semi minor distance / semi major distance * 2). 
Derivation of distance for a and b can be done below:

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/b843c322-0d10-4aa5-bed6-28d45943d3b1)

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/3dece488-a8e9-4d4e-b528-0215bda89933)

Where:
h is the angular momentum vector
a is the semi major axis distance
b is the semi minor axis distance
e is the eccentricity (eccentric vector magnitude)
Mu is the gravitational parameter GM of the parent.

*Section to be updated in code.

To determine the apogee, we can use the semi major axis where the distance between apogee and perigee is 2 times of the semi major axis. 

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/4dc6a24d-7ff7-4fbe-8789-a6020b3bddce)

Where :
Center Point position vector (Relative to parent origin) =  semi major axis basis * semi major axis * eccentricity

Apogee vector = centerpoint vector - (semi major basis direction vector * a)
Perigee vector = centerpoint vector + (semi major basis direction vector * a)

Apogee and perigee distance can then be calculated from the magnitude of apogee vector or perigee vector minus the planetary parent radius.

Step 3: True anomaly 
True anomaly can be calculated easily by taking the angle between the current position vector from the focal point or (0,0,0) in the inertial frame and the semi-major axis basis direction vector.

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/dc0e25cf-ab89-435b-b971-39c7a520b6f3)

Inclination (Theta) = Acos(h dot h_z)

Where:
h is the normal of the orbital plane
H_z is the up axis of the parent’s equatorial plane 
Theta is the inclination in radians

Step 5: Longitude of ascending node

First we have to obtain the ascending node vector. This vector is the intersection of the object's orbital plane and the parent's equatorial plane. The direction of the ascending node depends on the clockwise or counter clockwise rotation of the orbit (prograde or retrograde).

Then we have to obtain the vector n, pointing from the parent's origin. This can be calculated from the -y and x component of the specific angular momentum vector as shown below or the cross product between the orbital plane and parent’s equatorial plane plane. 

To determine the angle, since we have the ascending node direction vector, and the reference direction of the parent, basically the right transform of the parent body game object, we can determine the angle difference between them using the dot product as shown on the below : 

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/efc52d8f-42fd-4bcd-823b-117edc184883)

Recall that the dot product angle is always <180 degrees, for LANs of above 180 degrees we need to find out which side it is on. By taking the cross product of the ascending node and the reference direction, we have a 3rd vector that is perpendicular to those two vectors in the up if the ascending node direction is left, and vice versa.

Taking the dot product of the 3rd vector against the equatorial plane up direction, we get an angle positive or negative. If the ascending node is greater than 180, suppose it is 190 then the result angle will be 10 and the dot product will be positive hence choosing which if statement to run.

# N-Body integrator

In the orbit.cs , the implemented option allows the user to select a simple keplerian orbit or complex N-body integrator. Especially for the player ship, we would want an N-body integrator to be the orbit propagator of choice. In order to support multi timescales, like time warps while maintaining accuracy, we implemented Newton's equations along with a Runge Kutta 4th and 8th order scheme. 

The core operation is Newton's law of universal gravitation which states that the force acting on one object is proportional to the Mu or the parent’s gravitational constant and its distance squared (Left). The vector form of this formula can be written to calculate the force vector given positions of the parent object (Right). To account for all bodies, the vector form equation can be modified to account for forces of attraction as a result of N number of celestial bodies. Note that for M2, since spacecraft mass is insignificant compared to the planet, it is assumed to be zero.

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/91be140d-02fc-4bac-a4d6-f52a08116ebd)

A discrete time step is used to calculate the resultant acceleration from the force over that discrete time. However, like all euler methods, we have to use very small time steps to achieve an acceptable level of accuracy , <0.01 seconds. 

# Numerical Runge Kutta Methods

Numerical methods like Runge Kutta allow us to use much larger discrete time steps to perform the calculations as shown above. When we are using the Euler method , if the timestep becomes too large, there will be large uncertainties in each iteration, or error. This error will be integrated and over many timesteps, builds up. This results in orbits not behaving as expected, like magically gaining or losing energy. 

In the RK 4th order scheme as shown on the right, we are using 4 discrete substeps within the timesteps to calculate the instantaneous gradient of the function, then by using the scheme we can combine the result of k1 ,k2, k3, k4 into a combined result, which provides accuracy that approximates euler methods with very small discrete time steps.

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/11bbd5c0-da23-40a5-9beb-50b7d384d64e)

# RK 4 Scheme

For most low time warp options, a RK4 scheme should provide reasonable accuracy. Currently the planetarium module utilizes RK4 for all integration time steps, perhaps in the future this would be dynamic based on an energy conservation epsilon like 1e-8. Nevertheless the first part of the scheme is to find the K coefficients, k1, k2, k3, k4. These are expressed below:

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/c8d4b899-6329-4ce3-a664-0b9f8b858207)

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/60b59538-1b3a-4b78-9bc9-69258d71ae33)

Where:
Tn -> not used
h is the timestep or 
delta time.


Some modifications are required to make this scheme work in order to integrate both position and velocity, for each k coefficient, there is the kn_v component and the kn_x component, where v is the velocity, x is the position. *Replaced r with x

For Velocity : 

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/7ac87d9f-84ef-43ca-a457-35a6ea48ad99)

For Positional :

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/b7af0eb6-68a2-4ac0-b917-5da31f0bed9c)

The left equation sets are the velocity component, or v in the code, the right side equation sets are the position component. The result of both v and x can be calculated using the equation sets 
below:

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/3a73c0d2-87f1-49f3-8f6b-ce139450c942)

# RK 8 Scheme

For higher timesteps, we need a scheme of higher order to maintain accuracy. This would be used for instance to draw orbit line predictions for interplanetary travel. The RK8 scheme in principle is similar to RK4 as described, however it consists of a set of 8 coefficients to obtain a result. RK8 scheme equation sets below showing the 8 K_n and its coefficients:

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/2e3254b2-71b1-4e79-ab9d-778c87dc4a8c)

# Heliocentric reference frame 

In order for the integrator to take into account all objects, we must perform all numerical integrations in a reference frame that contains all celestial bodies, this reference frame should be the heliocentric frame. During the initialization phase, the calculated position and velocity of objects are in a “parent centric” frame, that is the reference frame is the equatorial frame of the parent celestial object. In order to convert parent centric frame to a heliocentric frame, we have to iterate through the tree structure, starting from the current object, traversing up the tree, to its parent and its parent to its parent and integrate all parentric positions and velocity to obtain heliocentric reference frame. 

Below shows the implementation of a recursion to find parent orbits and iteration through each parent.

# RK4 Orbit prediction

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/cb03f68c-64cf-402a-966c-c683a29f12b7)

In order to predict the future orbit and thus drawing down orbit lines as shown above, we need to integrate the position of the object with respect to time separately to a defined future time, a future position and velocity. Furthermore we have to also integrate the position of all celestial bodies in the same future time. For celestial bodies, since it follows a classical Keplerian simple orbit, its position for any given future time can be calculated easily from the anomaly. 

Since we have already calculated the period from the previous segments, we can calculate the mean motion, that is what will be the angular displacement of the mean anomaly given a certain elapsed time if the mean anomaly rotates a complete circle for every period .


Once mean anomaly is determined, we can calculate the position of the celestial object at given elapsed time from start of the game by first converting the mean motion to a future mean anomaly, then convert mean anomaly to eccentric anomaly using the newton-raphson method as described in the orbital elements to state vector section. 

 
With the future eccentric anomaly is known, we can use that to obtain the future position, which would be used as input for the orbit prediction integrator.

# Planetarium Floating Objects 

In the planetarium which contains all orbital or celestial objects, there is one and only one object that is set as the floating origin. In all cases this is the player ship. The player ship anchor is a placeholder empty object that contains all properties of the playerships orbit or state vectors. The planetarium will calculate orbital mechanics in the heliocentric frame, then for displaying, all objects are scaled down and translated to the player’s frame. The player’s frame’s origin would always be (0,0,0) in the game editor. The scaling function makes sure that objects within the game scene are of a reasonable distance apart so as to not cause rendering artifacts. 

![image](https://github.com/Jacob19999/unity_n-body_runge-kutta/assets/26366586/0846d0b8-7600-4ae5-82f6-06e305a89f7b)

Notice in the editor screenshot above that the player ship is in the (0,0,0) position in the editor while the planet “Orbits” around the player. You can also see initial orbit parameter input in the inspector, this is where the player’s initial orbit is defined at the start of the game.

In the planetarium.cs a function runs at every update cycle for all tagged celestial or satellite objects, and the transformation vector calculated. The transformation can be applied for all objects .

