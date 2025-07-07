# Project1
import tkinter as tk
from tkinter import filedialog, messagebox, scrolledtext
import cv2
import numpy as np
import base64
import random
from cryptography.fernet import Fernet
import os
import time

# ========== AES UTILITIES ==========
def generate_key(password: str) -> bytes:
    return base64.urlsafe_b64encode(password.encode().ljust(32)[:32])

def encrypt_message(message: str, password: str) -> bytes:
    key = generate_key(password)
    return Fernet(key).encrypt(message.encode()) + b"####"

def decrypt_message(encrypted: bytes, password: str) -> str:
    key = generate_key(password)
    return Fernet(key).decrypt(encrypted).decode()

# ========== STEGANOGRAPHY FUNCTIONS ==========
def hide_message_in_image(image_path, message, password, output_path):
    image = cv2.imread(image_path)
    if image is None:
        raise Exception("Could not read the image.")

    encrypted = encrypt_message(message, password)
    binary = ''.join(format(byte, '08b') for byte in encrypted)

    flat = image.flatten().astype(np.int32)
    if len(binary) > len(flat):
        raise Exception("Message is too large to encode in this image.")

    random.seed(password)
    indices = list(range(len(flat)))
    random.shuffle(indices)

    for i in range(len(binary)):
        flat[indices[i]] = (flat[indices[i]] & ~1) | int(binary[i])

    flat = np.clip(flat, 0, 255).astype(np.uint8)
    encoded_image = flat.reshape(image.shape)

    success = cv2.imwrite(output_path, encoded_image)
    if not success:
        raise Exception("Failed to save encoded image.")
    return output_path

def reveal_message_from_image(image_path, password):
    image = cv2.imread(image_path)
    if image is None:
        raise Exception("Could not read the image.")

    flat = image.flatten()
    random.seed(password)
    indices = list(range(len(flat)))
    random.shuffle(indices)

    binary = ''.join(str(flat[i] & 1) for i in indices)
    bytes_data = [int(binary[i:i+8], 2) for i in range(0, len(binary), 8)]

    marker = [35, 35, 35, 35]  # ASCII ####
    for i in range(len(bytes_data)):
        if bytes_data[i:i+4] == marker:
            encrypted = bytes(bytes_data[:i])
            return decrypt_message(encrypted, password)
    raise Exception("No message found or incorrect password.")

# ========== SPLASH SCREEN ==========
def show_splash_screen(root):
    splash = tk.Toplevel(root)
    splash.overrideredirect(True)
    splash.configure(bg="black")

    text = tk.Label(splash, text="Launching Cybersecurity Tool...", font=("Consolas", 20, "bold"), fg="lime", bg="black")
    text.pack(padx=50, pady=50)

    splash.update_idletasks()
    w, h = splash.winfo_reqwidth(), splash.winfo_reqheight()
    x = (splash.winfo_screenwidth() // 2) - (w // 2)
    y = (splash.winfo_screenheight() // 2) - (h // 2)
    splash.geometry(f"{w}x{h}+{x}+{y}")

    for i in range(5):
        text.config(text=f"Loading{'.'*((i%4)+1)}")
        splash.update()
        time.sleep(0.5)

    splash.destroy()
    root.deiconify()

# ========== MAIN GUI ==========
class StegoApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Cybersecurity Stego Tool")
        self.root.geometry("700x600")
        self.root.configure(bg="#121212")
        self.main_menu()

    def clear(self):
        for widget in self.root.winfo_children():
            widget.destroy()

    def main_menu(self):
        self.clear()
        tk.Label(self.root, text="Secure Steganography Tool", font=("Helvetica", 18, "bold"), fg="cyan", bg="#121212").pack(pady=20)

        tk.Button(self.root, text="🔒 Encrypt Message", font=("Helvetica", 14), width=25, bg="#007acc", fg="white",
                  command=self.encode_ui).pack(pady=10)

        tk.Button(self.root, text="🔓 Decrypt Message", font=("Helvetica", 14), width=25, bg="#28a745", fg="white",
                  command=self.decode_ui).pack(pady=10)

    def encode_ui(self):
        self.clear()
        tk.Label(self.root, text="🔐 Encrypt and Hide Message", font=("Helvetica", 16, "bold"), fg="white", bg="#121212").pack(pady=10)

        text_box = scrolledtext.ScrolledText(self.root, height=4, width=60)
        text_box.pack(pady=10)

        tk.Label(self.root, text="Enter Password:", fg="white", bg="#121212").pack()
        key_entry = tk.Entry(self.root, show="*", width=40)
        key_entry.pack(pady=5)

        img_path_var = tk.StringVar()

        def select_image():
            path = filedialog.askopenfilename(filetypes=[("Images", "*.png *.jpg *.jpeg")])
            if path:
                img_path_var.set(path)
                tk.Label(self.root, text=os.path.basename(path), fg="lightgreen", bg="#121212").pack()

        def save_image():
            msg = text_box.get("1.0", tk.END).strip()
            pwd = key_entry.get().strip()
            path = img_path_var.get()

            if not msg or not pwd or not path:
                messagebox.showerror("Error", "All fields are required.")
                return

            out_path = filedialog.asksaveasfilename(defaultextension=".png", filetypes=[("PNG", "*.png")])
            if not out_path:
                return

            try:
                hide_message_in_image(path, msg, pwd, out_path)
                messagebox.showinfo("Success", f"Message hidden and saved to:\n{out_path}")
            except Exception as e:
                messagebox.showerror("Error", str(e))

        tk.Button(self.root, text="Choose Image", command=select_image, bg="#444", fg="white").pack(pady=5)
        tk.Button(self.root, text="Encrypt & Save Image", command=save_image, bg="#007acc", fg="white").pack(pady=10)
        tk.Button(self.root, text="⬅ Back", command=self.main_menu, bg="#555", fg="white").pack()

    def decode_ui(self):
        self.clear()
        tk.Label(self.root, text="🔓 Decrypt Hidden Message", font=("Helvetica", 16, "bold"), fg="white", bg="#121212").pack(pady=10)

        tk.Label(self.root, text="Enter Password:", fg="white", bg="#121212").pack()
        key_entry = tk.Entry(self.root, show="*", width=40)
        key_entry.pack(pady=5)

        output_box = scrolledtext.ScrolledText(self.root, height=5, width=60)
        output_box.pack(pady=10)
        output_box.config(state=tk.DISABLED)

        def decode():
            pwd = key_entry.get().strip()
            path = filedialog.askopenfilename(filetypes=[("PNG Images", "*.png")])
            if not path or not pwd:
                messagebox.showerror("Error", "Select image and enter password.")
                return

            try:
                message = reveal_message_from_image(path, pwd)
                output_box.config(state=tk.NORMAL)
                output_box.delete("1.0", tk.END)
                output_box.insert(tk.END, message)
                output_box.config(state=tk.DISABLED)

                tk.Label(self.root, text="Decrypted Data", fg="cyan", bg="#121212", font=("Helvetica", 14, "bold")).pack()
            except Exception as e:
                messagebox.showerror("Error", str(e))

        tk.Button(self.root, text="Select Image & Decrypt", command=decode, bg="#28a745", fg="white").pack(pady=10)
        tk.Button(self.root, text="⬅ Back", command=self.main_menu, bg="#555", fg="white").pack()

# ========== RUN APP ==========
if __name__ == "__main__":
    root = tk.Tk()
    root.withdraw()
    show_splash_screen(root)
    app = StegoApp(root)
    root.mainloop()
