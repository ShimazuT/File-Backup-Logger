# 🗂 Backup Manager (Python + Tkinter)

A simple backup utility with a GUI that copies or zips an entire folder.

## Features
- Select source & backup folders via GUI
- Option to create `.zip` backups
- Automatic naming with date & version

## Here is  the code 
```bash

# backup.py

import os
import shutil
import zipfile
import json
import logging
from datetime import datetime
from tkinter import Tk, filedialog, Button

# Configure logging
logging.basicConfig(filename='backup.log', level=logging.INFO, format='%(asctime)s - %(message)s')

class BackupManager:
    def __init__(self, source_folder, backup_folder, zip_backup=False):
        self.source_folder = source_folder
        self.backup_folder = backup_folder
        self.zip_backup = zip_backup

    def create_backup(self):
        try:
            if not os.path.exists(self.source_folder):
                raise FileNotFoundError("Source folder does not exist.")
            if not os.access(self.source_folder, os.R_OK):
                raise PermissionError("Permission denied to read the source folder.")

            timestamp = datetime.now().strftime("%Y-%m-%d")
            version = "v227-3-3"
            backup_name = f"backup_{timestamp}_{version}"
            backup_path = os.path.join(self.backup_folder, backup_name)

            if self.zip_backup:
                self._zip_backup(backup_path)
            else:
                shutil.copytree(self.source_folder, backup_path)

            logging.info(f"Backup created successfully at {backup_path}")
        except Exception as e:
            logging.error(f"Error during backup: {e}")

    def _zip_backup(self, backup_path):
        with zipfile.ZipFile(f"{backup_path}.zip", 'w') as zipf:
            for root, dirs, files in os.walk(self.source_folder):
                for file in files:
                    zipf.write(os.path.join(root, file), os.path.relpath(os.path.join(root, file), self.source_folder))
        logging.info(f"ZIP backup created successfully at {backup_path}.zip")

class BackupApp:
    def __init__(self, master):
        self.master = master
        master.title("Backup Manager")

        self.source_folder = ""
        self.backup_folder = ""

        self.select_source_button = Button(master, text="Select Source Folder", command=self.select_source)
        self.select_source_button.pack()

        self.select_backup_button = Button(master, text="Select Backup Folder", command=self.select_backup)
        self.select_backup_button.pack()

        self.backup_button = Button(master, text="Start Backup", command=self.start_backup)
        self.backup_button.pack()

    def select_source(self):
        self.source_folder = filedialog.askdirectory()

    def select_backup(self):
        self.backup_folder = filedialog.askdirectory()

    def start_backup(self):
        if self.source_folder and self.backup_folder:
            backup_manager = BackupManager(self.source_folder, self.backup_folder, zip_backup=True)
            backup_manager.create_backup()
        else:
            logging.error("Source or Backup folder not selected.")

if __name__ == "__main__":
    root = Tk()
    app = BackupApp(root)
    root.mainloop()

# config.json
{
    "source_folder": "",
    "backup_folder": "",
    "backup_interval": "daily"
}




