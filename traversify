import arcpy
import csv
import math
import os
from datetime import datetime

"""
Survey Traverse Tool for ArcGIS
-------------------------------
This script converts survey traverse data from CSV format into a point feature class.
It reads distance and azimuth measurements and calculates coordinates for each point
along the traverse, starting from an initial known point.

Requirements:
- ArcGIS Pro or ArcMap with arcpy
- CSV file with survey data (must include Distance and Azimuth columns)
- First row in CSV should contain the starting coordinates

Author: Your Name
Last Updated: March 3, 2025
"""

def create_traverse_points(input_csv, output_folder, output_name="traverse_output.shp", spatial_ref=4326):
    """
    Creates a point feature class from survey traverse data.
    
    Args:
        input_csv (str): Path to the CSV file containing traverse data
        output_folder (str): Directory where the output shapefile will be saved
        output_name (str): Name of the output shapefile
        spatial_ref (int/str): Spatial reference ID or path to projection file
        
    Returns:
        str: Path to the created feature class
    """
    # Set workspace and output file path
    arcpy.env.workspace = output_folder
    output_fc = os.path.join(output_folder, output_name)
    
    # Create log file for errors and info
    log_file = os.path.join(output_folder, "traverse_log.txt")
    with open(log_file, 'w') as log:
        log.write(f"Traverse Processing Log - {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}\n")
        log.write(f"Input file: {input_csv}\n")
        log.write(f"Output file: {output_fc}\n\n")
    
    try:
        # Create an empty point feature class with additional fields
        arcpy.CreateFeatureclass_management(
            output_folder, 
            output_name, 
            "POINT", 
            spatial_reference=spatial_ref
        )
        
        # Add fields to store traverse data
        arcpy.AddField_management(output_fc, "POINT_NUM", "LONG")
        arcpy.AddField_management(output_fc, "DISTANCE", "DOUBLE")
        arcpy.AddField_management(output_fc, "AZIMUTH", "DOUBLE")
        arcpy.AddField_management(output_fc, "NOTES", "TEXT", field_length=255)
        
        # Create a cursor to insert points
        fields = ['SHAPE@XY', 'POINT_NUM', 'DISTANCE', 'AZIMUTH', 'NOTES']
        cursor = arcpy.da.InsertCursor(output_fc, fields)
        
        # Process traverse data from CSV
        with open(input_csv, mode='r') as file:
            reader = csv.DictReader(file)
            rows = list(reader)  # Convert to list to access first row separately
            
            if not rows:
                raise ValueError("CSV file is empty or improperly formatted")
            
            # Get starting coordinates from first row
            try:
                start_x = float(rows[0]['Longitude'])
                start_y = float(rows[0]['Latitude'])
                
                # Log starting point
                with open(log_file, 'a') as log:
                    log.write(f"Starting point: {start_x}, {start_y}\n")
                
                # Insert the first point
                cursor.insertRow([(start_x, start_y), 1, 0, 0, "Starting Point"])
                
                # Loop through the rows and create traverse points
                for i, row in enumerate(rows, start=2):  # Start numbering from 2 (after starting point)
                    # Extract values, with error handling
                    try:
                        distance = float(row['Distance'])
                        azimuth = float(row['Azimuth'])
                        notes = row.get('Notes', '')  # Optional field
                    except (KeyError, ValueError) as e:
                        with open(log_file, 'a') as log:
                            log.write(f"Error at point {i}: {str(e)}\n")
                        continue
                    
                    # Convert azimuth to radians
                    azimuth_rad = math.radians(azimuth)
                    
                    # Calculate new coordinates using basic trigonometry
                    # Note: This is a simple calculation and might need adjustments
                    # for different coordinate systems or Earth curvature
                    delta_x = distance * math.sin(azimuth_rad)
                    delta_y = distance * math.cos(azimuth_rad)
                    
                    new_x = start_x + delta_x
                    new_y = start_y + delta_y
                    
                    # Insert the new point
                    cursor.insertRow([(new_x, new_y), i, distance, azimuth, notes])
                    
                    # Update starting point for next iteration
                    start_x, start_y = new_x, new_y
                    
                    # Log progress every 10 points
                    if i % 10 == 0:
                        with open(log_file, 'a') as log:
                            log.write(f"Processed point {i}: {new_x}, {new_y}\n")
                
                print(f"Traverse complete. Created {i} points.")
                with open(log_file, 'a') as log:
                    log.write(f"Successfully completed traverse with {i} points\n")
                
            except KeyError:
                error_msg = "CSV must contain 'Longitude' and 'Latitude' columns for starting point"
                print(error_msg)
                with open(log_file, 'a') as log:
                    log.write(f"ERROR: {error_msg}\n")
            
    except Exception as e:
        error_msg = f"An error occurred: {str(e)}"
        print(error_msg)
        with open(log_file, 'a') as log:
            log.write(f"CRITICAL ERROR: {error_msg}\n")
    
    finally:
        # Clean up
        if 'cursor' in locals():
            del cursor
    
    return output_fc

if __name__ == "__main__":
    # Settings - update these with your actual paths
    workspace_dir = "C:/GIS/Workspace"
    input_csv_file = "C:/path/to/your/parsed_data.csv"
    
    # Run the tool
    result = create_traverse_points(
        input_csv=input_csv_file,
        output_folder=workspace_dir,
        output_name="traverse_output.shp"
    )
    
    if os.path.exists(result):
        print(f"Traverse file created successfully at: {result}")
    else:
        print("Failed to create traverse file. Check the log for details.")
