# Booking
```python
import tkinter as tk
from tkinter import ttk
from tkcalendar import Calendar
import json
import os

booking_file = "bookings.json"

if not os.path.exists(booking_file):
    bookings = {
        "rooms": ["room1", "room2", "room3"],
        "orgs": ["ООО Альфа", "ДОЛ Факел+", "Студия Танцев Вектор", "ИП Пироговы", "Ресторан Arcabaleno"],
        "bs": []
    }
    f = open(booking_file, "w", encoding="utf-8")
    json.dump(bookings, f)
    f.close()
else:
    f = open(booking_file, "r", encoding="utf-8")
    bookings = json.load(f)
    f.close()

def save():
    f = open(booking_file, "w", encoding="utf-8")
    json.dump(bookings, f)
    f.close()

root = tk.Tk()
root.geometry("1000x600")

left_frame = tk.Frame(root)
left_frame.grid(row=0, column=0)

center_frame = tk.Frame(root)
center_frame.grid(row=0, column=1)

right_frame = tk.Frame(root)
right_frame.grid(row=0, column=2)

label_room = tk.Label(left_frame, text="Выберите зал:")
label_room.pack()

room_var = tk.StringVar()
room_list = ttk.Combobox(left_frame, textvariable=room_var,
                         values=bookings["rooms"], state="readonly")
room_list.current(0)
room_list.pack()

cal = Calendar(left_frame, selectmode='day', date_pattern='yyyy-mm-dd')
cal.pack()

label_events = tk.Label(left_frame, text="Мероприятия на день:")
label_events.pack()

events_list = tk.Listbox(left_frame, width=40, height=15)
events_list.pack()

label_timeline = tk.Label(center_frame, text="Лента занятости зала", font=("Arial", 14))
label_timeline.pack()

timeline_frame = tk.Frame(center_frame)
timeline_frame.pack()

start_time = None
end_time = None


def make_time_str(total_minutes):
    hour = total_minutes // 60
    minute = total_minutes % 60
    if hour < 10:
        hour_str = "0" + str(hour)
    else:
        hour_str = str(hour)
    if minute < 10:
        minute_str = "0" + str(minute)
    else:
        minute_str = str(minute)
    return hour_str + ":" + minute_str


def slot_choose(event):
    global start_time, end_time
    clicked_cell = event.widget
    clicked_idx = slots.index(clicked_cell)

    if clicked_cell["bg"] == "red":
        info_label.config(text="Это время уже занято!", bg="lightcoral")
        return

    if start_time is not None and end_time is not None:
        for i in range(48):
            if slots[i]["bg"] == "lightblue":
                slots[i].config(bg="lightgreen")
        start_time = None
        end_time = None

    if start_time is None:
        start_time = clicked_idx
        clicked_cell.config(bg="yellow")
        info_label.config(text="Начало: " + time_slots[start_time] + "\nВыберите конец", bg="lightyellow")

    elif clicked_idx == start_time:
        slots[start_time].config(bg="lightgreen")
        start_time = None
        info_label.config(text="Время не выбрано", bg="white")

    else:
        end_time = clicked_idx
        if end_time < start_time:
            a = start_time
            start_time = end_time
            end_time = a

        conflict = False
        for i in range(start_time, end_time + 1):
            if slots[i]["bg"] == "red":
                conflict = True
                break

        if conflict:
            info_label.config(text="Ошибка: в интервале чужое время!", bg="lightcoral")
            for i in range(start_time, end_time + 1):
                if slots[i]["bg"] == "yellow" or slots[i]["bg"] == "lightblue":
                    slots[i].config(bg="lightgreen")
            start_time = None
            end_time = None
        else:
            for i in range(start_time, end_time + 1):
                slots[i].config(bg="lightblue")

            count = end_time - start_time + 1
            total_mins = count * 15
            hours = total_mins // 60
            mins = total_mins % 60
            text = "Выбрано:\nС " + time_slots[start_time] + " до " + time_slots[end_time] + "\n(" + str(hours) + " ч " + str(mins) + " мин)"
            info_label.config(text=text, bg="lightblue")


time_slots = []
slots = []

for i in range(48):
    total_minutes = 9 * 60 + i * 15
    time_str = make_time_str(total_minutes)
    time_slots.append(time_str)
    cell = tk.Label(timeline_frame, text=time_str, bg="lightgreen", bd=1, width=8, height=3)
    cell.bind("<Button-1>", slot_choose)
    row = i // 8
    col = i % 8
    cell.grid(row=row, column=col, padx=1, pady=1)
    slots.append(cell)

label_form = tk.Label(right_frame, text="Новая бронь", font=("Arial", 12))
label_form.pack()

label_org = tk.Label(right_frame, text="Организация:")
label_org.pack()

org_var = tk.StringVar()
org_list = ttk.Combobox(right_frame, textvariable=org_var, values=bookings["orgs"])
org_list.current(0)
org_list.pack()

label_event = tk.Label(right_frame, text="Название мероприятия:")
label_event.pack()

event_entry = tk.Entry(right_frame)
event_entry.pack(pady=5, padx=10)

info_label = tk.Label(right_frame, text="Время не выбрано", bg="white")
info_label.pack(pady=30, padx=10)


def refresh():
    global start_time, end_time
    start_time = None
    end_time = None

    current_room = room_var.get()
    current_date = cal.get_date()

    for i in range(48):
        slots[i].config(bg="lightgreen", fg="black", text=time_slots[i])

    events_list.delete(0, tk.END)

    for b in bookings["bs"]:
        if b["room"] == current_room:
            if b["date"] == current_date:
                for i in range(b["start"], b["end"] + 1):
                    slots[i].config(bg="red", fg="white", text=time_slots[i])
                time_text = time_slots[b["start"]] + " - " + time_slots[b["end"]]
                events_list.insert(tk.END, time_text + " | " + b["org"] + " — " + b["event"])

    info_label.config(text="Время не выбрано", bg="white")


def confirm_booking():
    global start_time, end_time
    if start_time is None or end_time is None:
        info_label.config(text="Ошибка: выберите время!", bg="lightcoral")
        return

    org_name = org_var.get().strip()
    event_name = event_entry.get().strip()
    current_room = room_var.get()
    current_date = cal.get_date()

    if org_name == "" or event_name == "":
        info_label.config(text="Ошибка: заполните данные!", bg="lightcoral")
        return

    if org_name not in bookings["orgs"]:
        bookings["orgs"].append(org_name)
        org_list["values"] = bookings["orgs"]

    new_booking = {
        "room": current_room,
        "date": current_date,
        "start": start_time,
        "end": end_time,
        "org": org_name,
        "event": event_name
    }
    bookings["bs"].append(new_booking)
    save()

    event_entry.delete(0, tk.END)
    refresh()
    info_label.config(text="Бронь добавлена!", bg="lightgreen")


def on_change(event):
    refresh()


agree = tk.Button(right_frame, text="Утвердить бронь", command=confirm_booking)
agree.pack(pady=10, padx=10, fill=tk.X)

room_list.bind("<<ComboboxSelected>>", on_change)
cal.bind("<<CalendarSelected>>", on_change)

refresh()

root.mainloop()
