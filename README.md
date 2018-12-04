# PolyKids
This repo will just be a document explaining my work. I will not be publishing the project as Otago Polytechnic owns it.

The behavior of any Django Application (or web frameworks, in general) can be broken down as follows:

1.The user visits a URL, such as /amazon/games
2.This triggers a request to that URL, which is looked up in our URL dispatcher
3.The URL dispatcher will map this URL to a function within our Views
4.The corresponding View function will then access models, perform queries or fetch data as needed, which is bundled up and passed into a template
5.The data passed into the template is then displayed to the user using HTML

By following this flow, it becomes easier to reason about our applications even when they begin to grow larger and larger.

"The core structure of any web framework can be broken into three parts: the Models, Views, and Controllers. This pattern allows developers to have a separation of concerns when building an application, and allows us to impose a logical structure to our projects. Let’s go over these parts in more detail."

Within a web framework, Models are your data layer. At any given time, your models will represent what your objects in your database will look like. For example, take a look at the following Django model shown below:

```
from django.db import models

class Student(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
    age = models.IntegerField()
```

"A view is typically a visual representation of our underlying data layer (models). Views can update models as well as retrieve data from them through a query, which in turn would be passed to an HTML template."

"In Django, controllers are generally described as the underlying mechanisms of the framework itself which sends web requests to the appropriate view in combination with the URL dispatcher. In a sense, Django is more of a “Model, Template, View” framework as there are no explicit “Controllers”."

JSON

JavaScript Object Notation (JSON) is a way to store information in a lightweight, organized structure, containing a series of key/value pairs which can easily be parsed (extracted).

Breaking down each of these files:

1.manage.py: A command-line program allowing users to interact with their Django applications.

2.init.py: An empty file signifying that this directory is considered a Python package.

3.settings.py: A file consisting of key-value pairs for configuring your Django application. You can configure your databases, setup paths to static files, and much more.

4.urls.py: Allows us to map view functions to URLs, which is essentially a table of contents for our application. You can read much more about it here.

5.wsgi.py: Allows us to deploy WSGI-compatible web servers, which you can read about more here.

##How to run?

Run the command 'python manage.py runserver' in the same directory as manage.py

# Views.py

## List of imported Libraries

Django shortcuts help send render requests through views.

```
from django.shortcuts import render, HttpResponse
from django.shortcuts import render, redirect
```

Authenication Library for login form.

```
from django.contrib import auth
```

Requests setup to pull & move JSON requests

```
import requests
import json
import urllib.request
```

Libraries to build mongoengine

```
from mongoengine import *
from pymongo import MongoClient
```

Libraries to turn sent strings into JSON packets

```
from bson import json_util
```

Libraries to build and run API view with MQTTs

```
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework.views import APIView
```

Paho library handles and sends MQTT packets.

```
import paho.mqtt.client as mqtt
from django.http import JsonResponse
```

Libraries to handle computation

```
import datetime
import base64
import numpy as np
```

## Simple Pymongo Connect

Function that connects to the mongo client

```
Pymongo
def PymongoConnect():
	client = MongoClient('10.118.26.11', 27017)
	db = client.duniot_database
	db.authenticate('opiot', 'P@ssw0rd')
	mqtt_collection = db.node_data
	print("Connected")
	all_data = json_util.loads(json_util.dumps(mqtt_collection.find()))
	print(all_data)
```

## Better Pymongo Connect

The same connect but splits up nodes into separate arrays to be processed.

```
def PymongoConnectTwo():
    USER = 'opiot'
    PASS = 'P@ssw0rd'
    IP = '10.118.26.11'
    PORT = 27017

    user = urllib.parse.quote_plus(USER)
    passw = urllib.parse.quote_plus(PASS)
    client = MongoClient('mongodb://%s:%s@%s:%s' % (user, passw, IP, PORT))
    print("connected")
    db = client.duniot_database     # connect to the duniot_database
    mqtt_collection = db.node_data   # connect to the node_data collection

    relevant_node_names = ["Gate1", "Gate2", "NoiseSenor1", "NioseSensor2", "room-sensor-D207", "room-sensor-D206b", "senor2", "sensor1", "sensor3"]
    relevant_nodes = {node_name:get_node_data(node_name, mqtt_collection) for node_name in relevant_node_names}
    return (relevant_nodes)	
```

