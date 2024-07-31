#Imports All of the Modules
import tkinter as tk
from tkinter import messagebox, Menu, Text, Scrollbar
from PIL import ImageGrab
import base64
import requests
import os
import subprocess
import pyautogui
from win32 import win32gui
import time
import json
import threading

#Gives Access to the API Key
api_key = "<YOUR_OPENAI_KEY_HERE>"

#Function to get the pathway to the downloads folder
def get_downloads_folder():
    return os.path.join(os.path.expanduser("~"), "Downloads")

#Takes the dimensions of the window and saves screenshots to the downloads folder
def screenshot():
    current_time = time.strftime("%Y%m%d%H%M%S")
    hwnd = win32gui.GetForegroundWindow()
    if hwnd:
        win32gui.SetForegroundWindow(hwnd)
        time.sleep(1)  
        rect = win32gui.GetWindowRect(hwnd)
        im = ImageGrab.grab(bbox=(rect[0], rect[1], rect[2], rect[3]))
        screenshot_path = os.path.join(get_downloads_folder(), f"screenshot_{current_time}.jpg")
        im.save(screenshot_path, "JPEG", quality=85)
        print(f"Screenshot saved at: {screenshot_path}")
        return screenshot_path
    else:
        print('Active window not found!')
        return None

#Function to monitor the application and take screenshots
def monitor_application(interval=2):
    global monitoring
    while monitoring:
        screenshot_path = screenshot()
        if screenshot_path:
            screenshots_lock.acquire()
            screenshots.append(screenshot_path)
            if len(screenshots) > 5:
                oldest_screenshot = screenshots.pop(0)
                if os.path.exists(oldest_screenshot):
                    os.remove(oldest_screenshot)
                    print(f"Deleted oldest screenshot: {oldest_screenshot}")
            screenshots_lock.release()
        time.sleep(interval)

#Function gets the name of the application and locates where it is
def start_monitoring():
    global monitoring
    app_name = entry_app_name.get()
    window_title = app_name

    if not app_name:
        messagebox.showerror("Error", "Please enter the application name.")
        return

    if not is_application_running(window_title):
        open_application(app_name)

    monitoring = True
    monitor_thread = threading.Thread(target=monitor_application)
    monitor_thread.start()

#Quits the monitoring and screenshotting process
def stop_monitoring():
    global monitoring
    monitoring = False

#Quit the Application
def quit_app():
    stop_monitoring()  
    root.destroy()

#Function to encode the image to base64
def encode_image(image_path):
    with open(image_path, "rb") as image_file:
        return base64.b64encode(image_file.read()).decode('utf-8')

#Function to check if an application is running
def is_application_running(window_title):
    hwnd = win32gui.FindWindow(None, window_title)
    return hwnd != 0

#Function to open the application
def open_application(app_name):
    try:
        subprocess.Popen(app_name)
        time.sleep(5)  
    except Exception as e:
        print(f"Error opening application: {e}")

#Function to delete all screenshots after OpenAI is asked
def delete_all_screenshots():
    global screenshots_lock, screenshots
    screenshots_lock.acquire()
    for screenshot_path in screenshots:
        if os.path.exists(screenshot_path):
            os.remove(screenshot_path)
    screenshots.clear()
    screenshots_lock.release()

#Function to interact with OpenAI's API using the API Key
def ask_openai():
    app_name = entry_app_name.get()
    window_title = app_name  
    prompt = entry_prompt.get()

    if not app_name:
        messagebox.showerror("Error", "Please enter the application name.")
        return

    if not is_application_running(window_title):
        open_application(app_name)

    if not screenshots:
        messagebox.showerror("Error", "No screenshots captured.")
        return

    try:
        base64_images = [encode_image(path) for path in screenshots]
        image_messages = [
            {"type": "image_url", "image_url": {"url": f"data:image/jpeg;base64,{image}"}}
            for image in base64_images
        ]
        headers = {
            "Content-Type": "application/json",
            "Authorization": f"Bearer {api_key}"
        }
        payload = {
            "model": "gpt-4o-mini",
            "messages": [
                {
                    "role": "user",
                    "content": [
                        {
                            "type": "text",
                            "text": prompt
                        }
                    ] + image_messages
                }
            ],
            "max_tokens": 500
        }

        # Make request to OpenAI API
        response = requests.post(
            "https://api.openai.com/v1/chat/completions",
            headers=headers,
            json=payload
        )

        if response.status_code == 200:
            response_data = response.json()
            summary = response_data['choices'][0]['message']['content']
            text_output.config(state=tk.NORMAL)
            text_output.delete(1.0, tk.END)
            text_output.insert(tk.END, summary)
            text_output.config(state=tk.DISABLED)
            delete_all_screenshots()
        else:
            print(f"Error: {response.status_code}")
            messagebox.showerror("Error", f"An error occurred: {response.status_code} - {response.json().get('error', {}).get('message', 'Unknown error')}")
    except Exception as e:
        messagebox.showerror("Error", f"An error occurred: {e}")

#About the Application
def show_about():
    messagebox.showinfo("About", "Make sure you have the application open, not minimized or else this app will not be able to scan for it.")

#Initialize the screenshot list, monitoring flag, and lock
screenshots = []
screenshots_lock = threading.Lock()
monitoring = False

root = tk.Tk()
root.title("OpenAI Image Chat Application")
root.geometry("600x400")

#All text input sections, buttons, and response section
menu_bar = Menu(root)
root.config(menu=menu_bar)

help_menu = Menu(menu_bar, tearoff=0)
help_menu.add_command(label="Close App", command=quit_app)
help_menu.add_command(label="Help", command=show_about)
menu_bar.add_cascade(label="Help", menu=help_menu)

label_app_name = tk.Label(root, text="Enter the application name:")
label_app_name.pack(pady=5)

entry_app_name = tk.Entry(root, width=50)
entry_app_name.pack(pady=5)

label_prompt = tk.Label(root, text="Enter the prompt:")
label_prompt.pack(pady=10)
entry_prompt = tk.Entry(root, width=50)
entry_prompt.pack(pady=10)

monitor_button = tk.Button(root, text="Start Monitoring the Application", command=start_monitoring)
monitor_button.pack(pady=10)

stop_button = tk.Button(root, text="Stop Monitoring the Application", command=stop_monitoring)
stop_button.pack(pady=10)

ask_button = tk.Button(root, text="Ask OpenAI", command=ask_openai)
ask_button.pack(pady=20)

label_output = tk.Label(root, text="OpenAI Response:")
label_output.pack(pady=5)

scrollbar = Scrollbar(root)
scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

text_output = Text(root, wrap=tk.WORD, yscrollcommand=scrollbar.set, height=20, state=tk.DISABLED)
text_output.pack(padx=10, pady=5, fill=tk.BOTH, expand=True)

scrollbar.config(command=text_output.yview)

root.mainloop()
