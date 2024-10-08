﻿# Markov-Assumption
 import requests
import random

API_KEY = 'your_openweathermap_api_key'
location = 'Sharjah, AE'


def gethistoricalweather(api_key, location):
    url = f"http://api.openweathermap.org/data/2.5/weather?q={location}&appid={api_key}&units=metric"

    try:
        response = requests.get(url)
        response.raise_for_status()  # Raises an exception for 4xx/5xx errors
        data = response.json()


        if 'weather' in data and len(data['weather']) > 0 and 'main' in data:
            current_weather = data['weather'][0]['main']
            temp = data['main']['temp']
            humidity = data['main']['humidity']
            print(f"Current weather: {current_weather}, Temp: {temp}°C, Humidity: {humidity}%")
            return current_weather, temp, humidity
        else:
            print("Weather data not available.")
            return 'Sunny', 30, 40  # Default to Sunny weather with average temp and humidity

    except requests.exceptions.RequestException as e:
        print(f"Error: {e}")
        return 'Sunny', 30, 40  # Default to Sunny if there's an error



transitionm = [
    [0.75, 0.2, 0.05],  # Sunny to Sunny, Cloudy, Rainy (less likely to rain)
    [0.3, 0.6, 0.1],  # Cloudy to Sunny, Cloudy, Rainy (cloudy has moderate rain chance)
    [0.2, 0.3, 0.5]  # Rainy to Sunny, Cloudy, Rainy (rainy days tend to cluster)
]

states = ['Sunny', 'Cloudy', 'Rainy']


def nextstatereal(current_state):
    state_index = states.index(current_state)
    next_state = random.choices(states, transitionm[state_index])[0]
    return next_state


def simulateweatherreal(days, current_state, current_temp, current_humidity):
    print(f"Day 1: {current_state}, Temp: {current_temp}C, Humidity: {current_humidity}%")
    for day in range(2, days + 1):
        current_state = nextstatereal(current_state)

        #  realistic temperature changes based on weather
        if current_state == 'Sunny':
            current_temp += random.uniform(-1, 3) # generates a random number
        elif current_state == 'Cloudy':
            current_temp += random.uniform(-2, 0)
        elif current_state == 'Rainy':
            current_temp += random.uniform(-3, -1)  #

        # Simulate humidity based on weather
        if current_state == 'Sunny':
            current_humidity += random.uniform(-5, 0)
        elif current_state == 'Cloudy':
            current_humidity += random.uniform(0, 5)
        elif current_state == 'Rainy':
            current_humidity += random.uniform(5, 10)

        #  temperature and humidity in reasonable bounds
        current_temp = max(15, min(current_temp, 45))
        current_humidity = max(10, min(current_humidity, 90))

        print(f"Day {day}: {current_state}, Temp: {current_temp:.2f}C, Humidity: {current_humidity:.2f}%")



current_weather, current_temp, current_humidity = gethistoricalweather(API_KEY, location)
days_to_predict = 61
simulateweatherreal(days_to_predict, current_weather, current_temp, current_humidity)

