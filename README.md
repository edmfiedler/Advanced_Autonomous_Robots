# Demonstration

https://youtu.be/QykghXVawko

# Setup based on a provided VM

Put the plugins relocate and scanright into the folder:

/home/smr/mobotware/aurs-plugins

In /home/smr/offline/sim388, add course_line, course_obst, and
simconfigcourse. Also, in ulmsserver.ini, add the locations of the plugins as:

module load="/home/smr/mobotware/aurs-plugins/scanright/scanright.so.0"
module load="/home/smr/mobotware/aurs-plugins/relocate/relocate.so.0"

at the bottom.

To run, open a terminal open the terminal on the desktop and do:

cd offline/sim388

Then, start the simulator:

simserver1 simconfigcourse.xml

Open another terminal in the same location (Ctrl+Shift+T) and start the 
laser server:

ulmsserver

Check if the modules loaded in correctly using:

module list

It should show that scanright and relocate have loaded.

Next, open another terminal and load the script:

mrc -s8000 course_graphplanner

# Route planning

The robot will go to whatever guidemark is specified as rt[0] first (set to
12), and will then read the gmno of that guidemark in simconfigcourse.xml 
to decide to which guidemark it is going to navigate to next. 

Based on simconfigcourse.xml, the only path that leads to either 13 or 14
is as follows:

2->7->11->3->6->12->13

Going to any other guidemark (other than directly to 14) would lead to
gettting stuck at 1, 4, 5, due to the specified gmno values being read by
the robot.

The navigation works by knowing the positions it should drive to to see the
guidemark in the arrays gmx and gmy and the actual position of the guidemark
in the arrays gx and gy, in order to face the guidemark.

# Scanning 

After reading the gmno of guidemark 13 or 14, the robot goes roughly into 
the position (2,2.25,-1.57) and starts the object detection. It first uses
the plugin relocate to determine a position facing perpendicular to the 
right-most side of the shape, in between the left and right-most points of
the shape that are being seen. It then uses the plugin scanright, to scan
the right-most point from that position and determines the location and 
orientation of where to take the next scan from. 

The scans are corners of the shape, but the data has to be treated in 
order to analyse it:
1) A mean position of points closer than 0.05 is taken (for when 4 scans
are taken for the triangle
2) The order of the points scanned is reversed to obtain positive internal
angles (not sure if this is always required, in MATLAB this works
consistently.)

Then, based on internal angles, and side length, the type, position, and
orientation can be determined. This is shown in the executed code in the 
terminal e.g. Rectangle 04x0.15 (x,y,ori). The validity of this output
can be checked using the MATLAB script course_detect in the folder 
coursedetect.

The script chooses a random pose for a random shape and places it within
the boundaries. The output in the command window gives the connections of 
points used to make the shape which can be copy-pasted into course_obst
such that the same shape can be drawn in the simulator. It then gives the 
calculated shape classification and pose, and then the actual. The 
calculation is based on the same principle of what is used in the SMR.
The actual classification and pose can be compared to what is found in the 
SMR.

The robot then drives to the checkpoint it is closest to and drives to the
start position.
