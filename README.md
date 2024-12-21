# rainfall-prediction-using-ml-by-user-input
import numpy as np
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression
import os
import tkinter as tk
from tkinter import messagebox
from tkinter import filedialog
from PIL import Image, ImageTk


if not os.path.exists('static'):
    os.makedirs('static')


def train_model(months, rainfall):
    X = np.array(months, dtype=float).reshape(-1, 1)  
    y = np.array(rainfall, dtype=float)
    model = LinearRegression()
    model.fit(X, y)  
    return model


def predict_rainfall(model, month):
    future_month = np.array([[month]], dtype=float)
    prediction = model.predict(future_month)  
    return future_month, prediction


def plot_rainfall(months, rainfall, future_month, prediction):
    try:
        prediction = np.array(prediction)
        
        fig, axs = plt.subplots(2, 2, figsize=(14, 12))  
        
        
        axs[0, 0].scatter(months, rainfall, color='blue', label='Actual Rainfall', marker='o')
        axs[0, 0].scatter(future_month, prediction, color='green', label='Predicted Rainfall', s=100)
        axs[0, 0].plot(np.append(months, future_month), np.append(rainfall, prediction.flatten()), linestyle='dashed', color='red', label='Trend Line')
        axs[0, 0].axvline(x=1, color='gray', linestyle='--', label='Prediction Boundary')
        axs[0, 0].set_xlabel('Month')
        axs[0, 0].set_ylabel('Rainfall (mm)')
        axs[0, 0].set_title('Rainfall Prediction for Next Month')
        axs[0, 0].legend()
        axs[0, 0].grid(True, linestyle='--', linewidth=0.6)

        
        axs[0, 1].hist(rainfall, bins=10, color='orange', edgecolor='black')
        axs[0, 1].set_title('Distribution of Rainfall Data')
        axs[0, 1].set_xlabel('Rainfall (mm)')
        axs[0, 1].set_ylabel('Frequency')
        axs[0, 1].grid(True, linestyle='--', linewidth=0.6)

        
        axs[1, 0].plot(months, rainfall, marker='o', color='purple', linestyle='-')
        axs[1, 0].set_title('Rainfall Trend Over Time')
        axs[1, 0].set_xlabel('Month')
        axs[1, 0].set_ylabel('Rainfall (mm)')
        axs[1, 0].grid(True, linestyle='--', linewidth=0.6)

        
        axs[1, 1].boxplot(rainfall, vert=True, patch_artist=True, boxprops=dict(facecolor='lightblue', color='black'))
        axs[1, 1].set_title('Box Plot of Rainfall Variability')
        axs[1, 1].set_ylabel('Rainfall (mm)')

        
        plot_path = 'static/rainfall_analysis.png'
        plt.tight_layout()  
        plt.savefig(plot_path)
        plt.close()

        return plot_path
    except Exception as e:
        print(f"An error occurred while plotting: {e}")
        return None


class RainfallPredictionApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Rainfall Prediction")

        self.root.geometry('5407x3604') 
        self.root.config(bg="#F0F8FF")

        
        self.bg_image = Image.open("weather.jpg")  
        self.bg_image = self.bg_image.resize((2500,978))   
        self.bg_photo = ImageTk.PhotoImage(self.bg_image)
        
        
        self.bg_label = tk.Label(root, image=self.bg_photo)
        self.bg_label.place(x=0, y=0, relwidth=1, relheight=1) 

       
        self.main_frame = tk.Frame(root, bg="peach puff", padx=20, pady=20)  
        self.main_frame.place(relx=0.5, rely=0.5, anchor="center", width=570, height=650) 

    
        self.title_label = tk.Label(self.main_frame, text="Rainfall Prediction App", font=("Helvetica", 18, "bold"), fg="#2C3E50", bg="#F0F8FF")
        self.title_label.grid(row=0, column=0, columnspan=2, pady=10, sticky="nsew")

        
        self.label_months = tk.Label(self.main_frame, text="Enter number of months:", font=("Arial", 12), bg="#F0F8FF")
        self.label_months.grid(row=1, column=0, padx=10, pady=10, sticky="w")

        self.entry_months = tk.Entry(self.main_frame, font=("Arial", 12))
        self.entry_months.grid(row=1, column=1, padx=10, pady=10)

        self.label_rainfall = tk.Label(self.main_frame, text="Enter rainfall data (comma-separated):", font=("Arial", 12), bg="#F0F8FF")
        self.label_rainfall.grid(row=2, column=0, padx=10, pady=10, sticky="w")

        self.entry_rainfall = tk.Entry(self.main_frame, font=("Arial", 12))
        self.entry_rainfall.grid(row=2, column=1, padx=10, pady=10)

        
        self.submit_button = tk.Button(self.main_frame, text="Predict Rainfall", font=("Arial", 12, "bold"), bg="#3498DB", fg="white", command=self.on_predict, relief="raised", width=20)
        self.submit_button.grid(row=3, column=0, columnspan=2, pady=20)

        
        self.result_label = tk.Label(self.main_frame, text="Predicted Rainfall: Not available", font=("Arial", 14), bg="#F0F8FF", fg="#27AE60")
        self.result_label.grid(row=4, column=0, columnspan=2, pady=10)

        
        self.canvas = tk.Canvas(self.main_frame, width=600, height=600, bg="white") 
        self.canvas.grid(row=5, column=0, columnspan=2, pady=10)

    
    def on_predict(self):
        try:
            
            num_months = int(self.entry_months.get())
            rainfall_data = self.entry_rainfall.get().split(',')
            rainfall = list(map(float, rainfall_data))

        
            months = list(range(1, num_months + 1))
            model = train_model(months, rainfall)
            future_month, prediction = predict_rainfall(model, month=num_months + 1)

           
            self.result_label.config(text=f"Predicted Rainfall for Next Month: {prediction[0]:.2f} mm", fg="#27AE60")

            
            plot_path = plot_rainfall(months, rainfall, future_month, prediction)

            
            self.display_image(plot_path)

        except Exception as e:
            messagebox.showerror("Error", f"An error occurred: {e}")

    
    def display_image(self, plot_path):
        try:
            
            img = Image.open(plot_path)
            img = img.resize((550,350))  
            img_tk = ImageTk.PhotoImage(img)
            
            # Display the image on the canvas
            self.canvas.create_image(0, 0, anchor='nw', image=img_tk)
            self.canvas.image = img_tk  
        except Exception as e:
            messagebox.showerror("Error", f"Failed to display the image: {e}")

if __name__ == "__main__":
    root = tk.Tk()
    app = RainfallPredictionApp(root)
    root.mainloop()
