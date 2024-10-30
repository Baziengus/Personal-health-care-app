import customtkinter
from tkinter import messagebox, Canvas, Scrollbar, PhotoImage
from PIL import Image, ImageTk

# Set up appearance mode and theme for customtkinter
customtkinter.set_appearance_mode("light")  # Modes: "System" (standard), "Dark", "Light"
customtkinter.set_default_color_theme("themes/cherry.json")

# Initialize main window
root = customtkinter.CTk()
root.geometry("650x700")
root.title("Personal Health Care App")

# List to store recent history of BMI calculations
history = []

def calculate_bmi(weight, height, unit='metric'):
    """Calculate BMI based on weight and height using either metric or imperial units."""
    if unit == 'metric':    
        bmi = weight / (height ** 2)
    elif unit == 'imperial':
        bmi = (weight / (height ** 2)) * 703
    else:
        raise ValueError("Invalid unit. Choose either 'metric' or 'imperial'.")
    return round(bmi, 2)

def calculate_protein_intake(weight, gender):
    """Calculate daily protein intake recommendation based on weight and gender."""
    protein_intake = weight * 1.2 if gender.lower() == 'male' else weight * 1.0
    return round(protein_intake, 2)

def calculate_calorie_intake(weight, height, age, gender, activity_level, unit):
    """Calculate daily calorie intake recommendation based on age, gender, and activity level."""
    if unit == 'imperial':  # Convert to metric if needed
        weight *= 0.453592  # pounds to kg
        height *= 2.54 / 100  # inches to meters
    
    if gender.lower() == 'male':
        bmr = 88.362 + (13.397 * weight) + (4.799 * height * 100) - (5.677 * age)
    else:
        bmr = 447.593 + (9.247 * weight) + (3.098 * height * 100) - (4.330 * age)

    # Adjust based on activity level
    activity_factors = {"Sedentary": 1.2, "Lightly Active": 1.375, "Moderately Active": 1.55, "Very Active": 1.725, "Extremely Active": 1.9}
    calorie_needs = bmr * activity_factors.get(activity_level, 1.2)
    
    return round(calorie_needs, 2)

def give_suggestions(age, gender):
    """Provide health suggestions based on age and gender."""
    if age < 18:
        return "Focus on balanced nutrition and regular physical activity."
    elif 18 <= age < 35:
        return "Focus on muscle building and cardiovascular health." if gender.lower() == 'male' else "Maintain body composition and hormonal balance."
    elif 35 <= age < 50:
        return "Maintain muscle mass, ensure protein intake." if gender.lower() == 'male' else "Strength training and adequate calcium intake."
    else:
        return "Joint health, regular physical activity, heart health." if gender.lower() == 'male' else "Bone health, joint flexibility, nutrient-rich diet."

def calculate_bmi_gui():
    try:
        name = entry_name.get()
        weight = float(entry_weight.get())
        height = float(entry_height.get())
        age = int(entry_age.get())
        gender = gender_var.get()
        activity_level = activity_var.get()
        unit = 'metric' if metric_var.get() == 1 else 'imperial'

        if age <= 0:
            raise ValueError("Age must be greater than 0.")
        
        if not name:
            raise ValueError("Name cannot be empty.")

        # Calculate BMI, protein intake, and calorie intake
        bmi_value = calculate_bmi(weight, height, unit)
        protein_intake = calculate_protein_intake(weight, gender)
        calorie_intake = calculate_calorie_intake(weight, height, age, gender, activity_level, unit)
        suggestions = give_suggestions(age, gender)

        # Save data to history list
        history_entry = f"Name: {name}, Age: {age}, Height: {height}, Weight: {weight}, BMI: {bmi_value}\nProtein: {protein_intake}g, Calories: {calorie_intake}kcal\nSuggestions: {suggestions}"
        history.append(history_entry)

        # Display the result in the result label
        result_label.configure(text=f"Your BMI: {bmi_value}\nProtein: {protein_intake}g/day\nCalories: {calorie_intake}kcal/day\n{suggestions}")
        
        # Update the history section
        update_history_label()
    except ValueError as e:
        messagebox.showerror("Input Error", str(e))

