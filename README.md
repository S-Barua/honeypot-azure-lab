# honeypot-azure-lab
A simple Windows honeypot setup using PowerShell and Azure Log Analytics to capture and analyze failed RDP login attempts. This project is designed for educational purposes, providing hands-on experience in setting up a basic honeypot to monitor and study potential security threats.
# Honeypot Azure Log Analytics Lab

## Overview

This project demonstrates a simple Windows honeypot setup using PowerShell and Azure Log Analytics. It captures failed RDP login attempts, retrieves geolocation data for the source IP addresses, and visualizes the data in Azure Log Analytics. This is a basic setup for educational purposes and should not be used as a replacement for real security measures.

## Prerequisites

*   **Azure Subscription:**  You need an active Azure subscription.
*   **Log Analytics Workspace:** Create a Log Analytics workspace in your Azure subscription. Note the Workspace ID and Primary Key.
*   **Windows Virtual Machine:**  Create a Windows VM that will act as the honeypot.
*   **ipgeolocation.io API Key:**  Sign up for a free API key at [https://ipgeolocation.io/](https://ipgeolocation.io/).  The free tier is sufficient for testing.

## Setup Instructions

1.  **Create a Windows VM:** Create a Windows Virtual Machine.
2.  **Configure Log Analytics Workspace:**
    *  Navigate to your Log Analytics workspace in the Azure portal.
    *  Go to `Agents configuration` under `Settings` and find the `Workspace ID` and the `Primary Key`.
      
3.  **PowerShell Script:**
    *   Download the ['(https://drive.google.com/file/d/1i-JyOlP98sN-FJyVfl_uAa_laaCkKPbi/view?usp=sharing)'] (./powershell/failed_rdp_logger.ps1) script.
    *   Edit the script and replace `"YOUR_IPGEO_API_KEY_HERE"` with your actual ipgeolocation.io API key.
    *   Save the script.
      
4.  **Run the PowerShell Script:**
    *   On your Windows VM, open PowerShell as an administrator.
    *   Navigate to the directory where you saved the `failed_rdp_logger.ps1` file.
    *   Run the script: .\failed_rdp_logger.ps1'
    *   The script will start monitoring the Windows Event Viewer for failed RDP login attempts.  It will retrieve the geolocation data for each failed attempt and write the data to `C:\ProgramData\failed_rdp.log`.
      
5.  **Data Collection Rule Configuration**
    * Go to your Log Analytics workspace. Then navigate to `Data collection rules` and click on `Create data collection rule` button.
    * Give your rule a `Name`.
    * On the `Resources` tab, click on `Add resources` and select the virtual machine where the PowerShell script is running. Click `Apply`.
    * Go to the `Collect and transform` tab and click `Add data source`.
    * Select the `Text logs` for `Data source type`.
    * Enter the file path `C:\ProgramData\failed_rdp.log`.
    * Click on `Next`.
    * On the `Destination` tab, select your Log Analytics workspace.
    * Click on `Review + create` then click `Create`.
  
6.  **Configure Log Analytics Data Ingestion:**
    *   In the Azure portal, navigate to your Log Analytics workspace.
    *   Go to `Tables` under the `Settings` menu.
    *   Create a custom log table. (You may need to manually create the custom log table.
      
7.  **Apply Data Ingestion**
    *  If the table is created successfully then wait to receive the logs.
    *  Navigate to `Logs` and then run the KQL query.

## Configuration

*   **API Key:**  Make sure to replace the placeholder in the `failed_rdp_logger.ps1` script with your actual ipgeolocation.io API key.

## Usage

1.  **Failed RDP Attempts:**  Let the honeypot run and wait for failed RDP login attempts to occur.  You can simulate failed attempts to test the setup.
2.  **View Data in Azure Log Analytics:**
    *   In the Azure portal, navigate to your Log Analytics workspace.
    *   Go to "Logs".
    *   Run the following KQL query to view the captured data:

    ```kusto
    FAILED_RDP_WITH_GEO_CL
    | extend latitude = extract("latitude:([0-9.-]+)", 1, RawData),
             longitude = extract("longitude:([0-9.-]+)", 1, RawData),
             destinationhost = extract("destinationhost:([^,]+)", 1, RawData),
             username = extract("username:([^,]+)", 1, RawData),
             sourcehost = extract("sourcehost:([^,]+)", 1, RawData),
             state = extract("state:([^,]+)", 1, RawData),
             country = extract("country:([^,]+)", 1, RawData),
             label = extract("label:([^,]+)", 1, RawData),
             timestamp = extract("timestamp:([^,]+)", 1, RawData)
    | project TimeGenerated, Computer, latitude, longitude, destinationhost, username, sourcehost, state, country, label, timestamp
    ```

## Example Output

<img width="904" alt="Screenshot 2025-03-08 173323" src="https://github.com/user-attachments/assets/6bc60377-83e3-4f78-9112-ba05883f27c8" />
<img width="1280" alt="Screenshot 2025-03-08 173239" src="https://github.com/user-attachments/assets/22f5a4e2-0c33-449e-a947-2534bd6f71c8" />
<img width="1280" alt="Screenshot 2025-03-08 173202" src="https://github.com/user-attachments/assets/9ad9e1dd-6fdf-4a02-9175-f3d4390f9c36" />
<img width="1280" alt="Screenshot 2025-03-08 173050" src="https://github.com/user-attachments/assets/ce17a1a5-a5bd-430d-8ca9-ab98e1bd5bab" />
<img width="233" alt="Screenshot 2025-03-08 173022" src="https://github.com/user-attachments/assets/389d582d-e143-414e-9df3-745110cf1051" />
<img width="1242" alt="Screenshot 2024-12-13 144335" src="https://github.com/user-attachments/assets/de0cf195-44fa-43c7-904c-9592d598744b" />


## Disclaimer

This is a basic honeypot setup for educational and testing purposes. It should not be used as a replacement for proper security measures.

## Credits

*   This project was inspired by the following YouTube video: [https://www.youtube.com/watch?v=k_Z-ty0xeno](https://www.youtube.com/watch?v=k_Z-ty0xeno)
*   Thanks to ipgeolocation.io for providing the geolocation API.
