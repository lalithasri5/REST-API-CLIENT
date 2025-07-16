# REST-API-CLIENT
// Import necessary Java libraries for HTTP client and JSON parsing
import java.io.IOException;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import org.json.JSONObject; // We'll use the org.json library for JSON parsing

/**
 * WeatherFetcherApp is a Java application that consumes the OpenWeatherMap REST API
 * to fetch weather data for a specified city and displays it in a structured format.
 *
 * To run this application:
 * 1. Ensure you have the 'org.json' library in your classpath.
 * You can download the JAR from: https://mvnrepository.com/artifact/org.json/json/20240303
 * (Look for the 'jar' link under the 'Files' section on that page, or use the direct link:
 * https://download.dcache.org/nexus/service/rest/repository/browse/public/org/json/json/20240303/json-20240303.jar)
 * 2. Save the downloaded 'json-20240303.jar' file to a known location, e.g., C:\libs\ or /home/user/libs/.
 * 3. Compile:
 * Windows: javac -cp ".;C:\path\to\your\json-20240303.jar" WeatherFetcherApp.java
 * Linux/macOS: javac -cp ".:/path/to/your/json-20240303.jar" WeatherFetcherApp.java
 * (Replace 'C:\path\to\your\' or '/path/to/your/' with the actual directory where you saved the JAR.)
 * 4. Run:
 * Windows: java -cp ".;C:\path\to\your\json-20240303.jar" WeatherFetcherApp
 * Linux/macOS: java -cp ".:/path/to/your/json-20240303.jar" WeatherFetcherApp
 *
 * Replace "YOUR_API_KEY" with your actual OpenWeatherMap API key.
 */
public class WeatherFetcherApp {

    // IMPORTANT: Replace "" with your actual OpenWeatherMap API key.
    // You can get one for free from: https://openweathermap.org/api
    private static final String API_KEY = "a34e186311dfb6f745af453fae605666";
    private static final String BASE_URL = "https://api.openweathermap.org/data/2.5/weather";

    public static void main(String[] args) {
        // Define the city for which to fetch weather data
        String city = "London"; // You can change this to any city you want

        System.out.println("Fetching weather data for: " + city);

        try {
            // Construct the API URL with the city and API key
            String apiUrl = String.format("%s?q=%s&appid=%s&units=metric", BASE_URL, city, API_KEY);

            // Create an HttpClient instance
            HttpClient client = HttpClient.newHttpClient();

            // Build the HttpRequest
            HttpRequest request = HttpRequest.newBuilder()
                    .uri(URI.create(apiUrl))
                    .build();

            // Send the request and get the HttpResponse
            // HttpResponse.BodyHandlers.ofString() specifies that the response body should be treated as a String
            HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

            // Check if the request was successful (HTTP status code 200)
            if (response.statusCode() == 200) {
                // Parse the JSON response body
                JSONObject jsonResponse = new JSONObject(response.body());

                // Extract and display the weather information
                displayWeather(jsonResponse, city);
            } else {
                // Handle API errors (e.g., invalid city, invalid API key)
                System.err.println("Error fetching weather data. HTTP Status Code: " + response.statusCode());
                System.err.println("Response Body: " + response.body());
                // Attempt to parse error message if available
                try {
                    JSONObject errorJson = new JSONObject(response.body());
                    if (errorJson.has("message")) {
                        System.err.println("API Error Message: " + errorJson.getString("message"));
                    }
                } catch (Exception e) {
                    System.err.println("Could not parse error response body.");
                }
            }

        } catch (IOException | InterruptedException e) {
            // Handle network-related errors or interruptions
            System.err.println("An error occurred during the HTTP request: " + e.getMessage());
            e.printStackTrace();
        } catch (org.json.JSONException e) {
            // Handle JSON parsing errors
            System.err.println("Error parsing JSON response: " + e.getMessage());
            e.printStackTrace();
        } catch (Exception e) {
            // Catch any other unexpected errors
            System.err.println("An unexpected error occurred: " + e.getMessage());
            e.printStackTrace();
        }
    }

    /**
     * Parses the JSON response and prints the weather details in a structured format.
     *
     * @param jsonResponse The JSONObject containing the weather data.
     * @param city The name of the city for which weather data was fetched.
     */
    private static void displayWeather(JSONObject jsonResponse, String city) {
        System.out.println("\n--- Weather Report for " + city + " ---");

        // Extract main weather information
        if (jsonResponse.has("main")) {
            JSONObject main = jsonResponse.getJSONObject("main");
            System.out.println("Temperature: " + main.getDouble("temp") + " °C");
            System.out.println("Feels Like:  " + main.getDouble("feels_like") + " °C");
            System.out.println("Min Temp:    " + main.getDouble("temp_min") + " °C");
            System.out.println("Max Temp:    " + main.getDouble("temp_max") + " °C");
            System.out.println("Humidity:    " + main.getInt("humidity") + " %");
            System.out.println("Pressure:    " + main.getInt("pressure") + " hPa");
        } else {
            System.out.println("Main weather data not found in response.");
        }

        // Extract weather description
        if (jsonResponse.has("weather") && jsonResponse.getJSONArray("weather").length() > 0) {
            JSONObject weather = jsonResponse.getJSONArray("weather").getJSONObject(0);
            System.out.println("Description: " + weather.getString("description"));
            System.out.println("Main:        " + weather.getString("main"));
        } else {
            System.out.println("Weather description not found in response.");
        }

        // Extract wind information
        if (jsonResponse.has("wind")) {
            JSONObject wind = jsonResponse.getJSONObject("wind");
            System.out.println("Wind Speed:  " + wind.getDouble("speed") + " m/s");
            if (wind.has("deg")) {
                System.out.println("Wind Degree: " + wind.getInt("deg") + "°");
            }
        } else {
            System.out.println("Wind data not found in response.");
        }

        // Extract clouds information
        if (jsonResponse.has("clouds")) {
            JSONObject clouds = jsonResponse.getJSONObject("clouds");
            System.out.println("Cloudiness:  " + clouds.getInt("all") + " %");
        } else {
            System.out.println("Cloud data not found in response.");
        }

        // Extract visibility
        if (jsonResponse.has("visibility")) {
            System.out.println("Visibility:  " + jsonResponse.getInt("visibility") + " meters");
        }

        System.out.println("----------------------------");
    }
}