Old leftover code from the Function above code saves into only one array and is not dynamic but could still be used if data processing is changed up.

```
    #print(mqtt_collection)
    #print(db.node_data.find({"deviceName":"roomSensorOTAA"}))
    #node_data = json_util.loads(json_util.dumps(db.node_data.find("deviceName":"roomSensorOTAA")))
    #print(node_data)

    #data = json_util.loads(json_util.dumps(db.node_data.find({"deviceName":"senor2"}, {"deviceName", "applicationID", "dataEntries", "applicationName"})))
    #slicedData = json_util.loads(json_util.dumps(db.node_data.find({"deviceName":"sensor1"}, {"deviceName", "applicationID", "dataEntries", "applicationName"})))
    #data = json.loads(json.dumps(db.node_data.find({"deviceName":"roomSensorOTAA"}, {"deviceName", "applicationID", "dataEntries", "applicationName"})))
    #print(data[0]["dataEntries"][0])
    #print(data[0]["applicationName"])
    # print("dataEntries")
    #return(data)
```

## Base64 to String

converts base64 encoded string to bytes then to string

```

def base64_to_str(data):
    try:
        decoded = base64.b64decode(data).decode('utf-8')
    except UnicodeDecodeError:
        decoded = "Payload could not be decoded"
    return decoded
	
```

## MQTTs Requests

Uses requests to run function, once run it connects to the MQTTs broker then subscribes to the message chain. Once a message is received it decodes the payload and turns it to JSON.

```

@api_view(['GET', 'POST'])
def paho(request):
   #broker = "iot.op-bit.nz"
   broker = "10.25.100.49"
   print("APIVIEWWORKING")
   if request.method =='GET':
        def on_message(client, userdata, message):
           print("received message =", str(message.payload.decode("utf-8")))
           payloadData = message.payload.decode("utf-8")
           jsonPayloadData = json_util.loads(json_util.dumps(payloadData))
           print("AAAAAAAAAAAAAAAAAAA" + jsonPayloadData["data"])
        def on_connect(client, userdata, flags, rc):
           print("Connected with result code" + str(rc))

        client = mqtt.Client()
        client.on_connect = on_connect
        client.on_message = on_message
        print("connecting to broker ", broker)
        client.connect(broker, 1883, 60)  # connect
        client.subscribe("#", 0)
        #client.username_pw_set("loraroot", "P@ssw0rd")
        client.loop_forever()
       # client.loop_start()  # start loop to process received messages
       #
       # client.disconnect()  # disconnect
       # client.loop_stop()  # stop loop

        return render(request, data)
```

## Routes

Plain on render routes for testing

```

def index(request):
    return HttpResponse('Hello World!')

def test(request):
	return render(request, 'polyapp/test.html')

def home(request):
	return render(request, 'polyapp/home.html')
	
```

## Data Processing Page

Saves and passes data into splitData, then passed back split into groups of 360 to create 24 hour sections. The data is then saved and passed to the template.

