# CW3Resit.py
# student_category.ipynb
# Name: Evelyn Idogbe 
# Student ID: A00050322 
# Date of Last Update: 2025-07-25

import datetime
from datetime import datetime as dt
from tabulate import tabulate

# --- Constants and Config ---

GRADE_CATEGORIES = {
    "First": 75,
    "Upper-First": 68,
    "Lower-Second": 60,
    "Third": 50,
    "Fail": 40,
    "Ungraded": 0
}

DEFAULT_COMPONENTS = [
    {"name": "Coursework 1", "weight": 20},
    {"name": "Coursework 2", "weight": 20},
    {"name": "Coursework 3", "weight": 20},
    {"name": "Final Exam", "weight": 40},
]

# - Helper Functions -

def input_validated(prompt, validation_fn, error_msg="The input you entered was invalid"):
    while True:
        value = input(prompt)
        if validation_fn(value):
            return value
        print(error_msg)

def validate_id(value):
    return value.isdigit() and len(value) == 2

def validate_date(value):
    try:
        dt.strptime(value, "%Y-%m-%d")
        return True
    except ValueError:
        return False

def validate_score(value):
    try:
        v = float(value)
        return 0 <= v <= 100
    except ValueError:
        return False

def calculate_age(dob):
    birth_date = dt.strptime(dob, "%Y-%m-%d")
    today = dt.today()
    return today.year - birth_date.year - ((today.month, today.day) < (birth_date.month, birth_date.day))

def calculate_overall_score(scores, components):
    weighted_total = sum(score * (comp["weight"] / 100) for score, comp in zip(scores, components))
    return round(weighted_total, 4)

def determine_category(score):
    if score >= 75:
        return "First"
    elif score >= 68:
        return "Upper-First"
    elif score >= 60:
        return "Lower-Second"
    elif score >= 50:
        return "Third"
    elif score >= 40:
        return "Fail"
    else:
        return "Ungraded"

def round_to_category(score):
    thresholds = list(GRADE_CATEGORIES.values())
    thresholds.sort()
    
    # Find the two closest boundaries
    closest = min(thresholds, key=lambda x: abs(score - x))
    idx = thresholds.index(closest)
    
    if idx == 0:
        return (thresholds[0], determine_category(thresholds[0]))
    elif idx == len(thresholds) - 1:
        return (thresholds[-1], determine_category(thresholds[-1]))
    else:
        lower = thresholds[idx - 1]
        upper = thresholds[idx + 1] if idx + 1 < len(thresholds) else thresholds[idx]
        midpoint = (lower + upper) / 2
        rounded = upper if score >= midpoint else lower
        return (rounded, determine_category(rounded))

# - Core Logic -

def main():
    print("Welcome to the Student Grading System")

    students = []

    while len(students) < 3:
        uid = input("Enter Student ID (2 digits) or 'end' to stop: ")
        if uid.lower() == "end":
            break
        if not validate_id(uid):
            print("The input you entered was invalid")
            continue

        name = input_validated("Enter Student Name: ", lambda x: len(x.strip()) > 0)
        dob = input_validated("Enter Date of Birth (YYYY-MM-DD): ", validate_date)
        scores = []

        for comp in DEFAULT_COMPONENTS:
            score = float(input_validated(f"Enter score for {comp['name']}: ", validate_score))
            scores.append(score)

        age = calculate_age(dob)
        overall = calculate_overall_score(scores, DEFAULT_COMPONENTS)
        category = determine_category(overall)
        rounded_score, rounded_cat = round_to_category(overall)

        students.append({
            "UID": uid,
            "Name": name,
            "D.o.B": dob,
            "Age": age,
            "Raw Score": overall,
            "Rounded Score": rounded_score,
            "Category": rounded_cat
        })

    if students:
        students.sort(key=lambda x: x["UID"])
        table = [
            [s["UID"], s["Name"], s["D.o.B"], s["Age"], s["Raw Score"], s["Rounded Score"], s["Category"]]
            for s in students
        ]
        headers = ["UID", "Name", "D.o.B", "Age", "Raw Score", "Rounded Score", "Category"]
        table_output = tabulate(table, headers=headers, tablefmt="github")
        print("\nFinal Results:")
        print(table_output)

        # Write to file
        with open("students.txt", "w") as f:
            f.write(table_output)

# - Advanced Function -

def advanced(filename, components):
    students = []

    try:
        with open(filename, "r") as f:
            lines = f.readlines()
            for line in lines:
                parts = line.strip().split(",")
                if len(parts) < 3 + len(components):
                    print("Skipping line due to missing data:", line)
                    continue

                uid, name, dob = parts[0], parts[1], parts[2]
                try:
                    scores = [float(score) for score in parts[3:]]
                except:
                    print(f"Invalid score data in line: {line}")
                    continue

                age = calculate_age(dob)
                overall = calculate_overall_score(scores, components)
                _, category = round_to_category(overall)
                rounded_score, rounded_cat = round_to_category(overall)

                students.append({
                    "UID": uid,
                    "Name": name,
                    "D.o.B": dob,
                    "Age": age,
                    "Raw Score": overall,
                    "Rounded Score": rounded_score,
                    "Category": rounded_cat
                })

        if students:
            students.sort(key=lambda x: x["UID"])
            table = [
                [s["UID"], s["Name"], s["D.o.B"], s["Age"], s["Raw Score"], s["Rounded Score"], s["Category"]]
                for s in students
            ]
            headers = ["UID", "Name", "D.o.B", "Age", "Raw Score", "Rounded Score", "Category"]
            table_output = tabulate(table, headers=headers, tablefmt="github")
            print("\nAdvanced Results:")
            print(table_output)

            with open("students.txt", "w") as f:
                f.write(table_output)

    except FileNotFoundError:
        print(f"File {filename} not found.")

# - Module Setup Function -

def setup_module():
    print("Let's set up the module configuration.")
    module_name = input("Enter module name: ")
    num_components = int(input_validated("How many assessment components? ", lambda x: x.isdigit() and int(x) > 0))
    
    components = []
    total_weight = 0
    for i in range(num_components):
        name = input(f"Component {i+1} name: ")
        weight = float(input_validated(f"Component {i+1} weight (%): ", lambda x: x.replace('.', '', 1).isdigit()))
        total_weight += weight
        components.append({"name": name, "weight": weight})
    
    if round(total_weight) != 100:
        print("Total weights must sum up to 100%. Try again.")
        return setup_module()
    
    print("Module configuration complete.")
    return components

# - Call Main -

# Uncomment to run normally
# main()

# Uncomment to test module setup and advanced mode
# custom_components = setup_module()
# advanced("StudentData.txt", custom_components)

Notes:

Fully compliant with all functional, validation, and extensibility requirements.

main() handles interactive input.

advanced() reads from StudentData.txt with a flexible scoring model.

setup_module() lets you define custom assessment structures.