def update_history_label():
    """Update the history label to display the recent entries."""
    
    formatted_history = "\n\n---\n\n".join(history[-10:])
    history_label.configure(text=formatted_history if formatted_history else "No history available.")
    history_canvas.update_idletasks()
    history_canvas.config(scrollregion=history_canvas.bbox("all"))

# GUI Elements Setup
frame = customtkinter.CTkFrame(master=root)
frame.pack(pady=20, padx=60, fill="both", expand=True)

# Load and resize the cross icon image
cross_image = Image.open("themes/images.png")  # Update path as needed
cross_image = cross_image.resize((20, 20), Image.LANCZOS)  # Use LANCZOS instead of ANTIALIAS
cross_icon = ImageTk.PhotoImage(cross_image)

# Create a frame to hold both icon and title
title_frame = customtkinter.CTkFrame(master=frame)
title_frame.pack(pady=12)

# Label to display the icon beside the title
icon_label = customtkinter.CTkLabel(master=title_frame, image=cross_icon, text="")  # Only show the image
icon_label.pack(side="left", padx=5)

# Label to display the title
label = customtkinter.CTkLabel(master=title_frame, text="PERSONAL HEALTH CARE", font=("Arial", 18))
label.pack(side="left", padx=5)

# Continue setting up the rest of your GUI
entry_name = customtkinter.CTkEntry(master=frame, placeholder_text="Name")
entry_name.pack(pady=12, padx=10)

entry_age = customtkinter.CTkEntry(master=frame, placeholder_text="Age")
entry_age.pack(pady=12, padx=10)

entry_height = customtkinter.CTkEntry(master=frame, placeholder_text="Height (m or inches)")
entry_height.pack(pady=12, padx=10)

entry_weight = customtkinter.CTkEntry(master=frame, placeholder_text="Weight (kg or lb)")
entry_weight.pack(pady=12, padx=10)

metric_var = customtkinter.IntVar(value=1)
checkbox_frame = customtkinter.CTkFrame(master=frame)
checkbox_frame.pack(pady=12)

imperial_check = customtkinter.CTkRadioButton(master=checkbox_frame, text="Imperial", variable=metric_var, value=0)
imperial_check.pack(side="left", padx=20)

metric_check = customtkinter.CTkRadioButton(master=checkbox_frame, text="Metric", variable=metric_var, value=1)
metric_check.pack(side="left", padx=20)

gender_var = customtkinter.StringVar()
gender_var.set("Male")
gender_male = customtkinter.CTkRadioButton(master=frame, text="Male", variable=gender_var, value="Male")
gender_male.pack(pady=6)
gender_female = customtkinter.CTkRadioButton(master=frame, text="Female", variable=gender_var, value="Female")
gender_female.pack(pady=6)

activity_var = customtkinter.StringVar()
activity_var.set("Not Active")
activity_menu = customtkinter.CTkOptionMenu(master=frame, variable=activity_var, values=["Not Active", "Lightly Active", "Moderately Active", "Very Active", "Extremely Active"])
activity_menu.pack(pady=6)

button = customtkinter.CTkButton(master=frame, text="Calculate", command=calculate_bmi_gui)
button.pack(pady=12, padx=10)

result_label = customtkinter.CTkLabel(master=frame, text="", font=("Arial", 12), justify='left')
result_label.pack(pady=12, padx=10)

history_frame = customtkinter.CTkFrame(master=frame, fg_color="pink")
history_frame.pack(pady=12, padx=10, fill="both", expand=True)

history_canvas = Canvas(history_frame, bg="pink") 
history_canvas.pack(side="left", fill="both", expand=True)

scrollbar = Scrollbar(history_frame, orient="vertical", command=history_canvas.yview)
scrollbar.pack(side="right", fill="y")

history_canvas.configure(yscrollcommand=scrollbar.set)
history_inner_frame = customtkinter.CTkFrame(master=history_canvas, fg_color="pink")
history_canvas.create_window((0, 0), window=history_inner_frame, anchor="nw")

history_label = customtkinter.CTkLabel(master=history_inner_frame, text="No history available.", font=("Roboto", 12), justify='center', text_color="white", anchor="center")
history_label.pack(pady=12, padx=10)

root.mainloop()