```
def data(request):
    sensor1 = PymongoConnectTwo()["sensor1"]
    sensor2 = PymongoConnectTwo()["senor2"]
    sensor3 = PymongoConnectTwo()["sensor3"]
    cleanedData = []
    senor1DataPerHour = []
    senor2DataPerHour = []
    senor3DataPerHour = []

    splitData(sensor1, senor1DataPerHour)
    splitData(sensor3, senor3DataPerHour)

    for i in sensor2['dataEntries']:
        nodeData = base64.b64decode(i['data']).decode('utf-8')
        cutNodeData = nodeData[20:24]
        cleanedData.append(float(cutNodeData))
    cutCleanedData = list(chunks(cleanedData, 360))[-24:]
    for j in cutCleanedData:
        senor2DataPerHour.append(round(np.mean(j), 2))
    sortedData = {
       'data': data,
       'senor1DataPerHour':senor1DataPerHour,
       'senor2DataPerHour':senor2DataPerHour,
       'senor3DataPerHour':senor3DataPerHour,
    }
    return render(request, 'polyapp/data.html', sortedData)
```

# URLs

The URLs map functions to URLs.

```
urlpatterns = [
    path('', views.index, name='index'),
    path('test/', views.test, name='test'),
    path('home/', views.home, name='home'),
    path('data/', views.data, name='data'),
    path('overview/', views.overview, name='overview'),
    path('blob/', views.blob, name='blob'),
    path('paho/', views.paho, name='paho')
]
```

#Dashindex

This file run the chart on the data page. 

Datasets loads each line in the graph, change the 'data:' field variable (eg senor1DataPerHour)

```
  datasets: [
    {
      label: "Front",
      fillColor: "rgba(247, 80, 90, 0.0)",
      strokeColor: "#F7505A",
      pointColor: "#F7505A",
      pointStrokeColor: "rgba(0,0,0,0.2)",
      pointHighlightStroke: "rgba(225,225,225,0.75)",
      data: senor1DataPerHour
    },
    {
      label: "Middle",
      fillColor: "rgba(255, 172, 100, 0.0)",
      strokeColor: "rgba(255, 172, 100, 1)",
      pointColor: "rgba(255, 172, 100, 1)",
      pointStrokeColor: "rgba(0,0,0,0.2)",
      pointHighlightStroke: "rgba(225,225,225,0.75)",
      data: senor2DataPerHour
    },
    {
      label: "Back",
      fillColor: "rgba(19, 71, 34, 0.0)",
      strokeColor: "rgba(88, 188, 116, 1)",
      pointColor: "rgba(88, 188, 116, 1)",
      pointStrokeColor: "rgba(0,0,0,0.2)",
      pointHighlightStroke: "rgba(225,225,225,0.75)",
      data: senor3DataPerHour
    }
```

base chart settings

```
var ctx = document.getElementById("salesData").getContext("2d");
window.myLineChart = new Chart(ctx).Line(salesData, {
  pointDotRadius : 6,
  pointDotStrokeWidth : 2,
  datasetStrokeWidth : 3,
  scaleShowVerticalLines: false,
  scaleGridLineWidth : 2,
  scaleShowGridLines : true,
  scaleGridLineColor : "rgba(225, 255, 255, 0.015)",
  scaleOverride: true,
  scaleSteps: 1,
  scaleStepWidth: 10,
  scaleStartValue: 0,

  responsive: true

})
```

#MQTT reading (PolyDecerypt)

Waits for the document to load then starts the MQTT process

```
$(document).ready(function () {
    client.connect(options);
});
```

Options to change connection & messages you will be 'subscribed' to / what messages you want to receive 

```
//Connect Options
var options = {
    timeout: 3,
    //Gets Called if the connection has sucessfully been established
    onSuccess: function () {
        client.subscribe('#', { qos: 2 });
    },
    //Gets Called if the connection could not be established
    onFailure: function (message) {
        alert("Connection failed: " + message.errorMessage);
    }
};
```

Once a message has been received convert the 'payload' into a json object then extract the values needed and passing them into a variable that will display frontend.

