# Bayesian based pose estimation of a discrete 1-dimensional traversing autonomous robot

## Assumptions
1. The robot only moves in the forward direction.
2. The robot moves only one step(1m) at a time in normal conditions.
3. Assuming the robot starts from the left most initial state (state 0 in my case).
4. Assume the path to be a loop, i.e, the successor of the the last state is the zeroth state, we must hardcode them as they could not be done during iteration.
5. The prior knowledge/belief at the start is assumed to be distributed equally likely within the given range,i.e, the prior belief = 1/100 = 0.01.
6. Assuming that the robot first moves and then measures.

## Bayes Filter Implementation

To estimate the probability of a robot being at a new state requires information such as the previous estimation of robot being at a given state, the action model and the measurement model. We use the Bayes filter algorithm to recursively calculate belief X<sub>t</sub> from X<sub>(t-1)</sub> <br> 
The action model and the measurement model was defined in the scope of this program. The action model remains the same in the whole process whereas the prior state and the measurement model changes with each timestep. Action model and the measurement model was hardcoded for the boundary conditions (for the first and the last steps).

#### Action model representation
> P(X<sub>(i+1)</sub> | X<sub>i</sub> , u<sub>i</sub>) <br> assuming Markov property, where X is the position or state and u is the action (moving forward, staying at the same position, moving 1 step forward, moving 2 steps forward) 

#### Measurement model representation
>P(Z<sub>i</sub> | X<sub>i</sub>) <br> assuming Markov property, where Z is the measurement and X is the current position.

#### Logic for ACTION_ARRAY code

<p> Action array is a 100x100 2D array 100 rows represent hundred 1m spaces where the action model is implemented for different timesteps. For example, in the 1st time step which involves the movement from 0th state to the 1st state (movement from the 0th column to the 1st column as per the logic of my code), the 1st(index value = 0) row represents the probability values for the robot to be in a particular state according to the action model. 100 columns represent the 100m movement space where the actual movement of the robot takes place between different states along with the inherent probability values assocaited with the movements. At any certain time I'm fixing the row, thus any operation(forward, backward) with respect to the motion of the car happens in a 1D space with alterations to the column values. </p>

#### Logic for MEASUREMENT_ARRAY code

<p> Measurement array is a 100x100 2D array 100 rows represent hundred 1m spaces where the measurement model is implemented for different timesteps. For example, in the 1st time step which involves the movement measurement from 0th state to the 1st state (movement measurement from the 0th column to the 1st column as per the logic of my code), the 0th row represents the probability values for the robot to be in a particular state according to the measurement model - the first column in the first row must be 0.8. 100 columns which represent the 100m movement space where the actual movement of the robot takes place between different states along with the inherent probability values assocaited with the movements. </p>

#### Initial prior knowledge

Initially the prior knowledge of the robot being at a state (location in the world) was considered to be spread equally likely over the full space and was defined as `belief_state = np.full(100,0.01)`

#### Prediction 

<p>Then we go ahead with the prediction which tells you the probability for the movement from previous posotion to your current position due to performance of an action. Here we predict the probability of moving to the consecutive state from the current state for the whole time step for all the given states given the robot does an action. The summation of the probabilities for each state is calculated and this gives you the possibility of being at the state at that particular time step irrespective of your start position. np.dot multiplies the elements of the array and sums it. This is defined as `bel_bar = np.dot(belief_state,action_array)`
  
#### Measurement update/ internal belief
<p>After knowing the probabilities of being at a particular state at a given timestep post making a move/action(Prediction), we multiply this probability with the measurement model(sensor model data) to end up at the internal belief of the robot being at a particular state at that given timestep. np.multiply function does element to element multiplication. This is defined as `y = np.multiply(bel_bar,measurement_array[j])`
  
#### Normalization 
The normalization factor (norm_fact) is calculated by dividing the sum of beliefs. This is defined as `norm_fact = 1 / (np.sum(y))`

#### Actual belief
We multiply the internal belief with the normalization factor to get the probabilities within the range (<1). This is defined as `belief_state = norm_fact * y`

###### This actual belief gets updated for each timestep and the actual belief of current state would be used as the prior belief for the next consecutive state (i.e. during the transformation from the current to the next consecutive state).
