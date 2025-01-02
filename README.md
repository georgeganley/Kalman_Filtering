## Acknowledgments

This project was completed as part of the AI for Robotics course at Georgia Tech. This repository does not include the code or solution to the project, but does illustrate the outcomes of a successful Kalman filtering implementation.

## Summary and Goal

### Scenario
While exploring the Milky Way in search of Earth, a spaceship’s engine malfunctions. The only remaining form of propulsion is a **jump** mechanism that lets you hop from one asteroid to another—like cosmic hopscotch. The spaceship's tasks include:

1. **Estimation:** Predict the positions of asteroids one timestep in the future.  
2. **Navigation:** Use those predictions to plan valid jumps from asteroid to asteroid until you reach homebase (the top of the field).

## Project Structure

1. **Asteroid Motion Model**

   Each asteroid follows a known motion model:
   
   $$x(t) = C_{pos_x} + C_{vel_x}\,t + \tfrac{1}{2} C_{acc_x}\,t^2,$$
   
   $$y(t) = C_{pos_y} + C_{vel_y}\,t + \tfrac{1}{2} C_{acc_y}\,t^2.$$
   - $(C_{pos})$ = position term $(x, y)$
   - $(C_{vel})$ = velocity term $(v_x, v_y)$
   - $(C_{acc})$ = acceleration term $(a_x, a_y)$
   - Each timestep **$\Delta t = 1$** second.

3. **World**
   - A rectangular field of variable width and height (e.g. 2x2, 4x2, etc.).
   - **(0, 0)** is the lower-left corner.
   - The **top boundary** of the field (the maximum \(y\)-value) is **homebase** (Earth).
   - Asteroids may enter and leave this field. Once an asteroid exits, it’s no longer tracked.

---

## Estimation

**Goal:**  
Predict each asteroid’s $(x, y)$ position one timestep into the future, given its current (possibly noisy) position. 

### Key Points
1. **Inputs & Outputs**  
   - Given a list of asteroid positions at each timestep ~
   - Return an *estimated* position for each asteroid ID for the *next* timestep.

2. **Noise and No-Noise Cases**  
   - Some scenarios have no noise, while others have noise following a gaussian distribution

---

## Navigation

**Goal:**  
Use asteroid predictions to plan hops for the spaceship until it reaches **homebase** at the top of the field.

### Key Points
1. **Spaceship State**  
   - Represented by a blue rocket icon at its true location.
   - Initially “riding” a particular asteroid (center blue dot).
   - Jump range = radius around the spaceship’s current position. Any asteroid outside this circle is unreachable.

2. **Jump Mechanics**  
   - When you jump onto an asteroid, the spaceship’s **true** state (position, velocity, etc.) becomes that asteroid’s state.  
   - **Important**: You do *not* get to see the true asteroid data directly; you still rely on noisy measurements to plan your jumps.

3. **Invalid Jumps**  
   - Jumping to an asteroid outside the jump radius.
   - Jumping to or from a location beyond the field boundaries.
   - Attempting to jump to an asteroid that has left the field.
 

## How Kalman Filtering Applies

Because many of the asteroid position measurements are **noisy**, a **Kalman filter** (or a variant) can be employed to track each asteroid’s true state (position, velocity, and possibly acceleration). A Kalman filter helps in providing an optimal estimate (in a least-squares sense) given a linear (or linearized) motion model and Gaussian noise assumptions.

### Important Features & Steps in a Kalman Filter

1. **State Representation**  
   A typical state vector for an asteroid might include:
   
   $$\mathbf{x} = \begin{bmatrix} x \\ y \\ v_x \\ v_y \\ a_x \\ a_y \end{bmatrix},$$
   
   where $(x, y)$ is position, $(v_x, v_y)$ is velocity, and $(a_x, a_y)$ is acceleration.

3. **Prediction (Time Update)**
   - **Predict the next state** using the known motion model (e.g., constant acceleration over a 1-second timestep).
   - In discrete form:
     
     $$\mathbf{x}_{k+1} = \mathbf{F}\,\mathbf{x}_k + \mathbf{u}_k,$$

     where $(\mathbf{F})$ is the state transition matrix derived from:
     
     $$x_{k+1} = x_k + v_x \Delta t + \tfrac{1}{2} a_x \Delta t^2, \quad v_{x,k+1} = v_{x,k} + a_{x,k} \Delta t, \quad \text{...}$$

     $(\mathbf{u}_k)$ could be external inputs or remain zero if none are present.
   - **Predict the next covariance** to account for process noise.

4. **Measurement Update**
   - Measurements might only be \((x, y)\) positions (i.e., partial observations of the state).
   - **Compute the innovation** (difference) between the measured position and the predicted position from the state vector.
   - **Update** the state vector and covariance using the Kalman Gain \(K\):
     
     $$\mathbf{x}_{k+1} \leftarrow \mathbf{x}_{k+1} + K \left( \mathbf{z}_{k+1} - \mathbf{H}\,\mathbf{x}_{k+1} \right),$$

     where $(\mathbf{z}_{k+1})$ is the actual measurement (observed $(x, y)$, and $(\mathbf{H})$ is the observation matrix mapping the state to the measured quantities.

5. **Noise Handling**
   - **Process Noise**: Represents modeling errors or unmodeled forces in the asteroid’s motion.
   - **Measurement Noise**: Represents sensor inaccuracies in the reported asteroid positions.

6. **Iterate**  
   At each timestep:
   1. Predict asteroid states using the motion model.
   2. Incorporate the new (possibly noisy) measurements to correct these predictions.
   3. Output the best estimate of each asteroid’s position for the next timestep.