```
client.onMessageArrived = function (message) {
    //Do something with the push message you received
    console.log("Got a message, it arrived!");
    jsonData = message.payloadString
    jsonData = jsonData.substring(jsonData.indexOf("{"));
    jsonData = JSON.parse(jsonData)
    console.log(jsonData)
    var dataVal = jsonData["data"]
    var dataName = jsonData["deviceName"]
    if (dataName == "sensor1") {
        dataVal = b64DecodeUnicode(dataVal)
        var slicedString = dataVal.slice(20, 24);
        console.log("Sliced String:" + slicedString)
        senor1Data = slicedString;
    }
    if (dataName == "senor2") {
        dataVal = b64DecodeUnicode(dataVal)
        var slicedString = dataVal.slice(20, 24);
        console.log("Sliced String:" + slicedString)
        senor2Data = slicedString;
    }
    if (dataName == "sensor3") {
        dataVal = b64DecodeUnicode(dataVal)
        var slicedString = dataVal.slice(20, 24);
        console.log("Sliced String:" + slicedString)
        senor3Data = slicedString;
    }
    console.log(dataVal)
}
```

Function used in MessageArrived to decode the payload string has it comes in encrypted b64

```
function b64DecodeUnicode(str) {
    try {
        // Going backwards: from bytestream, to percent-encoding, to original string.
        return decodeURIComponent(atob(str).split('').map(function (c) {
            return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2);
        }).join(''));
    }
    catch (err) {
        alert(err.message);
    }
}
```

# FusionCharts (FusionChartOne.js etc)

Builds the chart with set options, main changes are to the lower and upper limit, where to render the chart with 'renderAt' and the initial value with 'value: "1"'

```
FusionCharts.ready(function () {
      var fusioncharts = new FusionCharts({
        type: 'angulargauge',
        renderAt: 'chart-container',
        width: '100%',
        height: '300',
        dataFormat: 'json',
        dataSource: {
          "chart": {
            "caption": "Noise Sensor One",
            "subcaption": "Current Reading",
            "lowerLimit": "0",
            "upperLimit": "3",
            "theme": "fusion"
          },
          "colorRange": {
            "color": [{
              "minValue": "0",
              "maxValue": "1",
              "code": "#6baa01"
            }, {
              "minValue": "1",
              "maxValue": "2",
              "code": "#f8bd19"
            }, {
              "minValue": "2",
              "maxValue": "3",
              "code": "#e44a00"
            }]
          },
          "dials": {
            "dial": [{
              "value": "1"
            }]
          }

        },
		);
      fusioncharts.render();
    });
```

This event runs updates the front end graphics on the chart. '400' is where the interval is set lower it to speed up how many times the function runs '("value=" + senor1Data)' is where you feed the data value you want to update

```
        "events": {
          "initialized": function (evtObj, argObj) {
            evtObj.sender.changeInterval = setInterval(function () {
              evtObj.sender.feedData && evtObj.sender.feedData("value=" + senor1Data);
            }, 400);
          },
          "disposed": function (evtObj, argObj) {
            clearInterval(evtObj.sender.resetInterval);
            clearInterval(evtObj.sender.changeInterval);
          }
        }
      },
```

# Blob graphics (Home page)

maths to help the blobs move

```
	var radius = 8;
	TweenMax.staggerFromTo('.blob', 4 ,{
		cycle: {
			attr:function(i) {
				var r = i*90;
				return {
					transform:'rotate('+r+') translate('+radius+',0.1) rotate('+(-r)+')'
				}      
			}
		}  
	},{
		cycle: {
			attr:function(i) {
				var r = i*90+360;
				return {
					transform:'rotate('+r+') translate('+radius+',0.1) rotate('+(-r)+')'
				}      
			}
		},
		ease:Linear.easeNone,
		repeat:-1
	});
```

## Blob.css

This is the code you wanna change if you want to change the colour

```
.loading_cont {
 width: 100%;
 height: 100%;
 background: red;
 background: -webkit-linear-gradient(left top, #152a8e, #b1376c);
 background: -o-linear-gradient(bottom right, #152a8e, #b1376c);
 background: -moz-linear-gradient(bottom right, #152a8e, #b1376c);
 background: linear-gradient(to bottom right, #152a8e, #b1376c);
}
```

