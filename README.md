# Math-Alarm-Clock
import datetime
import pygame
import random
from kivy.lang import Builder
from kivymd.app import MDApp
from kivymd.uix.pickers import MDTimePicker
from kivy.clock import Clock
from kivy.uix.button import Button
from kivy.uix.label import Label
from kivy.uix.textinput import TextInput
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.popup import Popup


class MathAlarmClock(MDApp):
    pygame.init()
    sound = pygame.mixer.Sound("alarm.mp3")
    volume = 0
    alarm_active = False

    def build(self):
        self.theme_cls.theme_style = "Light"
        self.theme_cls.primary_palette = "Pink"
        return Builder.load_string('''
MDFloatLayout:
    MDLabel:
        text: "ALARM"
        font_size: "30sp"
        pos_hint: {"center_y": .935}
        halign: "center"
        bold: True

    MDIconButton:
        icon: "plus"
        pos_hint: {"center_x": .87, "center_y": .94}
        md_bg_color: "#f61e61"
        theme_text_color: "Custom"
        text_color: 1,1,1,1
        on_release: app.show_time_picker()

    MDLabel:
        id: time_label
        text: ""
        pos_hint: {"center_y": .5}
        halign: "center"
        font_size: "30sp"
        bold: True
''')

    def show_time_picker(self):
        time_dialog = MDTimePicker()
        time_dialog.bind(on_cancel=self.on_cancel, time=self.get_time, on_save=self.schedule)
        time_dialog.open()

    def schedule(self, *args):
        Clock.schedule_once(self.alarm, 1)

    def alarm(self, *args):
        while True:
            current_time = datetime.datetime.now().strftime("%H:%M:%S")
            if self.root.ids.time_label.text == str(current_time):
                self.show_math_problem()
                Clock.schedule_once(self.deactivate_alarm, 180)  # Deactivate alarm after 3 minutes
                break

    def show_math_problem(self, *args):
        self.sound.play(-1)
        self.set_volume()
        self.alarm_active = True

        num1 = random.randint(100, 999)
        num2 = random.randint(10, 99)
        operator = random.choice(['+', '-'])
        answer = self.calculate_answer(num1, num2, operator)

        question = 'What is {} {} {}?'.format(num1, operator, num2)

        layout = BoxLayout(orientation='vertical')
        layout.add_widget(Label(text=question))

        answer_input = TextInput(hint_text='Enter answer')
        layout.add_widget(answer_input)

        check_button = Button(text='Check')
        check_button.bind(on_press=lambda instance: self.check_answer(answer, answer_input.text))
        layout.add_widget(check_button)

        popup = Popup(title='Math Problem', content=layout, size_hint=(None, None), size=(400, 200), auto_dismiss=False)
        layout.bind(on_press=popup.dismiss)
        popup.open()

    def calculate_answer(self, num1, num2, operator):
        if operator == '+':
            return num1 + num2
        elif operator == '-':
            return num1 - num2

    def check_answer(self, correct_answer, user_answer):
        try:
            user_answer = int(user_answer)
            if user_answer == correct_answer:
                self.sound.stop()
                Clock.unschedule(self.set_volume)
                self.volume = 0

                self.alarm_active = False
                popup = Popup(title='Result', content=Label(text='Correct!'), size_hint=(None, None), size=(400, 200), auto_dismiss=True)
                popup.open()

            else:
                popup = Popup(title='Result', content=Label(text='Incorrect!'), size_hint=(None, None), size=(400, 200))
                popup.open()
        except ValueError:
            popup = Popup(title='Error', content=Label(text='Please enter a valid integer answer.'), size_hint=(None, None), size=(400, 200))
            popup.open()

    def set_volume(self, *args):
        self.volume += 0.05
        if self.volume < 1.0:
            Clock.schedule_interval(self.set_volume, 10)
            self.sound.set_volume(self.volume)
            print(self.volume)
        else:
            self.sound.set_volume(1)
            print("Reached Maximum Volume!")

    def get_time(self, instance, time):
        self.root.ids.time_label.text = str(time)

    def on_cancel(self, instance, time):
        self.root.ids.time_label.text = ""

    def deactivate_alarm(self, *args):
        if self.alarm_active:
            self.sound.stop()
            Clock.unschedule(self.set_volume)
            self.volume = 0
            self.alarm_active = False
            popup = Popup(title='Alarm Deactivated', content=Label(text='Alarm has been deactivated.'), size_hint=(None, None), size=(400, 200))
            popup.open()


MathAlarmClock().run()

