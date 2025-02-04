This report investigates fault injection and fault tolerance mechanisms in the co-simulated 
environment of Robotti, an autonomous farm vehicle. The goal is to test the robustness and 
dependability when injecting fault and implementing a fault tolerance mechanism to maintain 
the system in a valid state. Using VDM-RT and INTO-CPS tools, the study simulates a 
sensor failure by injecting a fault that causes a loss of positional data during navigation. With 
this, a last known position mechanism was implemented, which relies on substituting invalid 
GPS readings with the last-known valid position preventing from stopping the system. The 
results showed that the fault injection successfully disrupted normal operation, but the fault 
tolerance mechanism allowed the vehicle to continue operating with the same result. 
Thereafter, we highlight the importance of the approaching challenges, integrating insights 
from literature to suggest improvements for future work. 


Fault Injection 
Description of Fault Injection 
For this project, a fault was injected into a SensorGNSS class to simulate a loss of signal 
situation at a specified time. The sensor was modified to periodically output a fixed, 
erroneous position instead of the current position data. In the model, at a specified time 
(faultStartTime), the GPS outputted invalid data ([0.0, 0.0, 0.0, 0.0]) instead of real 
positional data. This disruption represented a realistic and critical issue in autonomous 
systems, particularly for agricultural robots navigating terrain with obstacles. Fault injection 
was controlled by triggering faultInjected in the read() method after Sync() updated the 
simulation time. Invalid GPS data replaced valid readings, effectively simulating GPS failure. 
The read() method provides invalid or zeroed data when faultInjected is true. Below is 
the relevant portion of the fault injection implementation: -- Simulate a fault injection when GPS signal is lost 
public read: () ==> seq of real 
read() == ( 
Sync(); -- Ensure time simulation updated before reading 
if faultInjected then ( 
local_val := [0.0, 0.0, 0.0, 0.0];  -- Return invalid data                
or any  invalid value) 
); 
return local_val; -- Return the local value (either valid 
or invalid based on faultInjected) 
); 
Description of Selected Fault 
The fault selected was GPS sensor failure because it is a realistic and critical issue in 
autonomous vehicles, particularly in agricultural settings where GPS interference can be 
caused by natural obstacles like trees or terrain. Autonomous vehicles rely on accurate 
satellite positional data in navigation systems. Signal loss is one of the most frequent failures 
in such scenarios supported by lecture and literature on GNSS faults in robotics [1]. The GPS 
sensor failure was compatible with the pre-existing controller and route logic, allowing for 
seamless integration of the fault injection. This ensured that the model remained functional 
for evaluating fault tolerance. 
Fault → Error → Failure Model: 
• Fault: Disruption in GNSS signal. 
• Error: Invalid values of GPS data. 
• Failure: Robotti’s inability to follow a defined route. 
Discussion of Design Patterns 
The Observer Pattern was used between SensorGNSS and the controller, monitoring data and 
triggering responses [2]. The Active Object Pattern ensured synchronization for precise fault 
activation during simulation. 
 
