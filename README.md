# text-to-speech
import tkinter as tk
from tkinter import scrolledtext
import threading
import speech_recognition as sr

class VoiceRecognitionApp:
    def _init_(self, root):
        self.root = root
        self.root.title("Voice Recognition - Speech to Text")
        self.root.geometry("600x400")
        self.root.configure(bg="#121212")  # dark background

        # Create UI components
        self.label = tk.Label(root, text="Press 'Start' and speak...", bg="#121212", fg="#e0e0e0", font=("Arial", 16))
        self.label.pack(pady=(20, 10))

        self.text_area = scrolledtext.ScrolledText(root, wrap=tk.WORD, font=("Arial", 14), bg="#1e1e1e", fg="#ffffff", insertbackground='white')
        self.text_area.pack(padx=20, pady=10, fill=tk.BOTH, expand=True)
        self.text_area.configure(state='disabled')

        self.status_label = tk.Label(root, text="Status: Idle", bg="#121212", fg="#888888", font=("Arial", 12))
        self.status_label.pack(pady=(5, 15))

        self.button_frame = tk.Frame(root, bg="#121212")
        self.button_frame.pack(pady=10)

        self.start_button = tk.Button(self.button_frame, text="Start", command=self.start_listening, font=("Arial", 14), bg="#10b981", fg="white", activebackground="#059669", relief=tk.FLAT, padx=20, pady=10)
        self.start_button.pack(side=tk.LEFT, padx=10)

        self.stop_button = tk.Button(self.button_frame, text="Stop", command=self.stop_listening, font=("Arial", 14), bg="#ef4444", fg="white", activebackground="#b91c1c", relief=tk.FLAT, padx=20, pady=10, state=tk.DISABLED)
        self.stop_button.pack(side=tk.LEFT, padx=10)

        self.listening = False
        self.recognizer = sr.Recognizer()
        self.microphone = sr.Microphone()
        self.listen_thread = None

    def start_listening(self):
        if not self.listening:
            self.listening = True
            self.status_label.config(text="Status: Listening...")
            self.start_button.config(state=tk.DISABLED)
            self.stop_button.config(state=tk.NORMAL)
            self.text_area.configure(state='normal')
            self.text_area.delete('1.0', tk.END)
            self.text_area.configure(state='disabled')
            self.listen_thread = threading.Thread(target=self.listen_loop, daemon=True)
            self.listen_thread.start()

    def stop_listening(self):
        if self.listening:
            self.listening = False
            self.status_label.config(text="Status: Stopped")
            self.start_button.config(state=tk.NORMAL)
            self.stop_button.config(state=tk.DISABLED)

    def listen_loop(self):
        with self.microphone as source:
            self.recognizer.adjust_for_ambient_noise(source)
            while self.listening:
                try:
                    audio = self.recognizer.listen(source, timeout=5, phrase_time_limit=7)
                    text = self.recognizer.recognize_google(audio)
                    self.append_text(text)
                except sr.WaitTimeoutError:
                    pass
                except sr.UnknownValueError:
                    self.append_text("[Unrecognized Speech]")
                except sr.RequestError as e:
                    self.append_text(f"[API unavailable: {e}]")
                except Exception as e:
                    self.append_text(f"[Error: {e}]")

    def append_text(self, text):
        def update_text():
            self.text_area.configure(state='normal')
            if self.text_area.index('end-1c') != '1.0':
                self.text_area.insert(tk.END, "\n")
            self.text_area.insert(tk.END, text)
            self.text_area.see(tk.END)
            self.text_area.configure(state='disabled')
        self.root.after(0, update_text)

if _name_ == "_main_":
    root = tk.Tk()
    app = VoiceRecognitionApp(root)
    root.mainloop()

        
   
   
