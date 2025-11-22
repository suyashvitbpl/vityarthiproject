# vityarthiproject
import tkinter as tk
import requests
from tkinter import messagebox
from tkinter import ttk
from PIL import Image, ImageTk
from tkinter import simpledialog


# IMPORTANT
API_KEY = "bb9df5083ba5dc0d4690631d62c78185"
BASE_URL = "http://api.openweathermap.org/data/2.5/weather?"

def get_city_prompt():
   
    user_input = simpledialog.askstring(title="Weather Application", prompt="Enter the City:")
    return user_input

def Err(message):
    
    #Displays a pop-up GUI error message window.
    
    messagebox.showerror("Application Error", message)




def GUI(temp,humidity,description,wind_speed,city,ID):
    # --- 1. Setup Main Window ---
    root = tk.Tk()
    root.title("Weather Dashboard")
    root.attributes('-fullscreen', True)
    
    #2. Set the TTK Theme & NEW CARD STYLE
    style = ttk.Style()
    style.theme_use('clam')
    CARD_BG = "#F0F0F0" 
    
    # Styles are now for the frame and its children
    style.configure('Card.TFrame', background=CARD_BG)
    style.configure('Card.TLabel', background=CARD_BG)
    style.configure('Card.TSeparator', background=CARD_BG)

    # --- 4. Define Fonts (Unchanged) ---
    FONT_TITLE = ("Segoe UI", 24, "bold")
    FONT_LARGE = ("Segoe UI", 44, "bold")
    FONT_SUBTITLE = ("Segoe UI", 14)
    FONT_DETAIL = ("Segoe UI", 11)

    
    # --- 5. Create Background Label (Unchanged) ---
    background_label = tk.Label(root)
    background_label.place(x=0, y=0, relwidth=1, relheight=1)

    
    # --- CHANGED: NO MORE Toplevel WINDOW ---
    # We create a Frame, parented to root.
    # We add padding inside the frame instead of the window.
    card_frame = ttk.Frame(root, style='Card.TFrame', padding=(30, 30))
    
    # --- CHANGED: Center the frame using .place() ---
    # This is much easier than manual math.
    card_frame.place(relx=0.5, rely=0.5, anchor='center')
    
    # All 'card_window.attributes()' lines are GONE.


    # --- 7. Add Widgets to the NEW card_frame ---
    # All widgets are now parented to 'card_frame'
    
    loc_label = ttk.Label(card_frame, text=str(city), font=FONT_TITLE, style='Card.TLabel')
    loc_label.pack(anchor="w")

    temp_label = ttk.Label(card_frame, text=str(temp)+"°C", font=FONT_LARGE, foreground="#000", style='Card.TLabel')
    temp_label.pack(anchor="w", pady=(10, 0))

    cond_label = ttk.Label(card_frame, text=str(description), font=FONT_SUBTITLE, foreground="#000", style='Card.TLabel')
    cond_label.pack(anchor="w")

    separator = ttk.Separator(card_frame, orient='horizontal')
    separator.pack(fill='x', pady=20)

    # This frame is parented to card_frame
    details_frame = ttk.Frame(card_frame, style='Card.TFrame')
    details_frame.pack(fill='x')

    hum_label_text = ttk.Label(details_frame, text="HUMIDITY", font=FONT_DETAIL, foreground="#000", style='Card.TLabel')
    hum_label_text.grid(row=0, column=0, sticky="w")
    
    # CHANGED: Parented to details_frame to use grid
    hum_label_val = ttk.Label(details_frame, text=str(humidity)+"%", font=FONT_DETAIL, style='Card.TLabel')
    hum_label_val.grid(row=1, column=0, sticky="w")

    wind_label_text = ttk.Label(details_frame, text="WIND", font=FONT_DETAIL, foreground="#000", style='Card.TLabel')
    wind_label_text.grid(row=0, column=1, sticky="w", padx=30)

    # CHANGED: Parented to details_frame to use grid
    wind_label_val = ttk.Label(details_frame, text=str(wind_speed)+"km/h", font=FONT_DETAIL, style='Card.TLabel') # (Fixed unit to km/h)
    wind_label_val.grid(row=1, column=1, sticky="w", padx=30)
    
    weather_image_files = {
    "Sunny": "sunny.jpg",
    "Rainy": "rainy.jpg",
    "Cloudy": "cloudy.jpg",
    "Partly Cloudy": "pcloudy.jpg",
    "Thunderstorm": "Thunderstorm.jpg",
    "Dizzle":"Dizzle",
    "default": "default.jpg"}

    if(ID>=200 and ID<=232):
        new_condition="Thunderstorm"
    elif(ID>=300 and ID<=321):
        new_condition="Dizzle"
    elif(ID>=500 and ID<=531):
        new_condition="Rainy"
    elif(ID>=600 and ID<=622):
        new_condition="Snow"
    elif(ID>=700 and ID<=781):
        new_condition="default"
    elif(ID==800):
        new_condition="Sunny"
    elif(ID==801):
        new_condition="Partly Cloudy"
    elif(ID>=802 and ID<=804):
        new_condition="Cloudy"
    else:
        new_condition="default"

    img_name = weather_image_files.get(new_condition, weather_image_files["default"])
    screen_width = root.winfo_screenwidth()
    screen_height = root.winfo_screenheight()
        
    try:
        img = Image.open(img_name)
        img_resized = img.resize((screen_width, screen_height), Image.LANCZOS)
        photo = ImageTk.PhotoImage(img_resized)
        
        background_label.config(image=photo)
        background_label.image = photo
    except FileNotFoundError:
        print(f"Warning: Image file '{img_name}' not found. Running without background image.")
    except Exception as e:
        print(f"Warning: Could not load image. Error: {e}")
        
    
    
    root.mainloop()




    
def get_weather(city_name):
    """Fetches weather data from OpenWeatherMap API and updates the UI."""
    
    if not city_name:
        Err("Please enter correct zcity ")
        return

    # Complete URL
    complete_url = BASE_URL + "appid=" + API_KEY + "&q=" + city_name+"&units=metric"

    try:
        # Send request
        response = requests.get(complete_url)
        # Raise an exception for bad status codes (4xx or 5xx)
        response.raise_for_status() 
        
        # Parse JSON response
        data = response.json()
        # Check if the city was found (cod 200)
        if data["cod"] == 200:
            main = data["main"]
            weather = data["weather"][0]
            wind = data["wind"]
            sys = data["sys"]

            # Get data
            temp = main["temp"]
            humidity = main["humidity"]
            description = weather["description"].capitalize()
            wind_speed = wind["speed"]
            city = data["name"]
            ID = weather["id"]

            

            # Format the output string
            
            #Call Your GUI meth and pass these method
            GUI(temp,humidity,description,wind_speed,city,ID)

        elif data["cod"] == "404":
            Err("City not found. Please check the spelling.")
        else:
            Err(data['message'])

    except requests.exceptions.HTTPError as http_err:
        if response.status_code == 401:
            Err("Error: Invalid API Key. Please check your API key in the code.")
        elif response.status_code == 404:
            Err("City not found. Please check the spelling.")
        else:
            Err("HTTP error occurred: "+str(http_err))
    except requests.exceptions.ConnectionError:
        Err("Error: Could not connect. Please check your internet connection.")
    except Exception as e:
        Err("An error occurred: " + str(e))


def main():
    City=get_city_prompt()
    get_weather(City)

main()
