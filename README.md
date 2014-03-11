### Real-time Streaming of ElectricImp Sensor Data to Plot.ly
##### Make HTTP posts with the Electric Imp and Visualize your sensor Data
[![Plotly-imp](readme_images/imp.jpg)](https://electricimp.com)

You'll need to have an account with Plot.ly to begin. 

You can sign up at https://plot.ly/

Make sure to also take note of your API key.

If you don't have one, you can find it at https://plot.ly/settings
Make sure to grab a stream token while you're in your settings pane!

##### Next, load up the agent.nut and device.nut files in the Electric Imp IDE.

This example features one sensor reading, but can easily be used to read as many sensors as you'd like. 
  
Once you've set up your Imp Device (device.nut) you can send some sensor data to the Imp Agent (agent.nut)

Example:
```c
// Initialize your Plotly stream tokens
local tokens = {
    sensortoken = "your_stream_token"
}

// Initialize Plotly Graph when the Device sends init message
// Here we are setting up the graph shell to prepare to
// stream data to it
device.on("init" function(msg) {
    // Init Plotly Data Object (No sensor data here)
    local data = [{
        x = [],
        y = [],
        stream = {
            token = tokens.sensortoken,
            maxpoints = 25
        }
    }];

    // Plotly Layout Object
    local layout = {
        fileopt = "extend",
        filename = "Your Clever Title"

    };

    // format object to be POSTed
    local payload = {
    un = "your_username",
    key = "your_apikey",
    origin = "plot",
    platform = "electricimp",
    args = http.jsonencode(data),
    kwargs = http.jsonencode(layout),
    version = "0.0.1"
    };

    // encode data and log
    local url = "https://plot.ly/clientresp";
    local headers = { "Content-Type" : "application/json" };
    local body = http.urlencode(payload);
    HttpPostWrapper(url, headers, body, true);

    // when a response is received and the graph is ready to
    // accept a stream, send the device a init:success message!
    device.send("initsuccess", "success");
});


device.on("new_reading", function(sensordata) {
    local headers = {"plotly-streamtoken" : tokens.sensortoken };
    local body = {
        x = sensordata.time_stamp,
        y = sensordata.sensor_reading
    }
    local data = http.jsonencode(body);

    // your middleware stream server
    // you can also roll your own with the provided server.js script
    // and simply change this URL to your server's URL
    local url  = "http://54.201.244.104:9999/"
    // Post data to streamserver
    HttpPostWrapper(url, headers, data, false);
});

// Http Request Handler
function HttpPostWrapper (url, headers, string, log) {
    local request = http.post(url, headers, string);
    local response = request.sendsync();
    if (log)
        server.log(http.jsonencode(response));
    return response;
}
```


