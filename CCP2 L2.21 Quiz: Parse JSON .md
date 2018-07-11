
Write the function to return the condition.
Given the following JSON, write a function to retrieve the weather "condition".

{
   "temp": {
      "min":"11.34",
      "max":"19.01"
   },
   "weather": {
      "id":"801",
      "condition":"Clouds",
      "description":"few clouds"
   },
   "pressure":"1023.51",
   "humidity":"87"
}


Here is the answer:

String getCondition(String JSONString) {
   JSONObject forecast = new JSONObject(JSONString); <--RL: Randomly chose object/variable name "forecast" for this JSON object.
   JSONObject weather = forecast.getJSONObject("weather"); <--RL: Within overall JSONObject "forecast", get subJSONObj "weather"?
   return weather.getString("condition"); <--RL: Within JSONObj "weather", get String VALUE that corresp to KEY "condition".
}

