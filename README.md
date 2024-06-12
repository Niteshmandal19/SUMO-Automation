# SUMO-Automation
automating process of making SUMO maps from a Excel sheet

Step 1:
Creating Nodes: 

Step 2:
Creating Edges

Step 3: 
Create Sumotest.net.xml using 
( netconvert --node-files=Nodes.nod.xml --edge-files=Edges.edg.xml \  --output-file=sumotest.net.xml)

Step 4:
Create trips using randomTrips.py
( randomTrips.py -n sumotest.net.xml -e 1000 -o sumotest.trips.xml)

Step 5:
create routes file 
( duarouter -n sumotest.net.xml --route-files sumotest.trips.xml -o sumotest.rout.xml --ignore-errors)

Step 6:
create sumotest.sumo.cfg by adding the below code in txt file and renaming it as (sumotest.sumo.cfg)
{ <?xml version="1.0" encoding="UTF-8"?>

<configuration xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://sumo.dlr.de/xsd/sumoConfiguration.xsd">

    <input>
        <net-file value="sumotest.net.xml"/>
        <route-files value="sumotest.trips.xml"/>
	    
	</input>

    <time>
        <begin value="0"/>
        <end value="1000"/>
    </time>

   

</configuration>
}
  
