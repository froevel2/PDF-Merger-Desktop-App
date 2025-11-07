# 📄 PDF Merger Desktop App

A Python desktop application that allows users to select, reorder, and merge multiple PDF files using a graphical interface. Built with Tkinter and PyPDF2, and packaged as a standalone `.exe` using PyInstaller.

---

## 🧩 Features

- Select multiple PDF files via dialog or drag-and-drop
- Reorder files using "Move Up" and "Move Down" buttons
- Merge selected PDFs into a single document
- Save the output to a custom location
- Simple, responsive GUI built with Tkinter

---

## 🛠️ Technologies Used

- Python 3.x
- Tkinter (GUI)
- PyPDF2 (PDF merging)
- PyInstaller (packaging)
- PowerShell (environment setup)

---

## 🪜 Setup Instructions

### 1. Clone the repository

```bash
git clone https://github.com/your-username/pdf-merger-app.git
cd pdf-merger-app
import os
import PyPDF2
import tkinter as tk
from tkinter import filedialog, messagebox, simpledialog, Listbox, Scrollbar

class PDFMergerApp:
    def __init__(self, root):
        self.root = root
        self.root.title("PDF Merger")
        self.root.geometry("450x380")
        self.root.resizable(False, False)

        self.pdf_files = []

        tk.Label(root, text="📄 Select or drag PDFs to merge", font=("Arial", 12)).pack(pady=10)

        self.select_button = tk.Button(root, text="Select PDFs", command=self.select_files, font=("Arial", 11), bg="#2196F3", fg="white")
        self.select_button.pack(pady=5)

        frame = tk.Frame(root)
        frame.pack()

        self.listbox = Listbox(frame, width=50, height=10)
        self.listbox.pack(side=tk.LEFT, padx=(10, 0))

        scrollbar = Scrollbar(frame, orient="vertical", command=self.listbox.yview)
        scrollbar.pack(side=tk.RIGHT, fill="y")
        self.listbox.config(yscrollcommand=scrollbar.set)

        control_frame = tk.Frame(root)
        control_frame.pack(pady=5)

        tk.Button(control_frame, text="↑ Move Up", command=self.move_up).grid(row=0, column=0, padx=5)
        tk.Button(control_frame, text="↓ Move Down", command=self.move_down).grid(row=0, column=1, padx=5)
        tk.Button(control_frame, text="🗑️ Remove", command=self.remove_file).grid(row=0, column=2, padx=5)

        self.merge_button = tk.Button(root, text="Merge PDFs", command=self.merge_pdfs, font=("Arial", 11), bg="#4CAF50", fg="white")
        self.merge_button.pack(pady=10)

        # Enable drag-and-drop (Windows only)
        try:
            self.root.drop_target_register(tk.DND_FILES)
            self.root.dnd_bind('<<Drop>>', self.drop_files)
        except:
            pass  # Drag-and-drop not supported on all systems

    def select_files(self):
        files = filedialog.askopenfilenames(filetypes=[("PDF files", "*.pdf")])
        self.add_files(files)

    def drop_files(self, event):
        files = self.root.tk.splitlist(event.data)
        self.add_files(files)

    def add_files(self, files):
        for file in files:
            if file.lower().endswith(".pdf") and file not in self.pdf_files:
                self.pdf_files.append(file)
                self.listbox.insert(tk.END, os.path.basename(file))

    def move_up(self):
        index = self.listbox.curselection()
        if index and index[0] > 0:
            i = index[0]
            self.pdf_files[i - 1], self.pdf_files[i] = self.pdf_files[i], self.pdf_files[i - 1]
            self.refresh_listbox(i - 1)

    def move_down(self):
        index = self.listbox.curselection()
        if index and index[0] < len(self.pdf_files) - 1:
            i = index[0]
            self.pdf_files[i + 1], self.pdf_files[i] = self.pdf_files[i], self.pdf_files[i + 1]
            self.refresh_listbox(i + 1)

    def remove_file(self):
        index = self.listbox.curselection()
        if index:
            i = index[0]
            del self.pdf_files[i]
            self.refresh_listbox()

    def refresh_listbox(self, new_index=None):
        self.listbox.delete(0, tk.END)
        for file in self.pdf_files:
            self.listbox.insert(tk.END, os.path.basename(file))
        if new_index is not None:
            self.listbox.select_set(new_index)

    def merge_pdfs(self):
        if not self.pdf_files:
            messagebox.showwarning("No PDFs", "Please select or drop PDF files first.")
            return

        output_name = simpledialog.askstring("Output File", "Enter name for merged PDF (without .pdf):")
        if not output_name:
            return

        output_path = filedialog.askdirectory()
        if not output_path:
            return

        try:
            merger = PyPDF2.PdfMerger()
            for pdf in self.pdf_files:
                merger.append(pdf)

            full_output = os.path.join(output_path, output_name + ".pdf")
            merger.write(full_output)
            merger.close()
            messagebox.showinfo("Success", f"Merged PDF saved as:\n{full_output}")
        except Exception as e:
            messagebox.showerror("Error", f"Failed to merge PDFs:\n{e}")

# Run the app
root = tk.Tk()
app = PDFMergerApp(root)
root.mainloop()
