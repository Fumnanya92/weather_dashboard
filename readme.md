### **Weather Dashboard Project **  

#### **1. Project Overview**  
This project fetches weather data from the **OpenWeather API** and stores it in an **AWS S3 bucket** for further analysis and visualization.

---

## **2. Prerequisites**  

### **2.1 Required Installations**  
Ensure you have the following installed on your machine:  

- **Python (3.x)** – [Download Here](https://www.python.org/downloads/)  
- **pip** (Comes with Python installation)  
- **AWS CLI** – [Install AWS CLI](https://aws.amazon.com/cli/)  
- **Git** – [Install Git](https://git-scm.com/)  
- **Visual Studio Code (VS Code)** – Recommended for writing and running Python scripts  

---

## **3. Project Setup**  

### **3.1 Create the Project Directory**  

Open your terminal and run:  

```bash
mkdir weather_dashboard
cd weather_dashboard
```

---

### **3.2 Setup Virtual Environment**  

Create and activate a virtual environment:  

```bash
python -m venv venv  
source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
```

---

### **3.3 Install Required Dependencies**  

```bash
pip install requests boto3 python-dotenv
```

---

### **3.4 Setup Environment Variables**  

Create a `.env` file in your project directory:  

```bash
touch .env
```

Edit the `.env` file and add the following credentials:  

```ini
OPENWEATHER_API_KEY=your_openweather_api_key
AWS_BUCKET_NAME=your_s3_bucket_name
```

Ensure `.env` is added to `.gitignore` to prevent accidental commits.

---

## **4. Implementing the Weather Fetching Script**  

### **4.1 Create a Python Script**  

Inside your project directory, create a folder named `src` and add the script file:  

```bash
mkdir src
cd src
touch weather_dashboard.py
```

Open `weather_dashboard.py` and add the following code:

```python
import os
import json
import boto3
import requests
from datetime import datetime
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

class WeatherDashboard:
    def __init__(self):
        self.api_key = os.getenv('OPENWEATHER_API_KEY')
        self.bucket_name = os.getenv('AWS_BUCKET_NAME')
        self.s3_client = boto3.client('s3')

    def create_bucket_if_not_exists(self):
        """Create S3 bucket if it doesn't exist"""
        try:
            self.s3_client.head_bucket(Bucket=self.bucket_name)
            print(f"Bucket {self.bucket_name} exists")
        except:
            print(f"Creating bucket {self.bucket_name}")
        try:
            self.s3_client.create_bucket(Bucket=self.bucket_name)
            print(f"Successfully created bucket {self.bucket_name}")
        except Exception as e:
            print(f"Error creating bucket: {e}")

    def fetch_weather(self, city):
        """Fetch weather data from OpenWeather API"""
        base_url = "http://api.openweathermap.org/data/2.5/weather"
        params = {
            "q": city,
            "appid": self.api_key,
            "units": "imperial"
        }
        
        try:
            response = requests.get(base_url, params=params)
            response.raise_for_status()
            return response.json()
        except requests.exceptions.RequestException as e:
            print(f"Error fetching weather data: {e}")
            return None

    def save_to_s3(self, weather_data, city):
        """Save weather data to S3 bucket"""
        if not weather_data:
            return False
            
        timestamp = datetime.now().strftime('%Y%m%d-%H%M%S')
        file_name = f"weather-data/{city}-{timestamp}.json"
        
        try:
            weather_data['timestamp'] = timestamp
            self.s3_client.put_object(
                Bucket=self.bucket_name,
                Key=file_name,
                Body=json.dumps(weather_data),
                ContentType='application/json'
            )
            print(f"Successfully saved data for {city} to S3")
            return True
        except Exception as e:
            print(f"Error saving to S3: {e}")
            return False

def main():
    dashboard = WeatherDashboard()
    
    # Create bucket if needed
    dashboard.create_bucket_if_not_exists()
    
    cities = ["Philadelphia", "Seattle", "New York"]
    
    for city in cities:
        print(f"\nFetching weather for {city}...")
        weather_data = dashboard.fetch_weather(city)
        if weather_data:
            temp = weather_data['main']['temp']
            feels_like = weather_data['main']['feels_like']
            humidity = weather_data['main']['humidity']
            description = weather_data['weather'][0]['description']
            
            print(f"Temperature: {temp}°F")
            print(f"Feels like: {feels_like}°F")
            print(f"Humidity: {humidity}%")
            print(f"Conditions: {description}")
            
            # Save to S3
            success = dashboard.save_to_s3(weather_data, city)
            if success:
                print(f"Weather data for {city} saved to S3!")
        else:
            print(f"Failed to fetch weather data for {city}")

if __name__ == "__main__":
    main()
```

---

## **5. Running the Script**  

### **5.1 Activate the Virtual Environment**  

```bash
source venv/bin/activate  # Windows: venv\Scripts\activate
```

### **5.2 Run the Script**  

```bash
python src/weather_dashboard.py
```

### **5.3 Expected Output**  

If successful, it should prompt the user for city then it should display:

```
Fetching weather for New York...
Temperature: 60.49°F
Feels like: 57.7°F
Humidity: 31%
Conditions: scattered clouds
Successfully saved data for New York to S3!
```

---

## **6. Verify Data in AWS S3**  

1. Go to [AWS S3 Console](https://s3.console.aws.amazon.com/s3/home).  
2. Open the bucket named as per your `.env` configuration.  
3. Inside the `weather-data/` folder, you should see `.json` files corresponding to the cities.  
4. Download and inspect the JSON file to ensure it contains correct weather data.  

---

## **7. Automating the Script Execution**  

To schedule the script execution, you can use **cron jobs** (Linux/macOS) or **Task Scheduler** (Windows).  

### **7.1 Linux/macOS (Cron Job)**
Open the crontab editor:

```bash
crontab -e
```

Add the following line to run the script every hour:

```
0 * * * * /path/to/venv/bin/python /path/to/weather_dashboard/src/weather_dashboard.py
```

### **7.2 Windows (Task Scheduler)**
1. Open **Task Scheduler**  
2. Create a new task  
3. Set the action to run the script:  

   ```
   C:\path\to\venv\Scripts\python.exe C:\path\to\weather_dashboard\src\weather_dashboard.py
   ```

4. Set the trigger to run every hour  

