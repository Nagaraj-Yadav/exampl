# exampl
# Ingest IoT data into ADLS

This project stores the streaming IoT data into ADLS gen2 storage account written in Python. The data would be stored in files of our desired format. Data from multiple IoT devices would be processed parallelly and stored in their respective destinations(ADLS) as per the given conventions.

# Getting Started


## Download

```
pip install azure-iot-device
pip install azure-storage-file-datalake
```
## Requirements

- An Azure Data Lake Storage Gen2 account

- Get the following ADLS account credentials and replace them accordingly in their respective fields in `configurations.py` file. 

```
configurations.py

config['Azure_Credentials'] = {
    'SAConnectionString' : '<Connection string>',
    'AzureSAName' : '<Storage account name>',
    'AzureSAAccessKey' : '<Key>', 
    }
```

- Add your preferred IoT devices in the `Devices.py` file.

  - > Strictly one functon per device.
  - If any authorization credentials are applicable to the device, add them in `configurations.py` file as shown below
  ```
  configurations.py

  config['<function name>'] = {
    'key' : '<value>',
    'key' : '<value>',
    }
  ``` 
  \<function name\>   - Name of the function that you give to that particular device.\
  \<keys and values\> - Credentials and fields related to that device.

```
Devices.py

def <function name>():
    outd = []
    directory = inspect.stack()[0].function
    file = directory+"-data"
    

    while True:
        
        --Add your logic to catch the IoT device data into outd array here--

        global a
        if a == directory:
            global b
            b += 1
            saving_data_in_storage(outd, directory, file)
            print(f"Device {directory} exitted successfully.")
            sys.exit()
        time.sleep(10)
```
>
> **NOTE** : No duplicate function names allowed.
>

Suppose, I have to add a simple Temperature-Humidity simulator, the structure would be, 

```
Devices.py

def run_tempHumidity_telemetry_sample():
    outd = []
    directory = inspect.stack()[0].function
    file = directory+"-data"
    
    TEMPERATURE = 20.0
    HUMIDITY = 60
    while True:
        temperature = TEMPERATURE + (random.random() * 15)
        humidity = HUMIDITY + (random.random() * 20)
        MSG_TXT = {
            "temperature" : temperature,
            "humidity" : humidity}
        
        message = Message(str(MSG_TXT), content_encoding='utf-8', content_type='application/json')

        if temperature > 30:
            message.custom_properties["temperatureAlert"] = "true"
        else:
            message.custom_properties["temperatureAlert"] = "false"
        
        if message.custom_properties:
            MSG_TXT.update(message.custom_properties)

        outd.append(str(MSG_TXT))        
        
        print("Sending message: {}".format(MSG_TXT))
        
        global a
        if a == directory:
            global b
            b += 1
            saving_data_in_storage(outd, directory, file)
            print(f"Device {directory} exitted successfully.")
            sys.exit()
        time.sleep(10)
```

## Design

Using Azure SDKs and storage account credentials, we interact with the storage

![](Images/Screenshot3.png)

## Running

Once done with all the prereqisites, run the `main.py` file.

- A list of existing containers in the provided storage account would be shown first and then, You would be prompted to enter the name of the container that you wish to consider to store the data. You can enter the name of an existing container or a new one too.

```
Existing containers in the selected storage account are listed below.
container-1
container-2

Enter the name of the container of your choice to work on (Existing/New) :
```

- Later, you would be shown the list of available devices in the project and then would be  
prompted to select the devices you would like to run.    

```
Enter the respective id of the devices you would like to run as space separated integers

Available Devices :
id | Devices
-----------------
1  |  Device-1
2  |  Device-2
3  |  Device-3

Here : 
```

`Here : 1 2 3` - To run the devices 1,2,3

- Once you select the devices, catching your device's data starts and you will be parallely be prompted as, which asks your respective device name(or function name of that particular device) to exit.

`Enter the name of the Device and hit 'Enter' to exit : `

