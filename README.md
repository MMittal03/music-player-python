import tkinter as tk
from tkinter import filedialog
import pygame
import os
from mutagen.mp3 import MP3

# ---------- SETUP ----------
pygame.mixer.init()
root = tk.Tk()
root.title("🎵 Music Player App")
root.geometry("420x500")

# ---------- VARIABLES ----------
song_folder = "songs"
songs = [f for f in os.listdir(song_folder) if f.endswith(('.mp3', '.wav'))]
current_index = 0
RECENT_FILE = "recent.txt"

# ---------- FUNCTIONS ----------
def play_selected_song():
    global current_index
    selected = playlist_box.curselection()
    if selected:
        current_index = selected[0]
        song = songs[current_index]
        full_path = os.path.join(song_folder, song)
        pygame.mixer.music.load(full_path)
        pygame.mixer.music.play()
        display_song_info(full_path, song)
        save_recent(song)

def display_song_info(path, song_name):
    try:
        audio = MP3(path)
        length = audio.info.length
        minutes = int(length // 60)
        seconds = int(length % 60)
        song_label.config(text=f"🎵 Now Playing: {song_name} ({minutes}:{seconds:02d})")
    except:
        song_label.config(text=f"🎵 Now Playing: {song_name}")

def pause_music():
    pygame.mixer.music.pause()

def resume_music():
    pygame.mixer.music.unpause()

def stop_music():
    pygame.mixer.music.stop()
    song_label.config(text="🎵 Stopped")

def change_volume(val):
    pygame.mixer.music.set_volume(int(val) / 100)

def next_song():
    global current_index
    current_index = (current_index + 1) % len(songs)
    song = songs[current_index]
    full_path = os.path.join(song_folder, song)
    pygame.mixer.music.load(full_path)
    pygame.mixer.music.play()
    playlist_box.select_clear(0, tk.END)
    playlist_box.select_set(current_index)
    display_song_info(full_path, song)
    save_recent(song)

def prev_song():
    global current_index
    current_index = (current_index - 1) % len(songs)
    song = songs[current_index]
    full_path = os.path.join(song_folder, song)
    pygame.mixer.music.load(full_path)
    pygame.mixer.music.play()
    playlist_box.select_clear(0, tk.END)
    playlist_box.select_set(current_index)
    display_song_info(full_path, song)
    save_recent(song)

def save_recent(song):
    with open(RECENT_FILE, "a") as f:
        f.write(song + "\n")

# ---------- UI ELEMENTS ----------
tk.Label(root, text="🎶 Python Music Player", font=("Arial", 18)).pack(pady=10)

playlist_box = tk.Listbox(root, width=40, height=10)
for song in songs:
    playlist_box.insert(tk.END, song)
playlist_box.pack(pady=10)

tk.Button(root, text="▶ Play Selected", width=20, command=play_selected_song).pack(pady=2)
tk.Button(root, text="⏸ Pause", width=20, command=pause_music).pack(pady=2)
tk.Button(root, text="⏯ Resume", width=20, command=resume_music).pack(pady=2)
tk.Button(root, text="⏹ Stop", width=20, command=stop_music).pack(pady=2)

tk.Button(root, text="⏮ Previous", width=20, command=prev_song).pack(pady=2)
tk.Button(root, text="⏭ Next", width=20, command=next_song).pack(pady=2)

volume_slider = tk.Scale(root, from_=0, to=100, orient=tk.HORIZONTAL, command=change_volume, label="🔊 Volume")
volume_slider.set(70)
volume_slider.pack(pady=10)

song_label = tk.Label(root, text="🎵 Now Playing: None", font=("Arial", 10))
song_label.pack(pady=10)

# ---------- RUN ----------
root.mainloop()