Figure 1.  The following diagram illustrates the Robotti's trajectory before(left) and after(right) fault injection. 
 
 



 
Fault Tolerance 
Description of Fault Tolerance in Controller model 
We address GPS signal loss to prevent system failure by adding fallback mechanism to the 
controller. The idea was to utilize the last known position from the GPS sensor with the given 
PosGNSS. When the fault was detected, which triggered the fallback, the controller would stop 
reading the GPS data and instead use the lastValidPos for navigation. This ensured the 
Robotti could continue navigating, albeit with increasing positional error over time, allowing 
it to continue its operation by relying on the last valid data, steering towards the current 
waypoint. This recovery strategy aimed to mitigate the impact of GPS failures by maintaining 
operational continuity but lacks adaptive fault correction, as noted in the literature on fault
tolerant systems [3]. The following code demonstrates the fault tolerance mechanism: -- Fault Detection Logic 
            if PosGNSS(1) = 0.0 and PosGNSS(2) = 0.0 then ( 
                IO`printf("GPS fault detected! Estimating position based on 
last valid data.\n", []); 
                predictedPos := [lastValidPos(1) + REQUESTED_SPEED * 0.1,               
                                 lastValidPos(2), lastValidPos(3),0.0]; 
                PosGNSS := predictedPos; -- Use predicted position 
            ) 
            else ( 
-- Update last valid position 
lastValidPos := PosGNSS; 
); 
Description of Selected Fault Tolerance 
The fallback mechanism was chosen for its simplicity and feasibility. This mechanism was 
inspired by redundancy strategies discussed in the lecture material [1]. The fallback 
mechanism was implemented by storing the last valid GPS position before the fault occurred. 
It prevented abrupt stops by substituting real-time positional data with static last-known 
values. While effective in maintaining continuity, the simplicity of this mechanism limited its 
ability to correct trajectory errors. By implementing fallback mechanism, it can imitate 
drivers or even better according to scholar articles [4]. The emerging software design and 
development can also maintain the autonomous vehicle to navigate accurately according to its 
desired route.  
Discussion of Design Patterns  
The State Pattern (Inheritance) handled transitions between valid and fallback modes, 
ensuring seamless operation. The Redundancy Pattern promotes modularity and flexibility in 
implementing fault recovery [5]. Hence, these patterns influence system design in more detail 
with fault injection. 
Results and Evaluation 
This section aims to describe and evaluate the implementation of fault injection and fault 
tolerance mechanisms in the GPS controller of an autonomous robotic system to follow a 
figure-8 trajectory. The system integrates a GPS sensor, actuators, and a controller 
responsible for detecting and mitigating GPS faults. The results revealed that while fault 
injection was successfully implemented, the fault tolerance mechanism did not restore the 
robot's intended behaviour. 
Behaviour after Fault Injection. 
The fault injection mechanism functioned as desired with the robot continues in a straight 
trajectory instead of the figure-8 path. Faults were triggered at a specified simulated time 
(faultStartTime = 10.0 seconds), with the GPS sensor returning invalid data to mimic 
the absence of a valid GPS signal. This behaviour was consistent throughout the simulation, 
ensuring a reliable test environment for evaluating fault tolerance. Upon the detection of GPS 
failure fault, Robotti’s current position data is reset and remained fixed [0, 0]. The logs 
confirmed that the system attempts to navigate to the waypoint [0, 20.444], but since the 
position is [0, 0], it effectively starts over and continues as if it's at the origin. The Robotti 
was able to continue moving, but the navigation became less accurate as it could not update 
its position in real-time. This result demonstrated the GNSS fault injection directly impacts 
position tracking. 
Simulation Logs: 
INFO - VdmOut OK Steering_control Fault injected at simulation time: 
10.050000000000008 seconds 
INFO - VdmOut OK Steering_control Steering angle: 0, speed: 1, current 
position: [0,0], current waypoint: [0,20.444] 
Figure 2 shows the behaviour after fault injection 
Behaviour after Fault Tolerance was Added 
After implementing the fault tolerance mechanism, the Robotti continued its path with 
minimal disruption. The system estimated the vehicle's position based on the last valid GNSS 
data after detecting a GNSS failure. The fault tolerance mechanism allowed the vehicle to 
continue moving, but it did so with noticeable limitations. Specifically, Robotti successfully 
completed half of the figure-8 path but then started circling, deviating from its intended route. 
While the vehicle was able to avoid sudden stops or errors in navigation, the trajectory 
became increasingly imprecise. It effectively ensured that the vehicle did not halt or crash 
due to the GNSS failure, but it struggled with realigning the vehicle's trajectory to the 
original figure-8 path. The fallback mechanism retained static positional data without 
recalibrating key variables such as steering angle. Multiplying REQUESTED_SPEED by a fixed 
value (0.1) assumes a constant relationship between the vehicle's speed and the deviation 
caused by the fault. This might work for minor deviations but fails in scenarios where the 
deviation magnitude varies significantly [6]. Although the system preserved movement 
continuity, the lack of trajectory correction led to a permanent deviation from the intended 
path. This became evident in the logs, where the robot's steering angle and speed remained 
unchanged despite the vehicle's departure from the desired route. This indicates that the robot 
continued in a circular pattern with no significant change in the control variables, despite the 
deviation from the trajectory. 
Simulation Logs: 
INFO - VdmOut OK Steering_control GPS fault detected! Estimating position 
based on last valid data. 
INFO - VdmOut OK Steering_control Steering angle: -0.1536559579383141, 
speed: 1, current position: [0.10000000007797853,19.79832526682544], 
current waypoint: [0,20.444] 
Figure 3 shows the behaviour after fault tolerance was added 
Evaluation of Fault Injection and Fault Tolerance Mechanism  
Fault injection successfully simulated a GPS failure, the expected behaviour makes the robot 
move in a straight line. The fault injection was effective due to its direct implementation in 
the SensorGNSS class. While the fault tolerance mechanism did mitigate the immediate 
impact of the GPS failure by holding the last known valid position, it failed to restore 
Robotti’s figure-8 trajectory due to static positional estimation and lack of real-time steering 
adjustments. The issue appears to be rooted in the interaction between the fault injection and 
the controller logic. The steering angle and speed, which were crucial for path-following, did 
not adjust based on the faulty data. Another potential issue lies in the fault injection timing. 
The fault was injected based on the simulated time step (Sync()) did not always synchronize 
with the controller’s update cycle, the controller would continue to use the previous data until 
the next synchronization. The waypoint navigation also contributed to the issue. While the 
controller used waypoints to guide the vehicle, Robotti could not recalculate its route 
effectively, continuing in a straight or circular path. While the fault tolerance mechanism 
prevented abrupt stops, its simplicity limited its ability to handle dynamic scenarios. 
Advanced strategies, such as predictive algorithms or sensor redundancy, are necessary for 
more robust fault recovery [7].  
The challenges implementing fault tolerance is due to strategies required to be compatible 
with types of fault injection. I failed to implement of nil values PosGNSS due to lack of VDM 
proficiency and time constraints. Implementing fallback strategies required balancing 
simplicity and functionality. While using last-known valid positions provided a baseline 
solution, it exposed limitations in trajectory correction and real-time adaptability. Injecting 
faults with precise timing required synchronizing simulation steps, particularly for complex 
control loops. Integrating faults without disrupting the system's functionality also posed 
challenges.




Conclusions 
Summary 
This study demonstrated the potential of fault injection and tolerance mechanisms in testing 
autonomous systems. While the fault injection accurately simulated GPS failures, the basic 
fault tolerance mechanism revealed significant limitations. Addressing these gaps with 
advanced strategies is critical for enhancing the reliability of autonomous vehicles in real
world scenarios. 
Future Work 
Implementing predictive models to estimate positional changes dynamically during faults can 
be helpful. Next, incorporating data from additional sensors, such as inertial measurement 
units (IMUs), to mitigate GPS failures. Working on Robotti’s software should prioritize 
heavy reliance on simulation tools like ROS and Gazebo for testing high-level control 
systems before field deployment. Software simulation for fault injection focus on the 
interplay between mechanical, electrical, and software systems. Adapting steering angles in 
real-time to maintain trajectory integrity using DSE can be considered in the future [7]. Also, 
exploring advanced fault-tolerant algorithms to improve response to positional errors helps 
with specific trajectories.  
Personal Reflection 
This coursework enhanced my understanding of fault injection and tolerance in autonomous 
systems that enables iterative software testing and debugging without risking expensive or 
unsafe physical tests. The integration of co-simulation tools with VDM-RT provided valuable 
hands-on experience. Challenges included applying various fault tolerance mechanism to 
make robot follows figure-8 trajectory and implementing log message to VDM. Writing 
limited-words report can be challenging if not addressed appropriately. 




References 
[1]  Lecture notes on Fault Tolerance in Cyber Physical System.  
[2]  P. Karsh, “7 Principles of Object-Oriented Design,” 27 Jun 2023.  
[3]  M. &. T. C. &. M. H. &. L. K. &. L. P. &. E. L. Frasheri, “Fault Injecting Co-simulations for Safety,” in 5th 
International Conference on System Reliability and Safety (ICSRS), 2021. 
[4]  Yu Zhanga, Linda Angell, Shan Bao, “A fallback mechanism or a commander? A discussion about the role 
and skillneeds of future drivers within partially automated vehicles,” Transportation Research 
Interdisciplinary Perspectives, p. 31, 2021. 
[5]  Igal Bilik, Oren Longman, Shahar Villeval, and Joseph Tabrikian, “The Rise of Radar for Autonomous 
Vehicles: Signal Processing Solutions and Future Research Directions,” IEEE Signal Processing Magazine, 
pp. 20-31, September 2019. 
[6] Byung-Hyun Lee, Jong-Hwa Song, Jun-Hyuck Im, Sung-Hyuck Im, Moon-Beom Heo and Gyu-In Jee, 
“GPS/DR Error Estimation for Autonomous Vehicle Localization,” p. 20, August 2015.  
[7] Interview with Frederick Foldager. 
[8] INTO-CPS Tool Chain User Manual.  
Appendix  
Changing Fault Timing 
To modify the fault timing in the model, update the faultTime parameter in the SensorGNSS 
component: 
protected faultStartTime: real := 25;  -- Change to desired time (seconds) 
Other Diagrams and Tables 
Figure 4 shows the Robotti’s original figure-8 trajectory without any changes to Controller model. 
Fault Injection Logs 
2024-12-02 21:58:31,075 INFO  - logAll OK Steering_control DoStep VDM internal time 
reached: 19850000000 at doStep completion 
2024-12-02 21:58:31,076 INFO  - logAll OK Steering_control DoStep VDM clock conversion. 
Internal time: 19850000000 External [s]: 19.9 
2024-12-02 21:58:31,083 INFO  - Protocol OK Steering_control DoStep waiting for next 
DoStep at: 19.9 
2024-12-02 21:58:31,089 INFO  - Protocol OK Steering_control DoStep called: 
19.900000000000148 
2024-12-02 21:58:31,090 INFO  - logAll OK Steering_control DoStep VDM internal stop 
time: 19900000000 
2024-12-02 21:58:31,091 INFO  - logAll OK Steering_control DoStep VDM internal time 
reached: 19900000000 at doStep completion 
2024-12-02 21:58:31,092 INFO  - logAll OK Steering_control DoStep VDM clock conversion. 
Internal time: 19900000000 External [s]: 19.900001 
2024-12-02 21:58:31,092 INFO  - Protocol OK Steering_control DoStep waiting for next 
DoStep at: 19.900001 
2024-12-02 21:58:31,100 INFO  - Protocol OK Steering_control DoStep called: 
19.95000000000015 
2024-12-02 21:58:31,101 INFO  - logAll OK Steering_control DoStep VDM internal stop 
time: 19950000000 
2024-12-02 21:58:31,108 INFO  - VdmOut OK Steering_control Steering angle: 0, speed: 1, 
current position: [0,19.798333333333467], current waypoint: [0,20.444] 
2024-12-02 21:58:31,109 INFO  - logAll OK Steering_control DoStep VDM internal time 
reached: 19950000000 at doStep completion 
2024-12-02 21:58:31,109 INFO  - logAll OK Steering_control DoStep VDM clock conversion. 
Internal time: 19950000000 External [s]: 20.0 
2024-12-02 21:58:31,110 INFO  - Protocol OK Steering_control DoStep waiting for next 
DoStep at: 20.0 
2024-12-02 21:58:31,116 INFO  - Protocol OK Steering_control DoStep called: 
20.00000000000015 
2024-12-02 21:58:31,118 INFO  - logAll OK Steering_control DoStep VDM internal stop 
time: 20000000000 
2024-12-02 21:58:31,119 INFO  - logAll OK Steering_control DoStep VDM internal time 
reached: 20000000000 at doStep completion 
2024-12-02 21:58:31,119 INFO  - logAll OK Steering_control DoStep VDM clock conversion. 
Internal time: 20000000000 External [s]: 20.000001 
2024-12-02 21:58:31,120 INFO  - Protocol OK Steering_control DoStep waiting for next 
DoStep at: 20.000001 
2024-12-02 21:58:31,127 INFO  - Protocol OK Steering_control DoStep called: 
20.05000000000015 
2024-12-02 21:58:31,127 INFO  - logAll OK Steering_control DoStep VDM internal stop 
time: 20050000000 
2024-12-02 21:58:31,130 INFO  - VdmOut OK Steering_control Fault injected at simulation 
time: 10.050000000000008 seconds 
2024-12-02 21:58:31,135 INFO  - VdmOut OK Steering_control Steering angle: 0, speed: 1, 
current position: [0,0], current waypoint: [0,20.444] 
.. 
2024-12-02 21:58:49,631 INFO  - logAll OK Steering_control DoStep VDM internal time 
reached: 105850000000 at doStep completion 
2024-12-02 21:58:49,632 INFO  - logAll OK Steering_control DoStep VDM clock conversion. 
Internal time: 105850000000 External [s]: 105.9 
2024-12-02 21:58:49,632 INFO  - Protocol OK Steering_control DoStep waiting for next 
DoStep at: 105.9 
2024-12-02 21:58:49,637 INFO  - Protocol OK Steering_control DoStep called: 
105.89999999999613 
2024-12-02 21:58:49,638 INFO  - Protocol OK Steering_control DoStep skipping execution 
next time is: 105.9 
2024-12-02 21:58:49,642 INFO  - Protocol OK Steering_control DoStep called: 
105.94999999999612 
2024-12-02 21:58:49,643 INFO  - logAll OK Steering_control DoStep VDM internal stop 
time: 105950000000 
2024-12-02 21:58:49,644 INFO  - VdmOut OK Steering_control Fault injected at simulation 
time: 52.999999999999126 seconds 
2024-12-02 21:58:49,648 INFO  - VdmOut OK Steering_control Steering angle: 0, speed: 1, 
current position: [0,0], current waypoint: [0,20.444] 
After fault tolerance log 
2024-12-02 21:40:58,817 INFO  - logAll OK Steering_control DoStep VDM internal time reached: 
19850000000 at doStep completion 
2024-12-02 21:40:58,818 INFO  - logAll OK Steering_control DoStep VDM clock conversion. Internal 
time: 19850000000 External [s]: 19.9 
2024-12-02 21:40:58,818 INFO  - Protocol OK Steering_control DoStep waiting for next DoStep at: 
19.9 
2024-12-02 21:40:58,824 INFO  - Protocol OK Steering_control DoStep called: 19.900000000000148 
2024-12-02 21:40:58,824 INFO  - logAll OK Steering_control DoStep VDM internal stop time: 
19900000000 
2024-12-02 21:40:58,829 INFO  - logAll OK Steering_control DoStep VDM internal time reached: 
19900000000 at doStep completion 
2024-12-02 21:40:58,829 INFO  - logAll OK Steering_control DoStep VDM clock conversion. Internal 
time: 19900000000 External [s]: 19.900001 
2024-12-02 21:40:58,831 INFO  - Protocol OK Steering_control DoStep waiting for next DoStep at: 
19.900001 
2024-12-02 21:40:58,840 INFO  - Protocol OK Steering_control DoStep called: 19.95000000000015 
2024-12-02 21:40:58,841 INFO  - logAll OK Steering_control DoStep VDM internal stop time: 
19950000000 
2024-12-02 21:40:58,851 INFO  - VdmOut OK Steering_control Steering angle: 
2.4855762781328394E-8, speed: 1, current position: [7.797852641012893E
11,19.79832526682544], current waypoint: [0,20.444] 
2024-12-02 21:40:58,852 INFO  - logAll OK Steering_control DoStep VDM internal time reached: 
19950000000 at doStep completion 
2024-12-02 21:40:58,853 INFO  - logAll OK Steering_control DoStep VDM clock conversion. Internal 
time: 19950000000 External [s]: 20.0 
2024-12-02 21:40:58,853 INFO  - Protocol OK Steering_control DoStep waiting for next DoStep at: 
20.0 
2024-12-02 21:40:58,863 INFO  - Protocol OK Steering_control DoStep called: 20.00000000000015 
2024-12-02 21:40:58,865 INFO  - logAll OK Steering_control DoStep VDM internal stop time: 
20000000000 
2024-12-02 21:40:58,866 INFO  - logAll OK Steering_control DoStep VDM internal time reached: 
20000000000 at doStep completion 
2024-12-02 21:40:58,867 INFO  - logAll OK Steering_control DoStep VDM clock conversion. Internal 
time: 20000000000 External [s]: 20.000001 
2024-12-02 21:40:58,868 INFO  - Protocol OK Steering_control DoStep waiting for next DoStep at: 
20.000001 
2024-12-02 21:40:58,876 INFO  - Protocol OK Steering_control DoStep called: 20.05000000000015 
2024-12-02 21:40:58,878 INFO  - logAll OK Steering_control DoStep VDM internal stop time: 
20050000000 
2024-12-02 21:40:58,882 INFO  - VdmOut OK Steering_control Fault injected at simulation time: 
10.050000000000008 seconds 
2024-12-02 21:40:58,884 INFO  - VdmOut OK Steering_control GPS fault detected! Estimating 
position based on last valid data. 
2024-12-02 21:40:58,889 INFO  - VdmOut OK Steering_control Steering angle: 
0.1536559579383141, speed: 1, current position: [0.10000000007797853,19.79832526682544], 
current waypoint: [0,20.444] 
… 
2024-12-02 21:41:43,487 INFO  - logAll OK Steering_control DoStep VDM internal time reached: 
105850000000 at doStep completion 
2024-12-02 21:41:43,487 INFO  - logAll OK Steering_control DoStep VDM clock conversion. Internal 
time: 105850000000 External [s]: 105.9 
2024-12-02 21:41:43,487 INFO  - Protocol OK Steering_control DoStep waiting for next DoStep at: 
105.9 
2024-12-02 21:41:43,495 INFO  - Protocol OK Steering_control DoStep called: 105.89999999999613 
2024-12-02 21:41:43,497 INFO  - Protocol OK Steering_control DoStep skipping execution next time 
is: 105.9 
2024-12-02 21:41:43,503 INFO  - Protocol OK Steering_control DoStep called: 105.94999999999612 
2024-12-02 21:41:43,504 INFO  - logAll OK Steering_control DoStep VDM internal stop time: 
105950000000 
2024-12-02 21:41:43,506 INFO  - VdmOut OK Steering_control Fault injected at simulation time: 
52.999999999999126 seconds 
2024-12-02 21:41:43,509 INFO  - VdmOut OK Steering_control GPS fault detected! Estimating 
position based on last valid data. 
2024-12-02 21:41:43,515 INFO  - VdmOut OK Steering_control Steering angle: 
0.1536559579383141, speed: 1, current position: [0.10000000007797853,19.79832526682544], 
current waypoint: [0,20.444]
