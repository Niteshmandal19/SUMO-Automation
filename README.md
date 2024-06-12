# SUMO-Automation
automating process of making SUMO maps from a Excel sheet

Step 1:
Creating Nodes: 
{
import pandas as pd
import xml.etree.cElementTree as ET
import os

def generate_nodes_xml(nodes_data, output_file):
    root = ET.Element("nodes")
    for _, row in nodes_data.iterrows():
        node = ET.SubElement(root, "node")
        node.set("id", str(row["node_id"]))
        node.set("x", str(row["x"]))
        node.set("y", str(row["y"]))
        if pd.notnull(row["z"]):
            node.set("z", str(row["z"]))
        if not pd.isna(row["node_type"]):
            node.set("type", row["node_type"])
    
    tree = ET.ElementTree(root)
    tree.write(output_file, xml_declaration=True, encoding="utf-8")

if __name__ == "__main__":
    try:
        # Print the current working directory
        print("Current working directory:", os.getcwd())
        
        # Full path to the Excel file
        file_path = r"C:\Users\nitesh mandal\Desktop\SUMO KA PY\data.xlsx"
        
        # Print available sheet names to debug
        with pd.ExcelFile(file_path, engine='openpyxl') as xls:
            print("Available sheets:", xls.sheet_names)
        
        # Read data from Excel file using openpyxl engine
        nodes_data = pd.read_excel(file_path, sheet_name="Nodes", engine='openpyxl')
        print("Data read successfully.")
        
        # Generate XML file
        generate_nodes_xml(nodes_data, "nodes.xml")
        print("XML file generated successfully.")
    
    except FileNotFoundError as e:
        print(f"File not found: {e}")
    except ValueError as e:
        print(f"Value error: {e}")
    except Exception as e:
        print(f"An error occurred: {e}")
}

Step 2:
Creating Edges
{
import openpyxl
from xml.etree.ElementTree import Element, SubElement, tostring
from xml.dom import minidom

def xlsx_to_edge_xml(xlsx_file, xml_file, sheet_name="Edges"):
    # Load the workbook
    wb = openpyxl.load_workbook(xlsx_file)
    
    # Select the Edges sheet
    if sheet_name not in wb.sheetnames:
        raise ValueError(f"Sheet '{sheet_name}' not found in the workbook.")
    sheet = wb[sheet_name]
    
    # Get column headers
    headers = [cell.value for cell in sheet[1]]
    
    # Create the root element
    edges = Element('edges')
    
    # Skip the header row
    for row in sheet.iter_rows(min_row=2, values_only=True):
        edge_data = dict(zip(headers, row))
        
        # Create an edge element
        edge = SubElement(edges, 'edge')
        
        # Required attributes
        edge.set('id', str(edge_data.get('Edge ID', '')))
        
        # Add ".0" to node IDs for 'from' and 'to' attributes
        from_node = str(edge_data.get('From Node', ''))
        to_node = str(edge_data.get('To Node', ''))
        edge.set('from', f"{from_node}.0")
        edge.set('to', f"{to_node}.0")
        
        # Optional attributes
        optional_attrs = [
            'Type', 'numLanes', 'Speed (m/s)', 'Priority', 'Length (m)',
            'Shape', 'SpreadType', 'Allow', 'Disallow', 'Width (m)', 'Name',
            'endOffset (m)', 'sidewalkWidth (m)', 'bikeLaneWidth (m)', 'distance (m)'
        ]
        
        attr_mapping = {
            'Type': 'type',
            'numLanes': 'numLanes',
            'Speed (m/s)': 'speed',
            'Priority': 'priority',
            'Length (m)': 'length',
            'Shape': 'shape',
            'SpreadType': 'spreadType',
            'Allow': 'allow',
            'Disallow': 'disallow',
            'Width (m)': 'width',
            'Name': 'name',
            'endOffset (m)': 'endOffset',
            'sidewalkWidth (m)': 'sidewalkWidth',
            'bikeLaneWidth (m)': 'bikeLaneWidth',
            'distance (m)': 'distance'
        }
        
        for xlsx_attr, xml_attr in attr_mapping.items():
            if xlsx_attr in edge_data and edge_data[xlsx_attr] is not None:
                edge.set(xml_attr, str(edge_data[xlsx_attr]))
    
    # Convert the ElementTree to a string
    rough_string = tostring(edges, 'utf-8')
    
    # Pretty-print the XML string
    reparsed = minidom.parseString(rough_string)
    pretty_xml = reparsed.toprettyxml(indent="  ")
    
    # Write to file
    with open(xml_file, "w") as f:
        f.write(pretty_xml)

# Usage
xlsx_to_edge_xml("data.xlsx", "edges.edg.xml")
}

Step 3: 
Create Sumotest.net.xml using 
(netconvert --node-files=Nodes.nod.xml --edge-files=Edges.edg.xml \  --output-file=sumotest.net.xml)

Step 4:
Create trips using randomTrips.py
(randomTrips.py -n sumotest.net.xml -e 1000 -o sumotest.trips.xml)

Step 5:
create routes file 
(duarouter -n sumotest.net.xml --route-files sumotest.trips.xml -o sumotest.rout.xml --ignore-errors)

Step 6:
create sumotest.sumo.cfg by adding the below code in txt file and renaming it as (sumotest.sumo.cfg)
(<?xml version="1.0" encoding="UTF-8"?>

<configuration xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://sumo.dlr.de/xsd/sumoConfiguration.xsd">

    <input>
        <net-file value="sumotest.net.xml"/>
        <route-files value="sumotest.trips.xml"/>
	    
	</input>

    <time>
        <begin value="0"/>
        <end value="1000"/>
    </time>

   

</configuration>)
  
